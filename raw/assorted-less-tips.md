---
url: https://blog.thechases.com/posts/assorted-less-tips/
date_fetched: 2026-07-05
backfilled: true
---

# Assorted less(1) tips

In
a
recent discussion on Reddit
I shared a number of tips
about the common utility
`less(1)`
that others found helpful
so I figured I'd aggregate some of those tips here.

### Operating on multiple files

While most folks invoke
`less`
at the tail of a pipeline like
you can directly provide one or more files to open

### Adding files after starting

When reading a document,
sometimes you want to view another file,
adding it to the file list.
Perhaps while reading some C source code
you want to also look over the corresponding header-file.
You can add that header-file to the argument list with
`:e file.h`

### Navigating multiple files

You can navigate between multiple files using
`:n`
to go to the next file in the argument-list, and
`:p`
for the previous file.
You can also use
`:x`
to rewind to the first file in the argument-list
similar to how
`:rewind`
behaves in
`vi`/`vim`.

### Removing files after starting

While I rarely feel the need to,
if you have finished with a file
and want to keep your argument list clean,
you can use
`:d`
to delete the current file from the argument-list.

## Navigating

### Jumping to a particular line-number

Use
`«count»`G`3141`G`3141`.
It helps to display
line numbers.

### Jumping to a particular percentage-offset

Similarly, using
`«count»`%`75%`
to jump right there.

### Searching

While many folks know you can search forward with
`/`
and some people know you can use
`«pattern»``?`
to search backwards,
or use
`«pattern»``n`/`N`
to search again for the next/previous match,
`less`
provides modifiers you can specify
before the pattern to modify its behavior:

- `!`
- 
  Find the next line that
  **doesn't**match the pattern
- `*`
- search across multiple files, starting from the current location in the current file
- `@`
- rewind to the first file and search from there
- `@*`
- rewind to the first file and search from there across multiple files

Thus you would use
`/@*`
to search for "pattern"
starting with the first file.
`«pattern»`

## Filtering lines

Using
`&`
lets you specify a pattern
and filter the displayed lines
to only those matching the pattern,
much like an internal
`grep`
command.
If you modify it with
`!`,
so it will display only those lines that do
*not match*
the pattern, like
`&!`.
I find this particularly helpful for browsing log-files.
`«pattern»`

## Bookmarking

You can bookmark points in a file with
`m`
followed by a letter,
then jump back to that bookmark with
`'`
followed by the same letter.
These apply globally across all open files,
so if you
`ma`
in the third file,
then navigate away to other files,
using
`'a`
will take you back to the marked location
in that third file.
I use marks most when reading man-pages,
dropping one mark at the
`OPTIONS`
section such as
`mo`,
and another at the
`EXAMPLES`
section,
such as
`me`,
then bounce back and forth between them with
`'o`
and
`'e`.
While you can use any of
the 26 lowercase or uppercase letters
(for a total of 52 marks),
I rarely use more than two or three
either in alphabetical order
("a", "b", "c"),
or assigning mnemonics like in the
`man`-page example above.

### Bracket matching

If the first line on the screen contains a
`(`,
`[`,
or
`{`,
typing that character
will jump to the matching/closing character,
putting it on the bottom line of the screen.
Similarly, if a closing
`)`,
`]`,
or
`}`,
character appears on the last line,
typing that closing character
will jump to the matching/opening character,
putting it at the top of the screen.
I find it a little disorienting
if they fall `less` than a screen-height apart
because what feels like a forward motion
to find the next matching close-bracket
might actually result in shifting the screen
*down*
rather than
*up*
which feels backwards.

While I don't use it much,
you can also specify match-pairs using
`alt`+`ctrl+f``alt`+`ctrl+b``alt`+`ctrl+f<>``(`/`)`,
`[`/`]`,
and
`{`/`)`
motions.

## Toggling options without restarting

While the
`man`-page
documents many flags you can pass
on the command-line,
you can also toggle boolean options from inside
`less`.
I find this particularly helpful
when I've fed the output of a long-running process to
`less`
and don't want to re-run it
because it will take a long time.
Instead of quitting,
you can type a literal
`-`
followed by the option you want to change.
I most commonly want to toggle word-wrap for long lines,
so instead of quitting and adding
`-S`
at the end of my pipeline,
I can type
`-S`
directly in
`less`.
Options I commonly toggle:

- `-S`
- word-wrap (mnemonic "splitting long lines")
- `-G`
- search-highlighting
- 
`-i`/`-I`
- smart-case/case-sensitivity for searches
- `-R`
- ANSI-color escaping
- 
`-N`/`-n`
- show/hide line-numbers

## Running external commands

The
`!`
lets you invoke an external command.
I don't do this often,
but occasionally I want some simple reference
like the current date
(`!date`)
or to do some simple math
(`!bc`).

## Default options with `$LESS`

You might find yourself regularly setting
a common group of
options
so you can put those in your environment
(usually in your shell startup file like
`~/.bashrc`)
like
`LESS="-RNe"`
if you want to
show ANSI colors,
show line-numbers,
and exit automatically when you reach the end of the file.

## Other misc

`less`
has a few other corners that I've never really used,
but figured I'd document here:

### Tags

While I've used tags in
`vi`/`vim`
to easily jump between definitions.
However, even though
`less`
provides support for tags generated by
`ctags`.
I've never found cause to use them.

### Editing the current document

The
`v`
command will open your
`$VISUAL`
editor on the current document.

### "Log" output

`less`
lets you redirect the output
it has gathered from
stdin
to a file using the
`o`
command
(or the
`O`
command to overwrite an existing file).
This might come in handy because
`less`
won't let you
edit stdin
in an external editor
but you can write it directly to a file.
