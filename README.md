# 🎾 Padel-Ready Dashboard

![Vue.js](https://img.shields.io/badge/vuejs-%2335495e.svg?style=for-the-badge&logo=vuedotjs&logoColor=%234FC08D)
![Vite](https://img.shields.io/badge/vite-%23646CFF.svg?style=for-the-badge&logo=vite&logoColor=white)
![Open-Meteo](https://img.shields.io/badge/API-Open--Meteo-blue?style=for-the-badge)

**Padel-Ready** es una plataforma de inteligencia meteorológica diseñada específicamente para jugadores de pádel. A diferencia de las aplicaciones de tiempo genéricas, esta herramienta utiliza un algoritmo termodinámico para predecir la jugabilidad en pista basándose en la condensación de los cristales y la dinámica del viento.



## 🧠 El Algoritmo: Padel Score ($PS$)

El núcleo del proyecto es el cálculo del **Padel Score**, un índice de 0 a 10 que evalúa la calidad del juego técnico.

### 1. Factor de Condensación (Cristales)
La app calcula el "Spread" entre la temperatura ambiente ($T_a$) y el punto de rocío ($T_{dp}$):
$$\Delta T = T_a - T_{dp}$$
Si $\Delta T < 2^\circ C$, el vapor de agua se condensa automáticamente en el vidrio, anulando el rebote de la bola.

### 2. Factor de Viento (Globos)
Se analizan las rachas de viento para determinar si el control de los globos es posible o si el partido se convierte en una "lotería" táctica.

## ✨ Características Principales

* **Glassmorphism UI:** Interfaz moderna y oscura con efectos de desenfoque y profundidad.
* **Hiper-localización:** Consultas de precisión mediante coordenadas geográficas exactas (Grid de 2-11km).
* **Métricas en Tiempo Real:** Datos directos de estaciones meteorológicas sin latencia.
* **Veredicto Inteligente:** Recomendaciones dinámicas según el estado de la pista.

## 🛠️ Tech Stack

* **Frontend:** Vue 3 (Composition API)
* **Build Tool:** Vite
* **Estilos:** CSS3 Avanzado (Variables dinámicas, Flexbox/Grid)
* **Datos:** Axios + Open-Meteo API

## 🚀 Instalación y Uso

1.  **Clonar el repositorio:**
    ```bash
    git clone [https://github.com/FranJDM/Padel-Ready-App.git](https://github.com/FranJDM/Padel-Ready-App.git)
    ```

2.  **Instalar dependencias:**
    ```bash
    cd my-vue-app
    npm install
    ```

3.  **Lanzar en modo desarrollo:**
    ```bash
    npm run dev
    ```

## 📈 Roadmap

- [ ] Integración de mapas interactivos con Mapbox.
- [ ] Histórico de jugabilidad por franjas horarias.
- [ ] Notificaciones push de "Pista Seca".
- [ ] Base de datos de clubes locales.

---
Desarrollado con ❤️ para la comunidad de pádel.
