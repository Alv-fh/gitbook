#  Escolares

![[Writeups/Images/Dark/1.png]]

## Conectividad con la máquina objetivo

Comprobamos que sí tenemos conectividad con la máquina y que el **TTL** es 64 por lo que hablamos de una máquina Linux.

![[Writeups/Images/Dark/2.png]]

## Escaneo de Puertos

Utilizamos la herramienta **nmap** y vemos los puertos abiertos.

![[Writeups/Images/Escolares/3.png]]

22 -> SSH

80 -> HTTP

## Fuzzing Web

![[Writeups/Images/Escolares/4.png]]

También vemos que si le hacemos fuzzing a la ruta `http://ip/wordpress/` nos enumera esto:

![[Writeups/Images/Escolares/5.png]]

Entramos en **trackback.php** y sale esto:

![[Writeups/Images/Escolares/6.png]]

En el fuzzing web descubro que hay un dominio ya que veo la ruta absoluta y veo que **http://escolares.dl/**, por lo que lo añado al fichero **/etc/hosts**

![[Writeups/Images/Escolares/7.png]]

Si busco **xmlrpc.php** sale lo siguiente:

![[Writeups/Images/Escolares/8.png]]

Inspecciono la página y veo que hay un comentario en donde hace referencia a una ruta, por lo que decido ir.

![[Writeups/Images/Escolares/9.png]]

Veo que luis es el admin, debajo en el correo pone luisillo por lo que puede que sea el usuario. Intento con **wpscan**.

![[Writeups/Images/Escolares/10.png]]

Tendremos que crear con la herramienta **[cupp](https://github.com/Mebus/cupp)** un diccionario, y pondremos los datos que salen. Una vez generado, lo ponemos en el **wpscan**.

`wpscan --url http://escolares.dl/wordpress -U luisillo --passwords luis.txt`.

![[Writeups/Images/Escolares/11.png]]

Encontramos la clave de luisillo. Y entramos

![[Writeups/Images/Escolares/12.png]]

Veo que hay un plugin llamado File Manager, por lo que podemos investigar ahí, y vemos que podemos subir archivos.

![[Writeups/Images/Escolares/13.png]]

Subimos una reverse shell en php -> [REVERSE-SHELL](https://github.com/pentestmonkey/php-reverse-shell)

Antes de nada, la editamos y ponemos nuestra ip de atacante y el puerto que vayamos a utilizar.

![[Writeups/Images/Escolares/14.png]]

Nos ponemos en escucha.

`nc -nlvp 443`

Y buscamos la reverse shell en el navegador. Si se queda cargando, es buena señal.

![[Writeups/Images/Escolares/15.png]]

Y conseguimos una shell. Hacemos el tratamiento de la **TTY** como aparece en pantalla.

![[Writeups/Images/Escolares/16.png]]

Vemos el **/etc/passwd**.

![[Writeups/Images/Escolares/17.png]]

Nos metemos en el home y encontramos un fichero llamado secret.txt en el parece ser la clave de luisillo.

![[Writeups/Images/Escolares/18.png]]

Entramos como luisillo y vemos que podemos ejecutar como cualquier usuario el binario **awk**.

Buscamos en la herramienta [searchbins](https://github.com/r1vs3c/searchbins).

![[Writeups/Images/Escolares/19.png]]

Lo ejecutamos en la máquina pero de esta manera para que no haya ningún error.

![[Writeups/Images/Escolares/20.png]]

Ya somos root!!
