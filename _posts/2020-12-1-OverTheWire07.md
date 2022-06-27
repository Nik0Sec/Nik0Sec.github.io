---
title: OverTheWire - Bandit 7
author: nik0
date: 2020-12-1 21:38:00 +0800
categories: [OverTheWire]
tags: [Bandit]
---

![OTW](/assets/img/sample/OTW.png)

## Bandit 7

Primero nos conectamos al servidor:

```terminal
ssh bandit7@bandit.labs.overthewire.org -p 2220
```

En la página de OTW nos dice que el hash está después de la palabra "millionth" así que usaremos el comando "grep" para filtrar el resultado entre todos esos archivos, el comando sería así: 

```terminal cat data.txt | grep "millionth"```

Así se vería por consola:

![xd](/assets/img/sample/COOL.png)


