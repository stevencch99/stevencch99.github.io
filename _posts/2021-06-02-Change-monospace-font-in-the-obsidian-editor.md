---
layout: post
title: "[Obsidian] Change Monospace Font In The Markdown Editor"
description: "[Obsidian] Change Monospace Font In The Markdown Editor"
crawlertitle: "[Obsidian] Change Monospace Font In The Markdown Editor"
date: 2021-06-03 13:18:07 +0800
categories: Editor
tags: Editor
comments: true

---

## Steps

- Choose your favorite theme from Obsidian community

   ![image.png](https://i.imgur.com/hZnxXTg.png)

- Add the CSS below in your theme's CSS file (`PATH_TO_YOUR_VAULT/.obsidian/themes/Obsidian Nord.css`):

  In my case, using 'Hack Nerd Font Mono' be like:

  ```css
  .markdown-source-view { font-family: 'Hack Nerd Font Mono', monospace; }
  ```

  ![image.png](https://i.imgur.com/5HvrrNK.png)

- This should effect immediately after the CSS style has been changed.

  ![image.png](https://i.imgur.com/1CKOC7y.png)

## References

- [forum.obsidian.md](https://forum.obsidian.md/t/monospace-font-in-the-editor/648/8)
