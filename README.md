## correr una imagen con docker

`docker run node:14-alpine`

## Error al intentar crear un proyecto nuxt

al intentar crear un proyecto nuxt con docker, este arrojaba problemas, pero
la solución la encontré en [github](https://github.com/nuxt/create-nuxt-app/issues/383)

el procesamiento es que primero se debe instalar de forma global el create app

```
npm i -g create-nuxt-app
```

luego la interacción con `create-nuxt-app`

```
npm init nuxt-app <my-project-name>
```

### `desde fuera de docker`

En teoría, para iniciar un nuevo proyecto con nuxt bastaría con correr:

```
docker run -it -v $(pwd):/app --name my-app-name -w /app node:14-alpine npm init nuxt-app <my-project-name>
```

`-v`: hace referencia al volumen para la persistencia de datos /path/del/host>:/path/en/el/container <br/>
`--name`: nombre al contenedor que vas a correr <br/>
`-w`: para indicar el workspace

Pero se requiere que exista el paquete `create-nuxt-app` por lo que primero se tendrá que construir la imagen de un dockerfile
para instalarle el paquete de forma global y luego crear el nuevo proyecto.

## Datos para docker

para entrar a la linea de comando o shell del contenedor de la imagen correr:
(se asume que es una imagen que contiene ubuntu, para alpine parece ser `/bin/ash`)

```
docker run -it <image> /bin/bash
```

`-i`: indica que sea interactivo <br/>
`-t`: indica que sea en un terminal emulado <br/>

- login en el grupo para evitar usar sudo:

```
newgrp docker
```

### `Construir una imagen de docker`

para construir una imagen de docker se requiere el siguiente comando básico

```
docker build .
```

pero con más opciones es:

```
docker build -f /path/to/a/Dockerfile . -t nombre/tag
```

# MOSTRANDO PASOS

## `CONSTRUIR UNA IMAGEN PARA INICIAR UN PROYECTO NUXT`

```
docker build -f ./Dockerfile.init.nuxt . -t wilder/init-nuxt-project
```

wilder es mi nombre, puede ser cualquier nombre y sin `/` si solo es para local.

el tag `-t` tambien puede ser init-nuxt-project:1.0.0

## `CREAR UN NUEVO PROYECTO NUXT`

imaginando que el tag de la imagen es `init-nuxt-project`

```
docker run -it -v $(pwd):/app -w /app --name my-nuxt-init-container init-nuxt-project <project-name>
```

si ya ha sido corrido el contenedor con `--name <container-name>`, solo hace falta volverlo a
llamar de manera simple

```
docker run <container-name> <project-name>
```

## `ACERCA DEL Dockerfile.init.nuxt`

la idea es crear una imagen que inicialice un proyecto de nuxt. Es como
una herramienta global inicial, que si yá está construido el proyecto con nuxt entonces no requiere tenerlo dentro.

```
#alpine es una distribución linux para contenedores
FROM node:14-alpine
#cuando se corra la imagen se deberá ingresar el volumen `-v` a /app
WORKDIR /app
RUN ["npm", "i", "-g", "create-nuxt-app"]
#ENV APP_NAME="my-app"
# ${variable:-word}  "echo $HOME"
#CMD [ "npm", "init", "nuxt-app", "."]
ENTRYPOINT [ "npm", "init", "nuxt-app"]
#la idea es que se ejecute nombre-docker <nombre-app>
```
