# Mendel · Pipeline MX — Dashboard con actualización diaria automática

Este repo publica el dashboard de pipeline en una URL fija (GitHub Pages). Todos los
días, a la hora que elijas, una automatización (GitHub Actions) entra a HubSpot,
trae los datos frescos, y actualiza el sitio solo — vos no tenés que descargar ni
tocar nada, simplemente abrís siempre la misma URL.

Lo único que **no** se actualiza solo es la parte de Diio (`data/diio_insights.json`)
y los objetivos (`data/quotas.json`), porque no son APIs públicas a las que la
automatización pueda llamar por su cuenta. Esos dos archivos los editás vos a mano
cuando haga falta (2 minutos, ver más abajo).

---

## Paso 1 — Crear cuenta y repositorio en GitHub

1. Andá a [github.com/signup](https://github.com/signup) y creá una cuenta (gratis).
2. Una vez adentro, arriba a la derecha tocá el **+** → **New repository**.
3. Nombre sugerido: `mendel-pipeline`. Dejalo **Public** (GitHub Pages gratis requiere
   que el repo sea público, salvo que tengas plan Pro/Team).
4. Creá el repo vacío (sin README, sin .gitignore — subimos todo nosotros).
5. Subí **todos los archivos de esta carpeta** tal cual están, respetando la
   estructura de carpetas (`data/`, `scripts/`, `.github/workflows/`). La forma más
   fácil sin usar la terminal: en la página del repo, "Add file" → "Upload files",
   arrastrás toda la carpeta.

## Paso 2 — Generar el token de HubSpot (Private App)

Este token es lo que le da permiso a la automatización para leer tu HubSpot. **Nunca
lo compartas en un chat ni lo pegues en el código** — va directo en la sección de
Secrets de GitHub (paso 3).

1. En HubSpot: **Configuración** (engranaje) → **Integraciones** → **Apps privadas**.
2. **Crear una app privada**. Ponele nombre "Dashboard Pipeline MX".
3. En la pestaña **Scopes**, activá (búsqueda como CRM):
   - `crm.objects.deals.read`
   - `crm.objects.contacts.read`
   - `crm.objects.owners.read`
4. Guardá y **copiá el token** que te muestra (empieza con `pat-...`). Solo se
   muestra una vez — si lo perdés, tenés que regenerarlo.

## Paso 3 — Guardar el token como Secret en GitHub

1. En tu repo de GitHub: **Settings** → **Secrets and variables** → **Actions**.
2. **New repository secret**.
3. Nombre exacto: `HUBSPOT_TOKEN`
4. Valor: pegá el token `pat-...` que copiaste.
5. **Add secret**.

## Paso 4 — Activar GitHub Pages

1. En el repo: **Settings** → **Pages**.
2. En "Build and deployment" → **Source**, elegí **GitHub Actions** (no "Deploy from
   a branch").
3. Con eso ya está — el workflow (`.github/workflows/update-dashboard.yml`) se
   encarga de publicar.

## Paso 5 — Correrlo por primera vez

1. En el repo: pestaña **Actions**.
2. Vas a ver el workflow "Actualizar dashboard Mendel". Tocá **Run workflow** (botón
   manual) para probarlo ahora mismo sin esperar al horario programado.
3. Esperá 1-2 minutos. Si se pone verde ✅, andá a **Settings → Pages** y ahí vas a
   ver la URL pública, algo como:
   `https://TU-USUARIO.github.io/mendel-pipeline/`
4. Guardá esa URL — es la que vas a abrir todos los días. Siempre va a mostrar los
   datos más recientes.

Si se pone rojo ❌, tocá el workflow para ver el error — lo más común es que el
Secret `HUBSPOT_TOKEN` esté mal escrito o el token no tenga los scopes correctos.

---

## ¿Cuándo corre?

Por defecto: **todos los días a las 07:00 UTC** (≈ 01:00 hora Ciudad de México).
Para cambiar el horario, editá esta línea en
`.github/workflows/update-dashboard.yml`:

```yaml
- cron: "0 7 * * *"
```

El formato es `minuto hora * * *` en **UTC**. Por ejemplo, para que corra a las
7:00 AM hora Ciudad de México (que está UTC-6), usarías `"0 13 * * *"`.

También podés correrlo manualmente cuando quieras desde la pestaña **Actions** →
**Run workflow**.

---

## Mantenimiento manual (lo que NO se actualiza solo)

### Objetivos del trimestre (`data/quotas.json`)

Al cambiar de trimestre, editá este archivo directamente en GitHub (tocá el
archivo → ícono de lápiz → editar → "Commit changes"):

```json
{
  "Agustín Merli": 11500,
  "Manon Fabre": 4600,
  "Rodrigo Salcedo": 4600,
  "Hernando Chávez": 4600
}
```

### Insights de Diio (`data/diio_insights.json`)

Es un diccionario `"id_del_deal": "texto del insight"`. El id del deal es el mismo
que usa HubSpot (lo ves en la URL del deal en HubSpot, o pedíselo a Claude). Ejemplo:

```json
{
  "58986349156": "Diio: última reunión fue muy positiva, contrato en firma..."
}
```

Para conseguir insights nuevos, pedile a Claude que revise Diio para deals
puntuales y te pase el texto — después lo pegás acá.

---

## Estructura del repo

```
mendel-pipeline/
├── index.html                        ← el dashboard (no lo edites salvo que sepas HTML/JS)
├── data/
│   ├── deals.json                    ← generado automáticamente cada día
│   ├── closed.json                   ← generado automáticamente cada día
│   ├── meta.json                     ← generado automáticamente cada día
│   ├── quotas.json                   ← editar a mano cuando cambian los objetivos
│   └── diio_insights.json            ← editar a mano cuando hay insights nuevos
├── scripts/
│   ├── fetch_hubspot.py              ← el script que trae los datos de HubSpot
│   └── requirements.txt
├── .github/workflows/
│   └── update-dashboard.yml          ← la automatización (horario, permisos, deploy)
└── README.md                          ← este archivo
```

## Si necesitás cambiar el equipo (owners) o las etapas del pipeline

Editá las constantes al principio de `scripts/fetch_hubspot.py`:
`OWNER_IDS`, `PIPELINES`, `STAGE_LABELS`, `CANONICAL_ACTIVE_STAGE_IDS`,
`VERBAL_WIN_STAGE_IDS`. Los IDs se consiguen en HubSpot → Configuración → Objetos →
Negocios → Pipelines (o pedíselo a Claude, que ya los tiene identificados de esta
conversación).
