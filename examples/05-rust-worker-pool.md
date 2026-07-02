# Example 5: A Rust Worker Pool with Channels

Rust concurrency code is famously safe and famously dense. Ownership moves,
channel clones, and mutex guards all serve one design, but the design is
nowhere written down - except in Pseudo.

## The original

```rust
use std::sync::{mpsc, Arc, Mutex};
use std::thread;

fn parallel_word_count(paths: Vec<String>, workers: usize) -> usize {
    let (job_tx, job_rx) = mpsc::channel::<String>();
    let job_rx = Arc::new(Mutex::new(job_rx));
    let (result_tx, result_rx) = mpsc::channel::<usize>();

    for _ in 0..workers {
        let rx = Arc::clone(&job_rx);
        let tx = result_tx.clone();
        thread::spawn(move || loop {
            let job = rx.lock().unwrap().recv();
            match job {
                Ok(path) => {
                    let text = std::fs::read_to_string(&path).unwrap_or_default();
                    tx.send(text.split_whitespace().count()).unwrap();
                }
                Err(_) => break,
            }
        });
    }
    drop(result_tx);

    for path in paths {
        job_tx.send(path).unwrap();
    }
    drop(job_tx);

    result_rx.iter().sum()
}
```

## The Pseudo translation

```python
# PARALLEL WORD COUNT WITH A WORKER POOL

Define "parallel_word_count", given [paths, workers]:

    Where paths is the list of files whose words should be counted.
    Where workers is how many counting threads to run at once.

    Set up two conveyor belts:
        one carrying file paths out to the workers,
        and one carrying finished counts back.

    Start the requested number of workers, each on its own thread:
        Each worker repeats forever:
            # Only one worker at a time may reach into the job belt; a lock
            # guarantees every file is claimed by exactly one worker.
            Take the next file path from the belt, waiting if none is ready.
            If the job belt has been closed and emptied:
                The worker finishes.
            Read the file, treating an unreadable file as empty.
            Count its words.
            Put the count on the result belt.

    Feed every file path onto the job belt.
    Close the job belt.
        # Closing the belt IS the shutdown signal. There is no "stop"
        # message: each worker discovers the end of the work by finding
        # the belt closed.

    Collect counts from the result belt until it closes too,
    which happens once every worker has finished and let go of it.
    Return the sum of all the counts.
```

## What the translation reveals

The two `drop` calls are the entire shutdown protocol, and in the Rust they
look like cleanup rather than communication. The Pseudo says it outright:
closing the belt is the shutdown signal. Likewise, `Arc<Mutex<Receiver>>` is
a five-word type expressing "workers share one job source and take turns" -
a sentence any reader can hold onto, where the type requires a Rust
education to decode.
