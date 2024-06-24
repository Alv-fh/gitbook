# Writeup

## Conocer la red

Lo primero de todos es encontrar equipos dentro de mi red.

`sudo arp-scan -I eth0 --localnet`

![captura-red](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/4e9f1b28-a21d-4768-9eeb-5f293d0107ee)

Hemos creado este script para hacerlo de manera más cómoda, **solo es efectivo si la máquina está en VMware o Virtualbox.**

![captura-script](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/6566b47f-4c4c-4191-97c5-dfb3d367ec7f)

### Resultado

![captura-vmware](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/55dddfab-31b7-4bf1-9fec-e989f94db0b3)

## Nmap

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.232.102 -oN allPorts`

![captura-nmap](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/1917069e-4a07-4383-83fc-58bd75749f0a)

## FTP

Entramos al FTP de manera anónima y nos descargamos el archivo que hay.

![captura-ftp](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/2a675bbf-9feb-4169-b62a-5cfa155f52e7)

Lo descomprimimos y vemos que es una carpeta de backups de firefox.

## Firefox-decrypt

**Firefox-decrypt es una herramienta para extraer contraseñas de perfiles de Mozilla.

Nos descargamos [Firefox-decrypt](https://github.com/unode/firefox_decrypt)

Y para usarlo.

`python firefox_decrypt.py`

Encontramos esto.

![captura-firefox](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/b3e5929f-b157-4d6e-a143-229f43a535af)

## Port 9090

![captura-9090](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/779ed7a1-0a6b-4af6-b0b3-b4e899c730b5)

Buscamos en el Navegador la IP pero por el puerto `9090`.

Aceptamos riesgos y:

![captura-debian](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/8c122009-a105-4951-bb25-f8a8eee33fe7)

Ahora ingresamos el usuario encontrado anteriormente.

Vemos una pestaña llamada terminal, tiene buena pinta.

![captura-marco](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/4f40e320-6b0a-4150-8c5e-7312233ef61e)

Tenemos la flag de user.

![captura-flag](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/5b70c6b6-1311-4262-9235-1bb615c7ec6f)

Vemos que comandos puede utilizar como root.

![captura-ayuda](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/cc3a2f08-a8fa-40b5-b5f0-8a5513828c09)

Vemos que puede leer archivos con el parámetro `-f`.

![captura-root-id](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/9be0a878-451c-46fc-bef1-c8e693ae9c9f)

Podemos ver la id_rsa de root, pero hay que ponerla correctamente. Hacemos uso del comando `awk`.

![captura-id](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/13c1a15a-4b1b-4275-b75c-bc1c2f9c0d44)

Y ahora completamos la clave le damos permisos **600** e intentamos entrar.

![captura-root-flag](https://github.com/Alv-fh/Vulnnyx_machines_writeups/assets/109484163/418ece7a-9571-4fd7-9a93-15319c729da6)

Y encontramos la flag de root.