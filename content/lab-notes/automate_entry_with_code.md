+++
date = '2026-05-14T22:43:45-03:00'
title = 'Automate HUGO labnotes'
tags = ["how-to"]
+++

## Use a Shell function (recommended — zero dependencies)

Add this to ~/.zshrc:

```bash
labnote() {
  local slug="${1:-}"
  if [[ -z "$slug" ]]; then
    read "slug?Lab note slug: "
  fi
  slug="${slug// /_}"   # spaces → underscores
  (cd ~/code/thetasigma-site && hugo new "lab-notes/${slug}.md" && code "content/lab-notes/${slug}.md")
}
```

Then `labnote thetasigma_labs_setup` or just `labnote` and it prompts you. Opens it in VSCode immediately. This is almost certainly what you want — it's a 30-second setup and you never leave the terminal.

## Setup to create lab notes

The `code` command isn't installed by default on macOS — VSCode ships it but you have to enable it. Two ways:

### Option 1: From inside VSCode (easiest)

1. Open VSCode
2. ⌘⇧P to open the command palette
3. Type: Shell Command: Install 'code' command in PATH
4. Hit enter — it adds a symlink to `/usr/local/bin/code`
5. Then `source ~/.zshrc` (or open a new terminal) and code will work.

### Option 2: Add it to PATH manually

If option 1 doesn't appear or fails, add this to `~/.zshrc`:

```zsh
export PATH="/Applications/Visual Studio Code.app/Contents/Resources/app/bin:$PATH"
Reload with source ~/.zshrc.
``
