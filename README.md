**Docker Hub (Registry)**

```bash
docker login
```

Incluir tu usuario y contraseña para autenticarte con tu repositorio de imágenes.

Lo siguiente es aprovechar nuestra pasada imagen que llamamos hello:v1 y pasarla para ser usada en nuestro contenedor, así que de igual manera vamos a nuestra linea de comandos y escribimos:

```bash
docker tag hello:v1 sennayrton/web-docker
```

Necesitamos volver a correr nuestro comando para consultar las imágenes que tenemos:

```bash
docker image ls
ó
docker images
ó
docker image list
```

La imagen hello:v1 aun existe y hay una nueva que se llama como tu repositorio en docker hub, algo así como sennayrton/web-docker:latest y, ¿por qué latest como tag de esta imagen? Esto es porque si no especificamos un tag, docker interpretara que esta es la ultima creada y la más actualizada, perfecto, pero si revisamos nuestra cuenta de docker hub aún esta vacía, esto es porque esta nueva imagen está en local, debemos de subirla, para esto solo debemos de ingresar:

```bash
docker push sennayrton/web-docker:latest
```

Y listo ya hemos creado nuestra primer imagen en un repositorio, imagen que podemos utilizar en cualquier momento para hacer un deploy o un run en local.

**Docker Services**

Un cluster está compuesto de varios nodos y dentro de estos nodos existen pod y dentro de estos pods existirán nuestras aplicaciones (containers).

Para está practica vamos a crear el siguiente código:

Dockerfile, requirements.txt, app.py

Ahora vamos a crear una imagen de esto con la siguiente instrucción:

```
docker build --tag=sennayrton/docker-demo2:latest .
```

Recuerda usar tu nombre de repositorio, bien ahora subamos esta imagen al repositorio:

```
docker push sennayrton/docker-demo2:latest
```

Bueno esto es para ir entendiendo el funcionamiento de este ecosistema, para este ejemplo utilizaremos un archivo de docker-compose.yml, el cual, nos dará lo necesario para exponer nuestra aplicación que acabamos de crear.

Ejemplo del archivo:

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: sennayrton/docker-demo2:latest
    deploy:
      replicas:2
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

### ¿Qué estamos diciendo en este archivo?

- Usa de la imagen que subimos con el tag part2 .
- Ejecute 2 instancias de esa imagen como un servicio llamado web,
limitando a cada una a usar, como máximo, el 10% de la CPU (en todos los núcleos) y 50 MB de RAM.
- Reinicie inmediatamente los contenedores si falla uno.
- Asigne el puerto 4000 en el host al puerto web 80 de nuestra aplicación.
- Indique a los contenedores de la web que compartan el puerto 80 a
través de una red de carga equilibrada llamada webnet. (Internamente,
los propios contenedores se publican en el puerto 80 de la web en un
puerto efímero).
- Defina la red web con la configuración predeterminada (que es una red de superposición con equilibrio de carga).

Bien  ahora vamos a iniciar un cluster aunque este proceso lo retomaremos luego con más profundidad. Ingresa el comando:

```
docker swarm init
```

Con eso iniciamos el cluster, ahora vamos a desplegar la aplicación con nuestro nuevo archivo y a esta app le llamaremos myApp, para esto ingresa por linea de comandos:

```
docker stack deploy -c docker-compose.yml myApp
```

Perfecto, ahora veamos nuestro servicios corriendo, ingresemos:

```
docker service ls
```

Deberían ver en pantalla algo similar a esto:


<img width="677" alt="Captura" src="https://user-images.githubusercontent.com/50055674/162172682-b0d9979c-e4f1-418e-863e-c165b074653d.png">


Bien ese es nuestro servicio corriendo, con nuestro balanceador de carga en local, ahora nosotros declaramos en nuestro archivo que queríamos 2 contenedores(aplicaciones) corriendo en simultáneo, para verlo usa en linea de comandos:

```
docker container ls -la
```

Nuestras aplicaciones corriendo:


<img width="1004" alt="Captura2" src="https://user-images.githubusercontent.com/50055674/162173165-f8c79ec0-73a3-4bc3-9c3b-14c926af6c8b.png">


Puedes ir a tu navegador e ingresar http://localhost:4000 y si refrescas varias veces veras que el Hostname cambia, claro están 
corriendo en contenedores diferentes, que tienen una misma salida. (Si estas corriendo esto en windows posiblemente la salida no sea 
localhost para ti, sino la ip de la maquina virtual que tenga linux instalado).


Ahora vamos a escalar nuestra aplicación, donde antes decía replicas : 2 ahora dile que son 5:

<img width="702" alt="Captura3" src="https://user-images.githubusercontent.com/50055674/162173484-97ef8220-8b40-4668-bdf9-632c005ae452.png">

<img width="1042" alt="Captura4" src="https://user-images.githubusercontent.com/50055674/162173693-2c3f8641-16c2-49a4-b483-70d5cbac42da.png">


Vemos el [localhost:4000](http://localhost:4000) y el cambio de hostname:

![Captura5](https://user-images.githubusercontent.com/50055674/162174237-a5e9537b-0c42-461c-b400-6e9cb2ed4293.png)

![Captura6](https://user-images.githubusercontent.com/50055674/162174325-de5226e3-ec9c-4f55-b163-abcd7267e9ce.png)


Perfecto a esto se le llama escalar una aplicación y es uno de los conceptos fundamentales a tener en cuenta, bien hasta aquí hoy aprendimos a crear un cluster, desplegamos nuestro propio balanceador de carga, escalamos una aplicación, ahora limpiemos este desastre:

Eliminamos la aplicación:

```bash
docker stack rm myApp
```

Eliminemos el cluster:

```bash
docker swarm leave --force
```


He utilizado la web: https://devopsfacilito.blogspot.com/2019/02/3-docker-services-devops-series.html , para seguir los pasos y aprender.

