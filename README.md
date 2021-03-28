![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Agustín Curcio Berardi

Docente:

* Yoel Yamil López

### Parte A: Replicación de bases de datos en MongoDB

---

**1. Implementar en MongoDB un "replica set" con tres servidores que contengan la información de la base de datos `finanzas`. Un nodo `primary`, un `secondary` y un `arbiter`.**

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

Luego resta ejecutar las tres instancias de `mongod` de la siguiente manera.

    mongod --replSet rs --dbpath /data/db/rs/0 --port 27017 --oplogSize 50
    mongod --replSet rs --dbpath /data/db/rs/1 --port 27018 --oplogSize 50
    mongod --replSet rs --dbpath /data/db/rs/2 --port 27019 --oplogSize 50

En la siguiente animación se observa como se reemplazan los `--dbpath` por los directorios correspondientes.

![Instanciación de mongod](doc/mongod-instances.gif)

**2. Conectarse al nodo `primary`.**

Ahora para configurar e inicializar el "replica set", tenemos que conectarnos desde un cliente de MongoDB al nodo `primary`. Para ello:

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

Por defecto, MongoDB reconoce al nodo como `secondary`. Apretamos `Enter` para que se reconfigure y tome ahora si al nodo como PRIMARY.

![Nodo primario](doc/primary.gif)

**3. Crear la base de datos `finanzas`.**

Como se necesita crear una base de datos relacionada con aplicaciones IoT, se creará la base de datos `cosmic`.

![Creación base de datos](doc/creacion-db.gif)

Para rellenar la base de datos con información, vamos a utilizar la estructura de datos que es empleada en el proyecto [Cosmic Pi](http://cosmicpi.org/). El mismo tiene como objetivo construir el telescopio de rayos cósmicos distribuido más grande del mundo. Al ser un proyecto de código abierto, utilizaremos la estructura de datos que generan los sensores y la completaremos con información ficticia. Dichos datos se encuentran definidos en `sensors.js`.

**4. Ejecutar el script `facts.js` cuatro veces para crear volumen de datos.**

Se utilizaron algunos de los campos que son enviados por los detectores de Cosmic Pi, tal cual se aprecia en el [repositorio oficial](https://github.com/CosmicPi/grafana_dashboards). Se generaron diez registros para dos sensores ficticios, los cuales se incluyeron en el archivo `sensors.js`. Para importarlos a la base de datos, solamente tenemos que correr el comando:

    load("sensors.js")

Tal cual se aprecia en la siguiente animación:

![Carga de datos](doc/load-json.gif)

**5. Buscar los datos insertados en el nodo `primary`.**

Para buscar los datos en el nodo primario, simplemente ejecutamos el comando:

    db.sensores.find()

Los datos que arroja dicho comando se pueden observar en el siguiente video.

![Muestra de datos en nodo primario](doc/show-collections.gif)

**6. Buscar los datos insertados en el nodo `secondary`.**

Para buscar los datos en el nodo secundario, primero nos conectamos  a él.

    mongo --port 27018

Antes de poder traer información debemos ejecutar el comando `rs.secondaryOk()` para que MongoDB habilite operaciones de lectura por parte del nodo secundario y pueda llevarse a cabo la replicación. Para ello, ejecutamos el comando en cuestión:

    rs.secondaryOk()

En esta instancia ya deberían estar copiados los datos al nodo secundario. Para buscarlos, primero simplemente seleccionamos la base de datos:

    use cosmic

Y por último mostramos los datos con el comando:

    db.sensores.find()

Todo el proceso se muestra a continuación:

![Lectura del secundario](doc/lectura-secundario.gif)

**7. Realizar un ejemplo de "fault tolerance", simulando una caída del servidor `primary`.**

Con el objetivo de mostrar un ejemplo de tolerancia a fallas, debemos disparar de algún modo el mecanismo de "failover". De esta manera, ante eventuales fallas que puedan ocurrir en el nodo primario, el sistema debería reaccionar y automáticamente seleccionar un nuevo nodo como primario.

Por este motivo, primero vamos a interrumpir la ejecución del nodo primario con la señal `Ctrl+C`. Luego vamos a verificar que en el cliente conectado a dicho nodo la comunicación se perdió, por ejemplo ejecutando `show dbs`. Y por último vamos a ver como el nodo secundario ahora se convierte en primario. Toda la secuencia de muestra debajo de estas líneas.

![Failover](doc/caida-primario.gif)

Así, se pudo comprobar que de forma "transparente" para el usuario el sistema realizó una votación entre los nodos que se encontraban activos y se seleccionó al nodo secundario como nuevo primario.

A continuación vamos a cargar nuevos datos en el nuevo nodo primario, que de ahora en más lo vamos a llamar `primary*`. Vamos a crear la base `cosmic1` y vamos a cargar los mismos datos que se encuentran en el archivo `sensors1.js`.

![Nuevo documento](doc/creacion-documento-primario.gif)

Como se pudo ver, el nodo `primary*` aceptó operaciones de escritura (si fuera secundario, esto no hubiera sido posible). Por lo que ahora si volvemos a inicializar el nodo `primary`, la información debería replicarse luego de ejecutar el comando `rs.secondaryOk()`, ya que solamente lo ejecutamos en el nodo secundario original.

El proceso completo de muestra a continuación:

![Nuevo secundario](doc/nuevo-secundario.gif)

Se observa que en los mensajes que se muestran en la consola de comandos que aparece `"newSyncSource":"localhost:27018"`, por lo que el sistema reconoce al `primary*`.

**8. Agregar un nuevo nodo con `slaveDelay` de 120 segundos.**

Para reutilizar los datos de la base de datos, antes de comenzar vamos a detener todo el cluster y borrar los contenidos de las directorios `/data/db/rs/0`, `/data/db/rs/1` y `/data/db/rs/2`. Una vez hecho eso, debemos crear un nuevo directorio donde el nuevo nodo guardará la información. Para ello, hacemos:

    cd /data/db/rs
    mkdir 3

Luego repetimos el punto 1) del trabajo, agregando la siguiente línea:

    mongod --replSet rs --dbpath /data/db/rs/3 --port 27020 --oplogSize 50

Una vez hecho esto, nos conectamos al nodo primario y creamos la variable `cfg`, incorporando esta vez al nuevo nodo de acuerdo a como se detalla a continuación:

    cfg = {
            _id:"rs",
            members:[
                {_id:0, host:"localhost:27017"},
                {_id:1, host:"localhost:27018"},
                {_id:2, host:"localhost:27019", arbiterOnly:true},
                {_id:3, host:"localhost:27020", slaveDelay:120, priority:0},
            ]
    };

Luego corremos el comando `rs.initiate(cfg)`. Todo el proceso se muestra a continuación:

![Nuevo miembro](doc/nuevo-miembro.gif)

> **NOTA**: A veces MongoDB demora en reconocer al nodo primario ni bien se inicializa el "replica set". Eso mismo se verificó en el video anterior.

**9. Ejecutar nuevamente el script `facts.js`. Asegurarse antes de ejecutarlo que el nodo con `slaveDelay` esté actualizado igual que PRIMARY.**

Vamos a crear la nueva base de datos con `use cosmic` y luego `load("sensors.js")`.

![Nuevos datos](doc/cargando-datos-nuevos.gif)

Y nos queda por último acceder al nodo con `id: 1` e `id: 3` para permitir que puedan realizarse lecturas en el nodo primario con `rs.secondaryOk()`.

En el caso del primero, al ejecutar el comando se deberían replicar los datos de forma instantánea, tal cual se muestra en la siguiente animación:

![Secundario sin delay](doc/lectura-secundario-1.gif)

Mientras que en el segundo, esto se hará también de forma inmediada, porque se lo indica así el comando `rs.secondaryOk()`, tal cual se aprecia a continuación.

![Secundario con delay](doc/lectura-secundario-2.gif)

Por este motivo, para verificar el funcionamiento del "delay" vamos a crear una nueva base con `use cosmic1` en el nodo `primary` y revisar que en el nodo sin `slaveDelay` la replicación se realiza inmediatamente, mientras que en el nodo con delay no ocurrirá lo mismo. El chequeo de los secundarios se realiza a las 20:58:35 horas.

![Revisión secundarios](doc/replicacion-con-delay.gif)

De la misma manera, a las 21:01:24 horas y habiendo transcurridos exactamente 180 segundos, se comprueba que los datos están presentes en el nodo con "delay".

![Revisión nodo con delay](doc/chequeo-luego-del-delay.gif)

### Parte B: Implementar "sharding" en MongoDB

---

**1.  Basados en la colección de `facturas` en la base de datos `finanzas`.**

Esto se implementó en la **Parte A** del presente trabajo. Se utilizará como base para esta segunda parte.

**2.  Levantar todas las instancias necesarias para tener un cluster distribuido con un único "shard".**

Como los "shards" y servidores de configuración requieren de un "replica set" cada uno, vamos a utilizar la siguiente 
estructura de carpetas. El proceso de creación es el mismo que en el punto 1) de la Parte A.

    ├── data                # Directorio raíz para almacenar los datos.
        └── db              # Directorio para los datos de la base de datos.
            └── rs          # Directorio para las instancias que conformarán el replica set.
                ├── 0       # Directorio donde la instancia PRIMARY del shard A.
                ├── 1       # Directorio donde la instancia SECONDARY del shard A.
                ├── 2       # Directorio donde la instancia ARBITER del shard A.
                ├── 3       # Directorio donde la instancia PRIMARY del shard B.
                ├── 4       # Directorio donde la instancia SECONDARY del shard B.
                ├── 5       # Directorio donde la instancia ARBITER del shard B.
                ├── 6       # Directorio donde la instancia PRIMARY del shard C.
                ├── 7       # Directorio donde la instancia SECONDARY del shard C.
                ├── 8       # Directorio donde la instancia ARBITER del shard C.
                ├── 9       # Directorio donde la instancia PRIMARY del replica set del servidor de configuración.
                ├── 10      # Directorio donde la instancia SECONDARY del replica set del servidor de configuración.
                └── 11      # Directorio donde la instancia ARBITER del replica set del servidor de configuración.

**Shard A**

Para inicializar el primer "shard" armamos el primer "cluster" de la siguiente manera, reemplazando el `--dbpath` 
por el que corresponde y agregamos la opción `--shardsvr` para indicar que forman parte de un "shard".

    mongod --shardsvr --replSet rs_A --dbpath /data/db/rs/0 --port 27017 --oplogSize 50
    mongod --shardsvr --replSet rs_A --dbpath /data/db/rs/1 --port 27018 --oplogSize 50
    mongod --shardsvr --replSet rs_A --dbpath /data/db/rs/2 --port 27019 --oplogSize 50

![Instanciación de mongod](doc/shard-instances.gif)

Luego nos conectamos al nodo primario:

    mongo --port 27017

Y configuramos el "cluster", tal cual se muestra aquí debajo:

    cfg = {
            _id:"rs_A",
            members:[
                {_id:0, host:"localhost:27017"},
                {_id:1, host:"localhost:27018"},
                {_id:2, host:"localhost:27019", arbiterOnly:true}
            ]
    };

Y se inicializa con `rs.initiate()`. A continuación se muestra el proceso completo.

![Instanciación de shard A](doc/initiate-shard-a.gif)

Vamos a repetir todo este proceso para los "shards" B y C. Para ello:

**Shard B**

    mongod --shardsvr --replSet rs_B --dbpath /data/db/rs/3 --port 27020 --oplogSize 50
    mongod --shardsvr --replSet rs_B --dbpath /data/db/rs/4 --port 27021 --oplogSize 50
    mongod --shardsvr --replSet rs_B --dbpath /data/db/rs/5 --port 27022 --oplogSize 50

Utilizando la siguiente configuración:

    cfg = {
            _id:"rs_B",
            members:[
                {_id:0, host:"localhost:27020"},
                {_id:1, host:"localhost:27021"},
                {_id:2, host:"localhost:27022", arbiterOnly:true}
            ]
    };

El proceso completo para el "shard" B se muestra a continuación:

![Configuración shard B](doc/configuracion-shard-b.gif)

**Shard C**

Los comandos serán:

    mongod --shardsvr --replSet rs_C --dbpath /data/db/rs/6 --port 27023 --oplogSize 50
    mongod --shardsvr --replSet rs_C --dbpath /data/db/rs/7 --port 27024 --oplogSize 50
    mongod --shardsvr --replSet rs_C --dbpath /data/db/rs/8 --port 27025 --oplogSize 50

Mientras que la variable `cfg` tendrá los siguientes datos:

    cfg = {
            _id:"rs_C",
            members:[
                {_id:0, host:"localhost:27023"},
                {_id:1, host:"localhost:27024"},
                {_id:2, host:"localhost:27025", arbiterOnly:true}
            ]
    };

Todo el proceso se muestra en el siguiente video:

![Configuración shard C](doc/configuracion-shard-c.gif)

**Servidores de configuración**

Para continuar con la instanciación vamos a configurar los servidores de configuración. En este caso solamente se requiere que 
sea un cluster, por lo que debemos reemplazar el "flag" `-shardsvr` por `--configsvr` para que MongoDB los considere como servidores
de configuración. Los comandos quedarían así:

    mongod --configsvr --replSet rs_C --dbpath /data/db/rs/6 --port 27023 --oplogSize 50
    mongod --configsvr --replSet rs_C --dbpath /data/db/rs/7 --port 27024 --oplogSize 50
    mongod --configsvr --replSet rs_C --dbpath /data/db/rs/8 --port 27025 --oplogSize 50

Algo a tener en cuenta es que este tipo de servidores no pueden ser árbitros, motivo por el cual debemos especificar en 
la variable `cfg` a los tres nodos como posibles secundarios.

    cfg = {
            _id:"C",
            members:[
                {_id:0, host:"localhost:27026"},
                {_id:1, host:"localhost:27027"},
                {_id:2, host:"localhost:27028"}
            ]
    };

Todo este proceso se muestra a continuación:

![Configuración controladores](doc/configuracion-rs-controlador.gif)

**Servidor de ruteo**

Para completar el proceso de inicialización del "shard", debemos levantar un servidor de ruteo. Esto es muy simple, ejecutando el 
siguiente comando:

    mongos --configdb c/localhost:27500

Una vez hecho eso, nos conectamos a dicho nodo:

    mongo --port 27500

Y agregamos al "shard" el que configuramos como partición A:

    sh.addShard("rs_A/localhost:27017")


**3.  Pensar las consultas que podrían realizarse a esta colección y definir una clave acorde para implementar "sharding".**

**4.  Implementar "sharding" en la base de datos `finanzas` sobre la colección `facturas`. Consultar la metadata del cluster.**

**5.  Agregar dos nuevos "shards" al cluster.**

**6.  Ejecutar el script `facts.js` cinco veces para crear volumen de datos.**

Por último, nos conectamos al nodo secundario del cluster A y habilitamos la replicación con `rs.secondaryOk()`.

![Habilitación réplica de shard A](doc/habilitacion-secundario-a.gif)

Antes de avanzar al servidor de configuración, cargamos datos en la base de datos `cosmic` utilizando el archivo `sensors.js` 
tal cual se ve debajo de estas líneas:

![Datos en shard A](doc/datos-en-shard-a.gif)

**7.  Consultar nuevamente la metadata del cluster. Ver los "shards" disponibles, los `chunks` creados y en que "shard" están estos.**

**8.  Definir dos consultas que obtengan cierta información de la base de datos e informar la salida del "explain". Una debe poder obtener**
**la información de un único "shard" y la otra debe tener que consultarlos a todos.**








---

![footer](doc/footer.png)