---
title: Tryhackme - Pickle Rick
author: nik0
date: 2021-01-17 15:02:00 +0800
categories: [Write up]
tags: [THM,PickleRick]
---

![banner!](/assets/img/sample/PickleRick/banner.jpg)

## Introducción

En este write up veremos la máquina Pickle Rick, donde hay que encontrar 3 "ingredientes" para ayudar a Rick a ser humano denuevo, esta máquina en particular no nos da pistas explícitas de cómo resolverla así que hay que ir indagando poco a poco

## Reconocimiento

Cómo es usual partimos escaneando puertos con Nmap ```sudo nmap -sS -sV -T4 IP``` y este es el output que nos muestra

![1](/assets/img/sample/PickleRick/1.png)

Vemos que hay un puerto SSH y un HTTP así que primero vamos a ver que nos muestra la web

![2](/assets/img/sample/PickleRick/2.png)

No vemos nada interesante en la página en si, entonces procedemos a ver el código fuente

![3](/assets/img/sample/PickleRick/3.png)

Nos muestra algo interesante, al final del código, nos muestra un usuario que vamos a guardar para más adelante

Seguimos enumerando con Gobuster con el comando ```gobuster dir -u http://ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,sh,txt,cgi,html,js,css,py``` (el parámetro -x sirve para especificar los tipos de extensiones que queremos buscar en los directorios/archivos)

![4](/assets/img/sample/PickleRick/4.png)

También se puede hacer un dirb

![5](/assets/img/sample/PickleRick/5.png)

Podemos ver que hay un login.php que nos arroja Gobuster y un robots.txt que también puede ser útil

## Acceso Inicial

Si nos vamos al /login.php nos sale esto:

![6](/assets/img/sample/PickleRick/6.png)

El usuario ya nos lo habían dado pero falta la contraseña, así que nos vamos a /robots.txt

![7](/assets/img/sample/PickleRick/7.png)

Probamos ese texto con el usuario y ya estamos dentro ;)

![8](/assets/img/sample/PickleRick/8.png)

Vemos un panel de comandos que si hacemos un "ls" nos arroja un output al igual que en consola

![9](/assets/img/sample/PickleRick/9.png)

Vemos el primer ingrediente pero no podemos hacer "cat" ni similares, así que probamos con usarlo cómo directorio

![10](/assets/img/sample/PickleRick/10.png

Y listo, ya tenemos nuestro primer ingrediente

Luego después de buscar un poco, me di cuenta de que usaba perl, bash y python3, con el comando ```whereis perl``` y me arrojaba que estaba en el sistema al igual que bash y python3

Entonces lo que vamos a hacer es tener una conexión reversa con la ayuda de un cheatsheet [Cheatsheet](https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/)

En este caso yo utilicé una de Perl

![11](/assets/img/sample/PickleRick/11.png)

La ejecutamos

![12](/assets/img/sample/PickleRick/12.png)

Y ya tenemos nuestra shell dentro del sistema

Ahora nos vamos a /home y vemos que hay 2 usuarios, ingresamos a "rick"

![13](/assets/img/sample/PickleRick/13.png)

y encontramos el segundo ingrediente

## Escalación de Privilegios

Partiendo primero por una enumeración básica con el comando ```sudo -l``` nos dice que todos los usuarios pueden ejecutar sudo

![14](/assets/img/sample/PickleRick/14.png)

Entonces simplemente escribimos el comando ```sudo -i``` y ya seríamos root y podemos encontrar el tercer ingrediente

![15](/assets/img/sample/PickleRick/15.png)








