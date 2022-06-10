---
layout: default
---
<h1>./issk.sh 2>/dev/null &;disown</h1>
Text can be **bold**, _italic_, ~~strikethrough~~ or `keyword`.



```
Bienvenidos a mi página de apuntes autodidácticos, esta página 
no está creada con intención de ser pública a pesar de tener soporte
como una página de dominio procedente de github.io, sin embargo 
sí que se creó con la intención de ser una web de repaso que pudiera
ver desde cualquier parte y dispositivo.

Si descubriste esta página y te apasiona la ciberseguridad espero 
que te sirvan de algo mis apuntes.
```
<h2>Apuntes:</h2>


| 1ª Fila        | 2ª Fila          
|:-------------|:------------------|:------|
| [Cifrado](./cifrado.html)           | [Comandos del Sistema](./another-page.html) 
| [Compiladores](./another-page.html)      | [Compresores](./another-page.html)          
| [Conceptos](./another-page.html)         | [Debugging](./another-page.html)            
| [Escalar privilegios](./another-page.html)|[ExploitError](./another-page.html)         
| [Games](./another-page.html)             | [Headers](./another-page.html)
| [Herramientas](./another-page.html)      | [Programacion](./another-page.html)
| [Mobile](./another-page.html)            | [Remoto](./another-page.html)
| [Servicios Web](./another-page.html)     | [Sites Web](./another-page.html)
| [SQL](./another-page.html)               | [Vulnerabilidades](./another-page.html)
| [Windows](./another-page.html)           | [Wrappers](./another-page.html)
| [Transferencia de zona](./another-page.html)|


# Metodologia

### Paso1

Primero lo que haremos es establecernos un entorno para poder trabajar, tener directorios específicos para el trabajo y las herramientas 
necesarias para el inicio del proceso.

Haremos el escaneo a la ip y también lanzaremos un whatweb a ciegas sobre el puerto 80 y 443, esto lo hacemos para ahorrarnos tiempo mientras se realiza el escaneo, tras esto nos fijaremos
si en el escaneo existe algún nombre de dominio o subdominio respectivo. Y proseguimos con la búsqueda de
versiones vulnerables en los servicios del escaneo, también comprobaremos si 
tiene algún puerto udp exclusivo asi como el 161 de snmp, los escaneos sobre puertos udp son mucho más lentos que tcp por lo que iremos directo a estos puertos mas conocidos.
En caso de haber encontrado un dominio con subdominio (admin.local.htb) buscaremos más posibles 
subdominios, fuzzeando por estos, existen varias herramientas como dnsenum, theHarvester etc. En caso de que la página tenga https buscaremos mas dominios y usuarios expuestos en 
el certificado ssl, también podemos lanzar un openssl s_client -connect sobre la ip para que nos reporte informacion sobre este.
Exploraremos distintos servicios squid-ldap-ftp-snmp-smpt-pop-imap etc antes de 
pasar a http en caso de no conocer un servicio accederemos a él mediante nc y 
probaremos pulsando enter, comilla si el servicio esta relacionado con una base
de datos o incluso help por si tiene un panel de ayuda, en caso de que no le guste
lo que le has puesto te saldra esto Ncat: Broken pipe. Si no encontramos nada
lo suyo sería buscar en hacktricks y payloadsallthethings


### Paso2 (Fijate en el wappalizer)


Tras saber si tiene alguna vulnerabilidad o no en los servicios nos aprovecharemos de ella en caso de que exista, después tendremos que acceder a la página y
buscar directorios manualmente al igual que haremos hovering sobre los recursos de esta web para saber sus redirecciones, todo
será a lo bruto pinchando en cualquier enlace abriendo todo lo posible, esto es una
buena práctica si se realiza con el site map de burpsuite, pero es preferible que
lo hagas dos veces la primera sin site map y la segunda con site map. Entre la primera
y la segunda haremos dos fuzzeo de la web, para encontrar directorios, estos fuzzeos 
serán por lo general con la misma wordlists siempre, pero existen ocasiones en las que 
tendremos que utilizar wordlists especificas como en el caso de saber que la 
página tiene un backdoor. En el primer fuzzeo no buscaremos por extensiones, en el segundo fuzzeo si, en 
caso de encontrar un directorio sin extension podriamos seguir
fuzzeando por hay, también viene bien utilizar wordlists pequeñas como #common.txt #big.txt
contenpladas en el repositorio de Seclists en github o acotadas para la web. Mientras se realizan los fuzzeos veremos el código fuente de la
web en el cual filtraremos por user, password, htb, api y debug, palabras clave a la hora de encontra información.
Por último veremos si mediante una petición post forzada en algún panel nos 
muestra algo la página y si algún framework que este usando es vulnerable.
También nos tendremos que haber percatado en si la url tiene algún parametro 
que pueda ser vulnerable como ?file= o ?path=, en esta busqueda hay que tener en cuenta que 
tambien podemos encontrar parametros que se igualen a un cierto recurso que si buscamos manualmente
con la extension .php podremos encontrar y si en algún directorio se 
pueden enumerar algunos objetos, como ?id=1 (SQLi). También buscaremos si 
tiene alguna subida de archivos.
Por último probamos diversos ataques como XXS, CSRF, SSRF, SSTI, XSLT, CRLF en diversos paneles y parametros, en formularios de inicio de sesion 
o registros multiples inyecciones como SQLi, XPATH, NoSQLi claro que es preferible tener en cuenta o saber cuando usar
estos ataques ya que son muchos hay diversos factores que te indican por donde pueden ir los tiros, también podemos recurrir
a hacktricks y PayloadsAllTheThins.

Ya deberiamos haber encontrado alguna vulnerabilidad en la web. 
Buscar desde fuera de la máquina en caso de tener arbitrary file read
proc/self/environ te informa de rutas importantes.


### Paso 3 (Intrusión)
Primero buscar lo que contiene el directorio actual al  que hemos accedido,
lo segundo que haremos es ver los comandos que podemos
utilizar con privilegios root o otro usuario, en caso de estar en el grupo sudoers usaremos 
sudo -l para verlos, también buscaremos SUID y SGID con find, en caso de no obtener nada
buscaremos de manera recursiva o no recursiva los archivos que más puedan llamar 
la atencion, aquellos como settings.php, config o .git etc de nombre, también podemos 
buscar con find los archivos de otro usuario que con suerte tu también tengas
permisos sobre el fichero. Por último podremos ver los procesos que se estan ejecutando a intervalos
de tiempo, con utilidades como pspy o procmon, en las busquedas recursivas de archivos grepea
por pass o username. También veremos los puertos abiertos internamente que tiene 
la maquina con netstat -antup o ss -tulwn. probaremos a buscar por distintas interfaces de red con 
las que tengamos conectividad, si no hay conectividad probablemente tengamos que jugar con rutas estaticas
Recuerda buscar algun archivo raro en la maquina, por ejemplo si es una maquina linux, un .exe no tiene sentido 
con lo cual podrias probar reversing sobre el o desbordamiento del buffer.
Asumo que ya nos hemos fijado en si tenemos permisos en directorios de otros usuarios 
recordemos fijarnos en el path de los procesos para un posible path hijacking o si podemos 
hacer library hijacking desde python en caso de utilizarse una libreria en un script .py.

### Estas son unas páginas muy buenas para hacer un repaso a la hora de hablar de escalada de privilegios y vulnerabilidades web.
[hacktricks](https://book.hacktricks.xyz/welcome/readme)

[GTFOBins](https://gtfobins.github.io/)

[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
## Certificado SSL -- TLS
En este documento que se encuentra en páginas https podemos
encontrar muchos datos valiosos, así como DNS altinativos 
también podemos enumerar usuarios con correos para el acceso a la página

Ejemplo; 
admin@europacorp.htb
https://admin-portal.europacorp.htb/login.php

Este correo es más que probable que vaya a ser válido, de tal manera
que solo tenemos que encontrar la contraseña, incluso si nos fijamos
el correo obtenido encaja como usuario para el dominio, también filtrado
por el certificado SSL.

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

## Comammando VM

Es una implementación que se le añade a los sistemas operativos
windows 7 y 10 para convertirlos en distribuciones enfocadas a la
ciberseguridad.

![CommandoVM](./ComandoVM.jpg) 
<h3>Tools:</h3>
<ul>
	<li>Nmap
    Wireshark
    Covenant
    Python
    Go
    Remote Server Administration Tools
    Sysinternals
    Mimikatz
    Burp-Suite
    x64dbg
    Hashcat
	 y muchas más de 140 herramientas de pentesting integradas.
	</li>
</ul>
[Mas informacion](https://blog.elhacker.net/2021/02/commando-vm-20-maquina-virtual-en-windows-para-pentesting-herramientas.html)


## Decodear hexadecimal ASCII a text 
Para decodear cualquier valor desconocido hayado en la tabla ASCII solo tenemos que
pasarlo previamente a hexadecimal ya sea binario decimal o cualquier otro, después de esto
lo decodeamos de hexadecimal a ascii text.

En python se haría de esta manera, le pasamos la cadena decimal y la encodeamos en hexadecimal
para posteriormente decodearla a ascii text.
```python
hex(24604052029401386049980296953784287079059245867880966944246662849341507003750)[2:].decode("hex")
```
Se utiliza [2:] Para quitarle los valores de los prefijos 0x
> Esta forma quizá este obsoleta

En dado caso simplemente podemos buscar un conversor online.


## Backups fuzzing
Estos backups se realizan con páginas en php por lo general, ya que estas paginas
tienen la necesidad de ocultar su codigo. Para averiguar la extension de los backups.

usamos: 
~ 
.bak
.bak2
.old
.1

Existen diversas wordlists que  tienen extensiones de backups para archivos, al igual
que herramientas como la siguiente, que te permite fuzzear por estas extensiones.

feroxbuster -u https://some-example-site.com --collect-extensions

## Identificar tipo de HASH
En muchas ocasiones la mejor opcion para identificar el tipo de hash es 
usar direcctamente john y te indicara que tipo de hash es, es una buena opción
en caso de que hashid o hash-identifier no te indique correctamente el tipo.

hashid tiene una parametro -m para indicarte el modulo que luego 
usariamos con hashcat.

## Descubrir hosts y puertos dentro de una red.
OS Unix...

El descubrimiento de hosts y puertos dentro de una red para un pentester es el pan de cada día, una tarea que 
se realiza muchas veces, de hecho uno de los primeros pasos a realizar es el escaneo de puertos y numeración de
servicios activos "banner grabbing", tenemos que recalcar que hay varias maneras de realizar este proceso, pongamonos
en un hipotético caso en el que hemos accedido a una red agena mediante la intrusion a un equipo, de primeras lo normal
es no tener conocimiento de como está configurada la red, si tiene reglas de firewall o simplemente las herramientas
que podemos usar desde dicho equipo/usuario, en caso de querer realizar una busqueda de hosts en la red o inclusive en otro
segmento de red, podriamos hacer uso de algún binario estático como el de nmap, pero si la red es restrictiva a la hora de 
transferir archivos esta idea esta descartada por completo, sin embargo existe una manera relativamente sencilla para 
el descubrimiento de hosts y puertos, esta es craer un script en bash con la siguiente estructura, en lo personal es 
mi primera opción, por la sencillez de la sintaxis de este mismo y velocidad de escaneo que realiza.

### Decubrir puertos de un equipo:
```bash
for port in $(seq 1 65535) ; do
        timeout 1 bash -c "echo ' ' >/dev/tcp/10.10.10.235/$port" &>/dev/null && echo "Este puerto esta abierto $port" & 
done; wait
```

### Descubrir ssh en multiples hosts:
```bash
hosts=(172.17.0.1 172.18.0.2 172.18.0.1 172.18.0.100 172.18.0.102 172.18.0.101 172.19.0.3 172.19.0.2 172.19.0.1)
for host in ${hosts[@]}; do
                 timeout 1 bash -c "echo '' >/dev/tcp/$host/22" &>/dev/null && echo "Tiene ssh abierto: $host" &
done; wait
```

### Descubrir en distintos segmentos de red los hosts conectados.
```bash
hosts=(172.17.0 172.18.0 172.19.0)
for host in ${hosts[@]}; do
        for last in $(seq 1 256); do
                 timeout 1 bash -c "ping -c1 $host.$last" &>/dev/null && echo "esta activa $host.$last" &
        done; wait
done
```
### Descubrimiento total de equipos existentes y sus respectivos puertos abiertos en una red "No recomendable"
```bash
hosts=(172.17.0 172.18.0 172.19.0)
for host in ${hosts[@]}; do    
        for last in $(seq 1 256); do
                 timeout 1 bash -c "ping -c1 $host.$last" &>/dev/null && echo "esta activa $host.$last" &
					  if [ $(echo $?) = 0 ]; then
							for port in $(seq 1 65535) ; do
								timeout 1 bash -c "echo ' ' > /dev/tcp/$host.$last/$port" &>/dev/null && echo "host $host.$last con puerto $port" &
					  		done ; wait
					  fi
			done; wait
done
```
----------------------------------------------------------
# Truco para parametros de PHP en la URL (SSRF)
En caso de no poder ver el contenido de un parametro de una página, prueba a 
poner el parametro detras de otro parametro.
Realmente esto ocurre gracias a una falla de SSRF mediante la cual se exponen
los servicios abiertos internamente de la maquina victima, tenemos que tener en cuenta
que los recursos que son accesibles mediante este puerto filtrado solo serán accesibles
por este puerto, con lo cual si al hacer hovering encima del recurso mostrado en la web vemos
otra dirección debido al uso de otro puerto, lo normal es que pongamos el parametro correspondiente 
donde se esta generando la vulnerabilidad de SSRF para seguir la busqueda.

Quedaria asi la url en un hipotético caso de darse dicho problema:

http://10.10.10.55:60000/url.php?path=http:localhost:888/?doc=backup

_Tener en cuenta el urlencode en todo momento_

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

