<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'

// --- ESTADO ---
const selectedCity = ref('Almería (Zapillo)')
const loading = ref(false)
const weather = ref(null)
const error = ref(null)

// --- CIUDADES/CLUBES PARA EL SELECTOR ---
const locations = [
  { name: 'Almería (Zapillo)', lat: 36.82, lon: -2.46, type: 'Costa' },
  { name: 'Madrid (Retiro)', lat: 40.41, lon: -3.70, type: 'Interior' },
  { name: 'Barcelona (Port)', lat: 41.38, lon: 2.17, type: 'Costa' },
  { name: 'Sevilla (Pino Montano)', lat: 37.38, lon: -5.98, type: 'Interior' },
  { name: 'Londres (Canary Wharf)', lat: 51.50, lon: -0.12, type: 'Norte' }
]

// --- ALGORITMO DE JUGABILIDAD (EL "CEREBRO") ---
const padelScore = computed(() => {
  if (!weather.value) return 0
  const { temp, humidity, dewPoint, wind, precip } = weather.value
  let score = 10
  
  if (precip > 0.1) return 0 // Lluvia = Injugable
  
  // Condensación (Delta T entre ambiente y rocío)
  const deltaT = temp - dewPoint
  if (deltaT < 1.5) score -= 6
  else if (deltaT < 3) score -= 3
  
  // Viento (Globos)
  if (wind > 30) score -= 5
  else if (wind > 18) score -= 2
  
  return Math.max(0, score).toFixed(1)
})

const getStatusColor = (score) => {
  if (score >= 8) return '#22c55e' // Verde
  if (score >= 5) return '#f59e0b' // Ámbar
  return '#ef4444' // Rojo
}

// --- LÓGICA DE DATOS ---
const fetchData = async () => {
  loading.value = true
  error.value = null
  const loc = locations.find(l => l.name === selectedCity.value)
  
  try {
    const res = await axios.get(`https://api.open-meteo.com/v1/forecast?latitude=${loc.lat}&longitude=${loc.lon}&current=temperature_2m,relative_humidity_2m,dew_point_2m,precipitation,wind_speed_10m&timezone=auto`)
    const c = res.data.current
    weather.value = {
      temp: c.temperature_2m,
      humidity: c.relative_humidity_2m,
      dewPoint: c.dew_point_2m,
      wind: c.wind_speed_10m,
      precip: c.precipitation
    }
  } catch (e) {
    error.value = "Error conectando con la estación meteorológica."
  } finally {
    loading.value = false
  }
}

onMounted(fetchData)
</script>

<template>
  <div class="dashboard-root">
    <aside class="sidebar">
      <div class="brand">
        <div class="brand-icon">🎾</div>
        <h2>Padel<span>Ready</span></h2>
      </div>
      
      <nav class="nav-menu">
        <p class="nav-label">ZONAS CRÍTICAS</p>
        <div 
          v-for="loc in locations" 
          :key="loc.name"
          class="nav-item"
          :class="{ active: selectedCity === loc.name }"
          @click="selectedCity = loc.name; fetchData()"
        >
          <span>{{ loc.name }}</span>
          <small>{{ loc.type }}</small>
        </div>
      </nav>
      
      <div class="pro-tag">PRO VERSION</div>
    </aside>

    <main class="main-content">
      <header class="top-bar">
        <div class="search-box">
          <input type="text" placeholder="Buscar club o distrito..." disabled>
        </div>
        <div class="user-profile">
          <span>FJ</span>
        </div>
      </header>

      <section class="display-area">
        <div class="hero-card" :class="{ loading: loading }">
          <div class="hero-info">
            <span class="location-tag">📍 {{ selectedCity }}</span>
            <h1 v-if="!loading">Padel Score: {{ padelScore }}/10</h1>
            <h1 v-else>Analizando pista...</h1>
            <p v-if="padelScore >= 8">Condiciones perfectas. Globos permitidos y cristales secos.</p>
            <p v-else-if="padelScore >= 5">Cuidado: La bola podría resbalar en el cristal al fondo.</p>
            <p v-else>Mejor quédate en el bar. Los cristales están empañados o hay temporal.</p>
          </div>
          
          <div class="gauge-container">
            <div class="gauge" :style="{ '--score': (padelScore * 10) + '%', '--color': getStatusColor(padelScore) }">
              <span class="score-text">{{ padelScore }}</span>
            </div>
          </div>
        </div>

        <div class="metrics-grid">
          <div class="metric-card">
            <div class="m-icon">💧</div>
            <div class="m-data">
              <label>Humedad</label>
              <p>{{ weather?.humidity }}%</p>
            </div>
            <div class="m-status" :class="{ ok: weather?.humidity < 80 }"></div>
          </div>

          <div class="metric-card">
            <div class="m-icon">💨</div>
            <div class="m-data">
              <label>Viento</label>
              <p>{{ weather?.wind }} km/h</p>
            </div>
            <div class="m-status" :class="{ warn: weather?.wind > 20 }"></div>
          </div>

          <div class="metric-card">
            <div class="m-icon">🌡️</div>
            <div class="m-data">
              <label>Punto Rocío</label>
              <p>{{ weather?.dewPoint }}°C</p>
            </div>
            <div class="m-status" :class="{ ok: (weather?.temp - weather?.dewPoint) > 4 }"></div>
          </div>

          <div class="metric-card">
            <div class="m-icon">🎾</div>
            <div class="m-data">
              <label>Presión Bola</label>
              <p>{{ weather?.temp > 15 ? 'Alta' : 'Baja' }}</p>
            </div>
          </div>
        </div>

        <div class="analysis-panel">
          <h3>Análisis Térmico de Condensación</h3>
          <div class="logic-flow">
            <div class="step">
              <small>Temp Ambiente</small>
              <span>{{ weather?.temp }}°C</span>
            </div>
            <div class="connector">➖</div>
            <div class="step">
              <small>Dew Point</small>
              <span>{{ weather?.dewPoint }}°C</span>
            </div>
            <div class="connector">🟰</div>
            <div class="step highlighted">
              <small>Spread (Delta T)</small>
              <span>{{ (weather?.temp - weather?.dewPoint).toFixed(1) }}°C</span>
            </div>
          </div>
          <p class="expert-note">
            *Un Spread menor a 2°C garantiza que el vapor de agua se condense en el vidrio al primer contacto.
          </p>
        </div>
      </section>
    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;800&display=swap');

:root {
  --bg-deep: #080a0f;
  --bg-main: #0f111a;
  --glass: rgba(255, 255, 255, 0.03);
  --glass-border: rgba(255, 255, 255, 0.08);
  --accent: #6366f1;
}

* { box-sizing: border-box; }
body { margin: 0; background: var(--bg-deep); font-family: 'Plus Jakarta Sans', sans-serif; color: white; overflow: hidden; }

.dashboard-root { display: flex; height: 100vh; }

/* --- SIDEBAR --- */
.sidebar { width: 280px; background: var(--bg-main); border-right: 1px solid var(--glass-border); padding: 30px; display: flex; flex-direction: column; }
.brand { display: flex; align-items: center; gap: 12px; margin-bottom: 50px; }
.brand-icon { font-size: 1.5rem; background: var(--accent); padding: 8px; border-radius: 12px; }
.brand h2 { margin: 0; font-size: 1.4rem; font-weight: 800; }
.brand span { color: var(--accent); }

.nav-label { font-size: 0.7rem; color: #475569; letter-spacing: 2px; font-weight: 800; margin-bottom: 20px; }
.nav-item { padding: 15px; border-radius: 12px; cursor: pointer; transition: 0.3s; margin-bottom: 10px; border: 1px solid transparent; }
.nav-item:hover { background: var(--glass); }
.nav-item.active { background: rgba(99, 102, 241, 0.1); border-color: var(--accent); }
.nav-item span { display: block; font-weight: 600; }
.nav-item small { color: #64748b; font-size: 0.75rem; }

/* --- MAIN CONTENT --- */
.main-content { flex: 1; padding: 30px; overflow-y: auto; background: radial-gradient(circle at top right, rgba(99, 102, 241, 0.05), transparent); }

.top-bar { display: flex; justify-content: space-between; margin-bottom: 40px; }
.search-box input { background: var(--glass); border: 1px solid var(--glass-border); padding: 12px 25px; border-radius: 30px; color: white; width: 300px; outline: none; }
.user-profile { width: 45px; height: 45px; background: var(--accent); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 800; }

/* --- HERO CARD --- */
.hero-card { 
  background: linear-gradient(135deg, rgba(255,255,255,0.05) 0%, rgba(255,255,255,0) 100%);
  border: 1px solid var(--glass-border);
  border-radius: 30px;
  padding: 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  backdrop-filter: blur(10px);
}
.location-tag { background: var(--accent); font-size: 0.7rem; font-weight: 800; padding: 5px 12px; border-radius: 20px; text-transform: uppercase; }
.hero-info h1 { font-size: 2.5rem; margin: 15px 0; }
.hero-info p { color: #94a3b8; max-width: 400px; line-height: 1.6; }

/* GAUGE CSS */
.gauge { 
  width: 150px; height: 150px; border-radius: 50%; 
  background: conic-gradient(var(--color) var(--score), #1e293b 0);
  display: flex; align-items: center; justify-content: center;
  position: relative;
}
.gauge::after { content: ''; position: absolute; width: 120px; height: 120px; background: var(--bg-main); border-radius: 50%; }
.score-text { position: relative; z-index: 10; font-size: 2.5rem; font-weight: 800; }

/* --- METRICS --- */
.metrics-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; margin-bottom: 30px; }
.metric-card { background: var(--glass); border: 1px solid var(--glass-border); padding: 20px; border-radius: 20px; display: flex; align-items: center; gap: 15px; position: relative; }
.m-icon { font-size: 1.5rem; }
.m-data label { font-size: 0.7rem; color: #64748b; text-transform: uppercase; font-weight: 800; }
.m-data p { margin: 5px 0 0 0; font-size: 1.2rem; font-weight: 700; }
.m-status { width: 8px; height: 8px; border-radius: 50%; background: #334155; position: absolute; top: 15px; right: 15px; }
.m-status.ok { background: #22c55e; box-shadow: 0 0 10px #22c55e; }
.m-status.warn { background: #f59e0b; box-shadow: 0 0 10px #f59e0b; }

/* --- ANALYSIS PANEL --- */
.analysis-panel { background: var(--glass); border: 1px solid var(--glass-border); padding: 30px; border-radius: 25px; }
.logic-flow { display: flex; align-items: center; gap: 20px; margin: 25px 0; }
.step { background: #1e293b; padding: 15px 25px; border-radius: 15px; text-align: center; border: 1px solid var(--glass-border); }
.step.highlighted { border-color: var(--accent); background: rgba(99, 102, 241, 0.1); }
.step small { display: block; color: #64748b; font-size: 0.7rem; margin-bottom: 5px; }
.step span { font-size: 1.3rem; font-weight: 700; }
.expert-note { font-size: 0.8rem; color: #475569; font-style: italic; }

.pro-tag { margin-top: auto; background: #1e293b; padding: 10px; border-radius: 10px; text-align: center; font-size: 0.7rem; font-weight: 800; color: #94a3b8; }
</style>