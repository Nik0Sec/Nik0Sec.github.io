---
title: Tryhackme - Mr Robot
author: nik0
date: 2020-12-12 02:21:00 +0800
categories: [Write up]
tags: [THM, MrRobot]
---

![Elliot](/assets/img/sample/MrRobot/mrrobot.jpeg)

## Introducción

En este write up, vamos a resolver la máquina de Mr Robot en Tryhackme, veremos temas cómo enumerar directorios, fuerza bruta con Hydra y explotación de binarios SUID

## Reconocimiento 

Primero hacemos un escaneo de puertos con nmap, con el comando ```sudo nmap -sS -sV IP```:

![1](/assets/img/sample/MrRobot/1.png)

Vemos que el puerto 80 está abierto, procedemos a poner la ip de la máquina en la web para ver que nos muestra.

![2](/assets/img/sample/MrRobot/2.png)

Aparentemente, es un video tipo "terminal" donde nos da a escribir distintos comandos, si vemos el código fuente, no hay nada interesante que podamos encontrar.

## Encontrando la primera llave

Podemos hacerlo de 2 formas, la primera es ir directo a /robots.txt y la otra para encontrar /robots.txt podríamos usar los scripts de nmap, con el comando ```sudo nmap -sS -sV --script http-enum "ip"```, aquí les dejo la documentación del script de nmap para que puedan entenderlo mejor: [http-enum](https://nmap.org/nsedoc/scripts/http-enum.html)

Ingresamos a /robots.txt y nos muestra esto:

![3](/assets/img/sample/MrRobot/3.png)

Aquí encontramos la primera llave de tres y un archivo llamado "fsocity.dic", por el tipo de extensión que tiene podemos inferir que es un diccionario que nos va a servir más adelante

![4](/assets/img/sample/MrRobot/4.png)

## Enumerando directorios con Gobuster

Con el comando ```gobuster dir -u http://ip -w "ruta de la wordlist" ``` podemos ver lo siguiente:

![5](/assets/img/sample/MrRobot/5.png)

vemos muchos directorios, los más interesantes son los relacionados a Wordpress, también podemos ver uno llamado "license", así que ingresamos para ver que hay dentro.

![6](/assets/img/sample/MrRobot/6.png)

Hay unas frases sacadas de la serie, si seguimos bajando nos encontramos con lo siguiente:

![7](/assets/img/sample/MrRobot/7.png)

Vemos que es una cadena codificada en base64, la decodificamos de la siguiente manera:

![8](/assets/img/sample/MrRobot/8.png)

Nos muestra unas credenciales, las podemos usar más adelante. (AVISO: estas credenciales te dan acceso al panel de WP, para la gente que no quiera hacer la fuerza bruta, etc, pero yo lo explico detallado para poder ejecutar bien la máquina cómo la hizo el creador de esta misma.) 

## Fuerza bruta a WP con Hydra

Si ingresamos a /wp-login.php vemos un panel:

![9](/assets/img/sample/MrRobot/9.png)

donde debemos usar unas credenciales que no tenemos, primero vamos a analizar el login con Burpsuite para poder así obtener el usuario con Hydra:

Ingresamos cualquier dato mientras interceptamos con Burpsuite:
![9](/assets/img/sample/MrRobot/9.png)

![10](/assets/img/sample/MrRobot/10.png)

En la línea 1 vemos que tiene el método HTTP POST, y en la línea 14 vemos la request que nos va a servir con Hydra.

![11](/assets/img/sample/MrRobot/11.png)

Cómo se ven en la imagen, usamos el diccionario que nos descargamos en /robots.txt, editamos los parámetros de la request para que Hydra haga fuzzing en ellos y podemos ver que el usuario "Elliot" fué aceptado.

Ahora con las credenciales que tenemos, Usuario "Elliot", contraseña "ER28-0652" podemos ingresar al panel de WP.

![12](/assets/img/sample/MrRobot/12.png)

## Subiendo la shell

Si nos vamos a temas, el que usa es TwentyFifteen, muy importante recordar esto

Ahora nos vamos al editor y elegimos una página para subir nuestra reverse shell, en este caso yo elegí la 404.php

![13](/assets/img/sample/MrRobot/13.png)

Luego editamos nuestra shell en terminal: 

![14](/assets/img/sample/MrRobot/14.png)

Y la pegamos en el editor: 

![15](/assets/img/sample/MrRobot/15.png)

y le damos a update file.

Luego ponemos netcat a la escucha con el comando ```sudo nc -lvnp 443```

![17](/assets/img/sample/MrRobot/17.png)

Luego vamos al directorio donde está nuestra reverse shell, en este caso la mía está aquí:

![16](/assets/img/sample/MrRobot/16.png)

Ahora nos vamos a la terminal:

![18](/assets/img/sample/MrRobot/18.png)

ahora tenemos nuestra shell y la podemos hacer full interactiva para más comodidad

![20](/assets/img/sample/MrRobot/20.png)

Una vez dentro vemos un directorio llamado "robot" si accedemos a el vemos 2 cosas, la segunda llave y un archivo llamado "password.raw-md5" si hacemos cat a la llave no podemos ver porque no tenemos permisos, pero si le hacemos un cat al otro archivo nos muestra el usuario "robot" y la contraseña que es un hash en formato md5

Nos vamos a crackstation.net donde podemos crackear el hash:

![19](/assets/img/sample/MrRobot/19.png)

vemos que la contraseña es el abecedario completo, así que ahora podemos acceder al usuario Robot:

![21](/assets/img/sample/MrRobot/21.png)

Ahora podemos obtener la segunda llave

## Escalando privilegios

Ahora tenemos que obtener la tercera y última llave pero para eso tenemos que ser root, así que intentamos de lo más básico enumerando ficheros SUID

![22](/assets/img/sample/MrRobot/22.png)

En los ficheros podemos observar que está nmap, así que hay un método para escalar privilegios SUID en la página de [GTFOBINS](https://gtfobins.github.io/gtfobins/nmap/#limited-suid), seguimos los pasos: 

![23](/assets/img/sample/MrRobot/23.png)

En el usuario Robot escribimos ```nmap --interactive```, luego ```!sh``` y listo, hacemos un ```id``` para confirmar que somos root y ya podemos obtener nuestra tercera llave ;)










