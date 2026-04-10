# Radio Expres — Reloaded

Sitio web estático moderno para **Radio Expres**, emisora de Rock & Pop clásico de los 60s, 70s, 80s y 90s, transmitiendo desde **Espinar, Cusco, Perú**.

---

## Stack

| Tecnología | Uso |
|---|---|
| **Vue 3** (CDN) | Reactividad, componentes, estado del player |
| **Tailwind CSS** (CDN) | Estilos utility-first, diseño responsive |
| **Bebas Neue + Inter** | Tipografía (Google Fonts) |
| **Icecast2 API** | Metadata de la canción actual |
| **iTunes Search API** | Portada/artwork del álbum |
| **GitHub Pages** | Hosting estático |

Sin build process. Todo corre directo en el browser.

---

## Estructura

```
radio-expres-reloaded/
├── index.html              ← App completa (Vue 3 + Tailwind, ~1150 líneas)
├── CNAME                   ← radioexpres.pe (GitHub Pages custom domain)
└── assets/
    └── img/
        ├── logo.png         ← Logo guitarra principal
        ├── logo-player.png  ← Fallback portada del player
        ├── favicon.png      ← Fallback favicon (reemplazado por SVG inline)
        └── team/
            ├── jaime.jpg
            ├── jaimesantiago.jpg
            ├── javierportillo.jpg
            ├── luiseduardo.png
            ├── maragamero.jpg
            └── willy.png
```

---

## Secciones del sitio

1. **Navbar** — fija, transparente → negro con blur al hacer scroll, hamburger en mobile
2. **Hero** — full-screen, logo flotante con glow animado, botones "Escuchar en Vivo" y "App Android"
3. **Player** — reproductor de radio en vivo (ver detalles abajo)
4. **Nosotros** — grid de décadas 60s/70s/80s/90s + descripción
5. **Programas** — cards de los 6 DJs con foto, programa, horario y links
6. **Patrocinadores** — slots disponibles para marcas
7. **Publicidad** — CTA para anunciantes
8. **Contacto** — cards de WhatsApp, Email y Facebook
9. **Footer** — logo, links, copyright, "Espinar · Cusco · Perú"

---

## Player — detalle técnico

### Stream
- **URL**: `https://sonic.sistemahost.es:7118/live`
- **Tipo**: Icecast2
- **Metadata endpoint**: `https://sonic.sistemahost.es:7118/status-json.xsl`

### Metadata
- Consulta el endpoint Icecast2 cada **5 segundos**
- Primer intento: fetch directo (con timeout de 3s)
- Fallback: CORS proxy via `api.allorigins.win`
- Recuerda cuál método funcionó para evitar doble intento en cada tick
- Parsea formato `"Artista - Canción"` y los muestra por separado

### Artwork (portada)
- Busca en **iTunes Search API** con `artist + track`
- `limit=1` + `AbortController` de 4s para máxima velocidad
- **Cache en memoria** (`Map`): si la canción ya se buscó, respuesta instantánea
- Escala la imagen de `100x100` → `600x600` para alta resolución
- Mientras carga: animación de **respiración** (pulso de glow rojo en el contenedor)
- Al llegar: transición `fade` suave

### Anti-buffer-bloat (cortes/interferencia)
El browser acumula buffer del stream live indefinidamente, causando glitches después de varios minutos. Solución implementada:
- Monitor cada **15 segundos** revisa `audio.buffered.end - audio.currentTime`
- Si el buffer supera **8 segundos** adelante → `silentReload()`: recarga el src sin mostrar error ni interrumpir la UI

### Visualizer
- 18 barras CSS con `transform: scaleY()` (GPU-accelerated)
- Cada barra tiene duración y delay de animación distintos para efecto orgánico
- Se detienen (estado flat) cuando el player está pausado

---

## Paleta de colores

| Token | Valor | Uso |
|---|---|---|
| `brand.red` | `#e01020` | Acentos principales, botones, live dot |
| `brand.red-light` | `#f52535` | Hover states |
| `brand.red-dark` | `#b50d1a` | Pressed states |
| `bg-[#0a0a0a]` | `#0a0a0a` | Fondo principal |
| `bg-[#0d0d0d]` | `#0d0d0d` | Secciones alternas |
| `zinc-900` | `#18181b` | Cards, navbar scrolled |

---

## Programas / DJs

| Locutor | Programa | Horario |
|---|---|---|
| Jaime Santiago | Desayuno Express | Lun–Vie 6–8 am |
| Mara Gamero Rosas | Ciudad Sónica | Lun–Vie 8–10 am |
| Luis Eduardo | Retromanía | Sáb–Dom 6–8 am |
| Willy Toledo Solís | Retrospectiva | Sáb–Dom 8–10 am |
| Jaime Valencia | De Colección | Sáb–Dom 10 am–12 pm |
| Javier Portillo Zamora | Billboard Forever | Sáb–Dom 12:30–3 pm |

---

## Deploy en GitHub Pages

1. Subir este directorio a un repositorio en GitHub
2. En **Settings → Pages**, seleccionar `main` branch y `/` (root)
3. El `CNAME` apunta a `radioexpres.pe` — configurar el DNS del dominio con un registro CNAME a `<usuario>.github.io`
4. GitHub Pages servirá `index.html` directamente

> No se necesita ningún build step. El sitio es 100% estático.

---

## Origen

Rediseño completo del sitio anterior (`expres-landing-arsha`), que usaba el template Arsha con Bootstrap y el plugin `lunaradio-sincors.js` para el player. Este reloaded reemplaza todo eso con Vue 3 + Tailwind y un player propio con mejor control del stream y la metadata.
