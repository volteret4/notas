---
title : "Bloquea Comentarios Y Recomendaciones En Youtube"
date : "2026-03-23"
image : ""
tags : ["ublock", "youtube"]
categories : ["Pamplinas"]
description : ""
---


Añade esto a las reglas de ublock para bloquear dichas secciones:
```sh
www.youtube.com###comments
www.youtube.com###related
www.youtube.com###sponsor-button
www.youtube.com###donation-shelf
www.youtube.com##.ytp-endscreen-content
www.youtube.com###chat:remove()
www.youtube.com##ytd-reel-shelf-renderer.ytd-item-section-renderer.style-scope
www.youtube.com###chat-container
```
