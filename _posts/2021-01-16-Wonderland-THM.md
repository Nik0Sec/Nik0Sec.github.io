---
title: Tryhackme - Wonderland
author: nik0
date: 2021-01-16 02:10:00 +0800
categories: [Write up]
tags: [THM,Wonderland]
---

![Banner!](/assets/img/sample/Wonderland/alicia.png)

## Introducción

En este write up veremos la máquina Wonderland de Tryhackme, lo interesante de esta máquina es que no hay pistas de cómo resolverla así que debemos ir descubriendo las vulnerabilidades por nosotros mismos

## Reconocimiento

Primero vamos a hacer un escaneo con Nmap para ello utlizaremos el comando ```sudo nmap -sS -sV IP``` y el output que nos muestra es el siguiente:

![1!](/assets/img/sample/Wonderland/1.png)

vemos que se muestran 2 puertos abiertos, el 80(HTTP) y el 22 (SSH) así que primero procedemos a investigar el puerto 80 para ver que podemos encontrar

![2!](/assets/img/sample/Wonderland/2.png)

En un principio vemos la página estática donde no podemos interactuar con ella ni nada

## Enumerando directorios con Gobuster

Vamos a buscar en la página si es que el dominio tiene directorios escondidos para ello utlizamos Gobuster que enumera directorios acorde a una lista de palabras que nosotros
le brindemos

con el comando ```gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://``` vamos a escanear los directorios

![3!](/assets/img/sample/Wonderland/3.png)

Luego de enumerar, encontramos un directorio llamado "/r" que nos parece llamativo, así que ingresamos esto en la página para ver que nos sale

![4!](/assets/img/sample/Wonderland/4.png)

Vemos que es más texto relacionado con la historia y en el código fuente no hay nada interesante, seguimos enumerando

![5!](/assets/img/sample/Wonderland/5.png)

Encontramos otro directorio llamado "/a", ya podemos inferir cuales son las otras palabras verdad? ;)

Correcto! si seguimos enumerando, los directorios juntos conforman una sola palabra "/r/a/b/b/i/t"!

Examinamos la última letra o directorio y nos encontramos con esto

![6!](/assets/img/sample/Wonderland/6.png)

![7!](/assets/img/sample/Wonderland/7.png)

En la primera foto, se ve el típico texto de siempre, pero si nos vamos al código fuente, vemos unas credenciales "alice:HowDothTheLittleCrocodileImproveHisShiningTail", se acuerdan del puerto 22 que estaba abierto y correspondía a SSH? ;)

## SSH 

Tenemos una conexión SSH exitosa

![8!](/assets/img/sample/Wonderland/8.png)

Si hacemos un ls, vemos 2 archivos, el primero es un archivo cuyo nombre es "root.txt" si la hacemos un cat para poder verlo no nos va a dejar porque no tenemos los privilegios suficientes, luego el segundo archivo es un archivo con extensión de python entonces procedemos a ejecutarlo para ver que hace

![9!](/assets/img/sample/Wonderland/9.png)

Cuando ejecutamos, el output que nos muestra es como un tipo de poema

## Escalando privilegios

Ahora si vemos el código fuente nos sale un módulo llamado "random" y si nos vamos al final de script, podemos ver que el módulo "random" está escrito en el mismo script

![10!](/assets/img/sample/Wonderland/10.png)

![11!](/assets/img/sample/Wonderland/11.png)

Si intentamos editar el script, no podemos porque no tenemos suficientes permisos

Hay una técnica que está publicada en la página [Aquí](https://criminalminds.github.io/posts/Privesc/) y que nos permite poder elevar nuestros privilegios creando el mismo archivo "random.py" en el directorio actual que ejecute "/bin/bash" así de esta manera cuando se ejecute el script, se va a cargar nuestro "random.py" en vez del que está dentro del script "walrus_and_the_carpenter.py" en este caso

Ejecutamos el script cómo usuario rabbit (sabíamos que era rabbit porque al hacer sudo -l dice que rabbit tiene permiso para ejecutar el script)

![12!](/assets/img/sample/Wonderland/12.png)

Ya avanzamos al otro usuario, ahora somos rabbit

![13!](/assets/img/sample/Wonderland/13.png)

Si vemos dentro del directorio, hay un archivo llamado "./teaparty" si vemos su código fuente vemos que hay un "date" dentro del script sin una ruta absoluta

Podemos abusar de esta vulnerabilidad, exportando nuestro propio $PATH y escribiendo nuestro propio script llamado "date"

primero exportamos nuestro propio $PATH:

```terminal
export PATH=/home/rabbit:$PATH
```
Con eso, cada vez que un script no tenga una ruta absoluta, primero buscará dentro del direcotorio "rabbit" que es donde vamos a crear nuestro propio "date"

dentro de date quedaría así:

```terminal
#!/bin/bash
/bin/bash
```

le damos permisos de ejecución 

```terminal
chmod +x date
```
y ejecutamos el "./teaparty"

![14!](/assets/img/sample/Wonderland/14.png)

Ahora somos Hatter ;)

Ahora vemos en la carpeta de Hatter una contraseña, al probarla con sudo me daba error por lo que no era por ahí

![15!](/assets/img/sample/Wonderland/15.png)

Entonces nos conectamos denuevo a SSH pero ahora con el usuario hatter

![16!](/assets/img/sample/Wonderland/16.png)

y ahora si tenemos una shell cómo el usuario hatter

Ahora con el comando ```getcap -r / 2>/dev/null``` podemos enumerar los cap_setuid, nos podemos fijar que perl tiene "cap_setuid+ep" más información en [GTFOBINS](https://gtfobins.github.io/gtfobins/perl/#capabilities).

![17!](/assets/img/sample/Wonderland/17.png)

En la misma página de GTFOBINS nos dice cómo poder abusar de esta vulnerabilidad para poder obtener root, en todo caso les dejo el comando abajo

```perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'```

![18!](/assets/img/sample/Wonderland/18.png)

y ya somos r00t! ;)






















