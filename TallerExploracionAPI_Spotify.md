# Taller de exploración de APIs: Spotify Web API

**ISIS3710 Programación con tecnologías web, Ciclo 2**
**Integrantes:** _(completar)_

URL base de la API: `https://api.spotify.com/v1`
Documentación: https://developer.spotify.com/documentation/web-api

Todas las operaciones requieren un token OAuth 2.0 en el encabezado `Authorization: Bearer <token>`. Las respuestas de error comunes a todas las operaciones son `401 Unauthorized` (token inválido o expirado), `403 Forbidden` (falta un *scope* o el usuario no tiene permiso) y `429 Too Many Requests` (se superó el límite de peticiones).

---

## 1. Recursos más relevantes del API

| Recurso | URI base | Descripción |
|---|---|---|
| **Album** | `/albums/{id}` | Álbum con sus canciones, artistas, fecha de lanzamiento y portadas. |
| **Artist** | `/artists/{id}` | Artista, con sus álbumes (`/artists/{id}/albums`) y canciones top (`/artists/{id}/top-tracks`). |
| **Track** | `/tracks/{id}` | Canción individual. |
| **Playlist** | `/playlists/{id}` | Lista de reproducción creada por un usuario, con sus ítems (`/playlists/{id}/items`). |
| **User** | `/me`, `/users/{user_id}` | Perfil del usuario autenticado o de otro usuario. |
| **Library** | `/me/library`, `/me/tracks`, `/me/albums` | Elementos guardados por el usuario (“Tu biblioteca”). |
| **Player** | `/me/player` | Estado y control de la reproducción (play, pause, next, cola). |
| **Search** | `/search` | Búsqueda de álbumes, artistas, canciones, playlists, shows, etc. |
| **Show / Episode** | `/shows/{id}`, `/episodes/{id}` | Podcasts y sus episodios. |
| **Audiobook / Chapter** | `/audiobooks/{id}`, `/chapters/{id}` | Audiolibros y sus capítulos. |
| **Category** | `/browse/categories/{id}` | Categorías de navegación (p. ej. “Pop”, “Workout”). |

Para los puntos 2 a 4 se escogieron los recursos **Playlist** y **Track**.

---

## 2 y 3. Operaciones, verbo HTTP y código de estado

### Recurso Playlist

| Operación | Verbo | URI | Código de éxito | Cuerpo de respuesta |
|---|---|---|---|---|
| Crear una playlist para el usuario actual | `POST` | `/me/playlists` | `201 Created` | Objeto playlist creado |
| Obtener una playlist | `GET` | `/playlists/{playlist_id}` | `200 OK` | Objeto playlist |
| Cambiar los detalles (nombre, descripción, pública/privada, colaborativa) | `PUT` | `/playlists/{playlist_id}` | `200 OK` | Vacío |
| Obtener las playlists del usuario actual | `GET` | `/me/playlists` | `200 OK` | Página (*paging object*) de playlists |
| Obtener las playlists de otro usuario | `GET` | `/users/{user_id}/playlists` | `200 OK` | Página de playlists |
| Obtener los ítems de una playlist | `GET` | `/playlists/{playlist_id}/items` | `200 OK` | Página de ítems |
| Agregar ítems (canciones/episodios) | `POST` | `/playlists/{playlist_id}/items` | `201 Created` | `{ "snapshot_id": "..." }` |
| Reordenar o reemplazar ítems | `PUT` | `/playlists/{playlist_id}/items` | `200 OK` | `{ "snapshot_id": "..." }` |
| Eliminar ítems | `DELETE` | `/playlists/{playlist_id}/items` | `200 OK` | `{ "snapshot_id": "..." }` |
| Obtener la imagen de portada | `GET` | `/playlists/{playlist_id}/images` | `200 OK` | Arreglo de imágenes |
| Subir una imagen de portada personalizada | `PUT` | `/playlists/{playlist_id}/images` | `202 Accepted` | Vacío |

> Nota: las versiones anteriores de las operaciones sobre ítems usaban `/playlists/{id}/tracks`; Spotify las marcó como *deprecated* y las reemplazó por `/playlists/{id}/items`.

No existe una operación `DELETE /playlists/{id}`: para “borrar” una playlist el usuario deja de seguirla con `DELETE /playlists/{playlist_id}/followers` (`200 OK`).

### Recurso Track

| Operación | Verbo | URI | Código de éxito | Cuerpo de respuesta |
|---|---|---|---|---|
| Obtener una canción | `GET` | `/tracks/{id}` | `200 OK` | Objeto track |
| Obtener varias canciones | `GET` | `/tracks?ids={id1},{id2},...` | `200 OK` | `{ "tracks": [ ... ] }` |
| Obtener las canciones guardadas del usuario | `GET` | `/me/tracks` | `200 OK` | Página de *saved tracks* |
| Guardar canciones en la biblioteca | `PUT` | `/me/tracks?ids=...` | `200 OK` | Vacío |
| Eliminar canciones de la biblioteca | `DELETE` | `/me/tracks?ids=...` | `200 OK` | Vacío |
| Verificar si unas canciones están guardadas | `GET` | `/me/tracks/contains?ids=...` | `200 OK` | Arreglo de booleanos, p. ej. `[true, false]` |
| Obtener características de audio *(deprecated para apps nuevas desde nov. 2024)* | `GET` | `/audio-features/{id}` | `200 OK` | Objeto audio-features |

> Spotify también agregó endpoints genéricos de biblioteca que reciben URIs de cualquier tipo: `PUT /me/library?uris=...` (guardar, `200 OK`), su equivalente `DELETE` y `GET /me/library/contains`.

---

## 4. Representación de los recursos

Ambos recursos se representan en **JSON**.

### Playlist (resumido)

```json
{
  "id": "3cEYpjA9oz9GiPac4AsH4n",
  "type": "playlist",
  "uri": "spotify:playlist:3cEYpjA9oz9GiPac4AsH4n",
  "href": "https://api.spotify.com/v1/playlists/3cEYpjA9oz9GiPac4AsH4n",
  "external_urls": { "spotify": "https://open.spotify.com/playlist/3cEYpjA9oz9GiPac4AsH4n" },
  "name": "Para estudiar",
  "description": "Música tranquila",
  "public": true,
  "collaborative": false,
  "snapshot_id": "MTYsZjM2...",
  "images": [ { "url": "https://i.scdn.co/image/...", "height": 640, "width": 640 } ],
  "owner": {
    "id": "juan", "type": "user", "display_name": "Juan",
    "href": "https://api.spotify.com/v1/users/juan", "uri": "spotify:user:juan"
  },
  "items": {
    "href": "https://api.spotify.com/v1/playlists/3cEYpjA9oz9GiPac4AsH4n/items?offset=0&limit=100",
    "limit": 100, "offset": 0, "total": 1, "next": null, "previous": null,
    "items": [
      {
        "added_at": "2026-10-16T15:00:00Z",
        "added_by": { "id": "juan", "type": "user" },
        "is_local": false,
        "item": { "id": "11dFghVXANMlKmJXsNCbNl", "type": "track", "name": "Cut To The Feeling", "...": "..." }
      }
    ]
  }
}
```

Características: atributos propios (`name`, `description`, `public`, `collaborative`), un `snapshot_id` que identifica la versión de la playlist, el `owner` embebido como objeto *user simplificado*, y los ítems como un **objeto de paginación** (`limit`, `offset`, `total`, `next`, `previous`). Cada ítem envuelve la canción con metadatos de la relación (`added_at`, `added_by`).

### Track (resumido)

```json
{
  "id": "11dFghVXANMlKmJXsNCbNl",
  "type": "track",
  "uri": "spotify:track:11dFghVXANMlKmJXsNCbNl",
  "href": "https://api.spotify.com/v1/tracks/11dFghVXANMlKmJXsNCbNl",
  "external_urls": { "spotify": "https://open.spotify.com/track/11dFghVXANMlKmJXsNCbNl" },
  "name": "Cut To The Feeling",
  "duration_ms": 207959,
  "explicit": false,
  "disc_number": 1,
  "track_number": 1,
  "is_playable": true,
  "is_local": false,
  "external_ids": { "isrc": "USUM71703861" },
  "album": {
    "id": "0tGPJ0bkWOUmH7MEOR77qc", "type": "album", "name": "Cut To The Feeling",
    "album_type": "single", "release_date": "2017-05-26",
    "href": "https://api.spotify.com/v1/albums/0tGPJ0bkWOUmH7MEOR77qc"
  },
  "artists": [
    { "id": "6sFIWsNpZYqfjUpaCgueju", "type": "artist", "name": "Carly Rae Jepsen",
      "href": "https://api.spotify.com/v1/artists/6sFIWsNpZYqfjUpaCgueju" }
  ]
}
```

Características: atributos de la canción (`name`, `duration_ms`, `explicit`, `track_number`), identificadores externos (ISRC) y los recursos relacionados (`album`, `artists`) embebidos en versión **simplificada**, cada uno con su propio `id`, `uri` y `href` para consultarlo completo.

---

## 5. ¿La identificación de los recursos es clara y consistente?

**Sí, en general.** Razones:

- Se usan **sustantivos en plural** para las colecciones (`/albums`, `/artists`, `/tracks`, `/playlists`) y un identificador para el elemento (`/tracks/{id}`), siguiendo la convención REST.
- Todo recurso tiene el mismo esquema de identificación: un **ID de Spotify** (base-62), una **URI** (`spotify:track:{id}`) y un **href** (URL completa del API). Esto es igual para canciones, álbumes, artistas, playlists y usuarios.
- Las sub-colecciones se expresan con anidamiento jerárquico: `/playlists/{id}/items`, `/albums/{id}/tracks`, `/artists/{id}/albums`.
- El alias `/me` identifica al usuario autenticado de forma clara (`/me/playlists`, `/me/tracks`).

**Inconsistencias encontradas:**

- La biblioteca del usuario está dispersa: hay endpoints por tipo (`/me/tracks`, `/me/albums`, `/me/shows`) y además el nuevo endpoint genérico `/me/library`, que hacen lo mismo con diferente forma de identificar los recursos (`ids` vs. `uris`).
- Hay operaciones que no son recursos sino acciones: `/me/player/play`, `/me/player/pause`, `/me/player/next`. Son verbos en la URI, lo cual no es REST puro.
- Los ítems de una playlist cambiaron de `/tracks` a `/items` (porque también pueden ser episodios); el cambio mejora la semántica, pero conviven dos nombres para lo mismo mientras dura la deprecación.
- Para operaciones en lote los IDs van en la *query string* (`?ids=a,b,c`), no en la ruta.

---

## 6. ¿El verbo usado en las operaciones es el adecuado?

**Mayormente sí:**

- `GET` se usa solo para consultar y no modifica estado (es seguro e idempotente).
- `POST /me/playlists` crea un recurso nuevo cuyo ID asigna el servidor: es el uso correcto de `POST`.
- `PUT /playlists/{id}` cambia los detalles y `PUT /me/tracks` guarda canciones: son operaciones idempotentes (repetirlas deja el mismo estado), por lo que `PUT` es apropiado.
- `DELETE /me/tracks` y `DELETE /playlists/{id}/items` eliminan elementos, lo que corresponde a `DELETE`.

**Puntos discutibles:**

- `PUT /playlists/{id}` solo modifica algunos campos (los que se envíen), lo cual es semánticamente una **actualización parcial**; el verbo más preciso sería `PATCH`.
- `PUT /playlists/{id}/items` hace dos cosas distintas según el cuerpo: **reemplazar** todos los ítems (uso correcto de `PUT`) o **reordenar** un rango (`range_start`, `insert_before`), que no es idempotente ni un reemplazo completo; sería más adecuado `PATCH` o `POST` a un recurso de acción.
- `DELETE /playlists/{id}/items` lleva un **cuerpo JSON**; la especificación HTTP no le da semántica al cuerpo de un `DELETE`, y algunos clientes/proxies lo descartan.
- `POST /playlists/{id}/items` agrega ítems y no es idempotente (si se repite, la canción queda duplicada), lo cual es consistente con `POST`.

---

## 7. ¿El código de estado retornado es el adecuado?

**En su mayoría sí:**

- `200 OK` en las consultas `GET` es correcto.
- `201 Created` al crear una playlist es correcto, y se devuelve el recurso creado.
- `202 Accepted` al subir la imagen de portada es correcto, porque el procesamiento de la imagen es asíncrono.
- Los errores usan códigos adecuados: `401` (no autenticado), `403` (sin permiso/scope), `404` (no existe), `429` (límite de peticiones, con el encabezado `Retry-After`).

**Puntos discutibles:**

- `POST /playlists/{id}/items` responde `201 Created`, pero no crea un recurso nuevo con su propia URI (solo modifica la playlist y devuelve un `snapshot_id`); `200 OK` sería más preciso.
- `PUT /playlists/{id}`, `PUT /me/tracks` y `DELETE /me/tracks` responden `200 OK` **con cuerpo vacío**; para respuestas sin contenido el código apropiado es `204 No Content`.
- `POST /me/playlists` responde `201` pero la documentación no indica que devuelva el encabezado `Location` con la URI del nuevo recurso, que es lo esperado junto con `201` (aunque el `href` viene en el cuerpo).

---

## 8. ¿La representación de los recursos es la adecuada?

**Sí.** Razones:

- **JSON** es un formato ligero, legible y estándar en la web.
- Es **autodescriptiva**: cada objeto incluye `type`, y los recursos relacionados vienen con su `href`, lo que permite navegar el API siguiendo enlaces (un acercamiento a HATEOAS).
- Usa **objetos simplificados** para las relaciones (el `album` dentro de un track no trae todas sus canciones), evitando respuestas enormes y referencias circulares.
- Las colecciones grandes usan un **objeto de paginación** uniforme (`limit`, `offset`, `total`, `next`, `previous`) en todo el API.
- El `snapshot_id` de la playlist permite manejar concurrencia (editar contra una versión conocida).

**Aspectos mejorables:**

- Hay muchos campos *deprecated* que siguen apareciendo (`popularity`, `preview_url`, `available_markets`, y en playlists `tracks` en vez de `items`, `track` en vez de `item`), lo que genera ruido y confusión.
- Algunas respuestas son **arreglos sin envoltorio** (`GET /me/tracks/contains` → `[true, false]`), inconsistente con otras que envuelven en un objeto (`{ "tracks": [...] }`).
- El objeto playlist anida un objeto de paginación (`items`) dentro del recurso, mezclando la representación del recurso con la de su sub-colección.

---

## 9. Dos operaciones que relacionan dos recursos

1. **Agregar canciones a una playlist** (relación *Playlist – Track*):
   ```
   POST /playlists/{playlist_id}/items
   Content-Type: application/json

   { "uris": ["spotify:track:4iV5W9uYEdYUVa79Axb7Rh", "spotify:track:1301WleyT98MSxVHPZCA6M"], "position": 0 }
   ```
   Respuesta `201` → `{ "snapshot_id": "..." }`

2. **Obtener las canciones de un álbum** (relación *Album – Track*):
   ```
   GET /albums/{album_id}/tracks?limit=20&offset=0
   ```
   Respuesta `200` → página de objetos track simplificados.

Otras opciones válidas: `GET /artists/{id}/albums` (Artist – Album), `GET /users/{user_id}/playlists` (User – Playlist), `DELETE /playlists/{playlist_id}/items` (Playlist – Track).

---

## 10. ¿Cómo se identifican los recursos en esas operaciones?

- **Agregar canciones a una playlist:** el recurso **principal (playlist)** se identifica con su **ID en la ruta** (`/playlists/{playlist_id}`), y el sub-recurso `items` indica la colección que se modifica. Los recursos **relacionados (canciones)** se identifican con sus **URIs de Spotify** (`spotify:track:{id}`) enviadas en el **cuerpo JSON** (o en el parámetro `uris` de la query). Se usan URIs en vez de IDs simples porque una playlist puede contener tanto canciones como episodios de podcast, y la URI incluye el tipo.

- **Obtener las canciones de un álbum:** la relación se expresa **jerárquicamente en la ruta**: `/albums/{album_id}` identifica el álbum y `/tracks` la colección de canciones que le pertenecen. Las canciones no se identifican en la petición; vienen en la respuesta, cada una con su `id`, `uri` y `href` (`/tracks/{id}`) para consultarla de forma independiente.

En resumen, el **recurso padre** siempre va en la ruta con su ID y la relación como sub-colección (`/padre/{id}/hijos`); los **recursos relacionados** se referencian con su ID o URI en el cuerpo/query cuando se van a asociar, o se devuelven con su `href` cuando se consultan.
