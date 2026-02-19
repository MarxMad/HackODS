# HackODS — Propuestas de solución para ODS 7, 8, 12 y 15

Este documento resume problemas prioritarios de los ODS 7, 8, 12 y 15, propone soluciones con enfoque de MVP para un hackathon y describe cómo programar un prototipo funcional. Está pensado para construir rápido, demostrar impacto y escalar después.

## Cómo usar este README
- Elige 1 proyecto (o el “Hub” con 2–3 módulos).
- Trabaja con una ciudad piloto para reglas/datos locales.
- Implementa los endpoints y pantallas mínimas descritas.
- Mide impacto con las métricas sugeridas para el pitch.

---

## ODS 7 — Energía asequible y no contaminante

### Proyecto: Mapa de techos solares y ROI comunitario
- Problema: Hogares y pymes no saben si la energía solar “les conviene”.
- Solución (MVP): Web que estima potencia, ahorro y payback por dirección/techo.
- Impacto: kWh/año potenciales; tCO2e evitadas; tasa de leads a instalación.

Implementación técnica:
- Frontend: React/Next.js + Leaflet/MapLibre para mapa y selección de techo.
- Backend: API (Express/FastAPI) que calcula irradiancia y ahorro con reglas simples.
- Datos: Irradiancia (NASA POWER), polígonos/edificios (OSM), tarifas locales (CSV seed).

Endpoints ejemplo:
- GET `/api/solar/estimate?lat={lat}&lon={lon}&consumo_kwh_mes={k}`
- Respuesta:
```json
{
  "pv_kw": 2.5,
  "generacion_anual_kwh": 4100,
  "ahorro_anual_moneda": 13500,
  "payback_anios": 3.8,
  "co2_evitable_ton_anio": 1.9
}
```

Pseudocódigo backend:
```python
def estimate(lat, lon, consumo_kwh_mes):
    gsr = nasa_power_irradiance(lat, lon)  # kWh/m2/día
    pv_kw = max(consumo_kwh_mes / 120, 1.5)
    pr = 0.75
    generacion_anual = gsr * 365 * pv_kw * pr
    ahorro = tarifa_media() * min(generacion_anual, consumo_kwh_mes*12)
    capex = pv_kw * costo_kw()
    payback = capex / max(ahorro, 1)
    co2 = generacion_anual * factor_emision_red()
    return {...}
```

Roadmap 24–48h:
- Día 1: Mapas, selector de punto, endpoint de estimación con reglas y CSV de tarifas.
- Día 2: UI de resultados (kWh, $ y payback), export/compartir estimación, métricas de uso.

---

## ODS 8 — Trabajo decente y crecimiento económico

### Proyecto: Match de empleo verde y capacitación express
- Problema: Falta puente entre perfiles de oficio/tech y empleos “verdes”.
- Solución (MVP): Registro de habilidades + catálogo pequeño de vacantes verdes + rutas de upskilling.
- Impacto: matches generados; postulaciones; horas de capacitación iniciadas.

Implementación técnica:
- Frontend: Form de perfil (skills, ubicación), lista de vacantes con filtros y badges verdes.
- Backend: Motor de matching por etiquetas y distancia; recomendaciones de cursos cortos.
- Datos: Seed de 30–50 vacantes (CSV/JSON), catálogo de cursos (CSV/JSON).

Esquema mínimo:
```sql
users(id, name, skills_text, location_point)
jobs(id, title, tags_text, location_point, employer)
courses(id, title, tags_text, provider, url)
```

Endpoint match:
- GET `/api/jobs/match?user_id={id}`
- Regresa top N vacantes y cursos asociados.

Roadmap 24–48h:
- Día 1: CRUD de perfiles, seed de vacantes, endpoint de match por Jaccard/TF-IDF.
- Día 2: Página de recomendaciones, postulación/guardar favorito, tracking de métricas.

---

## ODS 12 — Producción y consumo responsables

### Proyecto: Asistente de separación de residuos (sin hardware)
- Problema: Baja separación por desconocimiento.
- Solución (MVP): Clasifica residuo por palabra clave o imagen y da guía local de disposición.
- Impacto: consultas; tasa de clasificación correcta; toneladas desviadas estimadas.

Implementación técnica:
- Frontend: Buscador + resultados por categorías (orgánico, reciclable, rechazo).
- Backend: Clasificación por reglas/etiquetas + tabla de “¿dónde disponer?” por municipio.
- Datos: Tabla local de flujos y puntos de reciclaje (CSV seed).

Endpoints:
- POST `/api/waste/classify` body `{ "query": "botella PET" }`
- GET `/api/waste/where?municipio=XX&categoria=reciclable`

Pseudocódigo de reglas:
```js
function classify(text) {
  const t = text.toLowerCase()
  if (t.includes('orgánico') || t.includes('cáscara')) return 'organico'
  if (t.includes('pet') || t.includes('botella') || t.includes('plástico')) return 'reciclable'
  return 'rechazo'
}
```

Roadmap 24–48h:
- Día 1: Motor de reglas, endpoints, seed de 50 ítems y 10 puntos de disposición.
- Día 2: UI de resultados y compartir, analítica mínima de consultas.

---

## ODS 15 — Vida de ecosistemas terrestres

### Proyecto: Priorización de reforestación urbana
- Problema: Se plantan árboles donde menos impacto generan.
- Solución (MVP): Mapa de “hotspots” por isla de calor, densidad y falta de verde.
- Impacto: sitios priorizados; población beneficiada; reducción potencial de °C (estimada).

Implementación técnica:
- Frontend: Mapa de calor + lista de manzanas prioritarias y especies sugeridas.
- Backend: Índice simple `score = z(isla_calor)+z(densidad)-z(cobertura_arbolada)`.
- Datos: Temperatura superficial (tiles públicos), población (censo), OSM (parques/viales).

Endpoints:
- GET `/api/reforest/hotspots?city=...`
- GET `/api/reforest/species?zone_id=...`

Roadmap 24–48h:
- Día 1: Cálculo de score en grid 250m, endpoint y visualización.
- Día 2: Sugerencia simple de especies por altitud/clima y export de lista priorizada.

---

## Proyecto transversal — Hub de Sostenibilidad Local

Un solo front con módulos activables:
- Módulos iniciales: solar (ODS 7), residuos (ODS 12), reforestación (ODS 15).
- Beneficio: mismo stack geoespacial, impacto compuesto y narrativa integral.

MVP:
- Landing + selector de ciudad + 3 módulos funcionales en nivel demo.
- Telemetría mínima: módulos usados por sesión, métricas de impacto agregadas.

---

## Arquitectura de referencia (para cualquier módulo)

- Frontend: React/Next.js, Leaflet/MapLibre, UI simple con componentes reutilizables.
- Backend: Node.js (Express) o Python (FastAPI).
- Base de datos: PostgreSQL/Supabase (o SQLite en demo).
- Datos externos: OSM, NASA POWER, tiles satelitales, catálogos seed.
- Despliegue local: Node/Python + .env para llaves (no commitear secretos).

Estructura sugerida:
```
HackODS/
├─ frontend/
│  ├─ pages/
│  ├─ components/
│  └─ public/
├─ backend/
│  ├─ src/
│  └─ data/           # CSV/JSON seed
└─ README.md
```

Ejemplo endpoint (Express):
```js
import express from 'express'
const app = express()
app.use(express.json())

app.get('/api/solar/estimate', (req, res) => {
  const { lat, lon, consumo_kwh_mes } = req.query
  const pvKw = Math.max(Number(consumo_kwh_mes) / 120, 1.5)
  const gsr = 5.0 // kWh/m2/día aprox. demo
  const pr = 0.75
  const generacion = gsr * 365 * pvKw * pr
  const tarifa = 3.3 // demo
  const ahorro = tarifa * Math.min(generacion, Number(consumo_kwh_mes) * 12)
  const capex = pvKw * 18000
  const payback = capex / Math.max(ahorro, 1)
  const co2 = generacion * 0.00046
  res.json({ pv_kw: pvKw, generacion_anual_kwh: generacion, ahorro_anual_moneda: ahorro, payback_anios: payback, co2_evitable_ton_anio: co2 })
})

app.listen(3001)
```

Ejemplo componente (Leaflet):
```jsx
import { MapContainer, TileLayer, Marker, useMapEvents } from 'react-leaflet'
import { useState } from 'react'

function Picker({ onPick }) {
  useMapEvents({
    click(e) { onPick(e.latlng) }
  })
  return null
}

export default function MapaSolar() {
  const [point, setPoint] = useState(null)
  return (
    <MapContainer center={[19.43, -99.13]} zoom={12} style={{ height: 400 }}>
      <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
      <Picker onPick={setPoint} />
      {point && <Marker position={point} />}
    </MapContainer>
  )
}
```

---

## Datos de ejemplo (seed)

Tarifas eléctricas (CSV):
```
tarifa,costo_por_kwh
residencial,3.3
comercial,4.2
```

Vacantes verdes (JSON):
```json
[
  { "id": 1, "title": "Técnico fotovoltaico", "tags": ["electricidad","solar"], "city": "CDMX" },
  { "id": 2, "title": "Gestión de residuos", "tags": ["logística","reciclaje"], "city": "GDL" }
]
```

Puntos de disposición (CSV):
```
municipio,categoria,nombre,lat,lon
CDMX,reciclable,Centro Recicla Roma,19.41,-99.16
```

---

## Métricas y telemetría para el pitch
- ODS 7: kWh/año potenciales, tCO2e evitadas, payback promedio.
- ODS 8: matches generados, postulaciones, horas de capacitación iniciadas.
- ODS 12: consultas de clasificación, tasa de acierto, toneladas desviadas (estimadas).
- ODS 15: hotspots priorizados, población beneficiada, potencial de reducción de °C.

Instrumentación mínima:
- Contador por endpoint y por módulo.
- Evento “estimación generada”, “match visto”, “clasificación hecha”, “hotspot consultado”.

---

## Plan de demo (guion rápido)
1) Seleccionar una dirección → mostrar ahorro solar y payback.
2) Crear perfil con 3 habilidades → ver 3 vacantes verdes y 2 cursos.
3) Buscar “botella PET” → mostrar categoría y punto de disposición.
4) Abrir mapa de reforestación → ver top 5 manzanas prioritarias y especies sugeridas.

---

## Próximos pasos
- Confirma ciudad piloto y módulos.
- Creamos el esqueleto de frontend/backend y sembramos datasets.
- Integramos los endpoints y pantallas del MVP.
- Preparamos las métricas y el pitch con datos reales/demos.

Si quieres, inicio el esqueleto de proyecto y subo los archivos base con estos endpoints y componentes. También puedo adaptar los datasets a tu ciudad.

# HackODS
