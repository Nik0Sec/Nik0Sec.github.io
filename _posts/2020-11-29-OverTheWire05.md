---
title: OverTheWire - Bandit 5
author: nik0
date: 2020-11-29 20:11:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)


Primero nos conectamos al servidor:

```terminal
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

## Bandit 5

Lo primero que vemos es el directorio "inhere", hacemos un "find ." y vemos más directorios, en la página de OTW, nos dice que el archivo es "legible por humanos", "1033 bytes de tamaño" y " no ejecutable" entonces ya nos dan una pista para poder filtrar el archivo por esos términos.

El comando a utilizar será ```find . -type f -readable ! -executable -size 1033c | xargs cat```

![OTW](/assets/img/sample/OTWCTM.png)

Si queremos eliminar los espacios que se ven ahí, podemos usar los siguientes comandos: 

![OTW](/assets/img/sample/OTWHDP.png)


