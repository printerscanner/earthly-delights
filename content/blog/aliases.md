---
title: Aliases
description: 
date: 2024-07-11
tags:
  - Engineering
---
As this is a developer blog, it is important I show off my aliases. Over the years, I have developed strong opinions about what makes a good alias. The most important component of a good alias is memorability. The alias has to be significantly easier to remember than its command. It should solve a memory problem, not create one. I'll start with an example of what I think bad aliases are.

## Bad Aliases

```sh
alias gp='git push'
alias gpoh='git push origin HEAD'
alias gl='git pull'
alias gca='git commit -v -a'
alias gco='git checkout'
alias gta='git tag -a'
```

Is it really easier to remember `gp` than `git push`? Are you really saving time by reducing keystrokes? These may work for some people, but for me, this is a type of alias I will forget immediately after making it and find unused in my alias graveyard a year later. 




## My Most Used Aliases

The marker of a good alias is that it is frequently used and well tended to. Over time, I've crafted a set of aliases and functions in my `.zshrc` file that help streamline my workflow and simplify repetitive tasks. Here's a look at some of my favorites and the reasoning behind them.


 **Clear and Destroy**
   ```sh
   alias clear="cd && clear"
   ```
  `clear` on its own isn't destructive enough. I use `clear` usually always in moments of desperation when I have gone too far in the wrong direction and need to start over. I recommend this to anyone that appreciates a blank slate.

**Ultra-Simple Server**
   ```sh
   alias serve="python3 -m http.server"
   ```
Python changed its start server syntax several years ago, and my brain wasn't able to catch up. I set this a few years back and live in bliss that I don't have to search for the command every few weeks. This is my most used alias anytime I need to spin up a simple server. This python server is in my opinion, the easiest way to start a local server.



**Ultimate Image Resizer**
   ```sh
   resize() {
       read "prefix?Enter prefix: "
       i=1
       for file in *; do
           cwebp -q 75 "${file}" -o "${prefix}_${i}.webp"
           ((i++))
       done
   }
   ```
I frequently need to resize images, and this function is ultra fast and allows me to batch-process images. Prompting for a prefix keeps my file names organized and prevents overwriting. 



 **Which Yarn command was it?** 
   ```sh
   go() {
       code .
       yarn
       yarn dev || yarn start || yarn serve
   }
   ```
   In 2024, yarn commands live in a fractured state. There is no consensus between templates on what command should be used to start a server, and I constantly need to re-remember the library I built for the project to know what command to use. This command instead tries everything for me, so I don't have to go look at the `package.json`. I would highly recommend it. 




### Conclusion

A great alias saves time. Refrain from trying to reduce keystrokes; you'll never remember your abbreviations. Here is a link to my [Github repository](https://github.com/printerscanner/dotfiles/blob/main/sh/aliases.sh) if you'd like to grab these. 