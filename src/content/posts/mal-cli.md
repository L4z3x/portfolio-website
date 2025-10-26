---
title: "Mal‑Cli: Terminal MyAnimeList Client"
published: 2025-06-11
description: "A fast, keyboard-driven terminal client for MyAnimeList, built in Rust using Ratatui — browse, search, and manage your anime/manga right from the terminal."
image: '/projects/mal-cli/demo.gif'
tags: ["Rust"]
category: 'Project'
draft: false 
lang: 'en'
---

## Overview

**mal‑cli** is a high-performance terminal interface for the official [MyAnimeList](https://myanimelist.net/) API, written in Rust. It provides fast, keyboard-driven navigation across anime and manga lists in a visually sleek TUI using the Ratatui crate. Designed for efficiency, it supports GPU-enhanced terminals and works across Linux, macOS, Windows, and musl environments.

## Key Features

- **Ratatui-based TUI**: Modern, widget-driven interface with popups and keyboard navigation.

- **Cross-platform support**: GPU rendering (kitty, Windows Terminal 1.22+), compiled for Linux, macOS, Windows, and musl.

- **Keyboard-first UX**: Navigate with keys like s, r, Ctrl-p, and Esc; fast paging and popup switching.

- **Easy installation**: Available on crates.io, AUR (yay -S mal-cli), Debian and through github releases

## Technologies Used:

- **Rust**: for performance and memory safety

- **Ratatui**: A Rust TUI crate

- **Tokio async runtime**: for API calls and event handling

- **Crossterm**: low-level terminal I/O

- **serde / reqwest / tokio-rustls**: for HTTP, JSON parsing, and TLS

- **Image support**: via ratatui-image for inline graphics on supported terminals


## Challenges and Lessons:

- Managing async events and popups required careful structuring to avoid input lag and race conditions

- Creating AUR/DPKG packages from Rust binaries taught me packaging nuances across Linux distributions

## Outcome:

- Gained 111 stars ⭐ on [GitHub](https://github.com/L4z3x/mal-cli).

- Featured on Rust forums and r/linux for delivering a sleek, native-like TUI experience
    
- Continuously released updates (latest v0.2.1 on 2025‑06‑11), with support for multiple OS targets and easy installation across platforms