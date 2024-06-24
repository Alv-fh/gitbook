# Writeup

## Conocer la red

Lo primero de todos es encontrar equipos dentro de mi red.

`sudo arp-scan -I eth0 --localnet`

![captura-arp](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/7d2bd547-ed76-4c3c-80cb-1c780eb41418)

Ahora podemos saber cuál es la máquina atacante ya que estamos en VMware y la MAC siempre empieza por **00:0c** Por lo tanto sería la **192.168.95.101**

## Conectividad con la máquina objetivo

Comprobamos que tenemos conectividad.

![captura-ping](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/0b997875-f5bf-4ada-9c1f-6114f83be5cb)

Podemos ver que el **ttl** es **64** por lo tanto se trata de una máquina Linux, esto no es 100% fiable. Vemos que sí tenemos conectividad.

## Escaneo de puertos

Ahora que sabemos que tenemos conectividad, podemos escanear los puertos que estan abiertos para poder hacer la intrusión. Utilizamos la herramienta nmap.

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.95.101 -oN allPorts`

Este comando lo que hace es buscar todos los puertos abiertos (1-65535) (`-p-`, lo hace de manera sigilosa (`-sS`), busca la versión de los servicios ( `-sV`), de manera rápida(`--min-rate=5000`), desactiva la resolución DNS para evitar problemas(`-n`), utiliza triple verbose para que tengamos más detalle del escaneo (`-vvv`)y omite la detección de host(`-Pn`). Lo guarda en el fichero allPorts (`-oN allPorts`).

![captura-nmap](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/60784049-41c4-4519-b4d1-4a692d79fb6b)

Como podemos ver, están abiertos tanto el puerto **22 (SSH)** como el puerto **80 (HTTP)**. Por lo tanto podemos empezar a probar.

También vemos que utiliza **Apache 2.4.57**
También podemos ver que utiliza para el servivio **OpenSSH 9.2** para el servicio **SSH**.

![captura-apache](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/abdd66c6-d05d-4439-abca-fd5b6f04debc)

Vemos que al buscar en el navegador web por la IP del atacante, nos devuelve la página por defecto de apache que viene al instalarse.

En este punto lo que se me ocurre es hacer un **fuzzing**, para conocer los directorios que hay. Para ello podemos utilizar la herramienta gobuster, o wfuzz. En mi caso voy a utilizar **gobuster**.

![captura-gobuster](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/d075a291-bc97-4429-8512-0d00a42f6904)

Como podemos ver, encuentra el fichero **[robots.txt](https://es.wikipedia.org/wiki/Est%C3%A1ndar_de_exclusi%C3%B3n_de_robots)**, que ya hemos hablado en otras máquinas de él.

Ahora buscamos en el navegador el fichero **robots.txt** para conocer los directorios que están denegados o rechazados.

![captura-robots](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/4c281690-8f0e-478e-a0c1-94aeccad7870)

Entramos en el directorio ritedev.

![captura-ritedev](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/20ab4a7e-0058-486e-bfbb-27b6cd4b5c88)

Y se nos abre la esta página que a simple vista parece un estilo de tienda.

En este punto lo que se me ocurre es volver a hacer un ataque de diccionario **(Fuzzing)** para ver los directorios o archivos que hay.

![captura-admin](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/0fad6954-e919-49d4-98ba-dacd056d4307)

Vemos que hay un admin.php. Vamos a buscar, haber que sale.

![captura-admin2](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/b287d6b5-a8d4-4872-adf9-2f8182c47d0c)

Se nos abre esta especie de login, vamos a probar credenciales, típicas, como **admin** **admin**

![captura-admin3](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/d358cce8-2aeb-4eac-99aa-8abd354f5407)

Parece que ha funcionado, ahora vamos a irnos al **Files Manager** que es el más útil creo yo, ya que vamos a ver todo lo que hay.

![captura-filemanager](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/741f7794-281a-4c47-8a21-66e97c97be99)

Podemos subir archivos, que quiere decir eso. Que podemos subir por ejemplo una **reverse shell** en php por ejemplo. Para ello vamos a descargarnos una herramienta de GitHub.

![captura-github](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/6ea3c705-6644-4819-8d8d-8e82273b0a0a)

Ahora editamos el script, y en donde pone **CHANGE THIS** es donde tenemos que cambiar las cosas. En este caso lo que tenemos que cambiar es la **IP**, en este caso es la IP nuestra, con la que estamos atacando, y luego el puerto en el que vamos a utilizar para la reverse shell.

![captura-reverse](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/ffd64cae-1677-4157-969c-c6e4a8398a62)

Antes de ejecutar la reverse shell, nos tenemos que poner en escucha por el puerto que hayamos especificado, en mi caso el 443.

`nc -nlvp 443`

![capura-escucha](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/59ee502a-35a0-48cc-a3f1-4b56f1921ae3)

Ahora tenemos que encontrar la ruta en donde se ejecuta la reverse shell. En este caso es **IP/ritedev/media/php-reverse-shell.php**.Si vemos que se queda cargando, es buena señal. Luego nos vamos al terminal y nos debe de salir algo así.

![captura-escucha2](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/6b84ba3d-b9fc-4383-8364-685e1512e5a5)

Ahora vamos a hacer la reverse sell manegable, ya que ahora mismo no podemos hacer muchas cosas con ella. Para ello ejecutamos los siguientes comandos.

`script /dev/null -c bash`

`CTRL + Z`

`stty raw -echo; fg`

`reset xterm`

`export SHELL=bash`

`export TERM=xterm`

Y ya tenemos una shell normal y corriente. Ahora buscamos la flag.

![captura-usuario](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/5a109c31-e623-41c8-bbf7-7f5efd3c7d4c)

Como podemos ver, no podemos entrar a la carpeta de travis, porque no tenemos permisos, entonces tenemos que hacer escalada de privilegios.

Hacemos `sudo -l` para ver los comandos que podemos ejecutar con sudo.

Vemos que podemos usar **crash**

Ejecutamos `sudo -u travis /usr/bin/bash/crash -h`

![captura-travis](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/49ca15c4-e017-4757-a229-2ac5d6d56e71)

Ponemos **!** para poder ejecutar comandos y nos ponemos una shell en bash, se nos pondrá como travis ya que ejecutamos el comando como travis.

![captura-flag](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/38da74a6-11fe-4e06-a5c9-702203f80776)

Ya tenemos la flag, vamos ahora a por la de root.

Volvemos a hacer `sudo -l` para ver los comando que podemos usar como root.

![captura-xauth](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/dd1ff2c9-b00b-4051-9157-6552d6afc5a7)

Vemos que podemos usar el comando xauth.

Si hacemos un `-h` vemos que este comando puede leer comandos de un archivo.

Después de probar he encontrado esto, y es que si hacemos un `sudo -u root /usr/bin/xauth source /root/.ssh/id_rsa`.

![captura-id_rsa](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/21055337-5b74-4d43-a5eb-06e8d968857c)

Podemos ver la id_rsa de root, por lo tanto, ya es fácil, todo lo demás, antes de nada tenemos que ponerla bien y completarla ya que sale recortada. Por tanto en la máquina local nos creamos un fichero y pegamos la id_rsa y hacemos lo siguiente:

![image](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/c6477d2c-6577-499a-a915-1a830cbb608b)

Ahora tenemos que completar la clave.

Para ello nos lo pasamos a un fichero llamado **id_rsa** y lo dejamos de esta manera.

![captura-id_rsa2](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/924a505d-e7d8-405c-9626-46842c8dfb04)

Antes de probar a entrar por **SSH** tenemos que darle permisos **600**

`sudo chmod 600 id_rsa`

![captura-permisos](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/063a1293-3de1-444a-9866-dd622eaa616e)

Ahora vamos a probar a entrar por **SSH** pero con la **id_rsa**, ya que no nos pide clave.

![captura-flagroot](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/002f7739-772c-42c5-bdbf-83f6f93fd9f4)

En este caso la flag estaba oculta.