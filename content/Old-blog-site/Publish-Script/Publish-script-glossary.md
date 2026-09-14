---
title: Publish-Script Glossary
description: Terms used across the Publish-Script bucket's pages.
tags:
  - Bash
  - Glossary
---
# Glossary

## `set -euo pipefail` 
A Bash safety idiom: `-e` exits on any command error, `-u` errors on use of an undefined variable, and `-o pipefail` makes a pipeline fail if any command in it fails (not just the last one). See the [Stack Overflow answer this was based on](https://stackoverflow.com/a/2871034) and the [Bash reference manual's Set Builtin section](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#The-Set-Builtin).

## `IFS= read -r line` 
A pattern for reading a command's output one line at a time inside a `while` loop, typically as `while IFS= read -r line; do ... done < <(some_command)`. Setting `IFS=` (empty) stops Bash from trimming leading/trailing whitespace off each line, and `-r` stops `read` from treating backslashes as escape characters. I relied on this in the publish script to iterate over grep/awk output (e.g. slug lookups, backlink matches) line-by-line without the loop silently mangling filenames or paths that contain spaces or backslashes.

## Backgrounding (`&`) and `/dev/null` 
Appending `&` to a command runs it in the background so the script can continue (see [background command basics](https://linuxize.com/post/how-to-run-linux-commands-in-background/)); redirecting output to `/dev/null` (see ["What is /dev/null?"](https://www.geeksforgeeks.org/linux-unix/what-is-dev-null-in-linux/)) silences a command that would otherwise clutter the terminal.

## `xdg-open` 
A Linux command that opens a given file or URL with whatever application the desktop environment has configured as the default for it (e.g. a browser for an `http://` URL, an image viewer for a `.png`). I use it in the improved publish script to automatically pop open the local Hugo preview server in the default browser (`xdg-open "http://localhost:1313/posts/"`) instead of having to switch windows and type the URL myself every time I want to preview a post before publishing. See the [Stack Overflow reference](https://stackoverflow.com/questions/38147620/shell-script-to-open-a-url/38147878#38147878).

## `|| true` 
Appending this to a command line makes the script treat that command's failure as a non-failure, so `set -e` won't abort the script over it. See [when to use `|| true` in Bash](https://unix.stackexchange.com/a/716559).

## Git worktree 
A Git feature that checks out an additional branch into its own separate working directory, linked back to the same repository. See [[GitHub-Pages/Github-pages-glossary#Git worktree|the GitHub-Pages glossary entry]] for how this was used (and later dropped) for this site.

For Hugo-specific terms referenced from this bucket's pages (leaf bundle, front matter, etc.), see [[Hugo/Hugo-glossary|the Hugo glossary]].
