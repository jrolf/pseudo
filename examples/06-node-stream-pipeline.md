# Example 6: A Node.js Stream Pipeline

Node's stream API solves a real problem - processing files far bigger than
memory - behind an API that reads like plumbing. This function extracts
error lines from a huge log and writes them to a compressed archive.

## The original

```javascript
const fs = require("node:fs");
const zlib = require("node:zlib");
const { pipeline, Transform } = require("node:stream");

function archiveErrors(inputPath, outputPath, done) {
  const onlyErrors = new Transform({
    transform(chunk, encoding, callback) {
      const lines = chunk.toString().split("\n");
      const errors = lines.filter((line) => line.includes(" ERROR "));
      callback(null, errors.length ? errors.join("\n") + "\n" : "");
    },
  });

  pipeline(
    fs.createReadStream(inputPath),
    onlyErrors,
    zlib.createGzip(),
    fs.createWriteStream(outputPath),
    done
  );
}
```

## The Pseudo translation

```python
# ARCHIVE ERROR LINES FROM A HUGE LOG, WITHOUT LOADING IT INTO MEMORY

Define "archiveErrors", given [inputPath, outputPath, done]:

    Where inputPath locates the huge log file to be read.
    Where outputPath locates the compressed archive to be written.
    Where done is the function to call exactly once at the end,
    with either the failure or nothing at all on success.

    Connect four stations into one assembly line:
        read the log file a chunk at a time,
        keep only the lines containing the error marker,
        compress whatever survives,
        and write the compressed result to the archive file.

    While the line runs:
        Each station takes a chunk, does its one job, and passes it on.
        # No station is allowed to outrun the next. If the output disk is
        # slow, the slowdown propagates backward station by station until
        # even the reading pauses. This backpressure is the reason a
        # ten-gigabyte log never needs ten gigabytes of memory.

    If any station fails:
        Shut down the entire line and destroy the partial output stream.
        Report the failure once, through the single completion callback.

    When the reader reaches the end of the log:
        Let the chunks still inside the line drain through to the archive.
        Report success through the completion callback.
```

## What the translation reveals

Backpressure - the single most important property of this code - is
completely invisible in the source. It is a guarantee `pipeline` provides,
mentioned nowhere on the page. The Pseudo promotes it to a voiceover
comment, because it is the answer to the question every reviewer should ask:
"why streams instead of reading the file?"

The translation also exposes a genuine bug worth discussing: the filter
station splits *chunks* into lines, but chunk boundaries fall anywhere, so a
line straddling two chunks gets cut in half and may be missed or mangled.
In the source, that bug hides inside an innocent `chunk.toString().split()`.
In Pseudo, "read the log file a chunk at a time" sitting directly above
"keep only the lines" makes the mismatch visible enough to catch in review.
