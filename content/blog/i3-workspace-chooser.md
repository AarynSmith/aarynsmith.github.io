---
title: "I3 Workspace Chooser"
subtitle: "Creating an workspace chooser for I3"
date: 2018-05-25
tags: ["i3wm", "linux", "scripting"]
draft: false
comments: true
---

After working with I3 I've got a few window layouts I like for various tasks.
<!--more-->

This script is available on my [Github](https://github.com/AarynSmith/I3-Workspace-Chooser).

I'm a creature of habit, and I like to have my window layouts in a consistent layout. I have one layout for working on this blog, two columns. The first column has VIM, a free terminal, and hugo server, and the second column is a browser window showing a preview of my blog as I update it. Another layout I've been using while writing the [Project Euler](/tags/euler/) posts, which is similar to the blog layout, but one less terminal window on the left, and all of the terminals are pointed to a different directory.

![Screenshot](/img/I3-Workspace-Chooser-Screenshot.png)_Here you can see my blogging layout, as well as my Workspace chooser._

I'm already using [rofi](https://github.com/DaveDavenport/rofi) for a program launcher, so I wanted to use it's dmenu replacement mode to display a list of predefined workspaces, and run a script based on the selection.

Now, when I want to setup a new workspace, I can open each of the windows I want and arrange them to my liking, then use [i3-save-tree](https://i3wm.org/docs/layout-saving.html) to save the layout to a JSON file and modifying it , then I create a shell script with a series of i3-msg commands to launch various programs. The issue I ran into however was I wanted the terminals to open in specific directories, and run a program, but then to remain open after the program (i.e. VIM) had closed. In order to accomplish this I had to add a small tweak to my shell's rc file.

```bash
if [[ $1 == eval ]]
then
    "$@"
fi
```

This snippet tells your shell that if the first argument is `eval`, to execute the following arguments. This allowed me to build up a command in my workspace shell script below to start a program in a specific target directory.

```bash
i3-msg 'workspace WorkSpaceName; exec "i3-sensible-terminal -e zsh -is eval cd ~/Target_Directory/\; programName"'
```

This runs `i3-msg` telling it the workspace context (WorkSpaceName), and telling i3 to execute a command. The command being executed is `i3-sensible-terminal` which launches my terminal `urxvt` with the `-e` option, which runs another command. That command is my shell `zsh` with the `-i` and `-s` options. The `-i` option forces the shell to be interactive, and the `-s` reads the input from the command line. The command being run is `eval` which the snippet from above tells the shell to execute the following arguments, in this case a `cd` to a directory and then a program name.

This combined with the `i3-msg` append_layout command to restore a `i3-save-tree` JSON layout to a workspace can give us the ability to restore a complete workspace. So I wrote a script that will look for the JSON layout files, format them for rofi, then execute the associated shell script with the layout chosen.

```bash
shopt -s extglob
declare -A dmenu
WS_DIR=~/Scripts/Workspaces/
for file in ${WS_DIR}*.json
do
    dmenu+=([`basename ${file}`]=${file})
done
workspace=$(echo "$(printf "%s\n" "${!dmenu[@]}")" | rofi -i -dmenu)

if [ ! -z "$workspace" ];
then
    scriptname="${WS_DIR}$(basename ${dmenu[${workspace}]} .json).sh"
    exec sh ${scriptname}
fi
```

This combined with your JSON and shell script files will show you a rofi menu, allowing you to select the workspace you want to launch, then launch the layout on the workspace defined in the shell script. Below is a sample JSON and shell script that will launch two terminals, one running vim and one open and a web browser on workspace 9. You will notice in the shell scripts, a short sleep between opening similar windows (i.e. two terminals). This is to prevent the terminals from opening in the wrong order.

```json
// vim:ts=4:sw=4:et
{
    // splitv split container with 2 children
    "border": "normal",
    "floating": "auto_off",
    "layout": "splitv",
    "percent": 0.5,
    "type": "con",
    "nodes": [
        {
            "border": "pixel",
            "current_border_width": 1,
            "floating": "auto_off",
            "geometry": {
               "height": 390,
               "width": 566,
               "x": 0,
               "y": 0
            },
            "name": "vim",
            "percent": 0.5,
            "swallows": [
               {
               "class": "^URxvt$"
            ],
            "type": "con"
        },
        {
            // splith split container with 1 children
            "border": "normal",
            "floating": "auto_off",
            "layout": "splith",
            "percent": 0.5,
            "type": "con",
            "nodes": [
                {
                    "border": "pixel",
                    "current_border_width": 1,
                    "floating": "auto_off",
                    "geometry": {
                       "height": 390,
                       "width": 566,
                       "x": 0,
                       "y": 0
                    },
                    "name": "URxvt",
                    "percent": 1,
                    "swallows": [
                       {
                       "class": "^URxvt$"
                    ],
                    "type": "con"
                }
            ]
        }
    ]
}

{
    "border": "pixel",
    "current_border_width": 1,
    "floating": "auto_off",
    "geometry": {
       "height": 600,
       "width": 800,
       "x": 50,
       "y": 50
    },
    "name": "DuckDuckGo — Privacy, simplified. - qutebrowser",
    "percent": 0.5,
    "swallows": [
       {
       "class": "^qutebrowser$"
       }
    ],
    "type": "con"
}
```

```bash
i3-msg 'workspace 9; append_layout ~/Path/To/Workspaces/Example.json'
i3-msg 'workspace 9; exec "i3-sensible-terminal -e zsh -is eval cd ~/Path/To/Project\; vim"'
sleep .1
i3-msg 'workspace 9; exec "i3-sensible-terminal -e zsh -i eval cd ~/Path/To/Project"'
i3-msg 'workspace 9; exec qutebrowser'
```
