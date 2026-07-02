# Example 10: A Log-Parsing Regular Expression

Regular expressions might be the least readable notation in mainstream use.
Here is one that parses a web server access log line - sixty-odd characters
that most engineers would rather rewrite than read.

## The original

```text
^(\d{1,3}(?:\.\d{1,3}){3}) - (\S+) \[([^\]]+)\] "(GET|POST|PUT|DELETE) ([^" ]+)[^"]*" (\d{3}) (\d+|-)
```

## The Pseudo translation

```python
# PARSE ONE LINE OF A WEB SERVER ACCESS LOG

Starting from the very beginning of the line, expect, in this exact order:

    The visitor's IP address:
        One to three digits,
        then three more groups of a dot followed by one to three digits.
        Capture the whole address.

    A spacer: space, hyphen, space.

    The username:
        Any unbroken run of non-space characters.
        Capture it.

    The timestamp, wrapped in square brackets:
        Capture everything up to the closing bracket.

    The request, wrapped in double quotes:
        The method, which must be exactly GET, POST, PUT, or DELETE.
        Capture it.
        A single space, then the path:
            everything up to the next space or quote. Capture it.
        Silently skip the rest of the quoted section,
        which holds the protocol version nobody asked about.

    The status code: exactly three digits. Capture it.

    The response size:
        either a run of digits,
        or a lone hyphen meaning "no body was sent".
        Capture it.

If the line deviates from this shape at any point:
    The entire match fails and nothing at all is captured.
```

## What the translation reveals

A regex is a contract about the shape of text, and the Pseudo states the
contract's most brutal clause in plain words: deviate anywhere and you get
nothing. It also surfaces the small editorial decisions buried in the
pattern - the protocol version is deliberately matched but not captured,
and a hyphen in the size field is a meaningful value, not a formatting
quirk. This is also the example where Pseudo most obviously beats
line-by-line code comments: there is no line structure here to comment on,
only a shape, and shapes want to be described top to bottom.
