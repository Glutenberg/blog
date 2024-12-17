---
title: How to Setup a Local Bucket for Scoop
date: 2024-12-04T22:18:25-05:00
draft:  false
categories: 
    - software
tags:
    - guide
---

## Introduction 
Scoop out-of-the-box has default buckets installed. These buckets are publicly available on github and give users access to hundreds of precompiled, free software distributions. 

## Assumptions 
This guide assumes the following: 
- you already have [scoop package manager](https://scoop.sh/)  installed on your machine. If you have not yet done this please do so by following the installation instructions in the scoop documentation. 
- you are familiar with how to use [PowerShell](https://learn.microsoft.com/en-us/powershell/). 

## Instructions 
### Creating a manifest. 
Put simply, a manifest is a file that contains information that tells scoop what is being installed, where to get it and how to install it. There are 2 ways to create a manifest

#### Option 1: Using `Scoop Create`

Scoop has a built in tool called `create`. It will present a series of prompts and then generate a manifest for your application based on the given responses.  

```sh
scoop create P:\\Software\\myapp.zip
```

#### Option 2: Manually create myapp.json

```json
{
  "description": "A custom tool for doing stuff",
  "url": "P:\\Software\\myapp.zip",
  "extract_dir": "myapp",
  "version": "0.0.0",
  "bin": "myapp.exe",
  "license": "MIT",
  "hash": "BA952D5C325F2DBEC4854A88EBB3F7179DE8FC96F84FFCA391CDAEE90BE89CB5",
  "homepage": "P:/path/to/instruction.pdf"
}
```

### Creating a Bucket




### Registering the bucket with Scoop. 

### Installing app.exe
