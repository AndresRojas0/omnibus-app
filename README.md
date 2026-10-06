# Ómnibus parejos · Montevideo

Exploración interactiva sobre el Sistema de Transporte Metropolitano (STM) de Montevideo que responde una pregunta: **¿pausar los ómnibus en las paradas puede reducir el agrupamiento (*bunching*) y la espera de los pasajeros, sin agregar ni un solo coche?** Combina datos medidos de subidas con resultados de simulación de estrategias de control, y lo cuenta con mapas, diagramas y animaciones — todo en HTML estático, sin frameworks ni build.

**Sitio en producción:** [app-omnibus.vercel.app](https://app-omnibus.vercel.app)

## Quick path

1. Cloná el repo: `git clone https://github.com/AndresRojas0/omnibus-app.git`
2. Servilo estático desde la raíz: `python3 -m http.server 8000`
3. Abrí `http://localhost:8000` — no uses `file://`, las páginas hacen `fetch()` de los JSON.

## Las cinco páginas

| Página | Qué muestra | Datos que usa |
|---|---|---|
| `index.html` | Landing con hero animado (canvas), tabla de impacto por línea y reproductor embebido | `hero.json`, `sim/impacto.json`, `anim/index.json` |
| `mapa.html` | Replay del día completo de la red: coches en la calle, carga a bordo, línea de tiempo y diagrama de Marey | `red.json`, `coches/dias.json`, `coches/<fecha>.json`, `lineas.json` |
| `linea.html` | Análisis por parada de una línea: subidas medidas vs. pasajeros estimados, carga por tramo | `lineas.json`, `linea_<id>.json` |
| `simulador.html` | Reporte de experimentos de control (corridas × seeds) con diagramas Marey pre-renderizados | `sim/index.json`, `sim/impacto.json`, `sim/<id>.json` |
| `animacion.html` | Dos mapas sincronizados comparando `sin_control` vs. `decisor_final` en una corrida simulada | `anim/index.json`, `anim/<id>.json`, `sim/impacto.json` |

## Arquitectura de datos

Todo es JSON estático servido tal cual; las páginas lo leen con `fetch()` y lo dibujan con MapLibre GL (vendido localmente en `maplibre-gl.js`/`.css`).

| Archivo | Contenido |
|---|---|
| `data/red.json` | La red completa: por cada variante, `l` (línea), `d` (destino), `o` (tiempos programados en segundos por parada), `x` (distancia acumulada en metros por parada) y `g` (trazado GeoJSON) |
| `data/coches/<fecha>.json` | Los coches observados de cada día: por viaje, `[vid, anclas, carga, subidas]`. El `vid` es la clave de `red.json`; las anclas son pares `[índice de parada, hora en segundos]`; la carga va por parada | 
| `data/lineas.json` | Catálogo de líneas con pasajeros/mes |
| `data/linea_<id>.json` | Por línea: variantes, paradas y matrices por franja horaria |
| `data/anim/` | Simulaciones por variante y seed: matrices de espera por paso de tiempo para cada estrategia |
| `data/sim/` | Índices de experimentos, impacto a nivel red y corridas del simulador |

Días con datos medidos: **2026-08-04, 2026-08-08 y 2026-08-09** (hábil, sábado y domingo). Los tags `MEDIDO` y `EST.` en la interfaz distinguen lo observado de lo estimado/simulado.

## Estructura del repo

```
.
├── index.html · mapa.html · linea.html · simulador.html · animacion.html
├── maplibre-gl.js / maplibre-gl.css     # MapLibre GL vendido localmente
├── data/
│   ├── red.json · hero.json · lineas.json
│   ├── linea_<id>.json                  # ~140 líneas
│   ├── coches/                          # posiciones y carga por día
│   ├── anim/                            # simulaciones (matrices de espera)
│   ├── sim/                             # experimentos del simulador
│   └── download_lineas_json.py          # descarga y actualiza los JSON
└── .venv/                               # opcional, solo para los scripts (ignorado)
```

## Deploy

El sitio se despliega automáticamente en **Vercel** con cada push a `main`. No hay build: Vercel sirve los archivos estáticos y los JSON con compresión Brotli.

## Regenerar datos

Los scripts de `data/` son Python 3 solo con stdlib (no requieren instalar nada): descargan los JSON del sitio de origen y los guardan con reintentos y escritura atómica.

```bash
python3 data/download_lineas_json.py
```

## Créditos

- Datos: [STM — Sistema de Transporte Metropolitano](https://www.impo.com.uy/bases/transporte-metropolitano) (Montevideo, Canelones y San José).
- Basemap: [CARTO Dark Matter](https://carto.com/attributions) sobre datos © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors.
- Autor: [Andrés Rojas](https://github.com/AndresRojas0).
