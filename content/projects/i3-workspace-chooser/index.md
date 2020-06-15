---
title: "I3 Workspace Chooser"
subtitle: ""
date: 2018-05-25
draft: false

tags: []
categories: ["Projects"]

hiddenFromHomePage: true

featuredImage: "featured-image.png"
featuredImagePreview: "preview-image.png"

comments:
  enable: false
toc:
  enable: false
  auto: true
math:
  enable: false
---

An interactive workspace layout restoring script.
<!--more-->

I created this script to have an automated way to apply repeatable window layouts to my I3 workspace. It uses a main shell script that gets a list of JSON and shell scripts in a directory that define a workspace, and the applications to start. It then feeds that list into rofi, for user selection, then runs the associated shell script to restore the workspace.

More information can be found in my blog post [here](/posts/i3-workspace-chooser/). The script is available on my GitHub [here](https://github.com/AarynSmith/I3-Workspace-Chooser)
