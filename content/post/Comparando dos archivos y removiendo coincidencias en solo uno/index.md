---
title : "Comparando Dos Archivos Y Removiendo Coincidencias En Solo Uno"
date : "2025-07-15"
image : ""
tags : []
categories : [arch]
description : "Problemas con firma PGP y pacman"
---


> [Origen](<https://stackoverflow.com/questions/37503186/comparing-two-files-by-lines-and-removing-duplicates-from-first-file" class="tc-tiddlylink-external>)

Remove lines from test1 because they are in test2:

```bash
grep -vxFf test2 test1
```


To overwrite test1:
```bash
grep -vxFf test2 test1 >test1.tmp && mv test1.tmp test1
```
