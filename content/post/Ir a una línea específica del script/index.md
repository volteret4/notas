---
title : "Ir A Una Línea Específica Del Script"
date : "2025-07-31"
image : ""
tags : ["GOTO_1", "GOTO1", "GOTO_2"]
categories : [scripts]
description : "Muevete entre lineas de un script"
---


> [stackoverflow](https://stackoverflow.com/questions/9639103/is-there-a-goto-statement-in-bash)

```sh
#!/bin/bash

# Crear alias para goto y usarlo si DEBUG=TRUE

shopt -s expand_aliases
if [ -n "$DEBUG" ] ; then
  alias goto="cat >/dev/null <<"
else
  alias goto=":"
fi

# Ir a tag #GOTO_1

goto '#GOTO_1'

echo "Don't run this"

#GOTO1

echo "Run this"

# Ir a tag #GOTO_2

goto '#GOTO_2'

echo "Don't run this either"

#GOTO_2

echo "All done"
```
