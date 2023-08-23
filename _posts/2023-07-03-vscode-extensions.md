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

## My TOP 5 extensions

### Extension: nhoizey.gremlins
#### **Spot Hidden Characters with Ease - nhoizey.gremlins Extension**
Uncover hidden characters that can cause formatting issues in your code effortlessly with the `nhoizey.gremlins` extension. This tool highlights whitespace, tabs, and other non-printable characters, ensuring your codebase remains clean and error-free.

---

### Extension: brunnerh.insert-unicode
#### **Enhance Code Expressiveness - brunnerh.insert-unicode Extension**
Elevate your code's expressiveness with the `brunnerh.insert-unicode` extension. Unlock a vast array of Unicode characters directly within your code editor, enabling you to add special symbols, icons, and unique text elements, enriching your code comments and documentation.

---

### Extension: aaron-bond.better-comments
#### **Elevate Code Comments to Insights - aaron-bond.better-comments Extension**
Transform your code comments into actionable insights using the `aaron-bond.better-comments` extension. This tool introduces customizable comment tags that allow you to categorize and highlight important notes, to-dos, explanations, and more, helping you and your team better understand and manage the codebase.

---

### Extension: johnpapa.vscode-peacock
#### **Personalize Your Workspace with Ease - johnpapa.vscode-peacock Extension**
Make your coding environment uniquely yours using the `johnpapa.vscode-peacock` extension. With this tool, you can colorize the title bar and other interface elements of your Visual Studio Code according to your preferences. Quickly distinguish between different instances and projects, enhancing your productivity and focus.

---

### Extension: sdras.night-owl
#### **Coding at Dusk - sdras.night-owl Extension**
Embrace the soothing darkness of coding at dusk with the `sdras.night-owl` extension. This meticulously crafted theme provides a gentle contrast that reduces eye strain during extended coding sessions. Enjoy a harmonious blend of colors that fosters a calming environment while you write code with unwavering concentration.
