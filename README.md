# Pendientes del lab

Dashboard estático (un solo `index.html`) para que la asesora vea qué pendientes
hay con cada alumno, y los marque como resueltos con un clic. Los alumnos
registran sus pendientes por un Google Form compartido; no crean su propio Sheet.

## Arquitectura

- **Un Form único** (preguntas: `Alumno`, `Pendiente`, `Contexto`, `Fecha límite`,
  `Notas`) que cualquier alumno llena cada vez que tiene un pendiente nuevo.
- Sus respuestas caen en **un Sheet único**, con una columna extra `Estado`
  agregada a mano (vacía = Pendiente).
- Una pestaña `AlumnosOcultos` (columna `Alumno`) para ocultar de la vista a
  alguien sin borrar su historial.
- Un **Apps Script Web App** que el HTML llama para marcar un pendiente como
  resuelto, u ocultar/mostrar un alumno — es la única escritura del sistema.
  Todo lo demás es lectura de CSV publicado.

Trade-off aceptado: el Apps Script se despliega con acceso "Cualquier usuario"
(sin login), así que el link queda visible en el código de la página — en
teoría cualquiera que lo encuentre podría marcar algo como resuelto. Se aceptó
este riesgo a cambio de que la asesora no tenga que abrir el Sheet directamente.

## Armar el Form (una sola vez)

Preguntas, en este orden:

| # | Pregunta | Tipo | Obligatoria |
|---|----------|------|:---:|
| 1 | `Alumno` | Respuesta corta | Sí |
| 2 | `Pendiente` | Selección múltiple: `Tesis`, `Artículo`, `Otro` (+ opción "Otro") | Sí |
| 3 | `Contexto` | Párrafo | Sí |
| 4 | `Fecha límite` | **Fecha** (tipo nativo, no texto) | No |
| 5 | `Notas` | Párrafo | No |

En Configuración: desmarcar "Limitar a 1 respuesta" (cada alumno lo llena
muchas veces). No pedir correo.

En Respuestas → ícono de Sheets → crear hoja nueva. Ahí:
1. Agregar a mano la columna `Estado` (vacía).
2. Agregar una pestaña nueva `AlumnosOcultos` con columna `Alumno` (vacía).
3. Publicar como CSV (Archivo → Compartir → Publicar en la web) **la hoja de
   respuestas** y **la pestaña `AlumnosOcultos`** por separado — dos links CSV.
4. Extensiones → Apps Script → pegar el código de `apps_script_marcar_resuelto.gs`
   (fuera de este repo, ver abajo) → Implementar → Aplicación web → Ejecutar
   como "Yo", acceso "Cualquier usuario" → copiar la URL `/exec`.
5. Poner los 3 links en `SHEET_URL`, `OCULTOS_URL` y `APPSCRIPT_URL` al inicio
   del `<script>` de `index.html`.

El script `.gs` vive fuera de este repo (no se sube a GitHub porque no aporta
nada público); pídeselo a quien armó el proyecto si hace falta reinstalarlo.

## Uso

- Selector con 3 vistas: **Activos** (default), **Todos**, **Solo resueltos**.
- Chips por alumno (derivados de los datos, no hay que registrar a nadie
  aparte): clic filtra, la "✕" lo oculta (con confirmación).
- Orden: primero vencidos/más próximos por `Fecha límite`, luego los sin fecha
  límite por `Fecha` (registro) más antigua.
- Marca ⚠ los pendientes con 14+ días abiertos sin resolver.
- Botón **"Marcar resuelto"** por fila activa — escribe `Resuelto` en el Sheet
  vía el Apps Script y recarga.
- `?alumno=<nombre>` en la URL abre directo filtrado a un alumno.

**Formato de fecha:** el alumno escribe la fecha en `Fecha límite` con el
selector nativo del Form (no texto libre), así que siempre es una fecha real.
El Sheet puede tener locale `dd/mm/aaaa` o `mm/dd/aaaa` según cómo esté
configurado; el HTML infiere el orden solo por columna (si algún valor pasa de
12, ese slot es el día).

## Publicar en GitHub Pages

Settings → Pages → Deploy from branch → `main` / `/ (root)`. La URL pública
sirve `index.html` directo.

## Autocheck

`index.html?selftest=1` corre verificaciones del parser de CSV/fechas, el
orden de urgencia y la normalización de nombres (no requiere red).
