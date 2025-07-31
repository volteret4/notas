---
title : "Detecta La RAM De Tu GPU"
date : "2025-07-31"
image : ""
tags : ["Arch", "Linux", "gpu-passtrough", "gpu", "video"]
categories : [pamplinas]
description : ""
---




> [Origen](https://www.cyberciti.biz/faq/howto-find-linux-vga-video-card-ram)


```sh
glxinfo | grep -E -i 'device|memory'
```

Ea, a merendar.
