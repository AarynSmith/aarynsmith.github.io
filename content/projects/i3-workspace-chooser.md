---
title: "I3 Workspace Chooser"
subtitle: ""
date: 2018-05-25
tags:  []
draft: false
comments: false
---

An interactive workspace layout restoring script.
<!--more-->

I created this script to have an automated way to apply repeatable window layouts to my I3 workspace. It uses a main shell script that gets a list of JSON and shell scripts in a directory that define a workspace, and the applications to start. It then feeds that list into rofi, for user selection, then runs the associated shell script to restore the workspace. 

More information can be found in my blog post [here](/blog/i3-workspace-chooser/). The script is available on my GitHub [here](https://github.com/AarynSmith/I3-Workspace-Chooser)
