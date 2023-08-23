---
layout: post
title: My VS code extensions
date: 2023-07-03 15:38
category: Programming
tags: ["extensions", "vscode", "programming"]
---

# Introduction
If you are an user of VS Code there might be times when you need to migrate your configuration, including your extensions. But is there an easy way to do so? The answer is yes and I'm going to show you how

## How do you migrate your extensions ?

There are some handy CLI commands that you can use to interact with VSCode. 
Follow the next snippets to get the list of extensions that are currently installed on your machine. Then you can save this file somewhere and reuse it to quickly re-install all your favorite extensions.

### Export your extensions first
```console
code --list-extensions > extensions.txt
```

### Install extensions

```console
cat extensions.txt| xargs -n 1 code --install-extension
```