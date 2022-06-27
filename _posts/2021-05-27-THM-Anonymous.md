---
title: Tryhackme - Anonymous
author: nik0
date: 2021-05-27 14:10:00 +0800
categories: [Write up]
tags: [THM,Anonymous]
---

![LULZ](/assets/img/sample/Anon/0.png)

## Introducción

En este write up, veremos la máquina Anonymous que está enfocada a principiantes, sin embargo en mi opinión personal creo que es muy útil para ir practicando metodologías e ir progresando en cuanto a pruebas de penetración a sistemas.

## Reconocimiento

Primero ejecutaremos un escaneo con Nmap, para ello escribimos en la terminal ```sudo nmap -sS -sV -n IP``` el output que nos muestra es el siguiente:

![LULZ](/assets/img/sample/Anon/1.png)

Podemos observar que hay 4 puertos abiertos, si utilizamos searchsploit para buscar vulnerabilidades en las versiones de los puertos abiertos solo encontraremos una enumeración de usuario para SSH, aunque por ahí no es donde nos queremos enfocar, por ende continuaremos con los otros puertos (FTP, SMB).

Con el comando ```smbmap -H IP``` veremos que recursos compartidos están disponibles en la máquina


![LULZ](/assets/img/sample/Anon/2.png)


Actualmente hay un recurso que es de lectura solamente y se llama "pics", así que para poder accesar a este recurso, intentaremos ver que hay en FTP así que utilizaremos el comando ```ftp IP``` para poder ingresar al sistema, si empleamos un poco de lógica podemos suponer que las credenciales para FTP son "anonymous:anonymous" aunque estas credenciales son bastante usuales para manejar un [FTP Anónimo](https://www.duiops.net/manuales/faqinternet/faqinternet15.htm).

![LULZ](/assets/img/sample/Anon/3.png)

Ahora que hemos accesado al FTP de la máquina, procedemos a indagar más en los directorios disponibles.

![LULZ](/assets/img/sample/Anon/4.png)

Examinando las carpetas disponibles, nos fijamos en una carpeta llamada "scripts" al ingresar en ella hay 3 archivos interesantes, así que utilizaremos el comando ```mget *``` para obtener todos los archivos del directorio actual.

![LULZ](/assets/img/sample/Anon/5.png)

Vamos punto por punto, primero analizamos el archivo "clean.sh" para poder saber que es lo que hace, primero se crea una variable llamada "tmp_files" y se le asigna un valor a 0 luego en el "if" si ese valor es igual a 0 entonces hace un echo en el log "removed_file.log", en caso de lo contrario (else) borra el archivo y escribe otro mensaje.

![LULZ](/assets/img/sample/Anon/6.png)


Si revisamos el otro archivo "removed_files.log" podemos ver el output que genera "clean.sh", dada la información entregada podemos deducir que esto es un [CronJob](https://www.hostinger.es/tutoriales/cron-job) así que vamos a sobreescribir "clean.sh" para obtener una reverse shell con bash

## Obteniendo shell

En la terminal de arriba, ponemos una reverse shell para bash y aquí les dejo una [cheatsheet de PentestMonkey]( http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) para Bash y la terminal de abajo, es para ver la ip que vamos a utilizar para nuestra carga útil.

![LULZ](/assets/img/sample/Anon/7.png)

Ahora que tenemos listo nuestro script, lo vamos a subir al servidor FTP para reemplazar el que ya está en el directorio, para ellos utilizaremos el comando ```put clean.sh clean.sh``` que se muestra en la terminal izquierda, luego de reemplazar el archivo nos pondremos a la escucha con netcat, yo utilicé rlwrap que me permite utilizar las teclas, pero de igual manera se puede hacer una [Shell full TTY](https://ironhackers.es/tutoriales/como-conseguir-tty-totalmente-interactiva/).

![LULZ](/assets/img/sample/Anon/8.png)

Ya que esto es un cronjob, solo tendremos que esperar a que se ejecute "clean.sh" en el servidor FTP, si todo salió bien deberíamos tener una conexión reversa.

![LULZ](/assets/img/sample/Anon/9.png)

## Escalación de privilegios

Primero vamos a buscar binarios SUID con el comando ```find / -perm -u=s -type f 2>/dev/null``` .

![LULZ](/assets/img/sample/Anon/10.png)

Luego de enumerar los binarios, observamos que "/usr/bin/env" tiene permisos SUID.

![LULZ](/assets/img/sample/Anon/11.png)

Para poder explotar este binario nos dirigimos a la página [GTFOBins](https://gtfobins.github.io/gtfobins/env/#suid) y así nos dará información de cómo sacar provecho de esta vulnerabilidad.

![LULZ](/assets/img/sample/Anon/12.png)

El primer comando de GTFOBins no nos sirve, ya que nuestro binario SUID ya existe por ende ejecutamos el comando ```env /bin/sh -p```.

![LULZ](/assets/img/sample/Anon/13.png)

Y ahora somos r00t ;)











