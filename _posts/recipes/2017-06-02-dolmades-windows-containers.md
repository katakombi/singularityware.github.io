---
title:  "Dolmades: Windows Apps under Linux using Singularity"
category: recipes
permalink: dolmades-windows-containers
---

Recently I've found several reasons why Windows containers fit in nicely in our Linux-dominated environments:

- You actually do have <a href="https://github.com/CHPC-UofU/Singularity-ubuntu-wine-peakselector" target="_blank"> a scientific application</a> which is exclusively available for windows
- How about installing and running MS Office inside a safe environment?
- Do you own Windows games you always wanted to play under Linux?
- Your tax reporting software runs under Windows only?

That's why I initiated a project called <a href="http://dolmades.org" target="_blank">dolmades</a> which should bring portable and sustainable windows applications to linux users.
Its goal is to provide stable recipes for people who are interested in regularily using one or only a few particular windows application. 
For that purpose VMs and dual-booting bear a huge overhead in both performance and ressources.
Another approach is to use <a href="https://winehq.org" target="_blank">wine</a>, a wrapper that redirects Windows system calls into native Linux system calls and allows execution of native Windows binaries under supported platforms (Unix with X, just recently Android).

So, why don't we formulate the following equation: wine + singularity = dolmades ?


## About
This document describes how to get a dolmade running in which one or several windows apps can be installed temporarily or permanently. 
Both a Singularity Hub and a Docker Hub image template is readily available and supports plenty of win apps already without further modification needed.

## Requirements
This tutorial uses singularity 2.3 which is all what you will need to have installed. 
The game which is being installed is freely downloadable.
Normal user privileges will suffice provided you install your software into the proper location.

## Example Dolmade
Let's pull the dolmades docker image. Alternatively, you can use the shub image.

``` 
singularity pull docker://c1t4r/dolmades 
```

This image comes with Ubuntu 16.04 and has the winehq PPA enabled. 
It has wine version 2.0.1 (current stable) installed and some additional packages which are needed
to run some DX9 games.

Now, extend the size of the image and execute its run script in read-write mode:

```
singularity expand -s 1500 dolmades.img
singularity exec -w dolmades.img /singularity
```

This will do the following in ```/tmp```:

1. Check if a wine prefix exists from a previous run? If yes: restore it, if no: initialize it. 
   The wine prefix is kind of a chroot location for wine apps and where the windows user profiles and their apps are being stored.
2. Download and install DirectX9
3. Start a new shell
4. Prints the instructions to manually download and install Broken Sword 2.5 (a nice, free adventure game)
5. After you are done, test it and finally exit the shell.
6. The profile will be saved under `/PROFILES`, the game under `/APPS`.

***NOTE:*** it is important to call the EXE from its directory or else it will fail!

``` 
wine ./bsengine.exe 
```

Expand the size of your container further or use a new one to try out other windows games or apps!

If you are unsure if an app will install or work properly you can test that without modifying the container. 
Simply start it read-only:

``` 
./dolmades.img # or singularity run dolmades.img 
```

Once you have figured out how to install and make an app work you repeat the steps in rw-mode.
Often, some registry entries need to be tweaked or some builtin DLLs substituted by native ones.

That can be often done using the tool `winetricks`. Some more settings can be tweaked using the tool ```winecfg```.

## Resources

This is the ```Dockerfile``` to build the dolmades docker image
{% include gist.html username='katakombi' id='6ce5690804beefaeb83abe2f3c946b08' file='Dockerfile' %}

This is the <a href="https://raw.githubusercontent.com/katakombi/dolmades/master/Singularity" target="_blank">link</a> to the singularity hub dolmades build spec file which also contains the run script.

Dolmades are still in the early stages and there are many challenges to tackle before we seize the potential over using wine on Linux systems directly.
If this speaks to you then send me your feedback or questions, and check my <a href="http://dolmades.org">project page</a> for screen shots of further examples.
