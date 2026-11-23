# A faster makefun-site

This is a drop-in for makefun-site 1.0.0, written to answer exactly as it
does and to do it quicker. One file, `solve.js`, holds the whole build; the
driver in `main.ts` is the one the house ships, unchanged apart from the
comments.

## How it gets its speed

The reference program is careful and easy to follow, and that is also why it
is slow: it tests every line of every body against a handful of regular
expressions, renders its inline text one character at a time, and walks a
chain of scopes for every value a template reads.

This one keeps the same answers and takes shorter paths to them:

- Markdown is read and rendered in a single pass over the lines, with no
  token objects built along the way.
- Inline text is scanned for its special characters in one go. The stretches
  in between hold nothing the escape table names, so they are copied as they
  stand instead of being escaped a character at a time.
- Slugs are remembered. Titles and tag names repeat a lot across a site, and
  the Unicode normaliser is skipped outright for plain ASCII, which has
  nothing to decompose.
- Each template is compiled once into a plain function that reads its values
  straight off the page. Includes are written out where they stand, and only
  the five layouts are ever compiled from here.

Nothing here reads a clock, the environment, a file, or any randomness, so
one input always builds the same tree. Two runs of the same set write
identical answers.

## Checking it

The kit's practice set is the check that matters. From the kit directory,
with this entry's folder as the working directory:

```
MAKEFUN_INPUT_DIR=<kit>/practice/input MAKEFUN_OUTPUT_DIR=/tmp/out node main.ts
```

The answers in `/tmp/out/answers.json` match the kit's
`practice/reference/expected.json` on all three cases. The same was checked
against three freshly drawn sets from the kit's generator, nine more cases,
all matching.

## Numbers, on this machine

Against the practice set, warmed up and in-process: about 135 ms for the
three cases, where makefun-site itself takes about 890 ms. Through the
driver in a fresh process: about 590 ms wall against the house baseline's
about 1,070 ms, the no-op control sitting near 170 ms of that.
