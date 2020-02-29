---
layout: post
title: "WSL nvim as external editor"
date: 2020-02-29 13:23:00 +1030
categories: nvim wsl godot unity3d vim
---
Recently I have been dabbling with both [Unity3D][unity-web] and [Godot][godot-web]. I use [nVim(neo-vim)][nvim-web] as my primary code editor in a [WSL(Windows Subsystem for Linux)][WSL-wiki] Ubuntu terminal.

One of my frustrations while using [Unity3D][unity-web] and [Godot][godot-web] was clicking on scripts or errors in the editor and it opening up Visual Studio (or whatever the default editor was set to). There was no info anywhere on how to have your external editor set to nvim (inside a terminal) and especially nothing out there for a nVim/WSL setup.

Through trial and error and eventually figured out a way to make it work.

### Neovim-Remote
The first step is to install a little executable called nvr(neovim-remote) on your WSL installation. Basically nVim starts a server by default. You can get it's address via `:echo $NVIM_LISTEN_ADDRESS`. nvr will use that server and pass whatever file you want to edit into the currently open nvim process. This works great when you're using nvr in the same WSL environment, but it has issues when you're going from Windows to WSL. It detects it fine (when using a localhost as the NVIM_LISTEN_ADDRESS) but you can't seem to be able to pass the correct linux directory structure through it.

I felt like I was so close but still so far but I was determined to make this work.

After some research I found that you can use the command `wsl` from a Windows command prompt to execute a command inside your WSL environment. This seemed promising. I tested running `wsl ~/.local/bin/nvr --servername /tmp/nvimsocket test.txt` from Windows and on WSL nVim successfully opened up a new buffer named `test.txt`. Success!

### Batch Script
The next issue was converting our Windows path into a Linux path. Thankfully WSL provides us a with an easy command `wslpath` which will convert a windows path to linux and visa versa. My next step was to create a batch file to piece all this together.

__nvim.cmd__
{% highlight powershell %}
@echo off
wsl wslpath "%1" > tmpfile
set /p filepath= < tmpfile
del tmpfile
wsl ~/.local/bin/nvr --servername /tmp/nvimsocket %filepath% %2
{% endhighlight %}

My batch scripting is pretty terrible but this is pretty basic. It runs the command `wsl wslpath` to a tmpfile using the first command line argument as the path. It then sets variable filepath with the contents of that file and then deletes the tmp file. It then runs our `nvr` command with our filepath as the argument.

### Setup using Godot

- Editor -> Editor Settings -> [Text Editor] -> External

- [Exec Path] = `path/to/nvim.cmd`
- [Exec Flags] = `{file} "+call cursor({line}. {col})"`

### Setup using Unity3D

- Edit -> Preferences -> External Tools

- [External Script Editor] = `Browse -> nvim.cmd`
- [External Script Editor Args] = `"$(File)" "+call cursor($(Line), $(Column))"`

[nvim-web]: https://neovim.io
[WSL-wiki]: https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux
[godot-web]: https://godotengine.org/
[unity-web]: https://unity.com/
[neovim-remote-gh]: https://github.com/mhinz/neovim-remote
