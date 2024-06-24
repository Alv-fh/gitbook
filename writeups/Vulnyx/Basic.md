# Writeup

## Conocer la red

Lo primero de todos es encontrar equipos dentro de mi red.

`sudo arp-scan -I eth0 --localnet`

![captura-red](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/5a5abd23-e602-4544-aabf-d874e63f1c31)

Como sabemos que la estamos utilizando VMware, la MAC siempre empieza por **00.0c**. Entonces la IP es **192.168.166.101**.

## Conectividad con la máquina atacante

Hacemos un ping con la máquina atacante.

![captura-ping](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/2b2f8e4a-0129-4f23-b54f-c8dee2efd523)

Vemos que el **ttl** es 64, por lo tanto se trata de una máquina **Linux**.

## Escaneo de puertos

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.166.101 -oN allPorts`

![captura-escaneo](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/da7b8a2c-a67e-4201-82cc-7a7f347417a2)

Está el puerto **22 (SSH)**, el puerto **80 (HTTP)** y el **631 ([IPP](https://es.wikipedia.org/wiki/Internet_Printing_Protocol)).

## Web

Buscamos la IP en el navegador pero con el puerto **631** ya que con el **80** nos la ventana de Apache que viene predeterminada.

![captura-pweb](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/4ec9004d-ff4c-4cbd-96ca-c6c5e2c4dc9b)

Si nos vamos a Printers, vemos un usuario llamado **dimitri_printer**. Supongo que el usuario será **dimitri**.
Entonces podemos intentar entrar por **SSH** pero antes tenemos que conseguir la contraseña.

## Hydra

`sudo hydra -l dimitri -P /usr/share/wordlists/rockyou.txt ssh://192.168.166.101`

![captura-mememe](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/f2e67d49-6feb-41dc-b312-87dc1e6cab30)

## SSH

Entramos por **SSH**

![captura-flag](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/44925e0b-f06e-4543-a24a-d5ea7b5b4a8a)

Y encontramos la flag. Ahora vamos a por la de **root**.

Busco binarios que pueda ejecutar como **SUID**.

`find / -perm -4000 2>/dev/null`

![captura-env](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/080e9af5-ad05-4d96-ab13-23de3c512c6f)

Encuentro uno fuera de lo normal llamado **env**.

Vamos a **[GTfobins](https://GTfobins.github.io)**.

![captura-gtfobins](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/9c431bce-23fd-47d7-a58b-ff1d2a278424)

Tenemos que hacer un pequeño cambio ya que si no lo hacemos, nos saldrá error.

![captura-root](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/ec779fe7-1465-442b-89c2-36eec862dde6)

Ponemos la ruta absoluta del comando **env**.

![captura-root_flag](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/1cc94cd9-c14d-4a18-ab44-d95ffb143988)

Encontramos la flag de **root**.