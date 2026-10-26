# GH_DRIVE — Disco en la nube sobre GitHub

Cada **disco** = un **repositorio** de GitHub (~5 GB recomendado). Cada archivo máx. **100 MB**.

> ⚠️ **INFORMACIÓN CONFIDENCIAL:** el token (PAT) da acceso total a tus repos. No lo compartas, no lo subas a GitHub ni lo pegues en capturas. Se guarda en `localStorage` de tu navegador: úsalo solo en tu equipo.

## 1. Crear el token (PAT)

1. GitHub → **Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)**.
2. Marca los scopes:
   - `repo` → leer, subir y borrar archivos, crear repos.
   - `delete_repo` → solo si quieres borrar discos (repos) desde la app.
3. Define una expiración y copia el token (`ghp_...`). Solo se muestra una vez.

> Los tokens *fine-grained* no sirven para crear repos nuevos desde la API; usa *classic*.

## 2. Configurar la app

1. Abre `index.html` → engranaje ⚙ (arriba a la derecha).
2. Rellena:
   - **Token:** `ghp_...`
   - **Username / Org:** tu usuario de GitHub (o la organización).
   - **Active Disk Repo:** nombre del repo por defecto (ej. `disk-alpha`).
3. **SAVE CONFIG**. El pie de la vista interior pasará de ámbar (simulación) a verde (conectado).

Sin token ni usuario la app funciona en **modo simulación** (datos solo en `localStorage`).

## 3. Qué hace cada acción en la API

Base: `https://api.github.com` · Cabeceras: `Authorization: Bearer <TOKEN>` y `Accept: application/vnd.github+json`.

| Acción en la app | Petición |
|---|---|
| New Disk | `POST /user/repos` (u `/orgs/{org}/repos`) con `{name, private:true, auto_init:true}` |
| Delete Disk | `DELETE /repos/{owner}/{repo}` |
| Tamaño del disco | `GET /repos/{owner}/{repo}` → campo `size` (KB) |
| Listar carpeta | `GET /repos/{owner}/{repo}/contents/{path}` |
| Subir archivo | `PUT /repos/{owner}/{repo}/contents/{path}` con `{message, content(base64), sha?}` |
| Nueva carpeta | `PUT .../contents/{carpeta}/.gitkeep` (Git no guarda carpetas vacías) |
| Borrar archivo | `DELETE .../contents/{path}` con `{message, sha}` |
| Borrar carpeta | Borra recursivamente cada archivo con el `DELETE` anterior |

Notas:
- `auto_init:true` crea un commit inicial; sin él la API de contenidos falla en repos vacíos.
- Para sobrescribir un archivo existente hay que enviar su `sha` (la app lo hace sola).
- Repos **privados**: el `download_url` incluye un token temporal; el enlace caduca.

## 4. Límites de GitHub

- 100 MB por archivo (la app avisa antes de subir).
- ~1 GB recomendado / 5 GB tope suave por repo.
- API: 5 000 peticiones/hora con token.
- La API de contenidos sube archivos de hasta ~100 MB en base64; los grandes pueden tardar o fallar.

## 5. Publicarla (opcional)

Subir `index.html` a un repo → **Settings → Pages** → rama `main`, carpeta `/root`. Quedará en `https://<usuario>.github.io/<repo>/`.
Usa un repo aparte de tus discos y no guardes el token en el código: se escribe en el engranaje y vive solo en tu navegador.

## 6. Seguridad

- **Token solo en sesión (por defecto):** se borra al cerrar la pestaña. Solo se guarda en el navegador si activas *Remember token*. El botón *Forget token now* lo borra de inmediato.
- **Borrar discos en modo conectado:** exige escribir `DELETE` para confirmar.
- **Anti doble clic:** los botones se bloquean mientras una operación está en curso (ya no se duplican discos).
- **Nombres validados:** discos solo `a-z 0-9 . _ -`; usuario validado; discos duplicados se limpian al cargar.
- **Anti-XSS:** nombres de archivos/carpetas se escapan antes de mostrarse; enlaces solo `https://` o `data:`.
- **CSP:** la página solo puede conectarse a `api.github.com`. Si algún día se ve sin estilos, revisa la etiqueta `<meta http-equiv="Content-Security-Policy">`.
- Usa un token con expiración corta y revócalo en GitHub si sospechas que se filtró.

## 7. Problemas comunes

| Error | Causa |
|---|---|
| `401 Bad credentials` | Token mal copiado o vencido |
| `403` / `404` al crear o borrar repo | Faltan scopes `repo` / `delete_repo` |
| `422 sha wasn't supplied` | Se intentó sobrescribir sin `sha` |
| `404` al listar | El repo no existe o el nombre/usuario es incorrecto |
