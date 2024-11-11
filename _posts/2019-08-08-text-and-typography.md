---
title: Exec spawning
description: How exec spawning works
author: cotes
date: 2024-10-17 11:33:00 +0800
categories: [Tech, Android]
tags: [GrapheneOS]
pin: false
math: true
mermaid: true
---
# Exec Spawning

GrapheneOS creates fresh processes (via `exec`) when spawning applications instead of using the traditional Zygote spawning model. This approach improves privacy and security at the expense of higher cold start app spawning time and increased initial memory usage. However, it does not impact runtime performance beyond the initial spawning time. 

On flagship devices, the additional spawning time is approximately **200 ms**, and it is more noticeable on lower-end devices with weaker CPUs and slower storage. The impact on spawning time only applies when the app does not already have an app process; the OS will attempt to keep app processes cached in the background until memory pressure necessitates their termination.

## Zygote Model Overview

In the typical Zygote model, a template app process is created during boot, and every app is spawned as a clone of this template. This leads to all apps sharing the same initial memory content and layout, including sharing secrets that should be randomized for each process. The Zygote model saves time by reusing the initialization work and reduces initial memory usage due to copy-on-write semantics, which allows memory written only during initialization to be shared between app processes.

### Security Implications of the Zygote Model

The Zygote model undermines the security provided by features that rely on random secrets, such as:

- **Address Space Layout Randomization (ASLR)**
- **Stack Canaries**
- **Heap Canaries**
- **Randomized Heap Layout**
- **Memory Tags**

This model significantly diminishes the effectiveness of these security features, as every app has access to the values of every other app, and these values do not change for fresh app processes until a reboot occurs. Additionally, much of the OS itself is implemented via non-user-facing apps with privileges reserved for OS components. The Zygote template is reused across user profiles, temporarily sharing a set of device identifiers across profiles for each boot through these shared randomized values.

## Configuration Option

This feature can be disabled via:

