# Brand Logistics — Control Tower

Dashboard operativo de logística que se actualiza solo cada 5 minutos.

## Cómo funciona

1. **GitHub Actions** (el robot de GitHub) descarga el CSV desde `vtex.brandlive.net` cada 5 minutos y lo guarda en este repo, en `data/ops-om-ar.csv`.
2. El **dashboard** (`index.html`) lee ese archivo local y recalcula todo (estados, SLA, quiebres, volúmenes) directo en el navegador.
3. Como el CSV vive en el mismo repo que el dashboard, no hay problema de CORS.

No consume tokens de Claude ni requiere que hagas nada manual una vez configurado.

---

## Estructura de archivos

```
tu-repo/
├── index.html                      ← el dashboard
├── data/
│   └── ops-om-ar.csv               ← lo genera/actualiza GitHub Actions solo
└── .github/
    └── workflows/
        └── update-data.yml         ← el robot que descarga el CSV cada 5 min
```

> **Importante:** el archivo `update-data.yml` tiene que quedar en la ruta `.github/workflows/update-data.yml`. Esa carpeta con el punto adelante es obligatoria para que GitHub reconozca el workflow.

---

## Puesta en marcha (paso a paso)

### 1. Crear el repositorio
- Entrá a github.com → botón **New repository**.
- Ponele un nombre (ej. `control-tower`), dejalo **Public**, y creá el repo.

### 2. Subir los archivos
Subí los tres archivos respetando las rutas:
- `index.html` en la raíz del repo.
- `update-data.yml` dentro de `.github/workflows/`.
- (La carpeta `data/` se crea sola cuando corre el robot por primera vez; no hace falta subirla.)

Para crear la carpeta `.github/workflows/` desde la web de GitHub: **Add file → Create new file**, y en el nombre escribí `.github/workflows/update-data.yml` — GitHub crea las carpetas automáticamente al poner las barras.

### 3. Activar GitHub Pages (para publicar el dashboard)
- En el repo: **Settings → Pages**.
- En **Source**, elegí **Deploy from a branch**.
- En **Branch**, elegí `main` y carpeta `/ (root)`. Guardá.
- En 1-2 minutos te da la URL pública: `https://TU-USUARIO.github.io/control-tower/`

### 4. Correr el robot por primera vez
El workflow corre solo cada 5 minutos, pero la primera vez conviene dispararlo a mano para que baje el CSV ya:
- Andá a la pestaña **Actions** del repo.
- Elegí **Actualizar datos del dashboard** en la lista de la izquierda.
- Botón **Run workflow → Run workflow**.
- En ~30 segundos vas a ver el check verde y aparece `data/ops-om-ar.csv` en el repo.

### 5. Listo
Abrí la URL de GitHub Pages. El dashboard carga con los datos y se va refrescando solo.

---

## Preguntas frecuentes

**¿Cada cuánto se actualiza realmente?**
El workflow está programado cada 5 minutos. GitHub a veces demora unos minutos extra en horarios de mucha carga — es normal, no es un error. El dashboard además se refresca en el navegador cada 5 minutos, así que toma el CSV más nuevo que haya.

**¿Y si el servidor pide contraseña?**
Este setup asume que el link descarga el CSV sin autenticación. Si en algún momento empieza a pedir credenciales, hay que agregarlas como *Secrets* del repo y modificar el `curl` del workflow. Avisá si pasa.

**¿Puedo cambiar la frecuencia?**
Sí, en `update-data.yml`, la línea `cron: "*/5 * * * *"`. Por ejemplo `*/10 * * * *` = cada 10 minutos. (GitHub no garantiza intervalos menores a 5 minutos en el scheduler.)

**El dashboard dice que no puede leer el CSV.**
Casi siempre es que el workflow todavía no corrió por primera vez. Andá a Actions y ejecutalo manualmente (paso 4).

**¿Esto tiene algún costo?**
No. GitHub Actions es gratis para repos públicos, y GitHub Pages también.
