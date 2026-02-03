# Optimización de patrullaje autónomo con drones (Etapa 1)

**¿Cómo vigilar la selva eficientemente cuando la batería es el límite?**

Este proyecto aplica técnicas de **Investigación Operativa** y **Teoría de Grafos** para diseñar rutas de vuelo autónomas en el Parque Nacional Iguazú. Utilizando Programación Lineal Entera (MIP), el sistema transforma un problema logísticamente inviable (cobertura total) en una misión estratégica que maximiza la vigilancia de puntos críticos bajo restricciones estrictas de batería.

> 🇺🇸 [English Version](./README.md)

![Demo Ruta Optima](./assets/demo_mision_iguazu.gif)

*Visualización de la ruta óptima resolviendo el 'Orienteering Problem'. El algoritmo prioriza 'Hotspots' (rojo) y descarta nodos de bajo valor (gris) para cumplir con la autonomía de 40 minutos.*

---

## Flujo de trabajo y resultados

El proyecto se estructura en dos fases analíticas que simulan la evolución en la toma de decisiones:

### Fase 1: Diagnóstico de factibilidad (El fracaso del TSP)
*Enfoque: Geodesia y grafos (Notebook `01_Analisis_Factibilidad_TSP`)*

Inicialmente, evaluamos la estrategia intuitiva de cubrir todo (Traveling Salesperson Problem). Al modelar el terreno y calcular distancias reales (Haversine), demostramos que esta estrategia es físicamente imposible: requiere 64 minutos de vuelo, superando la autonomía del dron (40 min) en un 60%.

<div align="center">
  <img src="./assets/grafo_tsp_fallido.png" width="70%">
  <p><i>La ruta roja (TSP) colapsa por falta de batería antes de volver a la base.</i></p>
</div>

### Fase 2: El duelo de algoritmos (Greedy vs. MIP)
*Enfoque: Programación matemática (Notebook `02_Optimizacion_Mision_OP`)*

Ante la inviabilidad física, implementamos dos estrategias para resolver el **Orienteering Problem (OP)**:

1.  **Heurística Greedy:** Simula una decisión humana rápida, yendo siempre al punto más valioso y cercano.
2.  **Optimización exacta (MIP con SCIP):** Busca la solución matemática perfecta evaluando todas las combinaciones posibles.

**Hallazgo técnico interesante:**
Descubrimos que, para este escenario específico, ambos algoritmos lograron capturar el mismo valor de vigilancia (Score). Sin embargo, el solver exacto fue menos eficiente en el uso del tiempo (consumió casi toda la batería disponible) comparado con el Greedy.

> *¿Por qué pasó esto?* El modelo MIP actual busca maximizar el Score sin importar cuánto tiempo gaste, siempre que sea $< 40$ min. Al no tener una penalización por tiempo, el solver no tuvo incentivo para "ahorrar" batería, un comportamiento que corregiremos en la Etapa 2.

| Métrica | Estrategia TSP (Fuerza bruta) | Heurística Greedy (Intuición) | Optimización MIP (Matemática) |
| :--- | :--- | :--- | :--- |
| **Objetivo** | Visitar todo | Mejor ratio local | Máximo score global |
| **Tiempo de vuelo** | 64.2 min (Inviable) | 38.62 min (Sobra batería) | **39.8 min (Límite)** |
| **Score de vigilancia** | 100% (Teórico) | Alto | **Máximo posible** |
| **Resultado** | Pérdida de dron | Misión exitosa | **Misión óptima** |

---

## Impacto y conclusiones de la Etapa 1
El modelo logró generar un plan de vuelo operativo que valida el uso de drones en el sector. Las conclusiones principales son:

* **De lo imposible a lo táctico:** Transformamos una misión fallida (TSP) en una operación viable seleccionando inteligentemente los objetivos.
* **Seguridad operativa:** Se garantiza matemáticamente el retorno a la base (Seccional Apepú), eliminando el riesgo de aterrizajes forzosos en la selva.
* **Validación de algoritmos:** Comprobamos que para grafos pequeños (10 nodos), una heurística bien diseñada puede competir con solvers complejos, aunque el MIP es necesario para garantizar optimidad en escalas mayores.

## Desafíos técnicos y soluciones
Como estudiante de Ciencia de Datos, este proyecto requirió ir más allá del análisis exploratorio:

* **Modelado de restricciones:** Implementé un modelo MIP con `PySCIPOpt`, definiendo variables binarias ($x_{ij}$) para el flujo de ruta.
* **Eliminación de subtours:** Uno de los mayores retos fue evitar que el dron creara "anillos aislados" en el grafo. Lo solucioné implementando las restricciones matemáticas de **Miller-Tucker-Zemlin (MTZ)**.
* **Visualización operativa:** Transformé la salida abstracta del solver en un mapa interactivo con `Folium` y `AntPath`, permitiendo visualizar el flujo de la patrulla sobre el terreno real.

## Cómo ejecutar este proyecto localmente

### 1. Requisitos previos
Este proyecto utiliza `PySCIPOpt`, que requiere la instalación del solver **SCIP** (se recomienda usar Conda).

### 2. Instalación y ejecución

1. **Clonar el repositorio:**
   ```bash
   git clone [https://github.com/FernandaVil/drone-patrol-optimization.git](https://github.com/FernandaVil/drone-patrol-optimization.git)
