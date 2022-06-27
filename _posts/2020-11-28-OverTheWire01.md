---
title: OverTheWire - Bandit 1
author: nik0
date: 2020-11-28 20:53:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)

## Bandit 1

Primero nos conectamos al servidor:

```terminal
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

La contraseña es el mismo hash/flag que conseguimos en el nivel anterior en el archivo readme

Una vez dentro, hacemos ls y podemos ver un archivo llamado "-"


![OTW1](/assets/img/sample/OTW1.png)

Si intentamos "ls" o "cat" se va a quedar trabado porque si hacemos ```cat -``` el comando va a creer que le estamos dando instrucciones de parámetros ej: "cat -o" "cat -e" y así, también si intentamos con el tab no nos autocompleta el archivo "-".

![OTW1B](/assets/img/sample/OTW1B.png)


Ahora, hay unas alternativas con cat para poder hacer esto:

para poder llamar al archivo desde la ruta absoluta ```cat /home/bandit1/-```

o con asterisco(*) podemos hacer un cat global, quiere decir que va a llamar a todos los archivos que estén en ese directorio ```cat /home/bandit1/*```

también esto sirve para llamar al archivo ```./-```

Y con eso obtendríamos la flag/hash ;)
