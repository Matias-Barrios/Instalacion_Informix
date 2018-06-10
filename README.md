# Instalacion_Informix
Cosas para instalar informix

Instalar Informix 12.50 en Centos7 : 

```
sudo groupadd informix                                    # Crear grupo informix
sudo useradd -g informix -d /dev/null informix            # Crear usuario informix y agregarlo al grupo informix
sudo passwd informix                                      # seteamos el password del usuario informix
sudo mkdir /opt/informix                                  # Crear el directorio de informix
sudo chown informix.informix /opt/informix                # Damos ownership a informix
sudo vi /etc/profile.d/informix.sh                        # Generar un archivo de perfil
```
Copiar este contenido a /etc/profile.d/informix.sh

```
SQLHOSTS=/opt/informix/etc/sqlhosts
INFORMIXDIR=/opt/informix
INFORMIXSERVER=miServidor
ONCONFIG=onconfig.std
PATH=$INFORMIX/bin:$PATH:/opt/informix/bin
export INFORMIXDIR PATH INFORMIXSERVER ONCONFIG SQLHOSTS
```
Seguir con los pasos siguientes :

```
sudo mv $HOME/iif*.tar /opt/informix						# Mover el tar que bajhan de IBM a la carpeta /opt/informix
cd /opt/informix									# Ir a /opt/informix
su informix -c "tar -xvf iif*.tar"						# Descomprimirlo como el usuario informix
sudo ./ids_install											# Instalamos el informix ejecutando el instalador. Cuando poregunte si queres crear una instancia decimos que NO!! 
sudo touch /opt/informix/etc/sqlhosts				# Crear el archivo sqlhosts
sudo chown informix.informix /opt/informix/etc/sqlhosts
sudo vi 	/opt/informix/etc/sqlhosts		# Si quieren otro nombre de server deben cambiar tambien todas las referencias anterirores a miServidor (Paso 5) )
``` 

Copiar esto a /opt/informix/etc/sqlhosts

```
	#dbservername    nettype       hostname      servicename      options
	miServidor       onsoctcp      localhost     informix 
```

```
sudo vi /opt/informix/etc/onconfig.std						# Editar el archivo onconfig con estos valores ( en vi se puede buscar con el ? )
```
```
	ROOTPATH /opt/informix/logdir/rootdbs
	LTAPEDEV  /matiasInformixDBSpaces
	DBSERVERNAME miServidor
	ROOTSIZE 1000000
```

```
su informix -c "mkdir /opt/informix/logdir"				# Creamos el directorio /opt/informix/logdir	
su informix -c "chmod 777 /opt/informix/logdir"			# Le cambiamos los permisos
cd /opt/informix/logdir 								# Nos movemos a /opt/informix/logdir	
su informix -c "touch rootdbs"							# Creamos el root dbs 
su informix -c "chmod 660 rootdbs"						# Le cambiamos los permisos al dbspace root
sudo chown informix.informix rootdbs					# Le cambiamos los owners al dbspace root
```
```
sudo vi /etc/services									# Añadimos esta linea al final del archivo /etc/informix ( Si ya tienen algun servicio en el puerto 50000 cambiar por otro numero mayor a 50000, por ej 53400)
```

Y pegar este contenido : 

```
	informix        50000/tcp		# Informix server
```

```
sudo mkdir /mis_dbspaces									# Creamos una carpeta para los dbspaces
sudo chown informix.informix /mis_dbspaces				# le damos ownership de esa carpeta a el usuario informix
sudo su -											# nos convertimos en root
27) cd /opt/informix/ 								# Nos vamos a la carpeta de informix ( por las dudas)
source /etc/profile.d/informix.sh  					# Por si las moscas, ahcemos source al perfil
oninit -ivy											# ACA EL SERVER DEBERIA LEVANTAR
```
En este paso, si todo quedo ok, el server deberia levantar.
Despues de eso deberiamos chequear si esta online con este comando :
    onstat -
Si todo quedo bien deberiamos ver un mensaje como este :
		
			"IBM Informix Dynamic Server Version 10.00.UC4 -- On-Line -- Up 00:00:07 -- 19508 Kbytes"
			.....

De ahora en mas creamos un dbspace propio, porque crear una base de datos 
en el espacio del root es una chanchada
	
```
cd /mis_dbspaces																			# Vamos a la carpeta que creamos para los dbspaces
touch mi_primer_dbspace.dbspace																# Creamos un archivo vacio que sera nuestro primer dbspace
chown informix.informix mi_primer_dbspace.dbspace 											# le damos el ownership a informix
chmod 660 mi_primer_dbspace.dbspace 										        		# Cambiamos los permissos a 660
onspaces -c -d mi_primer_dbspace -p /mis_dbspaces/mi_primer_dbspace.dbspace -o 0 -s 100000  # Crear el dbspace
exit 																						# dejamos de ser root 
cd 																							# vamos a nuestro home
```

Creamos una base de datos llamada 'mapas'				

```
touch crear_db.sql && vi crear_db.sql																		# Creamos un archivo sql con estas lineas 
```

Y pegamos este contenido en el archivo : 

```
CONNECT TO 'mapas@miServidor' USER 'matias'  USING '1234';
	

CREATE TABLE Institutos
		(
		  id_instituto  SERIAL PRIMARY KEY  CONSTRAINT Institutos_clave_primaria,
		  nombre  varchar(50) NOT NULL CONSTRAINT Institutos_nombre_vacio,
		  calle   varchar(50) NOT NULL CONSTRAINT Institutos_calle_vacio,
		  numero  INT,
		  telefonos varchar(100),
		  email varchar(80),
		  baja boolean NOT NULL CONSTRAINT Institutos_baja_vacio
		); 
		
INSERT INTO Institutos (nombre, calle, numero, telefonos, email, baja)
VALUES ('Escuela Técnica "Arroyo Seco"', "Av. Agraciada Esq. Aguilar", 2544, "29243865|29243856", "etas010@gmail.com", "f");

INSERT INTO Institutos (nombre, calle, numero, telefonos, email, baja)
VALUES ('Intituto Técnico Superior "Buceo"', "Av. Rivera Esq. Tomas de Tésanos", 3729 , "26285408|26285813", "itsbuceo@gmail.com", "f");

INSERT INTO Institutos (nombre, calle, numero, telefonos, email, baja)
VALUES ('Escuela Técnica Cerro "Mtro. Nicasio García"', "Portugal Esq. Carlos Mª Ramírez", 4257 , "23111056|23119407|23114949", "portugal4257@hotmail.com", "f");

INSERT INTO Institutos (nombre, calle, numero, telefonos, email, baja )
VALUES ('Escuela Técnica Colón  "Don Alberico Passadore"', "Cno. Colman Esq. Cesar Mayo Gutierrez", 5274 , "23209511|23205789", "estecolon@gmail.com", "f");

INSERT INTO Institutos (nombre, calle, numero, telefonos, email, baja )
VALUES ('Escuela Técnica Flor de Maroñas', "Andrés Latorre Esq. Veracierto", 4914 , "25148177|25148210", "utuflor2012@hotmail.com", "f");

INSERT INTO Institutos (nombre, calle, numero, telefonos, email, baja)
VALUES ('Escuela Técnica Artigas', "Bernabé Rivera Esq. Gral Rivera", 626 , "47723687|47725988", "etautu626@adinet.com.uy", "f");

```
 
