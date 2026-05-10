---
description: The editor of God?
pubDatetime: 2022-02-25
tags:
- Others
title: Emacs First Look
---

I start to use emacs for text editing and rust developing.

I choose Doom Emacs as the integrated configuration.

# Install Emacs

Install Emacs 27 or 28, nativecomp version better

# Install Rust

I use the UNIX like systems, Ubuntu/MacOS:

```bash
curl https://sh.rustup.rs -sSf | sh
```

Then try to run `cargo`

# Install Doom Emacs

## Install ripgrep and fd-find

```bash
cargo install ripgrep
cargo install fd-find
```

## Install Doom

```bash
git clone --depth 1 https://github.com/hlissner/doom-emacs ~/.emacs.d
~/.emacs.d/bin/doom install
```

Edit `init.el` to set rust and lsp

```bash
vim ~/.doom.d/init.el
```

Under `:tools` , uncomment `lsp`

Under `:lang`  , uncomment/add `(rust +lsp)`

Sync the config

```bash
~/.emacs.d/bin/doom sync
```