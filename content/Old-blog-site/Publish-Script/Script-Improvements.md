---
title: Publish Script Improvements
description: Later fixes to the publish script — preview before publishing, automatic dates, error handling, and required-field checks.
tags:
  - Bash
  - Automation
  - Fixing
---
# Context

After using the [[Script-Overview|original publish script]] for a couple of posts, I noticed four gaps:

- No preview before publishing.
- The publish date had to be set by hand (error-prone).
- The script kept going after hitting an error instead of stopping.
- The script didn't stop if a post was missing tags or a description.

I addressed these as follow-up changes to the same script, now archived at [`Archives/blog_publication.sh`](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Archives/blog_publication.sh) in the Project-Dump repo, not as a separate tool.

# Preview before publishing

I made the script start Hugo's dev server in the background and open it straight to the posts listing:

```bash
hugo server -D --disableFastRender > /dev/null &
xdg-open "http://localhost:1313/posts/"
```

`> /dev/null` silences the server's console output so it doesn't clutter the terminal, and the trailing `&` backgrounds the server so the script can continue instead of blocking on it.

![[blog-script-xdg-open-preview-browser-launch.png]]
*`xdg-open` launching the default browser straight to the local preview URL from the terminal — confirms the command opens the correct target rather than failing silently or opening the wrong app.*

I then had the script loop on a yes/no prompt asking whether to publish; either answer kills the background Hugo server, and answering "no" exits the script without publishing.

# Auto-updating the publish date

I now have the current date (`date +%Y-%m-%dT00:00:00`) inserted into each post's front matter automatically via `sed -i "s/^date:.*/date: $publish_date/" index.md`, removing a manual step I didn't trust myself to remember.

# Auto-exit on error

I added `set -euo pipefail` (see [[Publish-script-glossary#`set -euo pipefail`|glossary]]) to the top of the script so it stops instead of continuing past a failure.

Turning this on surfaced a real collision: the backlink-extraction regex from [[Script-Overview#Resolving cross-post links via a slug lookup file|the slug-lookup step]] was also matching unrelated `[[ ]]` Bash conditionals elsewhere in the script (e.g. `[[ "$choice" == "Y" || "$choice" == "y" ]]`). My fix checks whether the matched `[[ ]]` content contains a `$` — if so, it's a Bash conditional, not an Obsidian backlink, and gets skipped (`if [[ $line != *\$* ]];`).

A second issue: `git add *` was erroring because some files are excluded by `.gitignore`. Rather than enumerate every file explicitly, I changed the line to `git add * || true` (see [[Publish-script-glossary#`|| true`|glossary]]) so that specific, expected failure doesn't abort the whole script.

# Auto-exit on missing tags or description

Since the script already extracts a post's tags and description while converting it, I now have it check both immediately after extraction and abort (deleting the partially-copied post folder so a re-run starts clean) if either is empty:

```bash
if [[ -z "$tags" ]]; then
    echo "Error, the $md_article is missing tags. Add them in and go again. Will be deleting the article folder to make it easier to re-run this script."
    rm -r "$ARTICLE_PATH"
    exit 1
fi
```

The description check works the same way, using `grep`/`awk` to pull `description:` out of the front matter first.

# Related

- [[Script-Overview]] — the base script these changes were made to.
