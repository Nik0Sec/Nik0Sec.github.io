---
title: Tryhackme - BountyHacker
author: nik0
date: 2021-05-28 23:38:00 +0800
categories: [Write up]
tags: [THM,BountyHacker]
---


![XD](/assets/img/sample/BountyHacker/0.jpg)

## Introducción

En este write up, veremos temas cómo reconocimiento, fuerza bruta a SSH y escalación de privilegios via sudo, en mi opinión personal una máquina bastante entretenida sin dejar de lado las metodologías de los CTF más habituales.

## Reconocimiento

Primero comenzaremos escaneando con la herramienta de Nmap, para ello utilizamos el comando ```sudo nmap -sS -sV -n IP```.

![XD](/assets/img/sample/BountyHacker/1.png)

Podemos observar tres puertos abiertos, vamos a analizar que nos muestra el puerto 80.

![XD](/assets/img/sample/BountyHacker/2.png)

Nos cuenta una historia con nombres que podrían ser potenciales usuarios y/o contraseñas, sigamos analizando, esta vez seguiremos con el siguiente puerto abierto que sería FTP.

![XD](/assets/img/sample/BountyHacker/3.png)

Nos logueamos con usuario "anonymous" y logramos ingresar al sistema, al enumerar directorios vemos dos archivos los cuales vamos a exportar a nuestra máquina local, utilizamos el comando ```mget *``` para extraer todos los archivos del directorio seleccionado.

![XD](/assets/img/sample/BountyHacker/4.png)

Ahora observamos el contenido que hay dentro de los dos archivos "locks.txt" y "task.txt".

"locks.txt" contiene lo que podrían ser contraseñas las cuales utilizaremos más adelante para hacer fuerza bruta al servicio SSH.

![XD](/assets/img/sample/BountyHacker/5.png)

"task.txt" contiene un mensaje escrito por alguien llamado "lin" el cuál nos da entender que es un usuario potencial para el logueo de SSH.

![XD](/assets/img/sample/BountyHacker/6.png)

Procedemos a utilizar el usuario dado por el archivo "task.txt" y las contraseñas en el archivo "locks.txt" para realizar la fuerza bruta al servicio SSH con la herramienta Hydra, usamos el comando ```hydra -l lin -P locks.txt IP ssh```.

![XD](/assets/img/sample/BountyHacker/7.png)

Luego de hacer la fuerza bruta, accedemos al servicio SSH con el usuario correspondiente y la contraseña encontrada por Hydra.

## Escalación de privilegios

Ya que estamos dentro, comenzamos a escalar privilegios comprobando distintos métodos cómo ```sudo -l``` y el output nos muestra que podríamos ejecutar /bin/tar 

![XD](/assets/img/sample/BountyHacker/8.png)

Si investigamos en [GTFOBins](https://gtfobins.github.io/gtfobins/tar/#limited-suid) podemos ver que en la parte de sudo podríamos explotar esta vulnerabilidad.

![XD](/assets/img/sample/BountyHacker/9.png)

Ejecutamos el código ```sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin``` y obtenemos root.

![XD](/assets/img/sample/BountyHacker/10.png)










