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

# Problemas clave y 3 opciones de proyectos por ODS (software + hardware + monitoreo)

## ODS 7 — Energía asequible y no contaminante

Problemas clave:
- Alto costo inicial y desconocimiento del retorno (hogares/MIPYMES).
- Consumos ineficientes por falta de datos y hábitos.
- Zonas con servicio eléctrico inestable o caro.

Opciones de proyecto:
- Mapa de techos solares y ROI comunitario
  - Software: Web de estimación, API de irradiancia, simulador de ahorro.
  - Hardware: Kit fotovoltaico demostrativo (panel 200–400W, microinversor).
  - Monitoreo: Smart meter con ESP32 + sensor PZEM-004T, dashboard en Grafana/InfluxDB.
- Microred vecinal con almacenamiento
  - Software: Optimización de despacho básico, balance de carga, app de visualización.
  - Hardware: Baterías LiFePO4, inversor híbrido, controladores MPPT, medidores por casa.
  - Monitoreo: LoRaWAN/NB-IoT para telemetría; alerta de cortes; registro de kWh compartidos.
- Kit de eficiencia energética en escuelas/MIPYMES
  - Software: App con recomendaciones personalizadas y metas; analítica de uso.
  - Hardware: Sensores de corriente (CT), temperatura y presencia; relevadores para horarios.
  - Monitoreo: NILM básico, alarmas de pico de consumo, reportes semanales de ahorro.

## ODS 8 — Trabajo decente y crecimiento económico

Problemas clave:
- Brecha entre habilidades actuales y empleos verdes emergentes.
- Baja digitalización/productividad en MIPYMES.
- Ingresos locales estacionales y poca diversificación.

Opciones de proyecto:
- Plataforma de empleo verde y microcredenciales
  - Software: Matching por habilidades y cursos; portafolio y badge digital.
  - Hardware: Kits de práctica para instalación solar/eficiencia en makerspace.
  - Monitoreo: Seguimiento de progreso en LMS; métricas de colocación y certificación.
- Diagnóstico digital de MIPYMES con sensorización ligera
  - Software: Scorecard y plan 90 días con tablero operativo.
  - Hardware: Sensores de temperatura/humedad en frío, contador de personas, enchufes inteligentes.
  - Monitoreo: Alertas de mermas/consumo; comparativas antes/después; indicadores de productividad.
- Microturismo sostenible con conteo y señalización
  - Software: Rutas y reservas; reputación de negocios responsables; mapa de flujo.
  - Hardware: Balizas BLE/QR, sensores de paso en puntos clave, señalética física.
  - Monitoreo: Conteo de afluencia y gasto local estimado; encuestas en sitio.

## ODS 12 — Producción y consumo responsables

Problemas clave:
- Baja separación de residuos por desconocimiento y falta de infraestructura.
- Subproductos valiosos se tratan como desecho.
- Compostaje y orgánicos sin control, generan olores y emisiones.

Opciones de proyecto:
- Asistente de separación y contenedores inteligentes
  - Software: Clasificación por texto/imagen y guía local de disposición.
  - Hardware: Botes con sensor ultrasónico de llenado (ESP8266/ESP32) y apertura asistida.
  - Monitoreo: Mapa de llenado y rutas óptimas de recolección; alarmas de rebose.
- Marketplace de subproductos con trazabilidad
  - Software: Publicación y match “tengo/ofrezco”; contratos simples y logística.
  - Hardware: Básculas en punto de generación; etiquetado QR/NFC por lote.
  - Monitoreo: Registro de toneladas desviadas y calidad del material; auditoría básica.
- Kit de compostaje urbano monitoreado
  - Software: App con etapas, alertas y guía de mantenimiento; panel de comunidad.
  - Hardware: Sensores de temperatura (DS18B20) y humedad; aireación manual/automática.
  - Monitoreo: Curvas de temperatura/humedad; tiempo a compost maduro; reducción de peso.

## ODS 15 — Vida de ecosistemas terrestres

Problemas clave:
- Deforestación y cambio de cobertura detectados tarde.
- Calor urbano y baja cobertura arbórea.
- Monitoreo de biodiversidad limitado y poco participativo.

Opciones de proyecto:
- Alertas tempranas con satélite y cámaras trampa
  - Software: Detección de cambio con NDVI/NBR; portal de alertas y validaciones.
  - Hardware: Cámaras trampa con Raspberry Pi/ESP32-CAM y panel solar.
  - Monitoreo: Uplink LoRaWAN/LTE; revisión por cuadrantes; bitácora de eventos.
- Priorización y salud de arbolado urbano
  - Software: Score de hotspots (isla de calor, densidad, cobertura) y plan de plantación.
  - Hardware: Sensores de humedad de suelo y tensiómetros; riego por goteo.
  - Monitoreo: Inspecciones con drone (RGB/multiespectral) y dashboard de supervivencia.
- Biodiversidad ciudadana acústica
  - Software: App de avistamientos y análisis de audio básico (claves de aves).
  - Hardware: Grabadores acústicos de bajo costo; microfonía protegida en parques.
  - Monitoreo: Mapa de puntos con riqueza acústica; campañas gamificadas y ranking.

Tecnologías recomendadas transversales:
- Software: Next.js/React, Express/FastAPI, Supabase/PostgreSQL, Leaflet/MapLibre, Grafana.
- Hardware: ESP32/ESP8266, Raspberry Pi, sensores CT/ultrasonido/temperatura/humedad, paneles y microinversores.
- Conectividad: LoRaWAN, NB-IoT/LTE, WiFi comunitario; almacenamiento local con sincronización.
- Telemetría: InfluxDB/TimescaleDB, alertas por webhooks/Telegram, paneles de operación.

# CDMX — Enfoque de Machine Learning guiado por datos
- Contexto: Los ODS están interrelacionados y requieren estrategias específicas por región.
- Referencia: “Sustainable visions: unsupervised machine learning insights on global development goals” (PLOS ONE, 2025). Enfoque con aprendizaje automático no supervisado sobre series históricas e indicadores comparables para recomendar intervenciones por zona.
- Objetivo: Recomendar, por colonia/manzana en Ciudad de México, combinaciones óptimas de módulos (energía, empleo, residuos, arbolado) según patrones locales.
- Datos seed iniciales (carpeta backend/data):
  - indicadores_locales_cdmx.csv: índices normalizados por zona (energía, residuos, calor, empleo verde, cobertura arbórea).
  - tarifas_cdmx.csv: costos de energía por segmento.
  - vacantes_verdes_cdmx.json y cursos_verdes_cdmx.json: oferta de empleo y capacitación verde.
  - puntos_disposicion_cdmx.csv: lugares de disposición y reciclaje.
  - cdmx_hotspots_grid.csv: grid con score de priorización para reforestación.
- Pipeline de recomendación (MVP):
  - Normalizar indicadores (0–1).
  - Clustering no supervisado (K-Means/HDBSCAN) y reducción de dimensión (PCA/UMAP) para entender patrones.
  - Reglas de decisión sobre cada zona: sugerir módulos según umbrales (isla de calor alta, baja cobertura arbórea, alto consumo/tarifa, baja infraestructura de reciclaje, desempleo verde elevado).
  - Exponer endpoint “/api/recommendations/cdmx” con lista ordenada de intervenciones por zona.

# Plan de implementación para Ciudad de México (MVP con ML)
- Día 0: Configuración de proyecto (frontend + backend) y creación de datasets seed.
- Día 1: Endpoints de estimación solar, residuos, reforestación y recomendación ML por zona; ingestión de seed en base de datos.
- Día 2: UI del Hub con módulos activables, mapa por zonas y panel de métricas multiobjetivo; telemetría básica (eventos por módulo).
- Opcional: Demostraciones de hardware (ESP32 + PZEM para energía; sensor ultrasónico en contenedor; registro acústico).

# Lenguajes y tecnologías (descripciones completas)
- JavaScript (Node.js + Express): plataforma de tiempo de ejecución del lado del servidor y framework web para APIs rápidas.
- Python (FastAPI + pandas + scikit-learn): framework web de alto rendimiento y librerías para procesamiento de datos y aprendizaje automático.
- Next.js (sobre React): framework web para construir interfaces de usuario y páginas con renderizado del lado del servidor.
- Leaflet / MapLibre GL: librerías de mapas web para visualización geoespacial.
- PostgreSQL / Supabase: base de datos relacional (PostgreSQL) y plataforma administrada que añade autenticación y APIs fáciles de usar.
- InfluxDB / TimescaleDB: bases de datos optimizadas para series de tiempo (telemetría de sensores).
- Grafana: plataforma de dashboards para visualizar métricas y telemetría.
- LoRaWAN (Long Range Wide Area Network): red de largo alcance y bajo consumo para telemetría de sensores.
- NB‑IoT (Narrowband Internet of Things) / LTE (Long Term Evolution): tecnologías de conectividad celular para dispositivos IoT.
- CSV (Comma-Separated Values) / JSON (JavaScript Object Notation): formatos de intercambio de datos sencillos para el seed y APIs.
- GIS: Sistemas de Información Geográfica para el manejo de datos espaciales; NDVI (Normalized Difference Vegetation Index) y NBR (Normalized Burn Ratio) para análisis satelital.
- PCA (Principal Component Analysis) / UMAP (Uniform Manifold Approximation and Projection): técnicas de reducción de dimensión para visualizar y entender estructura en datos.
- K‑Means / HDBSCAN (Hierarchical Density-Based Spatial Clustering of Applications with Noise): algoritmos de agrupamiento no supervisado para detección de patrones por zona.
# HackODS
