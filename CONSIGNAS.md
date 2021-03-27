![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Yoel Yamil López

### Parte A: Replicación de bases de datos en MongoDB

---

1. Implementar en MongoDB un "replica set" con tres servidores que contengan la información de la base de datos `finanzas`. Un nodo `primary`, un `secondary` y un `arbiter`.

2. Conectarse al nodo `primary`.

3. Crear la base de datos `finanzas`.

4. Ejecutar el script `facts.js` cuatro veces para crear volumen de datos.

5. Buscar los datos insertados en el nodo `primary`.

6. Buscar los datos insertados en el nodo `secondary`.

7. Realizar un ejemplo de "fault tolerance", simulando una caída del servidor `primary`.

    1. Explicar qué sucedió.
    2. Verificar el estado de cada servidor.
    3. Insertar un nuevo documento.
    4. Levantar el servidor caído.
    5. Validar la información en cada servidor.

8. Agregar un nuevo nodo con `slaveDelay` de 120 segundos.

9. Ejecutar nuevamente el script `facts.js`. Asegurarse antes de ejecutarlo que el nodo con `slaveDelay` esté actualizado igual que `primary`.

    1. Luego de ejecutado chequear el `secondary`.
    2. Consultar el nuevo nodo y ver cuando se actualizan los datos.

### Parte B: Implementar "sharding" en MongoDB

---

1.  Basados en la colección de `facturas` en la base de datos `finanzas`.

2.  Levantar todas las instancias necesarias para tener un cluster distribuido con un único "shard".

3.  Pensar las consultas que podrían realizarse a esta colección y definir una clave acorde para implementar "sharding".

4.  Implementar "sharding" en la base de datos `finanzas` sobre la colección `facturas`. Consultar la metadata del cluster.

5.  Agregar dos nuevos "shards" al cluster.

6.  Ejecutar el script `facts.js` cinco veces para crear volumen de datos.

7.  Consultar nuevamente la metadata del cluster. Ver los "shards" disponibles, los `chunks` creados y en que "shard" están estos.

8.  Definir dos consultas que obtengan cierta información de la base de datos e informar la salida del "explain". Una debe poder obtener
la información de un único "shard" y la otra debe tener que consultarlos a todos.


![footer](doc/footer.png)