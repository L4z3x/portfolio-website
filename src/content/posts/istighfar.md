---
title: "Istighfar GNOME shell Extension: Periodic Adkar Tool"
published: 2025-05-20
description: 'A GNOME shell extension that reminds users to recite *Istighfar* at regular intervals, built with JavaScript and leveraging GNOME native UI.'
image: "/projects/istighfar/popup.png" 
tags: ["GNOME","JavaScript","Extension"]
category: 'Project'
draft: false 
lang: ''
---

## Overview

**Istighfar** is a GNOME Shell extension written in JavaScript that periodically reminds users to recite *Istighfar* (seeking forgiveness). It integrates seamlessly with the GNOME top panel, providing unobtrusive notifications at configurable intervals. This lightweight, open-source tool aims to bridge daily spiritual practice and desktop productivity.

---

## Key Features

- **Panel-integrated reminders** displayed as native GNOME notifications  
- **Configurable interval** for reminders via GNOME Extension settings  
- **Minimal resource usage** — built with standard GNOME JS APIs  

---

## Technologies Used

- **GNOME Shell / GJS** – for building the extension  
- **JavaScript** – primary language for logic and UI  
- **GNOME Settings Schema** – for extension configuration  
- **CSS** – for styling popup and indicator components


## Skills Learned

- Building GNOME Shell extensions and understanding `metadata.json`, `extension.js`, and `prefs.js` structure  
- Integrating with GNOME’s settings system and translating configurations into user-friendly popups  
- Using GJS (GNOME JavaScript) and CSS styling within a system-level environment  
- Managing timed tasks and notifications in a desktop context


## Challenges and Lessons

- Handling **GNOME Shell lifecycle events** (enable, disable) to ensure stable operation across sessions  
- Working with native **GSettings schemas** and translating toggles into actionable reminder state  
- Designing a **non-intrusive UX** that respects both GNOME guidelines and user context  


## Outcome

- nearly 20 stars ⭐ on [GitHub](https://github.com/L4z3x/Istighfar-gnome-extension)  
- used by nearly 200 people on [gnome extensions site](https://extensions.gnome.org/extension/7902/istighfar/)
- Successfully published and used daily by several GNOME Shell users  
- Demonstrates strong understanding of JS, desktop UX, and system integration through GNOME

