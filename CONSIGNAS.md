![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Yoel Yamil López

## Tema: Replicación de bases de datos en MongoDB

### Consignas

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


![footer](doc/footer.png)