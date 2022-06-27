---
title: OverTheWire - Bandit 3
author: nik0
date: 2020-11-29 01:10:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)


## Bandit 3


Primero nos conectamos al servidor:

```terminal
ssh bandit3@bandit.labs.overthewire.org -p 2220
```


Podemos ver que hay un directorio llamado "inhere" y dentro de el no se ve nada, por lo que el archivo está oculto

con find podemos ver los archivos que están en el directorio, los busca recursivamente, así serían los comandos:

```find .```

nos muestra que dentro del directorio "inhere" hay un archivo "./hidden"

Ahora podemos usar cat para ver ese hash cómo se muestra en la imagen:

![OTW](/assets/img/sample/OTWM.png)


