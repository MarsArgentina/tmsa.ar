# Acortador de Links [![Netlify Status](https://api.netlify.com/api/v1/badges/53ac9b8d-46e9-4974-86c7-a7db60c58f01/deploy-status)](https://app.netlify.com/sites/tmsa-short/deploys)

Dentro de Mars Society Argentina tenemos la necesidad de compartir enlaces a nuestras publicaciones, eventos, redes sociales, entre otras cosas, para lo cual contar con links breves y fáciles de recordar, resulta de gran ayuda.

Para esto contamos con el dominio https://tmsa.ar que funciona como acortador de vínculos. Por defecto redirige el tráfico a nuestra [página web][MarsSociety] sin embargo a través de este repositorio es posible agregar nuevos enlaces de acceso rápido, en muy poco tiempo.

## Cómo funciona (en 30 segundos)

No hay servidor ni base de datos. Todos los links viven en el archivo [`_redirects`](_redirects) (una línea por link: `/código  URL`). Al hacer push a `main`, [Netlify](https://www.netlify.com) despliega el sitio y `tmsa.ar/código` pasa a redirigir a la URL larga. Todo lo que no tenga regla propia cae en el comodín `/*` y va al mismo camino de [argentina.marssociety.org](http://argentina.marssociety.org/).

El comando `tmsa` (instalación abajo) solo automatiza editar `_redirects` + commit + push. También se puede agregar un link editando el archivo a mano.

## Documentación

- [docs/ARQUITECTURA.md](docs/ARQUITECTURA.md): cómo funciona cada pieza, qué hace el CLI y qué datos viven fuera del repo (cuenta de Netlify, DNS, dominio).
- [docs/OPERACION.md](docs/OPERACION.md): agregar, cambiar y borrar links, problemas frecuentes, cómo reconstruir el servicio desde cero y checklist de traspaso.
- [docs/NETLIFY.md](docs/NETLIFY.md): evaluación de Netlify (costos, límites, riesgos) y alternativas.

> **Ojo:** `tmsa` hace `commit` y `push` sin pedir confirmación, a la rama en la que estés. Parado en `main`, el link queda publicado al instante.

## Instalación

Para instalar este repositorio y poder agregar nuevos enlaces, deberás seguir los siguientes pasos:

### 0. Preparación

Antes de comenzar necesitarás tener [Git](https://git-scm.com/downloads) y [Node.js](https://nodejs.org/es/) instalados. Se sugiere que a la hora de instalar Git en Windows, el comando `git` esté disponible desde la terminal Cmd.

Para saber si Git y Node.js están correctamente instalados se pueden usar los siguientes comandos:

```sh
git --version
node --version
```

Una vez instalados, deberás [configurar las llaves SSH](https://docs.github.com/es/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh) para poder pushear el código a GitHub, usando Git. Este proceso es complejo así que deberás seguir los tutoriales del enlace.

### 1. Clonar

El primer paso de la instalación será clonar el repositorio.

**Es importante que lo clones, y no lo descargues.**

Al momento de clonar, se creará una carpeta con el nombre del repositorio, en el directorio actual. El comando a usar es el siguiente:

```sh
git clone git@github.com:MarsArgentina/tmsa.ar.git
```

### 2. Instalar Paquetes

Una vez clonado el repositorio ingresamos a la carpeta:

```sh
cd tmsa.ar
```

Y una vez allí deberemos instalar los paquetes usando NPM:

```sh
npm install
```

Con esto nuestro repositorio se encontrará instalado correctamente.

### 3. Link

El siguiente comando, crea un ejecutable `tmsa` que estará disponible en cualquier directorio. Este paso es opcional pero recomendado.

```sh
npm link
```

## Uso

### Si seguiste el paso 3

Si has seguido el [paso 3](#3-link) podrás correr el siguiente comando en cualquier terminal:

```sh
tmsa [link] [name]
```

Esto acortará el `link` a `https://tmsa.ar/name`. Si omitís `name`, se genera un código aleatorio de 5 caracteres. El link corto queda copiado en el portapapeles.

Antes de usarlo: tené el árbol de git limpio (el comando commitea todos los cambios pendientes) y elegí un `name` que no coincida con un archivo del repo (`package.json`, `README.md`, `docs`, etc.). Más detalles en [docs/OPERACION.md](docs/OPERACION.md).

### Si NO seguiste el paso 3

Para usar el comando deberás abrir una terminal en la carpeta del proyecto y correr el siguiente comando:

```
npm run shorten [link] [name]
```

Esto acortará el `link` a `https://tmsa.ar/name`

## Licencia

Este proyecto es una instancia de [Netlify Shortener](https://github.com/kentcdodds/netlify-shortener) por lo que la licencia de este proyecto es MIT, y el Copyright es de [Kent C. Dodds](https://github.com/kentcdodds)

[MarsSociety]: http://argentina.marssociety.org/
