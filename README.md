# Dotfiles Repository Guide

This repo is the top-level controller for your shell/editor/terminal config.
It is intentionally small and uses submodules for each config domain.

## Repository Layout

- `dotfiles` (this repo): meta repo that pins versions of child config repos.
- `nvim/`: Neovim config repo (`git@github.com:stirlingwdonaldson/nvim.git`).
- `tmux/`: tmux config repo (`git@github.com:stirlingwdonaldson/tmux.git`).
- `zsh/`: zsh config repo (`git@github.com:stirlingwdonaldson/zsh.git`).

Each child directory is a git submodule, not a normal folder.

## Core Conventions

1. Keep each tool's config in its own repo (`nvim`, `tmux`, `zsh`).
2. Use this parent repo only to pin submodule commits and document setup.
3. Do not vendor plugin repos inside dotfiles.
4. Install plugins at setup/runtime using each plugin manager.
5. Keep `main` as the default branch in parent and child repos.

## First-Time Setup (New Machine)

```bash
git clone --recurse-submodules git@github.com:stirlingwdonaldson/dotfiles.git ~/dotfiles
cd ~/dotfiles
git submodule update --init --recursive
```

## Link Config Files

This repo stores config in subfolders; your tools read config from home paths.
Use symlinks:

```bash
# zsh
ln -sfn ~/dotfiles/zsh/.zshrc ~/.zshrc

# tmux (choose one style and stay consistent)
ln -sfn ~/dotfiles/tmux/tmux.conf ~/.tmux.conf
mkdir -p ~/.config/tmux
ln -sfn ~/dotfiles/tmux/tmux.conf ~/.config/tmux/tmux.conf

# nvim
mkdir -p ~/.config/nvim
ln -sfn ~/dotfiles/nvim/init.lua ~/.config/nvim/init.lua
```

## tmux Plugins (TPM)

`tmux/plugins/*` is intentionally not tracked in git.

Install TPM and plugins:

```bash
# Remove old stale symlink if present from previous layout
if [ -L ~/.tmux/plugins ]; then rm ~/.tmux/plugins; fi

mkdir -p ~/.tmux/plugins
test -d ~/.tmux/plugins/tpm || git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
~/.tmux/plugins/tpm/bin/install_plugins
```

Manual install/update from tmux is still available via TPM keybindings.

## Daily Workflow

### Change one config repo

Example: tmux changes.

```bash
cd ~/dotfiles/tmux
git switch main
git pull --ff-only

# edit files

git add .
git commit -m "Update tmux config"
git push
```

### Update parent pointer after child repo commit

```bash
cd ~/dotfiles
git add tmux
git commit -m "Bump tmux submodule"
git push
```

Repeat the same pattern for `nvim` and `zsh`.

## How to Read Git Status Correctly

From `~/dotfiles`:

- `modified: tmux (new commits)` means submodule HEAD moved.
- `modified: tmux (modified content)` means uncommitted changes inside submodule.
- You will not see normal per-file diffs for submodule content in parent repo.

From `~/dotfiles/tmux` (or `nvim`, `zsh`):

- You see normal file-level git status and diffs.

## Syncing and Updating

To pull latest parent + submodule pointers:

```bash
cd ~/dotfiles
git pull --ff-only
git submodule update --init --recursive
```

To update all submodules to latest remote heads intentionally:

```bash
cd ~/dotfiles
git submodule foreach 'git switch main && git pull --ff-only'
git add nvim tmux zsh
git commit -m "Bump submodules"
git push
```

## Troubleshooting

### `fatal: could not create leading directories ... ~/.tmux/plugins/tpm`

Cause: parent directory missing or old broken symlink at `~/.tmux/plugins`.

Fix:

```bash
rm -f ~/.tmux/plugins
mkdir -p ~/.tmux/plugins
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### Submodule looks dirty in parent repo

Check inside the submodule:

```bash
git -C ~/dotfiles/tmux status
```

Commit/push there, then stage pointer in parent:

```bash
cd ~/dotfiles
git add tmux
git commit -m "Bump tmux submodule"
```

### Fresh clone missing submodule content

```bash
git submodule update --init --recursive
```

## Optional Quality-of-Life Aliases

```bash
alias df='cd ~/dotfiles'
alias dfn='cd ~/dotfiles/nvim'
alias dft='cd ~/dotfiles/tmux'
alias dfz='cd ~/dotfiles/zsh'
```

## Safety Rules

- Make config edits in child repos, not by bypassing submodule git history.
- Do not commit secrets in any repo.
- Avoid force-push on shared branches unless you explicitly mean to rewrite history.
- Prefer small, focused commits in each child repo.

## Quick Command Reference

```bash
# parent status + submodule summary
git -C ~/dotfiles status
git -C ~/dotfiles submodule status --recursive

# child repo status
git -C ~/dotfiles/nvim status
git -C ~/dotfiles/tmux status
git -C ~/dotfiles/zsh status
```
