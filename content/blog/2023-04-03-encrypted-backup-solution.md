---
layout: post
title: "An idiot's guide to encryption and backup on linux"
date: 2023-04-03
draft: true
tags:
    - encryption
    - linux
    - backups
    - devops
---

This guide is an exploration of tools I've tried in the encryption and backup space, and the tools I've settled on as of early 2023. To give some context:

- I'm an idiot when it comes to tooling and have no patience for detailed configurations I can easily break
- I cannot be trusted to remember anything except a handful of passwords so backups must be automatic and simple
- I'm terrified of lock-in so my tools must be open and I should be able to switch tools in less than a week
- I recently moved away from hosted cloud to a simple NAS server running UnRAID where I store most backups

Generally a backup solution should incorporate three copies of any *important* data, preferably with one being stored offsite. For me, that looks like:

- local files on my laptop
  - some files synced to my phone
- Backups on my NAS
- Backups on a Raspberry Pi at a friend's house

## Encryption: from ecryptfs to LUKS

Being

In order to make sure my files aren't stolen 


- Drive encryption
- Backups
- Filesystems

Recently I've become more and more worried about data corruption, not just bitrot, which is rare enough to not be a risk for me personally, but corruption through files being incorrectly copied, transferred or moved, files being used incorrectly by software, or other ways that files could be messed with. For this reason, I've been researching software that can help mitigate corruption and prevent file loss. 

## My previous setup and why I was unhappy

Since its release I've used [EndeavourOS](https://endeavouros.com/), a variant of Arch Linux that is "A terminal-centric distro with a vibrant and friendly community at its core." I use it because its easy to install[^lfs], comes bundled with a good-enough-looking XFCE config, but I still have access to the well-oiled Arch underbelly (and the excellent wiki that comes with Arch). For this exploration 

[^lfs]: Before anyone complains my geek credentials are invalid because I like an easy install, I setup and ran a Linux From Scratch machine for a while. The easier something is to install, the less likely I am to break it and I would rather spend my life doing other things than installing OSs incorrectly. [LFS](https://www.linuxfromscratch.org/) is loads of fun if you're into that though, I do recommend it.
