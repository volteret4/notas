---
title : "Hacer Referencia A Argumentos Especificos Del Comando Anterior"
date : "2026-01-17"
image : ""
tags : ["shell"]
categories : ["terminal"]
description : ""
---


Puedes usar `!:#` para referenciar segmentos del commando anterior
```sh
touch archivo.txt
echo !:1 
archivo.txt

touch 1.txt 2.txt
echo !:2
2.txt
```
