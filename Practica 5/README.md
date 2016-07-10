#Práctica 5: Replicación de bases de datos MySQL
Juan Borja Álvarez Peralta

##Crear una BD e insertar datos.
A continuación como se muestra en la imagen accedemos a mysql y creamos una base
de datos llamada contactos y una fila llamada datos y la mostramos.
Todos los comandos están mostrados en las capturas de pantalla.

![imagen](Capturas/Captura_1.png)

Procedemos a introducir datos y visualizar los datos.
Ya tenemos datos (un registro) insertados en nuestra BD llamada “datos”. Podemos
haber insertado más registros. Veamos cómo entrar y hacer una consulta:

![imagen](Capturas/Captura_2.png)

##Replicar una BD MySQL con mysqldump.

Tenemos que tener en cuenta que los datos pueden
estar actualizándose constantemente en el servidor de BD principal. En este caso,
antes de hacer la copia de seguridad en el archivo .SQL debemos evitar que se
acceda a la BD para cambiar nada.

Asi que escribimos FLUSH TABLES WITH READ LOCK;
![imagen](Capturas/Captura_3.png)

Una vez bloqueada la escritura en la db, volcamos la copia con el siguiente comando:

mysqldump contactos -u root -p > /root/contactos.sql

Una vez hecho procedemos a desbloquear la db como podemos observar en la siguiente captura.
![imagen](Capturas/Captura_4.png)

A continuación desde la otra maquina virtual vamos a proceder a copiar los datos.

![imagen](Capturas/Capture_5.png)

Con el archivo de copia de seguridad en el esclavo ya podemos importar la BD
completa en el MySQL. Para ello, en un primer paso creamos la BD:

# mysql -uroot -p
mysql> create database contactos;
mysql> quit

![imagen](Capturas/Captura_6.png)

##Replicación de BD mediante una configuración maestro-esclavo:

Lo primero que debemos hacer es la configuración de mysql del maestro. Para ello
editamos, como root,  /etc/mysql/mysql.conf  el archivo mysqld.cnf 
![imagen](Capturas/Captura_7.png)
y modificamos los siguientes parámetros:
![imagen](Capturas/Captura_8.png)

Comentamos el parámetro bind-address que sirve para que escuche a un servidor:
![imagen](Capturas/Captura_9.png)
Le indicamos el archivo donde almacenar el log de errores. Por ejemplo al reiniciar el
servicio, si cometemos algún error en el archivo de configuración en el archivo de log
nos mostrará con detalle lo sucedido.

Después de esto reiniciamos el servicio con: "/etc/init.d/mysql restart"
![imagen](Capturas/Captura_10.png)

Y procedemos a configurar la máquina 2, igual que en la anterior salvo por el server-id=2:
![imagen](Capturas/Captura_11.png)
Procedemos a reiniciar el servicio: "/etc/init.d/mysql restart"
![imagen](Capturas/Captura_12.png)

Ahora volvemos a nuestra máquina 1 (swap1) que va a ser configurada como nuestro maestro para garantizar el acceso al esclava:

mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
FLUSH PRIVILEGES;
FLUSH TABLES;
FLUSH TABLES WITH READ LOCK;

![imagen](Capturas/Captura_13.png)

Volvemos a la máquina esclava (swap2), entramos en mysql y le damos los datos del maestro.
CHANGE MASTER TO MASTER_HOST='192.168.1.139', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=980, MASTER_PORT=3306;
START SLAVE;

![imagen](Capturas/Captura_14.png)

Comprobamos que todo ha ido bien en el esclavo con: "“SHOW SLAVE STATUS\G”"

![imagen](Capturas/Captura_15.png)

a continuación mostramos que todo funciona correctamente insertando algunos datos y comprobando que se actualiza,en la siguiente imagen podemos comprobarlo.
![imagen](Capturas/Captura_16.png)

##Replicación de BD mediante una configuración maestro-maestro:

Solo tenemos que configurar otro maestro-esclavo pero invirtiendo las máquinas ahora *swap1* será esclavo y *swap2* maestro, es decir:

Crear el usuario en *swap2* que antes era esclavo.

CREATE USER esclavo IDENTIFIED BY 'esclavo';
GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
FLUSH PRIVILEGES;
FLUSH TABLES;
FLUSH TABLES WITH READ LOCK;

![imagen](Capturas/Captura_17.png)

Crear el esclavo en *swap1* que antes era maestro.


CHANGE MASTER TO MASTER_HOST='192.168.1.138', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=980, MASTER_PORT=3306;
START SLAVE;

![imagen](Capturas/Captura_18.png)

A continuación muestro pruebas de inserciones y borrados de datos en ambas máquinas y vemos como se actualiza en ambas:

![imagen](Capturas/Captura_19.png)
