---
title: "Crear Commit Aleatorio"
date: "2025-07-09"
image: ""
tags: ["linux", "commit", "git", "oneline", "terminal"]
categories: [git]
description: "Añade un mensaje divertido a los commits"
---

Puedes usar estos comandos para añadir aleatoriedad a diferentes programas.

1. Commits aleatorios
   `curl -s https://whatthecommit.com/index.txt`

2. Chistes de padres
   `curl https://icanhazdadjoke.com`

Puede usarse por ejemplo en un **alias**:

```bash
# Para chezmoi
alias chzu='chezmoi git add . ; chezmoi git -- commit -m "$(curl -s https://whatthecommit.com/index.txt)" ; chezmoi git push'

# Para git
alias gitodo='git add . ; git commit -m "$(curl -s https://whatthecommit.com/index.txt)" ; git push'
```
