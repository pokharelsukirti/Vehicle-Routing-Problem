# Vehicle-Routing-Problem
# Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)
## Fuel Distribution Model
---
## Overview
The **Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)** is an operations research model used to optimize the routing of a fleet of vehicles with varying capacities and characteristics for fuel delivery. This document outlines the core presumptions, vehicle parameters, fuel types, and operational constraints that govern the model.

---
## Presumptions
The following assumptions underpin the HCVRP model:

1. **Terminal Fulfillment Guarantee**
   Terminals always maintain sufficient stock to fulfill any incoming order. Supply-side shortages at the terminal are not considered within this model.
2. **Vehicle Heterogeneity**
   Vehicles in the fleet differ from one another in maintenance cost, speed, and fuel consumption per kilometre. They are categorized into 5 categories based on these parameters.
3. **Depot Return Policy**
   Every vehicle must return, at the end of each trip, to the **same terminal** it departed from.
4. **Split Delivery Permitted** *(carried over from prior discussion)*
   A customer's demand for a given fuel may be fulfilled by more than one compartment, vehicle, or trip, provided every contributing compartment is fuel-compatible.
5. **Multi-Trip Operation** *(carried over from prior discussion)*
   A vehicle may be dispatched up to twice in a single day, provided each individual trip's duration (terminal departure to terminal return) does not exceed 5 hours.

> If assumption 3 doesn't match reality — e.g. each vehicle has one fixed home terminal rather than a choice — the model simplifies (see Modeling Notes at the end).

---
## Fuel Types
The model handles three distinct fuel types:

| # | Fuel Type | Notes |
|---|-----------|-------|
| 1 | **Petrol** | Requires a compatible compartment |
| 2 | **Diesel** | Requires a compatible compartment |
| 3 | **Kerosine** | Requires a compatible compartment |

> Fuel types must be matched to designated vehicle compartments. Cross-contamination between fuel types is not permitted.

---
## Vehicle Types
To be added later — once specified, each type $v$ will carry its own $Qcap_{v,k}$ (per compartment), $Cm_v$, $Cf_v$, $Sp_v$, and fleet count $m_v$ (all defined below).

---
# Notation

## Indices and Sets

| Symbol | Description |
|---|---|
| $v_1,\cdots,V$ | set of vehicle **types** (categories) — kept as your symbol, clarified to mean the 5 categories from Presumption 2, consistent with how $N_v$ is defined below |
| $k_1,\cdots,K$ | set of compartments within a vehicle |
| $f_1,\cdots,F$ | set of fuel types |
| $c_1,\cdots,C$ | set of customers |
| $t_1,\cdots,T$ | set of terminals |
| $j$ | generic index for a node, where a node is either a terminal or a customer location |
| $i \in I_v = \{1,\dots,m_v\}$ | **(new)** physical vehicle instances of type $v$ — needed because several individual trucks of the *same* type can be on the road at once, each on its own route |
| $g \in G=\{1,2\}$ | **(new)** trip/run index per vehicle per day — supports Presumption 5 (double trips) |
| $(j,j') \in A$ | **(new)** set of feasible arcs between nodes, used for routing |

## Parameters

| Symbol | Description |
|---|---|
| $Qcap_{v,k}$ | capacity of compartment $k$ in a vehicle of type $v$ |
| $\Delta_{f,k,v}$ | **(new — split out from your $\delta$)** fixed, physical compatibility: 1 if compartment $k$ of vehicle type $v$ is engineered to carry fuel $f$. Your original $\delta$ was described as a "binary variable," but compatibility itself is a fixed property of the truck, not a decision — so it's split into this exogenous parameter, with $\delta$ retained below as the operational decision. |
| $AC_{f,v}$ | alternative/aggregate capacity for fuel $f$ in vehicle type $v$; $AC_{f,v} = \sum_{k} Qcap_{v,k}\cdot \Delta_{f,k,v}$ |
| $Qd_{f,c}$ | quantity of fuel $f$ demanded by customer $c$ |
| $TQd_f$ | total demanded quantity of fuel $f$ across all customers; $TQd_f = \sum_{c} Qd_{f,c}$ (definitional, not an independent input) |
| $EA_j,\ LA_j$ | earliest / latest acceptable arrival time at node $j$ |
| $m_v$ | **(new)** maximum number of physical vehicles available of type $v$ |
| $Cm_v$ | **(new)** maintenance cost per km for vehicle type $v$ |
| $Cf_v$ | **(new)** fuel/running cost per km for vehicle type $v$ |
| $Sp_v$ | **(new)** average operating speed of vehicle type $v$ |
| $Dist_{jj'}$ | **(new)** distance of arc $(j,j')$ |
| $s_j$ | **(new)** service/unloading time at node $j$ |
| $\theta_v$ | **(new)** reload/turnaround time at a terminal for vehicle type $v$ between trips |
| $T^{max}$ | **(new)** maximum permitted duration of a single trip = 5 hours |
| $F_v$ | **(new)** fixed dispatch cost per vehicle of type $v$ used |
| $M$ | **(new)** large constant (big-M) |

## Variables

| Symbol | Description |
|---|---|
| $Ql_{k,f,v,i,g}$ | quantity of fuel $f$ loaded into compartment $k$ of vehicle-instance $i$ of type $v$ on trip $g$ *(extended with $i,g$ so simultaneous trucks of the same type don't collide)* |
| $\delta_{f,k,v,i,g}\in\{0,1\}$ | 1 if compartment $k$ of vehicle-instance $i$ (type $v$) is **actually loaded** with fuel $f$ on trip $g$ *(now a pure operational decision — compatibility lives in $\Delta$)* |
| $N_v$ | number of vehicles of type $v$ used; $N_v = \sum_{i\in I_v} w_{v,i,1}$ |
| $TA_{v,i,g,j}$ | arrival time of vehicle-instance $i$ (type $v$) at node $j$ on trip $g$ *(extended with $i,g$)* |
| $w_{v,i,g}\in\{0,1\}$ | **(new)** 1 if vehicle-instance $i$ of type $v$ is dispatched on trip $g$ |
| $z_{v,i,g,j}\in\{0,1\}$ | **(new)** 1 if vehicle-instance $i$ (type $v$) visits node $j$ on trip $g$ |
| $x_{v,i,g,jj'}\in\{0,1\}$ | **(new)** 1 if vehicle-instance $i$ (type $v$) traverses arc $(j,j')$ on trip $g$ |
| $q_{c,k,f,v,i,g}\geq 0$ | **(new)** quantity of fuel $f$ delivered to customer $c$ from compartment $k$ of vehicle-instance $i$ (type $v$) on trip $g$ — this is what makes split delivery possible |
| $\xi_{v,i,g,t}\in\{0,1\}$ | **(new)** 1 if vehicle-instance $i$ (type $v$) departs from / returns to terminal $t$ on trip $g$ |
| $Dep_{v,i,g},\ Ret_{v,i,g}\geq 0$ | **(new)** departure time from / return time to terminal, vehicle-instance $i$ (type $v$), trip $g$ |
| $u_{v,i,g,c}$ | **(new)** MTZ subtour-elimination auxiliary variable |

---
# Objective Function

$$
\min Z = \sum_{v\in V} F_v\, N_v \;+\; \sum_{v\in V}\sum_{i\in I_v}\sum_{g\in G}\sum_{(j,j')\in A} (Cm_v + Cf_v)\cdot Dist_{jj'}\cdot x_{v,i,g,jj'}
$$

Fixed dispatch cost per vehicle used, plus distance-based running cost — both driven by vehicle type, which is exactly where Presumption 2's heterogeneity (maintenance cost, speed, fuel consumption) shows up.

---
# Constraints

### C1 — Demand Satisfaction (split delivery)
$$
\sum_{v\in V}\sum_{i\in I_v}\sum_{g\in G}\sum_{k} q_{c,k,f,v,i,g} = Qd_{f,c} \qquad \forall c\in C,\ f\in F
$$
Total fuel $f$ delivered to customer $c$, summed across every compartment/vehicle/trip, equals demand — splitting is implicit in the summation.

### C2 — Compartment–Fuel Compatibility
$$
\delta_{f,k,v,i,g} \leq \Delta_{f,k,v} \qquad \forall f,k,v,\ i\in I_v,\ g
$$

### C3 — Single Fuel per Compartment per Trip
$$
\sum_{f\in F} \delta_{f,k,v,i,g} \leq 1 \qquad \forall k,v,i,g
$$

### C4 — Compartment Capacity
$$
Ql_{k,f,v,i,g} \leq Qcap_{v,k}\cdot \delta_{f,k,v,i,g} \qquad \forall k,f,v,i,g
$$

### C5 — Aggregate Vehicle Capacity per Fuel (this is where $AC_{f,v}$ does its job)
$$
\sum_{k} Ql_{k,f,v,i,g} \leq AC_{f,v} \qquad \forall f,v,i,g
$$
C4 caps each compartment individually; this caps the vehicle's *total* carrying capacity for fuel $f$ across all its compatible compartments — useful if there's a reason it's tighter than the simple compartment sum (e.g. a weight-based de-rating for a dense fuel).

### C6 — Load–Delivery Balance
$$
\sum_{c\in C} q_{c,k,f,v,i,g} = Ql_{k,f,v,i,g} \qquad \forall k,f,v,i,g
$$
Whatever's loaded into a compartment gets fully distributed across the customers it serves that trip — the mechanism enabling one compartment to serve several stops.

### C7 — Visit–Delivery Linking
$$
q_{c,k,f,v,i,g} \leq M\cdot z_{v,i,g,c} \qquad \forall c,k,f,v,i,g
$$
No delivery without an actual visit.

### C8 — Flow Conservation at Customer Nodes
$$
\sum_{j'} x_{v,i,g,c,j'} = \sum_{j'} x_{v,i,g,j',c} = z_{v,i,g,c} \qquad \forall c\in C,\ v,i,g
$$

### C9 — Terminal Departure / Same-Terminal Return
$$
\sum_{c\in C} x_{v,i,g,t,c} = \xi_{v,i,g,t} \qquad \forall t\in T,\ v,i,g
$$
$$
\sum_{c\in C} x_{v,i,g,c,t} = \xi_{v,i,g,t} \qquad \forall t\in T,\ v,i,g
$$
$$
\sum_{t\in T} \xi_{v,i,g,t} = w_{v,i,g} \qquad \forall v,i,g
$$
A dispatched trip leaves from exactly one terminal and returns to that same terminal (Presumption 3).

### C10 — Subtour Elimination (MTZ)
$$
u_{v,i,g,c} - u_{v,i,g,c'} + |C|\cdot x_{v,i,g,c,c'} \leq |C|-1 \qquad \forall c\neq c' \in C,\ v,i,g
$$

### C11 — Time Windows
$$
EA_j \leq TA_{v,i,g,j} \leq LA_j \qquad \forall j,\ v,i,g \text{ with } z_{v,i,g,j}=1 \text{ (or } \xi_{v,i,g,j}=1 \text{ if } j \text{ is the terminal)}
$$

### C12 — Arrival-Time Propagation
Vehicle speed is type-specific, so travel time is too:
$$
\tau^v_{jj'} = \dfrac{Dist_{jj'}}{Sp_v}
$$
$$
TA_{v,i,g,j'} \geq TA_{v,i,g,j} + s_j + \tau^v_{jj'} - M(1-x_{v,i,g,jj'}) \qquad \forall (j,j')\in A,\ j,j'\in C,\ v,i,g
$$
$$
TA_{v,i,g,c} \geq Dep_{v,i,g} + \tau^v_{tc} - M(1-x_{v,i,g,tc}) \qquad \forall t\in T,\ c\in C,\ v,i,g
$$
$$
Ret_{v,i,g} \geq TA_{v,i,g,c} + s_c + \tau^v_{ct} - M(1-x_{v,i,g,ct}) \qquad \forall c\in C,\ t\in T,\ v,i,g
$$

### C13 — Trip Duration Cap (5 hours)
$$
Ret_{v,i,g} - Dep_{v,i,g} \leq T^{max}\cdot w_{v,i,g} \qquad \forall v,i,g
$$

### C14 — Multi-Trip Sequencing
$$
w_{v,i,2} \leq w_{v,i,1} \qquad \forall v,i
$$
$$
Dep_{v,i,2} \geq Ret_{v,i,1} + \theta_v - M(1-w_{v,i,2}) \qquad \forall v,i
$$

### C15 — Fleet Size per Type
$$
N_v = \sum_{i\in I_v} w_{v,i,1} \leq m_v \qquad \forall v\in V
$$

### C16 — Variable Domains
$$
x_{v,i,g,jj'},\ z_{v,i,g,j},\ w_{v,i,g},\ \delta_{f,k,v,i,g},\ \xi_{v,i,g,t} \in \{0,1\}
$$
$$
Ql_{k,f,v,i,g},\ q_{c,k,f,v,i,g},\ TA_{v,i,g,j},\ Dep_{v,i,g},\ Ret_{v,i,g} \geq 0
$$

---
# Modeling Notes

1. **Type vs. instance.** Your $V$/$N_v$ pairing only makes sense if $N_v$ counts *distinct trucks of type v*, not arc-traversals — which means routing variables can't be indexed by type alone (two trucks of the same type need different routes simultaneously). That's the entire reason $I_v$ and $i$ exist here. If in practice you never run more than one vehicle of a given type at once, $I_v$ collapses to a single element and most of the added complexity disappears.
2. **$\delta$ split into $\Delta$ (parameter) and $\delta$ (variable).** Your draft listed $\delta_{f,k,v}$ as a single binary variable doing two jobs — "is this compartment built for this fuel" and "is this compartment currently filled with this fuel." Those need to be separate: one is a fixed truck spec, the other changes trip to trip.
3. **Terminal return assumption.** C9 assumes a vehicle can choose which terminal to operate from each trip, but must return to that same one. If each vehicle actually has one fixed home terminal, you can drop $\xi_{v,i,g,t}$ entirely and just hard-code which terminal each $(v,i)$ belongs to — simpler model, fewer variables.
4. **$TQd_f$** is shown as a definition, not an independent constraint — it's just $\sum_c Qd_{f,c}$, useful for reporting/sanity-checks rather than optimization.

Flag any of these if your actual operating policy differs — they're the assumptions filling gaps your draft hadn't specified yet.
