---
title : "Cómo Usar El Comando Tar En Linux"
date : "2025-12-18"
image : ""
tags : ["commands", "shell"]
categories : ["terminal"]
description : ""
---



Veamos qué operaciones básicas puedes llevar a cabo utilizando Tar. Antes de comenzar, necesitarás SSH en tu servidor VPS. ¡Aquí hay una [guía](https://www.hostinger.es/tutoriales/conectar-usando-terminal-putty-ssh/) para ayudarte!

## Crear un archivo .tar en Linux

Puedes crear compresiones .tar tanto para un archivo como para directorios. Un ejemplo de este tipo de archivo es:

```sh
tar -cvf sampleArchive.tar /home/sampleArchive
```

Aquí **/**home**/**sampleArchive es el directorio que necesita set comprimido creando **sampleArchive.tar**.

Este commando usa las opciones **–**cvf que significan:

- **c** – crear un nuevo archivo .tar
- **v** – muestra una descripción detallada del progreso de la compresión
- **f** – nombre del archivo

## Crear un archivo .tar.gz en Linux

Si deseas una mejor compresión, también puedes usar **.tar.gz**. Un ejemplo de esto es:

```sh
tar -cvzf sampleArchive.tar.gz /home/sampleArchive
```

La opción adicional **z** representa la [compresión gzip](https://www.hostinger.es/tutoriales/compresion-gzip). Alternativamente, puedes crear un archivo **.tgz** que sea similar a **tar.gz**. Te mostramos un ejemplo de esto último a continuación:

```sh
tar -cvzf sampleArchive.tgz /home/sampleArchive
```

## Crear un archivo .tar.bz2 en Linux

El archivo **.bz2** proporciona más compresión en comparación con gzip. Sin embargo, esta alternativa tomará mas tiempo para comprimir y descomprimir. Para usarla, debes usar la opción **-j**. Un ejemplo de cómo se vería la operación es el siguiente:

```sh
tar -cvjf sampleArchive.tar.bz2 /home/sampleArchive
```

Dicha operación es similar a **.tar.tbz** o **.tar.tb2**. Te mostramos un ejemplo a continuación:

```sh
tar -cvjf sampleArchive.tar.tbz /home/sampleArchive

tar -cvjf sampleArchive.tar.tb2 /home/sampleArchive
```

## Cómo descomprimir archivos .tar en Linux

El commando Tar de Linux también se puede utilizar para extraer un archivo. El siguiente commando extraerá los archivos en el directorio actual:

```sh
tar -xvf sampleArchive.tar
```

Si deseas extraer tus archivos a un directorio diferente, puedes usar la opción **-C**. Te mostramos un ejemplo de esto a continuación:

```sh
tar -xvf sampleArchive.tar -C /home/ExtractedFiles/
```

Puedes usar un commando similar para descomprimir archivos **.tar.gz**, tal como se muestra a continuación:

```sh
tar -xvf sampleArchive.tar.gz

tar -xvf sampleArchive.tar.gz -C /home/ExtractedFiles/
```

Los archivos **.tar.bz2** o **.tar.tbz** o **.tar.tb2** pueden descomprimirse de manera similar. Para esto deberás teclear el siguiente commando en la línea de commando:

tar -xvf sampleArchive.tar.bz2

## Cómo listar el contenido de un archivo en Linux

Una vez que hayas creado el archivo, puedes listar el contenido mediante un commando similar al siguiente:

```sh
tar -tvf sampleArchive.tar
```

Esto mostrará la lista completa de archivos junto con las marcas de tiempo y los permisos. Del mismo modo, para **.tar.gz**, puedes usar un commando como:

```sh
tar -tvf sampleArchive.tar.gz
```

Esto también funcionaría para archivos **.tar.bz2** como se muestra a continuación:

```sh
tar -tvf sampleArchive.tar.bz2
```

## Cómo descomprimir un único archivo .tar

Una vez que creas un archivo comprimido, puedes extraer un único archivo de ese comprimido. Esto lo puedes lograr con el commando que te mostramos a continuación:

```sh
tar -xvf sampleArchive.tar example.sh
```

Aquí **example.sh** es un archivo único que se extraerá del comprimido sampleArchive.tar. Alternativamente, también puedes usar el siguiente commando:

```sh
tar --extract --file= sampleArchive.tar example.sh
```

Para extraer un solo archivo de un comprimido .tar.gz puedes usar un commando similar al mostrado a continuación:

```sh
tar -zxvf sampleArchive.tar.gz example.sh
```

O alternativamente:

```sh
tar --extract --file= sampleArchive.tar.gz example.sh
```

Para extraer un solo archivo de un comprimido .tar.bz2 puedes usar un commando como este:

```sh
tar -jxvf sampleArchive.tar.bz2 example.sh
```

O, alternativamente, uno como este:

```sh
tar --extract --file= sampleArchive.tar.bz2 example.sh
```

Como puedes ver, el commando tar tiene mucha flexibilidad en su sintaxis.

## Cómo extraer múltiples archivos de los archivos .tar

En caso de que desees extraer varios archivos, usa el siguiente formato del commando:

```sh
tar -xvf sampleArchive.tar "file1" "file2"
```

Para .**tar.gz** puedes usar:

```sh
tar -zxvf sampleArchive.tar.gz "file1" "file2"
```

Para **.tar.bz2** puedes usar:

```sh
tar -jxvf sampleArchive.tar.bz2 "file1" "file2"
```

## Extraer múltiples archivos con un patrón

Si deseas extraer del comprimido patrones específicos de archivos como solo los **.jpg**, usa el commando **wildcards**. Una muestra de dicho commando se muestra a continuación:

tar -xvf sampleArchive.tar --wildcards '*.jpg'

```sh
Para **.tar.gz** puedes usar:
```

tar -zxvf sampleArchive.tar.gz --wildcards '*.jpg'

Para **.tar.bz2** puedes usar:

```sh
tar -jxvf sampleArchive.tar.bz2 --wildcards '*.jpg'
```

## Cómo agregar archivos a un archivo .tar

Si bien puedes extraer archivos específicos, también puedes agregar archivos nuevos a un archivo comprimido existente. Para hacerlo, debes usar la opción **-r** que significa agregar. El commando Tar puede agregar tanto archivos como directorios.

A continuación se muestra un ejemplo en el que estamos agregando example.jpg al **sampleArchive.tar** existente.

```tar -rvf sampleArchive.tar example.jpg```

También podemos agregar un directorio. En el ejemplo que te mostramos a continuación, el directorio image_dir se agrega al archivo sampleArchive.tar

`tar -rvf sampleArchive.tar image_dir`

No puedes agregar archivos o carpetas a comprimidos **.tar.gz** o .**tar.bz2**.

## Cómo verificar un archivo .tar en Linux

Usando Tar puedes verificar un archivo. Esta es una de las formas en que puedes hacerlo:

`tar -tvf sampleArchive.tar`

Esto no se puede aplicar en archivos **.tar.gz** o **.tar.bz2**.

## Cómo verificar el tamaño del archivo en Linux

Una vez que crees un archivo, puedes verificar su tamaño. Este se mostrará en KB (Kilobytes).

A continuación te mostramos varios ejemplos del commando a usar para verificar el tamaño de diferentes tipos de archivos comprimidos:

`tar -czf - sampleArchive.tar | wc -c`

`tar -czf - sampleArchive.tar.gz | wc -c`

`tar -czf - sampleArchive.tar.bz2 | wc -c`