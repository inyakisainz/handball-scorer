# Handball Scorer (Anotador de balonmano)

Aplicación web instalable para anotar partidos de balonmano en directo y analizarlos después. Funciona sin conexión y guarda los datos solo en el dispositivo. No tiene servidor de aplicación ni cuentas de usuario.

## Estructura del repositorio

| Ruta | Para qué sirve |
| --- | --- |
| `public/index.html` | Toda la aplicación (interfaz, lógica y estilos) en un solo archivo |
| `public/sw.js` | Service worker: caché para funcionar sin conexión |
| `public/manifest.webmanifest` | Hace que se pueda instalar como app |
| `public/icon-192.png`, `public/icon-512.png` | Iconos |
| `CHANGELOG.md` | Historial de versiones |

No hay dependencias ni paso de compilación: lo que se despliega es la carpeta `public/` tal cual.

## Probar en tu ordenador

```
cd public
python3 -m http.server 8080
```

Abre `http://localhost:8080`. En `localhost` el navegador permite el service worker sin HTTPS.

## Publicar

Sirve la carpeta `public/` como sitio estático **con HTTPS**. Sin HTTPS no se instala ni funciona sin conexión.

Caddy (certificado automático):

```
anotador.tudominio.com {
    root * /srv/handball-scorer/public
    file_server
    header /sw.js Cache-Control "no-cache"
    header /index.html Cache-Control "no-cache"
}
```

Nginx (dentro de un `server` con TLS ya configurado):

```
root /srv/handball-scorer/public;
location = /sw.js      { add_header Cache-Control "no-cache"; }
location = /index.html { add_header Cache-Control "no-cache"; }
location /             { try_files $uri $uri/ =404; }
```

Docker, detrás de tu proxy con TLS:

```
docker run -d --name anotador -p 8080:80 -v /srv/handball-scorer/public:/usr/share/nginx/html:ro nginx:alpine
```

Puede servirse en una subcarpeta (`https://dominio/anotador/`): todas las rutas son relativas.

## Instalar en la tablet o el móvil

1. Abre la dirección con conexión y recarga una vez. En Datos, "funcionamiento sin conexión" debe decir "listo".
2. Android (Chrome): menú, "Instalar aplicación" o "Añadir a pantalla de inicio".
3. iPad o iPhone (Safari): Compartir, "Añadir a pantalla de inicio".
4. Antes del primer partido real, pon el dispositivo en modo avión y comprueba que abre.

## Uso

Bajo la cabecera hay una barra que dice siempre qué partido está abierto: equipo, rival, competición y temporada. Al tocarla, o al tocar el icono de personas, se abre la gestión.

Arriba de la gestión se elige el **equipo** (un botón por equipo, más "+ Equipo") y la **temporada**. Todo lo que aparece debajo corresponde a esa combinación.

- **Partidos:** botón "Partido nuevo" y la lista agrupada por competición, con el balance de cada grupo. Los filtros de arriba dejan ver solo una competición o solo los amistosos.
- **Competiciones:** una tarjeta por competición con partidos, balance y goles, y otra fija para amistosos y entrenamientos. Desde cada tarjeta se crea un partido en esa competición o se ven sus partidos. El lápiz permite renombrarla, cambiar el formato o borrarla. Debajo se gestionan las temporadas.
- **Plantilla:** nombre del equipo, jugadores y, si hay más de un equipo, borrado del equipo.
- **Rivales:** rivales guardados con dorsales y nombres opcionales, y el balance de enfrentamientos.
- **Datos:** copia completa, importación y CSV de todos los partidos.

### Amistosos y entrenamientos

Al crear un partido se elige la competición. "Amistoso" y "Entrenamiento" no pertenecen a ninguna competición, pero sí a una temporada. También se puede crear una competición nueva desde el propio formulario.

### Registro y análisis

- **Registro:** siempre se empieza por quién (bloque 1). Si es un tiro, se marca zona y cuadrante en el bloque Tiro y se pulsa el resultado. Si es otra acción (pérdida, robo, falta, sanción…), basta un botón del bloque Otra acción, sin zona ni portería. Las acciones de equipo no necesitan jugador. En móvil, un selector alterna entre Tiro y Otra acción; en tablet se ven los dos a la vez.
- **Jugadores en pista:** en el bloque Quién, los que están en pista aparecen arriba y el banquillo debajo, atenuado. Con Cambios se marca quién está en pista (máximo 7) y se confirma con Hecho. Un partido nuevo arranca con un siete inicial (el primer portero y los seis primeros jugadores de la plantilla) que se puede corregir antes de empezar. Una exclusión o una roja saca al jugador de pista automáticamente; cuando vuelve hay que marcarlo en Cambios. Si se anota una acción de alguien que figura en el banquillo, aparece un aviso.
- **Estadísticas** tiene dos niveles, que se eligen arriba: Este partido, y Temporada y competición.
- **Este partido:** Resumen, Jugadores, Tiro, Porteros, Minutos y Evolución.
  - *Jugadores:* tabla ordenable con minutos, goles y tiros, porcentaje, goles por 60 minutos, paradas, asistencias, pérdidas, recuperaciones, exclusiones, +/− y valoración. Al tocar un jugador se abre su ficha: indicadores, tiro por distancia, por fase y por parte, mapa de zonas y la lista de sus acciones. Para los porteros la ficha se centra en las paradas.
  - *Minutos:* tiempo en pista por jugador y las alineaciones usadas con su parcial.
- **Temporada y competición:** Jugadores y Equipo. Arriba se elige equipo, temporada y ámbito: toda la temporada, solo amistosos y entrenamientos, o una competición.
  - *Jugadores:* la misma tabla sumando los partidos elegidos, con partidos jugados, minutos por partido, goles por partido y valoración media. La ficha añade la gráfica de goles por partido y el detalle partido a partido.
  - *Equipo:* evolución de eficacia y portero, partido a partido, comparación con la media y enfrentamientos con el rival.

### Valoración

Una sola fórmula en toda la aplicación. Los pesos están en un único sitio del código (`VW` en `index.html`).

| Acción | Puntos |
| --- | --- |
| Gol | +2 |
| Gol de 7 m | +1 |
| Tiro fallado (parado, fuera, palo o blocado) | −1 |
| 7 m fallado | −2 |
| Asistencia | +1 |
| 7 m provocado | +1 |
| Blocaje | +1 |
| Robo | +1,5 |
| Pérdida | −1,5 |
| Falta en ataque | −1,5 |
| 7 m cometido | −1 |
| Exclusión de 2 minutos | −2 |
| Roja | −4 |
| Parada (portero) | +2 |
| Parada de 7 m (portero) | +3 |
| Gol encajado (portero) | −0,75 |
| Gol encajado de 7 m (portero) | 0 |

- No puntúan las amarillas ni las faltas defensivas.
- Con estos pesos un portero queda en cero con un 27 % de paradas en juego.
- **Val/60′** es la valoración por cada 60 minutos en pista. Solo se muestra si el jugador tiene 5 minutos o más y se han marcado los jugadores en pista.
- En el acumulado, Val/PJ es la valoración total entre los partidos jugados, y Val/60′ la valoración total entre el tiempo total en pista.
- En las tablas, Pérdidas incluye las faltas en ataque y Recuperaciones suma robos y blocajes.

## Datos y copias

- Los datos viven en el almacenamiento del navegador de ese dispositivo (IndexedDB, con copia en localStorage). Borrar los datos del navegador o desinstalar la app los elimina.
- En iOS, Safari puede borrar los datos de un sitio que lleva semanas sin abrirse si no está instalado en la pantalla de inicio.
- **Exporta una copia (JSON) después de cada partido.** Ese archivo se puede importar en otro dispositivo. Importar sustituye todo lo que haya en el dispositivo de destino.
- Las copias y los datos de la versión anterior (un solo equipo) se convierten solos: el equipo, sus rivales y una competición por cada tipo de partido que no fuera amistoso.
- El CSV usa punto y coma y abre directamente en Excel en español.
- Los nombres de jugadores son datos personales. Con menores, informa al club y no publiques las copias.

## Actualizar

1. Modifica `public/index.html` y añade una entrada en `CHANGELOG.md`.
2. Cambia `VERSION` en `public/sw.js` (ahora es `anotador-v8`; la siguiente sería `anotador-v9`).
3. Despliega `public/`. En cada dispositivo, abre la app con conexión y ciérrala y ábrela otra vez: la versión nueva entra en la segunda apertura. Los datos se conservan.

## Límites de esta versión

- Un solo anotador por partido y sin sincronización entre dispositivos.
- Los datos de un partido ya creado (rival, competición, fecha) no se pueden editar; hay que borrarlo y crearlo de nuevo.
- Sin clasificaciones ni calendario de liga: las competiciones solo agrupan tus propios partidos.
- Sin detalle avanzado del tiro (tipo, trayectoria, presión), sin consola remota, sin vídeo y sin informe PDF.
- El tiempo en pista depende de que el anotador marque los cambios: los cambios rápidos ataque-defensa no se registran solos, y el reloj de la exclusión no devuelve al jugador a pista.
- Los eventos guardan el dorsal, no un identificador de jugador: cambiar un dorsal no actualiza partidos anteriores.
- Las tipografías se descargan de Google Fonts la primera vez y quedan en caché. Sin ellas se usa la fuente del sistema.
