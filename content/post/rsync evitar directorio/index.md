---
title : "Rsync Evitar Directorio"
date : "2026-01-17"
image : ""
tags : ["rsync"]
categories : ["terminal"]
description : ""
---



## [Exclude a Specific Directory](https://linuxize.com/post/how-to-exclude-files-and-directories-with-rsync/#exclude-a-specific-directory)

```sh
rsync -a --exclude 'dir1' src_directory/ dst_directory/
```