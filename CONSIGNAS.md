![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Yoel Yamil López

## Tema: Replicación de bases de datos en MongoDB

### Consignas

1. Implementar en MongoDB un ReplicaSet con 3 servidores que contengan la información de la base de datos `finanzas`. Un nodo PRIMARY, un SECONDARY y un ARBITER.
2. Conectarse al nodo PRIMARY.
3. Crear la base de datos `finanzas`.
4. Ejecutar el script `facts.js` cuatro veces para crear volumen de datos.
5. Buscar los datos insertados en el nodo PRIMARY.
6. Buscar los datos insertados en el nodo SECONDARY.
7. Realizar un ejemplo de "Fault Tolerance", simulando una caída del servidor PRIMARY.
    1. Explicar qué sucedió.
    2. Verificar el estado de cada servidor.
    3. Insertar un nuevo documento.
    4. Levantar el servidor caído.
    5. Validar la información en cada servidor.
8. Agregar un nuevo nodo con `slaveDelay` de 120 segundos.
9. Ejecutar nuevamente el script `facts.js`. Asegurarse antes de ejecutarlo que el nodo con `slaveDelay` esté actualizado igual que PRIMARY.
    1. Luego de ejecutado chequear el SECONDARY.
    2. Consultar el nuevo nodo y ver cuando se actualizan los datos.

![footer](doc/footer.png)