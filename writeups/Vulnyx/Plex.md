# Writeup

## Conocer la red

Lo primero de todos es encontrar equipos dentro de mi red.

`sudo arp-scan -I eth0 --localnet`

![captura-red](https://github.com/AlvarooFh/Plex/assets/148774363/f8868383-1b04-4df1-98d9-29cd3690c682)

Ahora podemos saber cuál es la máquina atacante ya que estamos en VMware y la MAC siempre empieza por **00:0c** Por lo tanto sería la **192.168.95.104**

## Conectividad con la máquina atacante

Comprobamos que tenemos conectividad.

`ping -c1 192.168.95.104`

![captura-conect](https://github.com/AlvarooFh/Plex/assets/148774363/94c0ee16-a204-4669-834f-52d790faa339)

Podemos ver que el **ttl** es **64** por lo tanto se trata de una máquina Linux, esto no es 100% fiable. Vemos que sí tenemos conectividad.

## Escaneo de puertos

Ahora que sabemos que tenemos conectividad, podemos escanear los puertos que estan abiertos para poder hacer la intrusión. Utilizamos la herramienta nmap

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.95.104 -oN allPorts`

Este comando lo que hace es buscar todos los puertos abiertos (1-65535) (`-p-`, lo hace de manera sigilosa (`-sS`), busca la versión de los servicios ( `-sV`), de manera rápida(`--min-rate=5000`), desactiva la resolución DNS para evitar problemas(`-n`), utiliza triple verbose para que tengamos más detalle del escaneo (`-vvv`)y omite la detección de host(`-Pn`). Lo guarda en el fichero allPorts (`-oN allPorts`).

![captura-nmap](https://github.com/Alv-fh/Plex/assets/109484163/2c352940-0cee-4dd9-8c7b-0277b9a0aa3d)

Como podemos ver, el puerto 21 está abierto. Es el puerto de **FTP**. Pero hay algo raro, y es que si nos fijamos, en el servicio, pone **SSH**. En este caso parece una especie de **multiplexación**.
Comprobamos que efectivamente está utilizando el servicio SSH, ya que nos pide contraseña.

![captura-ssh](https://github.com/Alv-fh/Plex/assets/109484163/de4cf6cb-675d-42c8-bbc8-aae1bf0394c6)

## SSLH

Como no hay ningún otro puerto abierto, me da a pensar de que utiliza **SSLH** ya que usa distintos servicios por ese puerto.

[SSLH](https://github.com/yrutschle/sslh)

Como podemos ver dice que acepta las conexiones de puertos específicos y que las reenvia. 

![captura-sslh](https://github.com/Alv-fh/Plex/assets/109484163/52af3ff2-622d-4ab8-a7cc-b23027271ee4)

Entonces nosotros podemos hacer un **curl -I** para que solo nos muestre el **HEAD**, es decir la cabecera.

![captura-curl_-i](https://github.com/Alv-fh/Plex/assets/109484163/c393cf9a-6372-46ca-aa32-641453f06b2c)

Vemos que utiliza Apache2 por este puerto, entonces hacemos un **curl** de tipo **GET** para ver lo que responde.

![capura-get](https://github.com/Alv-fh/Plex/assets/109484163/98abd722-e30c-44f6-8439-3be1a584ebd4)

En este punto lo que se me ocurre es hacer fuzzing, para ello podemos utilizar tanto gobuster como wfuzz, en mi caso lo voy a hacer con gobuster.

![captura-gobuster](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/d7f56720-658a-48e8-8679-0468391d5104)

Como se puede ver, también he puesto el parámetro -x para que busque con extensiones. En esto caso ha encontrado el fichero **[robots.txt](https://es.wikipedia.org/wiki/Est%C3%A1ndar_de_exclusi%C3%B3n_de_robots)**.

Entonces ahora que sabemos que hay un fichero llamado robots.txt, pues podemos enviar un curl de tipo GET para ver lo que responde.

![captura-robots](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/c79450eb-a4fc-40f4-a6c7-ed41e6537a3e)

## JSON WEB TOKEN

### ¿Qué es un JSON WEB TOKEN?
Un **Json Web Token (JWT)** es un mecanismo para poder propagar de forma segura entre dos partes, la identidad de un usuario, privilegios, etc. Esto viene todo codificado en **JSON**, que se encuentran dentro del **payload (cuerpo)**

Si le hacemos el mismo curl pero a la ruta que nos devuelve el curl a robots.txt, que parece un JSON Web Token.

![captura-token](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/1c253ff6-92de-4af9-9c4b-bb311413c85d)

Entonces podemos irnos a la página jwt, y pegamos la respuesta.

![captura-jwt](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/7fe910f3-9c6e-497e-845b-ef492a7f8a7d)

Ya tenemos el usuario y la password, entonces ya podemos entrar por ssh.

![captura-ssh2](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/44e17583-abd6-40a2-9e8b-481814f316f2)

Y ahí estaría la flag del usuario. Ahora vamos a por la de root.
Hacemos un **sudo -l** para ver los comandos que puede ejecuar como sudo.

![captura-sudol](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/f34d042c-d430-406e-b3b6-d75f89e9d98e)

Vemos que puede ejecutar **mutt** que es un cliente de correo electrónico, por tanto, ejecutamos mutt con sudo, y con el usuario root.

![captura-mutt](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/562f9544-5cb6-4c97-a650-3fbfe13c60ff)

Se nos va a abrir como una especie de editor de texto, por tanto vamos a probar a poner **!** para poder ejecutar comandos ya que en otros editores de texto se hace así. Escribimos `/usr/bin/bash`

![captura-mutt](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/92369ffa-9848-4786-86ca-09f1b3931f56)

Y funciona, ya somos root

![captura-root](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/1cf1b8bd-085a-48c6-b136-1eb43b7e5ea0)

Entonces nos vamos al directorio de root.

![captura-root2](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/d8b25291-a1ed-46fb-8c5e-3c3dffba0da4)

### ¿Por qué pasa esto?

Nosotros hemos ejecutado la herramienta mutt con los privilegios del usuario **root**. Entonces nosotros si ponemos `!` en el editor de texto podemos escribir comandos del sistema. Por lo tanto si ejecutamos `/usr/bin/bash`, estamos ejecutando una shell de **Bash** como **root**, de manera temporal ya que hemos ejecutado el comando mutt como root.