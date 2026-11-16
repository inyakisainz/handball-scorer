# Instalación

La aplicación es una web instalable (PWA). Se abre una vez con conexión, se añade a la pantalla de inicio y desde entonces funciona como una app, también sin conexión.

Dirección: **https://inyakisainz.github.io/handball-scorer/**

## Instalar como app

### Android (Chrome)

1. Abre la dirección en Chrome.
2. Toca el menú de tres puntos, arriba a la derecha.
3. Elige **Instalar aplicación** o **Añadir a pantalla de inicio**.
4. Confirma. El icono aparece junto al resto de tus apps.

### iPhone y iPad (Safari)

1. Abre la dirección en **Safari**. En iOS la instalación completa solo funciona desde Safari.
2. Toca el botón **Compartir** (el cuadrado con la flecha hacia arriba).
3. Elige **Añadir a pantalla de inicio**.
4. Toca **Añadir**.

### Ordenador (Chrome o Edge)

1. Abre la dirección.
2. Pulsa el icono de instalar que aparece en la barra de direcciones, o el menú y **Instalar Anotador de balonmano**.

## Comprobar que funciona sin conexión

Hazlo siempre antes del primer partido real; en muchos pabellones no hay cobertura.

1. Abre la app con conexión y ciérrala.
2. Vuelve a abrirla, también con conexión. En el menú (icono de personas), pestaña **Datos**, el estado debe decir que el funcionamiento sin conexión está **listo**.
3. Pon el dispositivo en modo avión y abre la app. Debe cargar con normalidad.

## Recomendaciones para el partido

- **Tablet en horizontal.** Se ven a la vez quién, el tiro y las demás acciones. En móvil funciona, alternando entre Tiro y Otra acción.
- **Batería y brillo.** La pantalla se mantiene encendida mientras corre el reloj, si el dispositivo lo permite.
- **Instalada mejor que en el navegador.** Hay más espacio en pantalla y, en iPhone y iPad, el sistema no borra los datos por falta de uso.

## Dónde están los datos

- En el almacenamiento del navegador de ese dispositivo. No se envían a ningún sitio.
- Se pierden si borras los datos de navegación del sitio o desinstalas la app.
- En iPhone y iPad, Safari puede borrar los datos de una web que lleva semanas sin abrirse si no está instalada en la pantalla de inicio.
- Exporta una copia después de cada partido: menú, pestaña **Datos**, **Exportar copia completa**. Ese archivo se puede importar en otro dispositivo.

## Actualizaciones

La app se actualiza sola. Cuando hay una versión nueva, se descarga al abrirla con conexión y entra en uso la siguiente vez que la abres. Los datos se conservan.

## Desinstalar

- **Android:** mantén pulsado el icono y elige Desinstalar.
- **iPhone y iPad:** mantén pulsado el icono y elige Eliminar app.
- **Ordenador:** en la ventana de la app, menú de tres puntos y Desinstalar.

Al desinstalar se borran los datos. Exporta antes una copia si quieres conservarlos.

## Alojarla en tu propio servidor

No hace falta para usarla, pero puedes servirla tú mismo. Solo hay que publicar la carpeta `public/` como sitio estático **con HTTPS**; sin HTTPS no se instala ni funciona sin conexión. Todas las rutas son relativas, así que puede ir en una subcarpeta.

Caddy, con certificado automático:

```
anotador.tudominio.com {
    root * /srv/handball-scorer/public
    file_server
    header /sw.js Cache-Control "no-cache"
    header /index.html Cache-Control "no-cache"
}
```

Nginx, dentro de un `server` con TLS ya configurado:

```
root /srv/handball-scorer/public;
location = /sw.js      { add_header Cache-Control "no-cache"; }
location = /index.html { add_header Cache-Control "no-cache"; }
location /             { try_files $uri $uri/ =404; }
```

Docker, detrás de tu proxy con TLS:

```
docker run -d --name anotador -p 8080:80 \
  -v /srv/handball-scorer/public:/usr/share/nginx/html:ro nginx:alpine
```

GitHub Pages: el repositorio incluye un flujo que publica `public/` en cada push a `main`. Si haces un fork, actívalo en Settings, Pages, eligiendo GitHub Actions como origen.

Los datos van ligados a la dirección: lo anotado en una dirección no aparece en otra. Para moverlos, exporta la copia en la primera e impórtala en la segunda.
