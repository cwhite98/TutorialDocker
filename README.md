# Tutorial Docker y AWS

## Introducción

## Objetivos

## Docker
* Docker es una plataforma para desarrollar, desplegar y correr aplicaciones con contenedores.

* La contenedorización es popular ya que es:
  - Flexible: incluso las aplicaciones mas complejas pueden ser contenidas en contenedores.
  - Ligera: los contenedores aprovechan y comparten el núcleo de host.
  - Intercambiable: puede implementar actualizaciones y actualizaciones sobre la marcha.
  - Portátil: se puede compilar localmente, implementarlo en la nube y ejecutar en cualquier lugar.
  - Escalable: puede aumentar y distribuir automáticamente réplicas de contenedor.
  - Apilable: puede apilar servicios verticalmente y sobre la marcha.

* Un contenedor se lanza corriendo una **imagen** (paquete ejecutable que incluye todo lo necesario para ejecutar una aplicación).

* Un **contenedor** es una instancia de tiempo de ejecución de una imagen.


### Instalación

#### 1. Instalar Docker
##### Ubuntu
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-
key add -
$ sudo add-apt-repository "deb [arch=amd64] 
https://download.docker.com/linux/ubuntu$(lsb_release -cs) stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

##### CentOS
```
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce
$ sudo systemctl start docker
$ sudo systemctl enable docker
$ sudo usermod -aG docker user1
$ sudo curl -L https://github.com/docker/compose/releases/download/1.24.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

##### Windows 
Tener en cuenta que Docker solo está disponible para Windows 10 64bit: Pro, Enterprise or Education.
[Docker Desktop para Windows](https://docs.docker.com/docker-for-windows/install/)

##### MacOS
[Docker Desktop para Mac](https://docs.docker.com/docker-for-mac/install/)

#### 2. Verificar la versión de Docker
```
$ docker –version 
```
#### 3. Conocer más a fondo información sobre Docker.
Contenedores pausados, contenedores corriendo actualmente, etc.
```
$ docker info
```
#### 4. Probar que docker fue instalado correctamente.
```
$ docker run hello-world
```
Este comando debería arrojar el siguiente resultado.
![](./docs/instalacion4.png)

#### 5. Listar las imágenes descargadas en el computador.
```
$ docker image ls
```

#### 6. Listar los contenedores existentes corriendo actualmente.
```
$ docker container ls
```
Listar todos los contenedores existentes, incluso los que no están corriendo. 
```
$ docker container ls –all
```

#### Cheat Sheet
![](./docs/cheatsheetinstalacion.png)

### Contenedores
* En el pasado, si comenzaba a escribir una aplicación de Python, su primera tarea era instalar un tiempo de ejecución de Python en su máquina. Con Docker, solo puede tomar un tiempo de ejecución Python portátil como una imagen, sin necesidad de instalación. Estas imágenes portátiles están definidas por algo llamado Dockerfile.

* Un **Dockerfile** define lo que sucede en el ambiente dentro de su contenedor. 

#### 1. Crear una aplicación en python (app.py) con el siguiente contenido. 
```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis                                                                                                    
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
app = Flask(__name__)
@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
               "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"),  hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=3000)
```
#### 2.	Crear el archivo requirements.txt con el siguiente contenido.
```
Flask
Redis
```
#### 3.	Instalar Flask y Redis.
```
$ pip install -r requirements.txt
```
#### 4.	Correr la aplicación.
```
$ python app.py
```
Ir al browser y buscar http://0.0.0.0:3000/ .
Deberían ver algo similar a la siguiente imagen.
![](./docs/contenedores4.png)

#### 5.	En este punto se tiene una aplicación funcional sin Docker.

#### 6.	Crear el Dockerfile con el siguiente contenido, este archivo sirve para crear una imagen de la aplicación. 
```Dockerfile
# Build an image starting with the Python 2.7
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD python app.py
```
Docker cuenta con diferentes instrucciones para componer el Dockerfile: ADD, COPY, ENV, EXPOSE, FROM, LABEL, STOPSIGNAL, USER, VOLUME y WORKDIR.

#### 7.	Correr el comando build para crear la imagen.
```
$ docker build -t nameApp .
```
#### 8.	Verificar si la imagen fue creada.
```
$ docker image ls
```
Este comando debería mostrar el siguiente resultado.
![](./docs/contenedores8.png)

#### 9.	Correr la app.
```
$ docker run -p 3000:3000 nameApp
```
Se puede acceder a la app por medio de http://localhost:3000,  debido a que el mensaje que imprime la app viene dentro del contenedor y este no mapea el puerto 3000 con el 3000. 
También se puede correr: 
```
$ curl http://localhost:3000
```
y se obtiene lo siguiente. 
Para correr la app en detached mode, usar el siguiente comando.

```
$ docker run -d -p 3000:3000 nameApp
```
#### 10.	Si se desea parar el contenedor.
```
$ docker container stop <id>
```
#### 11. Para compartir la imagen, crear una cuenta en https://hub.docker.com si aun no la ha creado.

#### 12.	Acceder a esta desde la terminal.
```
$ docker login
```
#### 13.	Ponerle un tag a la imagen.
```
$ docker tag <image> <username/repository:tag>
```
Un ejemplo de este comando es: 
```
$ docker tag friendlyhello cwhiter/get-started:part2
```
#### 14.	Verificar que la imagen creada con el tag anterior si exista.
```
$ docker image ls
```
#### 15.	Publicar la imagen.
```
$ docker push <username/repository:tag>
```
#### 16.	Correr la imagen desde cualquier lugar. 
```
$ docker run -p 3000:3000 <username/repository:tag>
```
Y desde cualquier browser accede a http://localhost:3000.
#### Cheat Sheet
![](./docs/compartirimagen8.png)

### Servicios
* Son contenedores en producción. 

* Corre solamente una imagen.

* Para esto, creamos un docker-compose, el cual define como los contenedores deberían comportarse en producción. 

* A un contenedor solo corriendo en un servicio se le llama task. 

#### 1.	Crear el docker-compose.yml con el siguiente contenido.	
```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    restart: always
    build: .
    ports:
      - "3000:3000"
```
Este archivo le dice a Docker que:
  * Obtenga la imagen creada previamente.
  * Reiniciar los contenedores siempre.
  *	Mapear el puerto 3000 al 3000. 

#### 2.	Correr la aplicación.
```
$ docker-compose build
$ docker-compose up
```
Para correr en detached mode
```
$ docker-compose up -d
```
#### 3.	Abrir la app en un browser.

## Desplegar en AWS

#### 1.	Crear una cuenta en [AWS Educate](https://aws.amazon.com/education/awseducate/).

#### 2.	Crear una maquina virtual EC2 y lanzarla. 

#### 3.	Escoger el sistema operativo Amazon Linux 2 64-bit(x86), el cual funciona como un CentOS. 
![](./docs/EscogerSO.png)

#### 4.	Escoger el Free tier y presionar *Review and Launch*.
![](./docs/FreeTier.png)
Volver a presionar *Launch*. 

#### 5.	Crear una nueva *key*, darle un nombre y descargarla. 
![](./docs/key.png)
Presionar *Launch Instance* una vez se haya descargado la *key*.

#### 6.	Presionar la instancia para verificar en que estado se encuenta. 
![](./docs/instancia.png)
Al ser una nueva máquina virtual, su estado es *pending* (amarillo) pero luego de unos segundos cambiará a *running* (verde). 

#### 7.	Darle un nombre a la máquina virtual.



###### Referencias
https://docs.docker.com/get-started/ 
