---
title : "Grep"
date : "2026-01-17"
image : ""
tags : ["grep"]
categories : ["terminal"]
description : ""
---





## Usar variables en grep

```sh
grep -io
```

## **Búsqueda En una carpeta (no recursiva)**


```sh
grep "tu_string" /ruta/a/la/carpeta/*
```

Esto buscará `"tu_string"` en todos los archivos dentro de la carpeta, pero **no** en subcarpetas.


## **Búsqueda Recursiva en subdirectorios**


```sh
grep -r "tu_string" /ruta/a/la/carpeta/
```

El flag `-r` (o `-R`) have que `grep` busque recursivamente en todos los subdirectorios.



## **Mostrar Número de línea en los resultados**


```sh
grep -rn "tu_string" /ruta/a/la/carpeta/
```

- `-n`: Muestra el número de línea donde aparece el string.


## **Ignorar Mayúsculas y minúsculas**

```sh
grep -ri "tu_string" /ruta/a/la/carpeta/
```

- `-i`: Ignore diferencias entre mayúsculas y minúsculas.



## **Buscar Archivos específicos (por extensión)**

Si solo quieres buscar en archivos `.txt`, usa:



```sh
grep -r "tu_string" /ruta/a/la/carpeta/ --include="*.txt"
```



## **Excluir Archivos o carpetas**

Si quieres excluir ciertos archivos o carpetas:


```sh
grep -r "tu_string" /ruta/a/la/carpeta/ --exclude-dir="logs"
```

Esto excluirá la carpeta `logs` de la búsqueda.



## **Mostrar Solo los nombres de los archivos que contienen el string**


```sh
grep -rl "tu_string" /ruta/a/la/carpeta/
```

El flag `-l` have que solo se muestren los nombres de los archivos que contienen la cadena, sin mostrar las líneas.



## **Usar `rg` (ripgrep) Para mayor velocidad**

Si tienes `ripgrep` (`rg`) instalado, es mucho más rápido:


```sh
rg "tu_string" /ruta/a/la/carpeta/
```

`rg` es recursivo por defecto y más eficiente que `grep`.