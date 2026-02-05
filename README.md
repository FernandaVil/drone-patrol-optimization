# Autonomous Drone Patrol Optimization (Stage 1)

**How to monitor the rainforest efficiently when battery life is the limit?**

This project applies **Operations Research** and **Graph Theory** techniques to design autonomous flight paths in Iguazú National Park. Using Mixed-Integer Programming (MIP), the system transforms a logistically unfeasible problem (total coverage) into a strategic mission that maximizes surveillance of critical hotspots under strict battery constraints.

> 🇪🇸 [Versión en Español](./README.es.md)

![Optimal Route Demo](./assets/demo_mision_iguazu.gif)

*Visualization of the optimal route solving the 'Orienteering Problem'. The algorithm prioritizes 'Hotspots' (red) and discards low-value nodes to comply with the 40-minute flight autonomy.*

---

## Context and Problem: The Fight Against Poaching
Iguazú National Park faces constant threats from poaching. Hunters utilize known strategic points called **saleros** (mineral salt licks where animals gather) and **picadas** (rudimentary trails opened in the jungle) for their illegal activities. 

Monitoring these areas via human foot patrols is dangerous, slow, and logistically expensive. Implementing drones would allow for a rapid response but presents an engineering challenge: a drone that runs out of battery over the dense canopy is a lost unit.

### Area of Study: Apepú Sector
For this project, we delimited a 25 km² critical area in the Apepú Sector. This sector was chosen due to its high biodiversity density and existing infrastructure (ranger stations). 

### Graph Construction: Real vs. Synthetic Data
One of the main technical challenges was **spatial data curation**, as there is no public dataset of "poaching points of interest." The 10-node network was built through:
* **Real Nodes:** Surveillance stations and known salt licks, obtained by cross-referencing ranger reports with official cartography.
* **Transit Nodes (Synthetic):** Calculated points to ensure drone connectivity and optimize trajectories between real objectives (e.g., camera trap coordinates).

> 💡 *Refer to **Notebook 01** for details on the georeferencing process and the scoring logic assigned to each node type.*

---

## Workflow and Results
The project is structured into two analytical phases that simulate the evolution of decision-making:

### Phase 1: Feasibility Diagnosis (The TSP Failure)
*Focus: Geodesy and Graphs (Notebook `01_Analisis_Factibilidad_TSP`)*

Initially, we evaluated the intuitive strategy of total coverage (Traveling Salesperson Problem). By modeling the terrain and calculating real distances (Haversine), we proved this strategy is physically impossible: it requires 64 minutes of flight, exceeding the drone's autonomy (40 min) by 60%.

<div align="center">
  <img src="./assets/grafo_tsp_fallido.png" width="70%">
  <p><i>The red route (TSP) collapses due to battery depletion before returning to base.</i></p>
</div>

### Phase 2: Algorithm Showdown (Greedy vs. MIP)
*Focus: Mathematical Programming (Notebook 02)*

Given the physical unfeasibility, we implemented two strategies to solve the **Orienteering Problem (OP)**:

1. **Greedy Heuristic:** Simulates a quick human decision, always moving to the nearest, most valuable point.
2. **Exact Optimization (MIP with SCIP):** Searches for the perfect mathematical solution by evaluating all possible combinations.

| Metric | TSP Strategy (Brute Force) | Greedy Heuristic (Intuition) | MIP Optimization (Exact) |
| :--- | :--- | :--- | :--- |
| **Objective** | Visit all nodes | Best local ratio | **Maximum global score** |
| **Flight Time** | 64.2 min (Unfeasible) | 38.62 min (Operational) | **36.99 min (Efficient)** |
| **Surveillance Score** | 100% (Theoretical) | High | **Maximum** |
| **Outcome** | Unit Lost | Successful Mission | **Optimal Mission** |

> 💡 **Technical Finding:**
While the MIP solver was faster, it is important to note that its current objective function is exclusively to maximize the score. In this scenario, the highest-scoring route also happened to be very time-efficient. However, in more complex problems, the solver might have chosen any route under 40 minutes regardless of battery savings. This "time indifference" is a latent risk that justifies the need for multi-objective optimization in the next stage.

---

## Technical Challenges and Solutions
* **Geospatial Data Curation:** I performed a cleaning and cross-referencing process between official geographic maps and patrol logs to extract precise coordinates in an environment where postal addresses and detailed commercial cartography do not exist.
* **Constraint Modeling:** Implemented a Mixed-Integer Programming (MIP) model using `PySCIPOpt`, defining binary variables ($x_{ij}$) for route flow.
* **Subtour Elimination:** One of the main challenges was preventing the drone from creating "isolated rings" in the graph. I solved this by implementing **Miller-Tucker-Zemlin (MTZ)** mathematical constraints.
* **Operational Visualization:** Transformed the abstract solver output into an interactive map using `Folium` and `AntPath`, allowing for the visualization of the patrol flow over real terrain.

## Impact and Conclusions of Stage 1
The model generated an operational flight plan that validates the use of drones in the sector. The main conclusions are:

* **From Impossible to Tactical:** We transformed a failed mission (TSP) into a viable operation by intelligently selecting targets.
* **Superiority of Global Analysis:** The MIP solver saved nearly 1.6 minutes of additional battery compared to Greedy by finding a more fluid visit sequence.
* **Operational Safety:** Base return (Apepú Station) is mathematically guaranteed, eliminating the risk of forced landings in the jungle.
* **Algorithm Validation:** We confirmed that for small graphs (10 nodes), a well-designed heuristic can compete with complex solvers, though MIP is necessary to guarantee optimality at larger scales.

This repository is designed to be read sequentially. The notebooks contain an extended technical narrative, mathematical justifications, and step-by-step documented source code.

---

## How to run this project locally

### 1. Prerequisites
This project uses `PySCIPOpt`, which requires the **SCIP** solver installation (Conda is highly recommended).

### 2. Installation and execution

1. **Clone the repository:**
   ```bash
   git clone https://github.com/FernandaVil/drone-patrol-optimization.git
2. **Create the environment and install dependencies:**
  ```bash
     cd drone-patrol-optimization
     conda create -n drones python=3.9
     conda activate drones
     conda install -c conda-forge pyscipopt  # Solver SCIP 
     pip install -r requirements.txt
   ```
3. **Execution:** Open the notebooks in sequential order.
  * notebooks/01_Analisis_Factibilidad_TSP.ipynb : Geospatial diagnosis.

  * notebooks/02_Optimizacion_Mision_OP.ipynb : Greedy vs. MIP comparison and final dashboard.

## Estructura del proyecto
  ```bash
      drone-patrol-optimization/
      ├── assets/
      │   ├── demo_mision_iguazu.gif
      │   └── grafo_tsp_fallido.png
      ├── notebooks/
      │   ├── 01_Analisis_Factibilidad_TSP.ipynb
      │   └── 02_Optimizacion_Mision_OP.ipynb
      ├── output/
      │   └── mision_iguazu__dashboard_final.html  <-- Interactive map
      ├── requirements.txt
      └── README.md
   ```
---
## Next steps: Stage 2 - Dynamic complexity
This first stage addressed static planning (Offline). The next phase of the project, currently under design, will address the changing reality of the rainforest:

* **Dynamic Nodes (Event-Driven):** What if a camera trap detects movement mid-flight? The system must recalculate the optimal route in real-time based on sensor alerts.
* **Terrain Risk ($R_{ij}$):** Flying over a trail is not the same as flying over virgin jungle. We will incorporate a risk cost matrix to penalize trajectories where equipment recovery would be impossible.
* **Multi-objective Function:** We will refine the MIP algorithm to not only maximize the score but also minimize flight time as a secondary objective. This ensures the mathematically fastest route among multiple options with the same score, avoiding sub-optimal solutions in more complex scenarios.

---

## References and sources
To ensure the technical validity of the optimization model and the precision of the geospatial calculations, the following sources were consulted:

* **Orienteering Problem:** Gunawan, A., Lau, H. C., & Vansteenwegen, P. (2016). *Orienteering Problem: A survey of recent variants, solution approaches and applications*. European Journal of Operational Research. [DOI: 10.1016/j.ejor.2016.04.059](https://doi.org/10.1016/j.ejor.2016.04.059)
* **Spherical Distance Calculation (Haversine):** Sinnott, R. W. (1984). *Virtues of the Haversine*. Sky and Telescope. (Mathematical basis for geodesic distance measurements in the Apepú Sector).
* **Cartography and Regional Context:**
    * Data obtained by cross-referencing information from the **National Geographic Institute (IGN Argentina)** and Iguazú National Park patrol reports.

---
*Project developed as a personal exploration in Operations Research and Decision Support Systems (DSS).*
