# DjangoDockerPostgreSQL
Desarrollando proyecto de django con docker, usando la bd PostgreSQL y PGadmin para visualizar la bd en una interfaz gráfica

# Vamos a usar docker compose
 Entonces tenemos que generar un archivo .yml en donde va a ir lo que utilizaremos, para que se creen los contenedor
 
 docker compose ayuda a organizar los contenedores y también a lanzar varios contenedores con poquísimos comando.

 # Para comenzar
    Creamos los archivos en nuestro directorio local:
    Dockerfile
    docker-compose.yml
    requirements.txt

# En requirements.txt:
    Django==4.2.9
    psycopg2-binary==2.9.10 (Porque voy a usar la bd PostgreSQL)
    pillow==11.1.0

# Ahora nos vamos a Dockerfile
    FROM python:3.13.2-alpine3.21

    ENV PYTHONUNBUFFERED=1
    
    WORKDIR /code

    COPY requirements.txt /code/

    RUN python -m pip install --upgrade pip && \
        pip install -r requirements.txt

    COPY . /code/

# Ahora el docker-compose.yml

    version: '3.8' 
    (Después de la versión de docker compose es 2.2, no es necesario colocarlo, pero por buenas prácticas se puede colocar, después de esa versión suele ponerse 3.8)

    services: 
    (son los paquetes que van a formar parte de mi app, en este caso el servicio de django, PostgreSQL, pgadmin)


# Entonces:
    El servicio de Django nos damos cuenta que depende del servicio de PostgreSQL
    Entonces primero tiene que instalarse
    PostgreSQL luego PGadmin y por último Django


#  Entonces sería así:
    version: '3.8'
    services:
        db:
            image: postgres:15.10
            restart: always (cuando hay un error en bd o derrepente se cae, con esto se levanta automaticamente)
            container_name: postgresql
            
            Volumes:(Para el tema de persistencia de datos, para que al momento que yo apague la máquina, esta siga)
                - ./data/db:/var/lib/postgresql/data
            environment: (Las variables de entorno de Postgre)
                - DATABASE_HOST=127.0.0.1
                - POSTGRES_DB=my_database
                - POSTGRES_USER=admin
                - POSTGRES_PASSWORD=admin
            ports:
                - "5432:5432"
        
        pgadmin:
            image: dpage/pgadmin4
            container_name: pgadmin
            environment:
                - PGADMIN_DEFAULT_EMAIL=jamer.omar.asencio.urcia@gmail.com
                - PGADMIN_DEFAULT_PASSWORD=admin
            ports:
                - "80:80"
            depends_on:
                - db
        
        web:
            build: .
            container_name:django
            command: python manage.py runserver 0.0.0.0:8000
            volumes:
                - .:/code
            ports:
                - "8000:8000"
            environment:
                - POSTGRES_NAME=my_database
                - POSTGRES_USER=admin
                - POSTGRES_PASSWORD=admin
            depends_on:
                - db


# TODO SERÍA:
version: '3.8'
services:
  db:
    image: postgres:15.10
    restart: always 
    container_name: postgresql
            
    volumes:
      - ./data/db:/var/lib/postgresql/data
    environment: 
      - DATABASE_HOST=127.0.0.1
      - POSTGRES_DB=my_database
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=admin
    ports:
      - "5432:5432"
        
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    environment:
      - PGADMIN_DEFAULT_EMAIL=jamer.omar.asencio.urcia@gmail.com
      - PGADMIN_DEFAULT_PASSWORD=admin
    ports:
      - "80:80"
    depends_on:
      - db
        
  web:
    build: .
    container_name: django
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    environment:
      - POSTGRES_NAME=my_database
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=admin
    depends_on:
      - db

# Ahora vamos a ejecutar:

    
