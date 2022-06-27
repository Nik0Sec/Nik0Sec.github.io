---
title: OverTheWire - Bandit 2
author: nik0
date: 2020-11-29 00:41:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)


## Bandit 2

Primero nos conectamos al servidor:

```terminal
ssh bandit2@bandit.labs.overthewire.org -p 2220
```
Si hacemos un "ls" vemos que hay un archivo llamado Spaces in this filename

![OTWC](/assets/img/sample/OTWC.png)

obviamente si escribimos "cat spaces in this filename" nos va a decir que no existe

Así que con estos comandos podemos ver el hash dentro del archivo:

```cat *``` para mostrar todo a nivel global dentro del archivo

también si hacemos cat + tab, tab sirve para el autocompletado 


![OTWD](/assets/img/sample/OTWD.png)


