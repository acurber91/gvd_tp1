![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Agustín Curcio Berardi

Docente:

* Yoel Yamil López

**1. Implementar en MongoDB un `replica set` con 3 servidores que contengan la información de la base de datos finanzas. Un nodo `PRIMARY`, un `SECONDARY` y un `ARBITER`.**

Antes de inicializar las tres instancias de `mongod` vamos a crear los siguientes directorios:

    ├── data                # Directorio raíz para almacenar los datos.
        └── db              # Directorio para los datos de la base de datos.
            └── rs          # Directorio para las instancias que conformarán el replica set.
                ├── 0       # Directorio donde la instancia PRIMARY almacenará sus datos.
                ├── 1       # Directorio donde la instancia SECONDARY almacenará sus datos.
                └── 2       # Directorio donde la instancia ARBITER almacenará sus datos.

Para ello, en una consola hacemos lo siguiente:

    mkdir data && cd data
    mkdir db && cd db
    mkdir rs && cd rs
    mkdir 0 1 2

Lo mismo se aprecia en el siguiente video:

![Creación de directorios](doc/create-directory.gif)

Luego resta instanciar las tres instancias de `mongod` de la siguiente manera.

    mongod --replSet rs --dbpath /data/db/rs/0 --port 27017 --oplogSize 50
    mongod --replSet rs --dbpath /data/db/rs/0 --port 27018 --oplogSize 50
    mongod --replSet rs --dbpath /data/db/rs/0 --port 27019 --oplogSize 50

En la siguiente animación se observa como se reemplazan los `--dbpath` por los directorios correspondientes.

![Instanciación de mongod](doc/mongod-instances.gif)

**2. Conectarse al nodo PRIMARY.**

Ahora para configurar e inicializar el "replica set", tenemos que conectarnos desde un cliente de Mongo al nodo PRIMARY. Para ello:

    mongo --port 27017

Una vez conectados, definimos la variable `cfg`, la cual contendrá la estructura de nuestro "replica set". El parámettro `_id` define el nombre del "replica set" mientras que `members` contiene a los servidores miembros de dicho conjunto.

    	cfg = {
		        _id:"rs",
                members:[
                    {_id:0, host:"localhost:27017"},
                    {_id:1, host:"localhost:27018"},
                    {_id:2, host:"localhost:27019", arbiterOnly:true}
                ]
	    };

Por último, se requiere ejecutar el método `rs.initiate()` para inicializar el "replica set", pasándole como parámetro la estructura que contiene la configuración del mismo.

    rs.initiate(cfg)

Todo esto lo podemos observar en la siguiente animación:

![Inicialización de replica set](doc/inicializacion-replica-set.gif)

Por defecto, MongoDB reconoce al nodo como SECONDARY. Apretamos `Enter` para que se reconfigure y tome ahora si al nodo como PRIMARY.

![Nodo primario](doc/primary.gif)

**3. Crear la base de datos `finanzas`.**

![footer](doc/footer.png)