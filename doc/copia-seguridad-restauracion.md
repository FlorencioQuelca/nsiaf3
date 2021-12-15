# COPIAS DE SEGURIDAD SISTEMA DE ALMACENES
## PROCESO PARA REALIZAR UNA COPIA DE SEGURIDAD DE LOS DATOS
### **Sistema deplegado con docker**
1. Obtener el identificador de contenedor de la base de datos ejecute el siguiente comando

    ```
    docker ps 
    ```

    Obtendra una respuesta similar a está
    ```
    CONTAINER ID   IMAGE          COMMAND                 CREATED      STATUS      PORTS     NAMES
    59d44e33fb0a   nsiaf:1.0.0    "/app/docker/entrypo…"  3 weeks ago  Up 3 weeks  3000/tcp  nsiaf_backend.1.spa0lfmuu5qg01cc5cn1xighd
    adc30c22967a6  mariadb:10.2   "docker-entrypoint.s…"  3 weeks ago  Up 3 weeks  3306/tcp  nsiaf_db.1.luvognyf9zqd56u2xzr7z5chr
    ```
    Debemos obtener el identificador de contenedor de la base de datos **mariadb:10.2** que tenga el prefijo **nsiaf_db** en el nombre de contenedor.

2. Ejecutar el comando para obtener la copia de seguridad.

    ```
    docker exec -i [ID_CONTENEDOR] mysqldump -u root -p nsiaf_production > [NOMBRE_DE_ARCHIVO]
    ```
    Donde:

    **ID_CONTENEDOR**, es el **identificador de contenedor de la base de datos** que identificamos en el **primer paso**.

    **NOMBRE_DE_ARCHIVO**, es el nombre de archivo que le pondremos a la copia de seguridad, puede tomar el siguiente estandar [AAAAMMDDHHmm]_[TIPO_SISTEMA].sql donde **AAAAMMDDHHmm** es la fecha y hora en que se realiza la copia de seguridad indicando el **año, mes, dia, hora y minutos** y **TIPO_SISTEMA** el sistema del que se hace la copia de seguridad **activos** o **almacenes**, por ejemplo: **202110111136_almacenes.sql**

    Un ejemplo de la ejecución del comando es el siguiente:
    ```
    docker exec -i adc30c22967a6 mysqldump -u root -p nsiaf_production > 202110111136_almacenes.sql
    ```
3. Verificación del contenido del backup:
    ```
    cat [NOMBRE_DE_ARCHIVO]
    ```
    Verifique que el contenido del archivo contenga los querys correspondiente para crear las tablas y el llenado de los datos de las mismas.





    ---
    **NOTA**

    Sí por algún motivo, al realizar la verificación del archivo nota que no se encuentran los querys para regenerar la base de datos, verifique si el usuario **activos** tenga los permisos adecuados, ingrese a la base de datos 
    y ejecute lo siguiente: 
    ```
    GRANT ALL ON *.* TO 'activos'@'%' IDENTIFIED BY 'complex-password';
    FLUSH PRIVILEGES;
    ```
    ---


## PROCESO PARA REALIZAR UNA RESTAURACION DE DATOS
### **Sistema deplegado con docker**

1. Obtener el identificador de contenedor de la base de datos ejecute el siguiente comando

    ```
    docker ps 
    ```

    Obtendra una respuesta similar a está
    ```
    CONTAINER ID   IMAGE          COMMAND                 CREATED      STATUS      PORTS     NAMES
    59d44e33fb0a   nsiaf:1.0.0    "/app/docker/entrypo…"  3 weeks ago  Up 3 weeks  3000/tcp  nsiaf_backend.1.spa0lfmuu5qg01cc5cn1xighd
    adc30c22967a6  mariadb:10.2   "docker-entrypoint.s…"  3 weeks ago  Up 3 weeks  3306/tcp  nsiaf_db.1.luvognyf9zqd56u2xzr7z5chr
    ```
    Debemos obtener el identificador de contenedor de la base de datos **mariadb:10.2** que tenga el prefijo **nsiaf_db** en el nombre de contenedor.

2. Para realizar la restauración de datos se lo debe hacer en una base de datos vacia sin tablas, por lo tanto debemos borrar y crear una base de datos nueva.

    Ingresar a la base de datos:
    ```
    docker exec -it [ID_CONTENEDOR] mysql -u root -p 
    ```
    Donde:
    **ID_CONTENEDOR**, es el **identificador de contenedor de la base de datos** que identificamos en el **primer paso**.

    
    Dentro de la consola de la base de datos ingresar el siguiente comando para eliminar la base de datos:
    ```
    drop database nsiaf_production;
    ```
    Y ejecutar el siguiente comando para crear la base de datos:
    ```
    create database nsiaf_production;
    ```
3. Copiar el archivo de la copia de seguridad dentro del contenedor de la base de datos. En este caso lo ponemos en el directorio **/tmp**
    ```
    docker cp [NOMBRE_DE_ARCHIVO] [ID_CONTENEDOR]:/tmp
    ```
    Donde:
    **NOMBRE_DE_ARCHIVO**, es el nombre de archivo de la copia de seguridad.

4. Ingresamos al contenedor de la base de datos
    ```
    docker exec -it [ID_CONTENEDOR] bash 
    ```

5. Una vez adentro del contenedor, ejecutamos el siguiente comando para regenerar la base de datos(antes debe salir de la consola de la base de datos, puede usar  **ctrl + d**): 

    ```
    mysql -u root -p nsiaf_production < /tmp/[NOMBRE_DE_ARCHIVO]
    ```
6. Ingresar a la base de datos y verificar que se hayan restaurado la base de datos, puede ejecutar el siguiente comando para ver las tablas de la base de datos:
    ```
    docker exec -it [ID_CONTENEDOR] bash
    ```
    Nos conectamos a la base de datos
    ```
    connect nsiaf_production;
    ```
    Y verificamos las tablas de la base de datos
    ```
    show tables;
    ```
7. Una vez adentro borrar algunos registros con el siguiente comando:
    ```
    delete from schema_migrations where version in ('20190206211353','20200128101819', '20201117134437','20201231085624', '20210112134500');
    ```
8. Salimos de la consola de base de datos e identificamos el contenedor del backend con el siguiente comando
    ```
    docker ps 
    ```

    Obtendra una respuesta similar a está
    ```
    CONTAINER ID   IMAGE          COMMAND                 CREATED      STATUS      PORTS     NAMES
    59d44e33fb0a   nsiaf:1.0.0    "/app/docker/entrypo…"  3 weeks ago  Up 3 weeks  3000/tcp  nsiaf_backend.1.spa0lfmuu5qg01cc5cn1xighd
    adc30c22967a6  mariadb:10.2   "docker-entrypoint.s…"  3 weeks ago  Up 3 weeks  3306/tcp  nsiaf_db.1.luvognyf9zqd56u2xzr7z5chr
    ```
    Debemos obtener el identificador de contenedor de la base de datos **nsiaf:1.0.0** que tenga el prefijo **nsiaf_backend** en el nombre de contenedor y ejecutar el siguiente comandos

    ```
    docker exec -i [ID_CONTENEDOR] bundle exec rake db:migrate
    ```
9. Bajar y levantar los servicios de docker swarm con los siguiente comandos
    Comando para bajar los servicios docker swarm
    ```
    docker stack rm nsiaf
    ```
    Comando para levantar los servicios docker swarm del sistema de almacenes
    ```
    docker stack deploy -c docker/docker-stack.yml nsiaf
    ```