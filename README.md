![header](doc/header.png)

# Trabajo Práctico Nº1

Autor:

* Agustín Curcio Berardi

Docente:

* Yoel Yamil López

1. Implementar en MongoDB un Replica Set con 3 servidores que contengan la información de la base de datos finanzas. Un nodo PRIMARY, un SECONDARY y un ARBITER.

Antes de inicializar las tres instancias de `mongod` vamos a crear los siguientes directorios:

    ├── data                # Directorio raíz para almacenar los datos.
        └── db              # Directorio para los datos de la base de datos.
            └── rs          # Directorio para las instancias que conformarán el Replica Set.
                ├── 0       # Directorio donde la instancia PRIMARY almacenará sus datos.
                ├── 1       # Directorio donde la instancia SECONDARY almacenará sus datos.
                └── 2       # Directorio donde la instancia ARBITER almacenará sus datos.

Para ello, en una consola hacemos lo siguiente:

    mkdir data && cd data
    mkdir db && cd db
    mkdir rs && cd db
    mkdir 0 1 2

Lo mismo se aprecia en el siguiente video:

![Creación de directorios](create-directory.mp4)

<!-- blank line -->
<figure class="video_container">
  <video controls="true" allowfullscreen="true" poster="path/to/poster_image.png">
    <source src="path/to/video.mp4" type="video/mp4">
    <source src="path/to/video.ogg" type="video/ogg">
    <source src="path/to/video.webm" type="video/webm">
  </video>
</figure>
<!-- blank line -->

![footer](doc/footer.png)