# Needle — Music Player

Reproductor musical web con estética editorial y una cola implementada mediante una **lista doblemente enlazada escrita a mano en TypeScript**. La interfaz ofrece controles de reproducción, altas en cualquier posición, eliminación, reordenación, búsqueda, cola persistente, descubrimiento de pistas y carga de archivos de audio locales.

## Requisitos

- Node.js 20 o superior.
- pnpm 9 o superior (recomendado). También puedes usar npm con workspaces.

## Desarrollo local

```bash
pnpm install
Copy-Item .env.example server/.env
Copy-Item .env.example client/.env.local
pnpm dev
```

Abre <http://localhost:5173>. La API arranca en <http://localhost:4000>. Los dos archivos de entorno se pueden rellenar después. Las pistas de muestra permiten probar navegación de nodos; puedes cargar tus propios audios desde la interfaz.

Para una configuración de PowerShell con el contenido del entorno principal:

```powershell
Copy-Item .env.example server/.env
Copy-Item .env.example client/.env.local
```

Edita `client/.env.local` y deja `VITE_API_URL=http://localhost:4000`. Edita `server/.env` para `PORT=4000`, `CLIENT_ORIGIN=http://localhost:5173` y, opcionalmente, las credenciales de Spotify.

## Estructura

```text
client/
  src/
    data/tracks.ts                 pistas de muestra
    structures/DoublyLinkedList.ts Node y lista doble genérica
    App.tsx                        interfaz, cola y reproducción
    styles.css                     sistema visual responsive
server/
  src/index.ts                     Express, CORS y endpoints Spotify
.env.example
```

## Lista doblemente enlazada

`Node<T>` contiene `value`, `next` y `prev`. `DoublyLinkedList<T>` mantiene `head`, `tail`, `current` y `size`. Sus métodos principales son `addFirst`, `addLast`, `insertAt`, `removeAt`, `removeById`, `moveNext`, `movePrevious`, `setCurrentById` y `clear`. `toArray()` solo produce una instantánea para que React pinte la vista: no se usa para navegar ni mutar la cola. La UI también representa las relaciones `prev` y `next` de cada nodo.

## Spotify y audio

El backend expone endpoints para el estado y login OAuth, búsqueda de catálogo y reproducción por dispositivo. El inicio de sesión usa Authorization Code + PKCE con el Client ID, sin Client Secret. El token de acceso vive en memoria del navegador y expira; refresh token y sesión persistida deben añadirse antes de un despliegue multiusuario o con más de una instancia del backend.

**Nota de API:** el Web Playback SDK requiere una aplicación registrada, Premium y acceso de Spotify al scope `streaming`. La app carga el SDK, registra un dispositivo en el navegador y envía las pistas seleccionadas a ese dispositivo. Cuando Spotify entrega una URL de preview, la app conserva el fallback; también admite audio local. Spotify puede restringir Web Playback SDK, scopes o streaming según elegibilidad y acceso de la aplicación; Premium por sí solo no garantiza acceso API. Las pistas de muestra son metadatos visuales.

## Build y producción

```bash
pnpm build
pnpm start
```

El build estático del cliente queda en `client/dist`; el servidor compilado queda en `server/dist`. Configura `CLIENT_ORIGIN` con la URL pública exacta del frontend. `VITE_API_URL` apunta al backend público y se incorpora durante el build del frontend. Para varios dominios permitidos, separa los orígenes por comas en `CLIENT_ORIGIN`.

## GitHub

```bash
git init
git add .
git commit -m "Build Needle music player"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/needle-music-player.git
git push -u origin main
```

## Despliegue sugerido

1. **Backend en Render:** crea un Web Service desde el repositorio. Build command: `corepack enable && pnpm install --frozen-lockfile && pnpm --filter @needle/server build`. Start command: `pnpm --filter @needle/server start`. Root directory déjalo vacío (raíz del monorepo). Añade `PORT` si la plataforma lo exige, `CLIENT_ORIGIN=https://tu-app.vercel.app`, `SPOTIFY_CLIENT_ID` y `SPOTIFY_REDIRECT_URI` en las variables del servicio. Activa health check en `/api/health`.
2. **Frontend en Vercel:** importa el mismo repositorio. Framework: Vite. Root directory: `client`; build: `pnpm build`; output: `dist`. Variable de build: `VITE_API_URL=https://tu-api.onrender.com`. Si Vercel no ve el workspace en la raíz seleccionada, usa la raíz del monorepo y ajusta el comando a `pnpm --filter @needle/client build`, salida `client/dist`.
3. Actualiza `CLIENT_ORIGIN` en Render con la URL de producción (y las URLs de preview que necesites) y vuelve a desplegar la API.
4. En Spotify Developer Dashboard registra exactamente `https://tu-api.onrender.com/api/spotify/callback` como Redirect URI. Define esa misma URL en `SPOTIFY_REDIRECT_URI`; en local ya usamos `http://127.0.0.1:4000/api/spotify/callback`. El flujo PKCE de este proyecto no requiere Client Secret; no lo añadas a `VITE_*`, Git, ni al bundle frontend.
5. Verifica `https://tu-api.onrender.com/api/health`, abre el frontend, prueba insertar/eliminar/avanzar/retroceder y confirma CORS en las herramientas del navegador.

Netlify sirve para el frontend también; Railway o AWS App Runner/Elastic Beanstalk son alternativas para la API. Antes de usar OAuth en varias instancias, configura almacenamiento compartido para los estados PKCE y sesiones.
