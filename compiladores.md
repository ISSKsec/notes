---
layout: default
---
<h1>./issk.sh 2>/dev/null &;disown</h1>
## Apuntes: Compiladores
----------------------------------------------------

## gcc

Es el compilador por defecto de c, su funcionalidad es sencilla 

```bash
cat << EOF > /tmp/rootshell.c
```
```c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```
```bash
EOF

gcc -o /tmp/rootshell /tmp/rootshell.c
```

Otra manera es esta, en este caso lo hacemos con una libreria
```bash
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c #una libreria
```

En este ejemplo con shared creamos una libreria compartida lo que 
significa que podrá ser ejecutada desde el interior de otro programa 
compilado, en la memoria.
El fPIC significa que la libreria carge independientemente de la posicion
,por ejemplo aqui si no estuviera especificado el fPIC no podria cargar en otra posicion que no sea la 100. 
offset position.

```
100: COMPARE REG1, REG2 
101: JUMP_IF_EQUAL 111 ... 
111: NOP
```

- stack Overflow desactivando protecciones
```bash
gcc -z execstack -g -fno-stack-protector -mpreferred-stack-boundary=2 exercise.c -o vulnapp
```
-z execstack 

> Como su propio nombre indica te permite 
> maniobrar sobre la pila.

## go

Si se te ha creado un direcctorio completo formado con aplicativos basados en go ("existira un main.go") podemos compilar la aplicación que 
queramos generar con go build.

Tambien podemos generar el aplicativo utilizando 

go get github.com/tomnomnom/httprobe 

De esta manera generaremos el archivo direcctamente en la carpeta go.


Tambien podemos usar esto: Cross-Compile
```bash
GOOS=linux GOARCH=amd64 go build .

#Cuando compilamos una aplicacion, se tiene en cuenta la arquitectura de la maquina, con cross-compile podemos definir la arquitectura

GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" .
```
go es la herramienta para administrar las dependencias y archivos en go.
```bash
go build . #Te permite compilar un archivo go.
go build -ldfalgs "-s -w" .
#Compilar con menor peso, luego podemos usar la combinatoria de upx brute en caso de que queramos reducir mas su peso.
```
go get te permite añadir una dependencia hecha en go y la instalaría inmediatamente.
-u = indica el uso de la red para actualizar el nombre de paquetes y sus dependencias.
-v = verbose

Con go run podemos ejecutar un programa go no compilado. 
go run file.go

## i686-w64-mingw32-gcc

Cross compiler, lo que quiere decir es que se puedes realizar una 
compilacion de un script que se ejecutara en una plataforma como windows 
como puede ser un .exe en linux u otra plataforma. 

```bash
i686-w64-mingw32-gcc text.c -o exploit -lws2_32
```

Este programa es un cross compiler

## javac
Es un compilador de java.

"jd-gui"

Es un comando con interfaz grafica que te permite ver el contenido que
tiene una archivo .class, este no es mas que un archivo java previemente
compilado.

javap
Es un comando que nos permite ver la información del código de un archivo 
.class, en otras palabras desensambla el código.

## Make

Este es un compilador que utiliza un programa en c con un Makefile.


![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

