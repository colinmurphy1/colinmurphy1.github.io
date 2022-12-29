---
title: 'Installing OptiFine without installing Java'
date: 2019-07-22T00:00:00-06:00
draft: false
authors: "Colin"
categories: ['notes']
tags: ['gaming']
keywords: ['minecraft', 'optifine', 'installing optifine without java']
description: "How to install the Minecraft OptiFine mod without installing the Java runtime."
---

I'm a huge fan of using the [OptiFine](http://optifine.net) mod while playing Minecraft. It makes the game run smoother on my Mac, and allows me to use really cool shaders on my gaming desktop. However, to install OptiFine, you need to have a Java runtime installed. If you don't want to pollute your system by installing Java, you can simply use the Java runtime that is included with Minecraft to install OptiFine.

Note that this article assumes that you are using the `.exe` or `.dmg` releases of Minecraft, and _NOT_ the `.jar` release. Linux and `.jar` releases of Minecraft require a Java runtime installed to play the game.

## Installation on macOS

1. Open your terminal
2. `cd` to this directory: `~/Library/Application Support/minecraft/runtime/jre-x64/jre.bundle/Contents/Home/bin`
3.  Run this command:
    `./java -jar ~/Downloads/OptiFine-version.jar`
4. Install as usual.

Output should be similar to this:

![Mac Terminal](/img/mac-java.png)

## Installation on Windows

1. Open Command Prompt or Powershell
2. `cd` to `C:\Program Files (x86)\Minecraft\runtime\jre-x64\bin\`
3. Type `java -jar` then press spacebar, and then drag your OptiFine .jar onto the command prompt window
4. Install as usual.
