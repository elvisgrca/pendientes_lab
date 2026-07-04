# Pendientes del lab

Herramienta estática (un solo `index.html`, sin backend) para que la asesora vea,
por alumno, qué pendientes hay conmigo. Lee directo de un CSV publicado de Google
Sheets; no escribe nada de vuelta.

## Por alumno: publicar el Sheet como CSV

En el Google Sheet del alumno: **Archivo → Compartir → Publicar en la web** →
elegir la hoja → formato **CSV** → Publicar. Copiar el link que te da (termina en
`output=csv`).

Encabezados exigidos en la fila 1, en este orden y con este texto exacto:

```
Fecha | Pendiente | Contexto | Fecha límite | Estado | Notas
```

`Estado` debe ser uno de: `Pendiente`, `En progreso`, `Resuelto`. Cada ronda de
corrección es una fila nueva; nunca se borra ni se reutiliza una fila.

**Formato de fecha:** el alumno escribe la fecha como sea más natural para él,
no hace falta pedirle que cambie el formato de la celda. Cada Sheet puede tener
un locale distinto (`dd/mm/aaaa` o `mm/dd/aaaa`), pero el HTML infiere el orden
solo: si en esa columna algún día pasa de 12, ese slot solo puede ser el día
(no el mes), y de ahí deduce el orden para toda la columna. Solo en el caso
ambiguo (ningún valor de la columna pasa de 12, ej. todas las fechas caen en la
primera mitad del mes) asume `dd/mm/aaaa` por default.

## Uso

Abrir `index.html` (o la versión en GitHub Pages), pegar el nombre del alumno y
el link CSV, "Agregar y cargar". Queda guardado en el navegador (localStorage) de
quien lo abre, no hay servidor ni login.

- Por defecto se ocultan los `Resuelto`; hay un check para verlos todos.
- Orden: primero vencidos/más próximos por `Fecha límite`, luego los sin fecha
  límite por `Fecha` más antigua.
- Marca ⚠ los pendientes con 14+ días abiertos sin resolver.
- "Ver todos combinados" junta los pendientes activos de todos los alumnos
  guardados.
- `?sheet=<url-csv>&nombre=<nombre>` en la URL abre directo ese alumno (para
  compartir un link ya cargado), sin depender de la memoria local.

## Ver la lista de alumnos en cualquier dispositivo

`localStorage` es por navegador/dispositivo: si agregas un alumno en la laptop,
no aparece solo en el celular. Dos formas de resolverlo, sin backend:

- **Exportar / Importar lista**: copia un texto en un dispositivo y pégalo en
  el otro (una sola vez). Útil para un traspaso puntual.
- **Sheet índice (recomendado si son varios dispositivos)**: crea un Google
  Sheet aparte con columnas `Nombre` y `Link` (un renglón por alumno, con su
  link CSV ya publicado), publícalo también como CSV, y pon esa URL en la
  constante `REGISTRY_URL` al inicio del `<script>` de `index.html`. Con eso
  cualquier dispositivo que abra la página carga sola la lista completa —
  vos administras el índice editando ese Sheet directamente, el HTML solo lo
  lee.

## Publicar en GitHub Pages

Settings → Pages → Deploy from branch → `main` / `/ (root)`. La URL pública
sirve `index.html` directo.

## Autocheck

`index.html?selftest=1` corre unas verificaciones del parser de CSV/fechas y
del orden de urgencia (no requiere red).
