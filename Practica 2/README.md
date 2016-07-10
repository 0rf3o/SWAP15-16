Juan Borja Álvarez Peralta
###Práctica 2:  Clonacion de la informacion de un sitio web

El entorno de desarrollo de la práctica se va a realizar en virtualbox.
Primeramente debemos de configurar las máquinas para que en la red se vean entre sí.
Para ello configuramos la red de cada una de ellas, nos metemos en configuración, red
y en modo promiscuo le damos a permitir todo.

A continuación inicio sesión en modo root, haciendo uso del siguiente comando le asignaremos una constraseña al usuario root
"sudo passwd root"->activacion de root.
y ya podremos loguearnos con el usuario root.

##Crear un tar con ficheros locales en un equipo remoto

Copiamos el directorio "borja" en la maquina swap1comprimido en tar.gz
tar czf - home/borja | ssh borja@192.168.0.198 ‘cat >  ~/tar.tgz’  
![captura](img/captura_1.png)


