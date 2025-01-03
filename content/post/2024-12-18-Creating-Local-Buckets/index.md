---
title: Creating Local Buckets for Scoop
description: This guide covers how to create a local repositories for scoop. That is, manifests and packages stored and accessable on network attached drives.
date: 2024-12-04T22:18:25-05:00
draft: false
categories:
  - Software
tags:
  - guide
image: Screenshot 2024-12-19 215642.png
---
    

> This guide covers how to create a local repositories for scoop. That is, manifests and packages stored and accessable on network attached drives. 

##  Introduction 
Scoop is a package manager for windows. It allows users to install, update and uninstall software via a windows terminal with basic commands such as: 
- `scoop install <Application>`
- `scoop update <Appliction>`     
- `scoop uninstall <Application>`.  

Users do not have to go to each webpage for an application they want, find the download, and run the installer. 

Scoop installs packages by reading an JSON file which it calls a **"manifest"**. This document contains information that tell scoop where to obtain the package and how to install, update and uninstall it. 

Every package has its own manifest and a collection of manifest is called a **"Bucket"**. Scoop out-of-the-box has default buckets installed. These buckets are publicly available on github and give users access to hundreds of precompiled, free software distributions. 

However, users may find themselves wanting to create their own manifest and buckets. reasions may include: 
- Distributing custom tools to coworkers or accross machines 
- Security reasons. Users may want to compile programs from sources

In order to not disclose any propriety information, this document will demonstrate the process using an open-source application ([clickpaste](https://github.com/Collective-Software/ClickPaste)) thats does not have a publically available manifest. 

Users will need have [scoop package manager](https://scoop.sh/) installed on their machine. If it is not yet installed, please do so by following the installation instructions in the scoop documentation. 

## Packaging a Program

Lets assume we created the program ClickPaste and we want to store it on our Network for anyone within our organisation to use. If you want to follow along clickpaste is available under the releases page of its github repository... Or you can get it at this [link](https://github.com/Collective-Software/ClickPaste/releases/download/v1.3.0/ClickPaste_v1.3.0.zip)

Now store it somewhere on your network (example: `F:\Software\` ) keeping the package zipped or zipping it if its not already zipped (right click > send to > compressed).  

Below is the folder structure of the ClickPaste Package. 

```powershell
ClickPaste_v1.3.0
├── AutoItX3.Assembly.dll
├── AutoItX3.Assembly.xml
├── AutoItX3.dll
├── AutoItX3_x64.dll
├── AutoIt_License.html
├── ClickPaste.exe
├── ClickPaste.exe.config
└── Gma.System.MouseKeyHook.dll
```

With your package "zipped" you need to get the hash of the package. The hash is basically a string of characters generated by looking at all the data in a package. Whats important to understand is that it is unique to that package and its reproducable. If anything changes withing that package it will produce an different hash. This allow scoop to validate the integrity of that package ensure nothing was corrupted during download or unwanted/malicious files are installed. 

You can get the hash via powershell with: 
```powershell
Get-FileHash -Path '.\ClickPaste_v1.3.0.zip' -Algorithm SHA256
```

## Creating a Manifest. 
First thing we need to do is create a manifest for our application. Put simply, a manifest is a file that contains information that tells scoop what is being installed, where to source it from and how to install it. There are 2 ways to create a manifest

**Option 1: Using `Scoop Create`**

Scoop has a built in tool called `create`. It will present a series of prompts and then generate a manifest for your application based on the given responses.  

```powershell
scoop create P:\\Software\\ClickPaste_v1.3.0.zip
```

**Option 2: Manually create myapp.json**
```json
{
  "description": "Windows 10 notification area app in C# that can paste clipboard contents as keystrokes to whatever location you click.",
  "url": "P:\\Software\\ClickPaste_v1.3.0.zip",
  "version": "1.3.0",
  "bin": "ClickPaste.exe",
  "license": "BSD 3-Clause",
  "hash": "5D0E853E9275B728ECC87BE65A96FC2ED881F2F58560CF6D5783391DAA2CBDE6",
  "homepage": "https://github.com/Collective-Software/ClickPaste?tab=readme-ov-file",
}
```

### Advanced: Pre and Post Install/Uninstall 
ClickPaste is an application that runs in the background and is accessible in the system tray or through shortcuts. It would be nice if its started up automatically when installed and everytime the computer started.

My prefered way to do this is to add a shortcut in hte `Startup` directory of the users `AppData` directory. To do this automatically, paste the following block into the ClickPaste manifest. 

```json
 "post_install": [
    "$shortcutPath = \"$env:APPDATA\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\$app.lnk\"",
    "$targetPath = \"$scoopdir\\shims\\$app.exe\"",
    "$wshShell = New-Object -ComObject WScript.Shell",
    "$shortcut = $wshShell.CreateShortcut($shortcutPath)",
    "$shortcut.TargetPath = $targetPath",
    "$shortcut.Save()",
    "if (Test-Path $targetPath) {",
    " Start-Process -FilePath $targetPath",
    "} else {",
    " Write-Host \"Executable not found at $targetPath\"",
    "}"
    ],
```

Next we need to clean up by removing this shortcut when the program is uninstalled. The following block will do that. 

```json
  "post_uninstall": [
    "$targetPath = \"$env:APPDATA\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\$app.lnk\"",
    "Remove-Item -Path $targetPath"
  ], 
```

And finally, since this program runs in the background we need to kill the process before uninstalling it. 

```json
  "pre_uninstall": [
    "Stop-Process -Name \"ClickPaste\" -Force -ErrorAction SilentlyContinue"
  ]
```

## Creating a Bucket
As mentioned before, scoop looks for manifests in a bucket. We therefore need to make a bucket on out network for our applications. We will add any manifests we create to it, including `ClickPaste.json`

Its important to note that Buckets are git repositories. you will therefore need `git` installed. It is installable via `scoop`

`scoop install git`

Create a new directory in your P:\Software or where every you choose to store your bucket. 

```powershell
New-Item -Path "P:\Software\ScoopBucket\bucket" -ItemType Directory -Recurse && `
Set-Location "P:\Software\ScoopBucket\" && `
git init && `
Set-Location "P:\Software\ScoopBucket\bucket" && `
Start-Process .
```

Place any manifests inside `P:\Software\ScoopBucket\bucket`. When you register a bucket to scoop it clones the repository over you the users machine so all changes must be commited in git

```powershell
git add *
git commit -m "clickpaste: Initial" 
```

## Installing ClickPaste

Ok so I may have lied. You dont necessarily need a bucket to install a program. Its a nice to have but you can get away with just the manifest. 

```scoop
scoop install ".\ClickPaste.json"
```

### Registering the bucket with Scoop. 

When managing lots of applications buckets give you the added benefits of: 
- Searching for applications 
- Automatic updates 
- Cleaner more userfriendly syntax. 

To install your package. In this case "ClickPaste", users must first register the bucket that contains its manifest to their instance of scoop. 

First, since this repository is being served by your network and not a git service like github, you`ll need to tell git the repository is "Safe". 

```powershell
# set as safe
git config --global --add safe.directory '//server/drivename/Software/ScoopBucket/.git'

# verify change
git config --global --get-all safe.directory
```

Now you'll notice the path is different here. This command required the path in UNC format. In order to find the name of your drive use the command `net use`

Heres an example of the output: 

```powershell
❯ net use
New connections will be remembered.

Status       Local     Remote                    Network
-------------------------------------------------------------------------------
             Z:        \\Mac\Home                Parallels Shared Folders
```

You should have no problem resistering the bucket now. 

```pwsh
# register the bucket
scoop bucket add internal "//server/drivename/Software/ScoopBucket"

# verify change 
scoop bucket list
```

`internal` is the name being given to the bucket. You can change this to something you prefer like your company's name. 

Now you should be able to install ClickPaste with the following command: 

```pwsh
scoop install clickpaste
```

# Conclusion
For me, distribution has always been the bane of developing tools for my team to use. Scoop, for the most part, has fixed than for me and I hope with this guide it can for you too. 

