![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Agustín Curcio Berardi

Docente:

* Yoel Yamil López

**1. Implementar en MongoDB un `replica set` con 3 servidores que contengan la información de la base de datos `finanzas`. Un nodo `primary`, un `secondary` y un `arbiter`.**

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

---

![footer](doc/footer.png)