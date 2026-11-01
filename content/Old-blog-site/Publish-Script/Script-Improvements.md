---
title: Publish Script Improvements
description: Later fixes to the publish script — preview before publishing, automatic dates, error handling, and required-field checks.
tags:
  - Bash
  - Automation
  - Fixing
---

## Overview

This page documents later fixes to the publish script — preview before publishing, automatic dates, error handling, and required-field checks.

## Context

After using the original publish script for a couple of posts, four gaps were noticed:

- No preview before publishing
- The publish date had to be set by hand (error-prone)
- The script kept going after hitting an error instead of stopping
- The script didn't stop if a post was missing tags or a description

These were addressed as follow-up changes to the same script, now archived at [`Archives/blog_publication.sh`](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Archives/blog_publication.sh) in the Project-Dump repo, not as a separate tool.

## Preview Before Publishing

The script was modified to start Hugo's dev server in the background and open it straight to the posts listing:

```bash
hugo server -D --disableFastRender > /dev/null &
xdg-open "http://localhost:1313/posts/"
```

`> /dev/null` silences the server's console output so it doesn't clutter the terminal, and the trailing `&` backgrounds the server so the script can continue instead of blocking on it.

See [xdg-open preview browser launch](Images/blog-script-xdg-open-preview-browser-launch.png) for `xdg-open` launching the default browser straight to the local preview URL from the terminal — confirming the command opens the correct target rather than failing silently or opening the wrong app.

The script then loops on a yes/no prompt asking whether to publish; either answer kills the background Hugo server, and answering "no" exits the script without publishing.

## Auto-Updating the Publish Date

The current date (`date +%Y-%m-%dT00:00:00`) is now inserted into each post's front matter automatically via `sed -i "s/^date:.*/date: $publish_date/" index.md`, removing a manual step that wasn't trusted to be remembered.

## Auto-Exit on Error

`set -euo pipefail` was added to the top of the script so it stops instead of continuing past a failure.

Turning this on surfaced a real collision: the backlink-extraction regex was also matching unrelated `[[ ]]` Bash conditionals elsewhere in the script (e.g. `[[ "$choice" == "Y" || "$choice" == "y" ]]`). The fix checks whether the matched `[[ ]]` content contains a `$` — if so, it's a Bash conditional, not an Obsidian backlink, and gets skipped (`if [[ $line != *\$* ]];`).

A second issue: `git add *` was erroring because some files are excluded by `.gitignore`. Rather than enumerate every file explicitly, the line was changed to `git add * || true` so that specific, expected failure doesn't abort the whole script.

## Auto-Exit on Missing Tags or Description

Since the script already extracts a post's tags and description while converting it, it now checks both immediately after extraction and aborts (deleting the partially-copied post folder so a re-run starts clean) if either is empty:

```bash
if [[ -z "$tags" ]]; then
    echo "Error, the $md_article is missing tags. Add them in and go again. Will be deleting the article folder to make it easier to re-run this script."
    rm -r "$ARTICLE_PATH"
    exit 1
fi
```

The description check works the same way, using `grep`/`awk` to pull `description:` out of the front matter first.

## Related

- [[Script-Overview]] — the base script these changes were made to
