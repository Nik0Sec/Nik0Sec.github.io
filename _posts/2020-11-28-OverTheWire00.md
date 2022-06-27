---
title: OverTheWire - Bandit 0
author: nik0
date: 2020-11-28 17:29:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)

## Introducción

En esta sección vamos a ir resolviendo máquinas de la página [OverTheWire](https://overthewire.org) que nos van a ayudar en temas de shell scripting etc, el modus operandi de estas máquinas es ir resolviendo la primera y luego cuando obtengamos la flag, ponemos la flag en el siguiente nivel para acceder y así, es cómo un CTF pero vas subiendo de nivel a medida que vayas encontrando más flags

## Bandit 0 

Primero nos dice que tenemos que para pasar al siguiente nivel, nos tenemos que conectar al servidor mediante SSH, entonces escribimos en la consola: 

```terminal
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

y la contraseña: bandit0

hacemos un ```ls``` 

vemos que hay un readme

escribimos ```cat readme``` y ahí está el hash o la flag

la flag es un hash que se vería así: ```boJ9jbbUNNfktd78OO4sq3ltutMc3MY1```

Eso sería todo, ahora esa flag es la clave del siguiente usuario y así iriamos avanzando en los niveles ;)
