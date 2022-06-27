---
title: Tryhackme - Daily Bugle
author: nik0
date: 2021-05-31 23:38:00 +0800
categories: [Write up]
tags: [THM,DailyBugle]
---

![XD](/assets/img/sample/DailyBugle/0.png)


## Introducción

En este write up, veremos temas cómo vulnerabilidades SQL, crackear hashes y escalar privilegios via yum.

## Reconocimiento

Utilizaremos la herramienta Nmap para este cometido con el comando ```sudo nmap -sS -sV -n --script=http-enum IP``` (mas info del script [http-enum](https://nmap.org/nsedoc/scripts/http-enum.html)) también podríamos utilizar el script [vuln](https://nmap.org/nsedoc/categories/vuln.html) de Nmap.

![XD](/assets/img/sample/DailyBugle/2.png)

Podemos observar que hay tres puertos abiertos, pero el script de http-enum nos muestra ciertos directorios localizados en el puerto 80, de igual forma nos muestra una versión del CMS (Joomla 3.7.0) que va a ser muy importante saberlo para los siguientes pasos.

Si ingresamos a la web, nos muestra que Spiderman robó un banco y a la derecha podemos ver un login.

![XD](/assets/img/sample/DailyBugle/3.png)

Si recordamos la versión del CMS, podemos buscar si existen vulnerabilidades utilizando la herramienta Searchsploit y efectivamente tenemos una vulnerabilidad de tipo SQL que podemos explotar de dos formas, con SQLmap o con un [script](https://github.com/stefanlucas/Exploit-Joomla) hecho en Python para vulnerar esta falla específica en este servicio y versión.


![XD](/assets/img/sample/DailyBugle/4.png)


Yo utilizaré el [script](https://github.com/stefanlucas/Exploit-Joomla) anteriormente nombrado, lo voy a descargar a mi máquina local con [wget](https://linux.die.net/man/1/wget).

![XD](/assets/img/sample/DailyBugle/5.png)

Una vez descargado, nos dirigimos al login de la página. para copiar la URL.

![XD](/assets/img/sample/DailyBugle/6.png)

Ejecutamos el script en conjunto con la URL copiada, y podemos observar el usuario "jonah" y una contraseña en forma de hash que tenemos que desencriptar para el usuario especificado.

![XD](/assets/img/sample/DailyBugle/7.png)

Si usamos la herramienta hashid para identificar que tipo de hash se está utilizando, nos da tres alternativas la cual podemos decir que es bcrypt gracias al manual de [hashcat](https://gist.github.com/dwallraff/6a50b5d2649afeb1803757560c176401).

![XD](/assets/img/sample/DailyBugle/8.png)

Vamos a utilizar John para crackear el hash, especificando el archivo y el tipo de encriptación al cual vamos a aplicar los parámetros.

![XD](/assets/img/sample/DailyBugle/9.png)

Observamos que el resultado que nos arroja Jhon, es la contraseña del usuario "Jonah".

Ahora nos dirigimos al directorio el cuál Nmap nos dió en el anterior escaneo.

![XD](/assets/img/sample/DailyBugle/10.png)

Ingresamos el usuario y contraseña.

![XD](/assets/img/sample/DailyBugle/11.png)

## Subiendo la shell

Ya que estamos dentro del panel, necesitamos obtener una conexión reversa.

![XD](/assets/img/sample/DailyBugle/12.png)

Para ello nos dirigimos a templates (protostar).

![XD](/assets/img/sample/DailyBugle/13.png)

Podemos editar el index.php para poner nuestra reverse shell.

![XD](/assets/img/sample/DailyBugle/14.png)

La editamos acorde a nuestra IP y puerto, en la terminal derecha se ve mi ip de la VPN (tun0) y a la izquierda está la reverse shell la cuál estoy editando, acá les dejo la [reverse shell](https://github.com/tutorial0/WebShell/blob/master/Php/php-reverse-shell.php) que yo utilizo.

![XD](/assets/img/sample/DailyBugle/15.png)

Luego de configurar la shell, la pegamos en index.php reemplazando el existente.

![XD](/assets/img/sample/DailyBugle/16.png)

Antes de guardarlo y subirlo, nos pondremos a la escucha con Netcat.

![XD](/assets/img/sample/DailyBugle/17.png)

Ahora si podemos guardarlo.

![XD](/assets/img/sample/DailyBugle/18.png)

Si todo salió bien, tendríamos una conexión hacia la máquina.

![XD](/assets/img/sample/DailyBugle/19.png)

## Escalando privilegios

En esta ocasión, utilizaremos un script de Github llamado [Linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) para identificar los vectores débiles que podríamos tener dentro del sistema, para copiar el script a la máquina tendremos que iniciar un servidor local con python ```python -m SimpleHttpServer```, luego en la máquina remota(THM) nos vamos al directorio /dev/shm, y con wget descargamos el archivo dentro del directorio, le aplicamos los permisos necesarios ```chmod +x linpeas.sh``` y ejecutamos.

![XD](/assets/img/sample/DailyBugle/20.png)


El script nos da un resultado muy interesante, encontró una contraseña al interior de unos archivos de configuración de PHP.


![XD](/assets/img/sample/DailyBugle/21.png)


Si vamos al directorio home, hay un usuario llamado jjameson.

![XD](/assets/img/sample/DailyBugle/22.png)

Si intentamos loguearnos a la cuenta jjameson con la contraseña anteriormente encontrada, vamos a poder acceder a tal cuenta

![XD](/assets/img/sample/DailyBugle/23.png)

Vamos a probar distintos métodos de escalación de privilegios, si hacemos un "sudo -l" podemos ver que el usuario jjameson tiene permiso para ejecutar yum con privilegios de sudo.


![XD](/assets/img/sample/DailyBugle/25.png)


Ahora nos dirigimos a [GTFOBins](https://gtfobins.github.io/gtfobins/yum/#sudo) para poder spawnear una shell con root interactiva cargando un plugin personalizado.

![XD](/assets/img/sample/DailyBugle/24.png)

Seguimos las instrucciones de GTFOBins y ahora somos r00t ;)

![XD](/assets/img/sample/DailyBugle/26.png)




























