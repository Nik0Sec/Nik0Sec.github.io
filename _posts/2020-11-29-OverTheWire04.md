---
title: OverTheWire - Bandit 4
author: nik0
date: 2020-11-29 02:24:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)

## Bandit 4

Primero nos conectamos al servidor:

```terminal
ssh bandit4@bandit.labs.overthewire.org -p 2220
```
En este nivel vemos el mismo directorio "inhere" pero al hacer un "find ." nos muestra varios archivos y no lo podemos listar todos, entonces ahora lo que vamos a hacer, es hacer uso del comando "file" y "xargs"

## Breve explicación de los comandos a usar

El comando "xargs" hace una llamada así mismo constantemente o es un comando recursivo, más info aquí: [XARGS](http://systemadmin.es/2009/04/uso-de-xargs-herramientas-unix-ii)

y el comando "file" sirve para ver que tipo de fichero hay, más info aqui: [FILE](https://cambiatealinux.com/file-informacion-sobre-el-tipo-de-fichero)

Ahora con el comando ```find . -name -file* | xargs file``` podemos listar los ficheros que hay y el tipo de ficheros que están en el directorio, el fichero número 07 tiene texto ASCII por lo que diferimos que el hash está ahí

Ahora lo que debemos hacer es mostrar el fichero número 07 pero ahora le ordenamos a xargs que le haga un cat, cómo se puede ver en la imagen:

![OTW](/assets/img/sample/OTWZ.png)

El comando es ```find . -name -file07 | xargs cat```

Y ahí tendríamos nuestro hash ;)



