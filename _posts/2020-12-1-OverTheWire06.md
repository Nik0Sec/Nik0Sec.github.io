---
title: OverTheWire - Bandit 6
author: nik0
date: 2020-12-1 02:24:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)

## Bandit 6

Primero nos conectamos al servidor:

```terminal
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

Si vemos en la página de OTW, nos dice que el archivo tiene las siguientes propiedades "propiedad del usuario bandit7" "propiedad del grupo bandit6" y "33 bytes de tamaño"

Si hacemos un "find ." no nos va a mostrar nada, por lo que hay que buscar desde la raíz y con los parámetros que ya mencionamos anteriormente, el comando sería ```find / -user bandit7 -group bandit6 -size 33c```

![XD](/assets/img/sample/GG.png)

Cómo podemos ver, nos arroja errores de que no tenemos permiso, cómo podemos solucionar esto?
## Breve explicación stderr y /dev/null
En bash hay un número "2", significa STDERR, es lo que controla los errores por terminal, es decir, si yo quiero sacar los errores que nos aparecen en consola tendríamos que mandar todos los errores con STDERR al /dev/null, cosa que solo me muestre los outputs exitosos, igual les voy a dejar unos enlaces donde explican mejor los términos "/dev/null" y "stderr"

[STDERR](https://blog.carreralinux.com.ar/2017/07/descriptores-de-archivo-stdin-stdout-stderr/)
[/DEV/NULL](https://aprendiendoausarlinux.wordpress.com/2013/01/23/el-archivo-devnull/)

Ahora, si hacemos lo anteriormente mencionado con el comando ```find / -user bandit7 -group bandit6 -size 33c 2>/dev/null``` nos arroja un directorio con un archivo "bandit7.password", entonces al igual que en nivel anterior usamos xargs para mostrar el hash ```find / -user bandit7 -group bandit6 -size 33c 2>/dev/null | xargs cat```

Aquí les dejo una imagen para que puedan ver mejor los comandos:

![;)](/assets/img/sample/GZ.png)

