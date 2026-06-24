# Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)
## Fuel Distribution Model

---

## Key Features

| Feature | Detail |
|---|---|
| Heterogeneous fleet | Vehicles differ in compartment count, per-compartment capacities, cost rates (maintenance, fuel-consumption, driver), and arc-specific speed |
| Multi-product, multi-compartment | Each compartment carries exactly one fuel type per day; mid-day top-up of the same fuel is allowed |
| Flexible endpoints | Start and end nodes are independently chosen from depots or eligible terminals; they need not match |
| Mid-route reloads | Any terminal may be visited multiple times within a day (visit-slots capped at $P$ per vehicle per terminal) |
| Split delivery | A customer's demand for a fuel may be split across multiple vehicles and/or compartments |
| Visit vs. load distinction | Binary $z_{v,i,(t,p)}$ records whether a terminal slot was visited; binary $\ell_{v,i,(t,p)}$ separately records whether fuel was actually loaded at that slot — a vehicle parked overnight at a terminal visits the slot but may not load |
| Route-end no-loading | A vehicle whose route ends at a terminal cannot load fuel it will not deliver — enforced via the $\ell$ variable gating all loading quantities |
| Terminal eligibility | Only terminals with $E_t = 1$ may serve as route start or end; any terminal may be used for mid-route reloads |
| Compartment fuel-identity lock | Once a compartment is assigned a fuel for the day ($\delta_{f,k,v,i} = 1$), it cannot switch fuels; it may only be topped up with more of the same fuel |
| Arrival time windows | Customer and terminal arrival windows are enforced via big-$M$ constraints; windows apply to every visited node including depots at route boundaries |
| Arrival-time propagation | A consistent clock is maintained along the entire route, with travel time varying by vehicle type and arc, and dwell time varying by quantity handled |
| Maximum daily duration | Total elapsed time from route start to route end is capped at $T^{max} = 10$ hours |
| MTZ subtour elimination | Miller–Tucker–Zemlin constraints applied over the visit-slot replicated node set $\mathcal{N} = C \cup \hat{T}$, using big-$M$ |
| No mid-route depot returns | Depots are strictly route boundaries — a vehicle cannot return to a depot as an intermediate stop |
| Compartment running-load tracking | Continuous variable $Lc_{k,f,v,i,j}$ tracks fuel level in each compartment at every node, enabling partial deliveries and top-ups to be represented exactly |
| Driver cost on travel time only | Driver pay covers moving time only (not dwell time), matching a typical per-hour-driven pay structure; this is a deliberate modeling choice that can be altered |
| Terminal-stay fixed cost | An overnight stop at a terminal incurs a fixed fee $FT_t$; ending at a depot does not |
| Fleet size enforcement | Number of vehicles dispatched per type cannot exceed the available physical fleet $m_v$ |
| Linear (MILP) formulation | All conditional logic is linearised via big-$M$; no product of two decision variables appears anywhere in the model |

---

## Overview

The **Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)** is an operations research model used to optimize the routing of a fleet of vehicles with varying capacities and characteristics for fuel delivery. This document lays out the key features, notation, objective, and constraints that define the model.

---

## Vehicle Types

Each vehicle type $v$ carries its own: compartment set $K_v$ (so the *number* of compartments differs by type), per-compartment capacities $Qcap_{v,k}$, maintenance cost $Cm_v$, fuel-consumption cost $Cf_v$, driver cost rate $Cd_v$, speed $Sp_{v,j,j'}$ (per arc), and fleet size $m_v$.

---

# Notation

## Indices and Sets

| Symbol | Description |
|---|---|
| $v \in V$ | set of vehicle types |
| $k \in K_v$ | set of compartments belonging to vehicle type $v$ — count and per-compartment specs vary by type |
| $f \in F$ | set of fuel types |
| $c \in C$ | set of customers |
| $t \in T$ | set of terminals |
| $d \in D$ | set of depots |
| $p \in \{1,\dots,P\}$ | visit-slot index — the $p$-th time a given terminal is used by a given vehicle that day |
| $\hat{T} = \{(t,p) : t \in T,\ p = 1,\dots,P\}$ | the set of terminal visit-slot nodes |
| $\mathcal{N} := C \cup \hat{T}$ | customer and terminal-visit-slot nodes — the nodes that get a "visited" indicator $z$ and a subtour-elimination value $u$ |
| $j,\ j'$ | generic node indices, ranging over $\mathcal{N} \cup D$ unless stated otherwise |
| $i \in I_v = \{1,\dots,m_v\}$ | physical vehicle instances of type $v$ |
| $(j,j') \in A,\ A \subseteq (\mathcal{N} \cup D) \times (\mathcal{N} \cup D)$ | feasible arcs. Depot-to-depot, depot-to-customer, and terminal-to-depot arcs are excluded. A vehicle cannot deliver before loading; it never travels directly between two depots; and once it departs a terminal it continues to a customer or another terminal — returning to a depot mid-route is infeasible since a depot is only ever a route's start or end, never an intermediate stop. |

> $Dist_{(t,p),(t,p')} = 0$ for any $p,p'$ of the same terminal — different slots of one terminal are the same physical place.

---

## Parameters

| Symbol | Description |
|---|---|
| $Qcap_{v,k}$ | capacity of compartment $k \in K_v$ in a vehicle of type $v$ |
| $\Delta_{f,k,v}$ | 1 if compartment $k \in K_v$ of vehicle type $v$ is engineered to carry fuel $f$ |
| $AC_{f,v}$ | aggregate capacity for fuel $f$ in vehicle type $v$; $AC_{f,v} = \sum_{k \in K_v} Qcap_{v,k} \cdot \Delta_{f,k,v}$ |
| $Qd_{f,c}$ | quantity of fuel $f$ demanded by customer $c$ |
| $TQd_f$ | total demanded quantity of fuel $f$; $TQd_f = \sum_c Qd_{f,c}$ (a reporting definition, not an independent input) |
| $EA_j,\ LA_j$ | earliest / latest acceptable arrival time at node $j$ |
| $m_v$ | maximum number of physical vehicles available of type $v$ |
| $Cm_v$ | maintenance cost per km for vehicle type $v$ |
| $Cf_v$ | fuel-consumption cost per km for vehicle type $v$ (driving fuel, not delivered fuel) |
| $Cd_v$ | driver cost per unit travel time for vehicle type $v$ |
| $Sp_{v,j,j'}$ | average speed of vehicle type $v$ on arc $(j,j')$ |
| $Dist_{j,j'}$ | distance of arc $(j,j')$ |
| $\beta$ | loading-rate constant — time per unit quantity loaded at a terminal |
| $\gamma$ | unloading-rate constant — time per unit quantity delivered to a customer |
| $T^{max}$ | maximum permitted duration of a vehicle's entire daily route, 10 hours |
| $E_t \in \{0,1\}$ | 1 if terminal $t$ may be used as a route's start or end |
| $TF_{f,t} \in \{0,1\}$ | 1 if terminal $t$ stocks fuel $f$ |
| $Pr_{f,t} \geq 0$ | price of fuel $f$ at terminal $t$ |
| $FT_t \geq 0$ | fixed cost incurred if a route ends at terminal $t$ |
| $P$ | cap on visit-slots per terminal per vehicle per day — a modeling device (e.g. $P = |C|$), not a real operating limit |
| $M$ | large constant (big-$M$) |

---

## Variables

| Symbol | Description |
|---|---|
| $Ql_{k,f,v,i,(t,p)}$ | quantity of fuel $f$ loaded into compartment $k$ of vehicle-instance $i$ (type $v$) at visit-slot $(t,p)$ |
| $Lc_{k,f,v,i,j} \geq 0$ | running quantity of fuel $f$ in compartment $k$ of vehicle-instance $i$ (type $v$) immediately after node $j$ |
| $\delta_{f,k,v,i} \in \{0,1\}$ | 1 if compartment $k$ of vehicle-instance $i$ (type $v$) is assigned fuel $f$ for the day |
| $\ell_{v,i,(t,p)} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) actually loads fuel at visit-slot $(t,p)$; 0 when the slot is visited but no fuel is taken on (e.g. the route ends there) |
| $N_v$ | number of vehicles of type $v$ used; $N_v = \sum_{i \in I_v} w_{v,i}$ |
| $TA_{v,i,j}$ | arrival time of vehicle-instance $i$ (type $v$) at node $j \in \mathcal{N} \cup D$ |
| $w_{v,i} \in \{0,1\}$ | 1 if vehicle-instance $i$ of type $v$ is dispatched on its (single) daily route |
| $z_{v,i,j} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) visits node $j \in \mathcal{N}$ |
| $x_{v,i,j,j'} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) traverses arc $(j,j') \in A$ |
| $q_{c,k,f,v,i} \geq 0$ | quantity of fuel $f$ delivered to customer $c$ from compartment $k$ of vehicle-instance $i$ (type $v$) |
| $\xi^{o}_{v,i,t} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) starts its route at terminal $t$ — necessarily at slot $(t,1)$ |
| $\xi^{o,D}_{v,i,d} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) starts its route at depot $d$ |
| $\xi^{d}_{v,i,(t,p)} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) ends its route at visit-slot $(t,p)$ |
| $\xi^{d,D}_{v,i,d} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) ends its route at depot $d$ |
| $u_{v,i,j}$ | MTZ subtour-elimination value, defined for $j \in \mathcal{N}$ |
| $s_{v,i,(t,p)} \geq 0$ | loading time vehicle-instance $i$ (type $v$) spends at visit-slot $(t,p)$; $s_{v,i,(t,p)} = \beta \cdot \sum_{k \in K_v} \sum_f Ql_{k,f,v,i,(t,p)}$ |
| $s_{v,i,c} \geq 0$ | unloading time vehicle-instance $i$ (type $v$) spends at customer $c$; $s_{v,i,c} = \gamma \cdot \sum_{k \in K_v} \sum_f q_{c,k,f,v,i}$ |

> Shorthand: $\xi^{d}_{v,i,t} := \sum_{p=1}^{P} \xi^{d}_{v,i,(t,p)}$ — whether terminal $t$ is the route's end at all, regardless of which slot.

---

# Objective Function

$$
\min Z = \underbrace{\sum_{v \in V} \sum_{i \in I_v} \sum_{(j,j') \in A} (Cm_v + Cf_v) \cdot Dist_{j,j'} \cdot x_{v,i,j,j'}}_{\text{driving cost}}
+ \underbrace{\sum_{v \in V} \sum_{i \in I_v} \sum_{t \in T} \sum_{p=1}^{P} \sum_{k \in K_v} \sum_{f \in F} Pr_{f,t} \cdot Ql_{k,f,v,i,(t,p)}}_{\text{fuel purchase cost}}
+ \underbrace{\sum_{v \in V} \sum_{i \in I_v} Cd_v \cdot \sum_{(j,j') \in A} \tau^v_{j,j'} \cdot x_{v,i,j,j'}}_{\text{driver cost}}
+ \underbrace{\sum_{v \in V} \sum_{i \in I_v} \sum_{t \in T} FT_t \cdot \xi^{d}_{v,i,t}}_{\text{terminal-stay fixed cost}}
$$

Total cost is driving cost, plus the fuel actually purchased, plus the driver's time on the move, plus a fixed cost only when a route ends at a terminal rather than back at a depot. There is no separate per-vehicle dispatch fee — running a vehicle at all is already captured by the other three terms.

---

# Constraints

### C1 — Demand Satisfaction

$$
\sum_{v \in V} \sum_{i \in I_v} \sum_{k \in K_v} q_{c,k,f,v,i} = Qd_{f,c} \qquad \forall\ c \in C,\ \forall\ f \in F
$$

The fuel a customer receives, added up across every vehicle, every instance of that vehicle, and every compartment that visits them, equals exactly what they ordered.

**Purpose:** guarantees every customer's order is fully met, without requiring any single vehicle or compartment to cover the whole thing — this is what makes split delivery possible.

---

### C2 — Compartment–Fuel Compatibility

$$
\delta_{f,k,v,i} \leq \Delta_{f,k,v} \qquad \forall\ f \in F,\ k \in K_v,\ v \in V,\ i \in I_v
$$

A compartment can only be assigned a fuel it was physically built to carry.

**Purpose:** stops the model from loading a fuel into a compartment that is not engineered for it.

---

### C3 — Single Fuel per Compartment per Day

$$
\sum_{f \in F} \delta_{f,k,v,i} \leq 1 \qquad \forall\ k \in K_v,\ v \in V,\ i \in I_v
$$

Each compartment is assigned at most one fuel for the whole day.

**Purpose:** enforces the no-mixing rule — a compartment can be topped up with more of its fuel, but never switched to a different one mid-day.

---

### C4 — Cumulative Load Propagation

The binary variable $\ell_{v,i,(t,p)} \in \{0,1\}$ indicates whether vehicle-instance $i$ (type $v$) actually loads fuel at visit-slot $(t,p)$. A vehicle that ends its route at a terminal visits that slot ($z_{v,i,(t,p)} = 1$) but must not load fuel it will not deliver ($\ell_{v,i,(t,p)} = 0$). This is enforced by:

$$
\ell_{v,i,(t,p)} \leq z_{v,i,(t,p)} \qquad \forall\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

$$
\ell_{v,i,(t,p)} \leq 1 - \xi^{d}_{v,i,(t,p)} \qquad \forall\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

The first constraint ensures loading can only occur at visited slots. The second prevents loading at a slot that is the route's final destination — if a vehicle parks overnight at a terminal it may not take on fuel it will never deliver.

The load propagation then uses $\ell$ to gate all loading quantities:

$$
Ql_{k,f,v,i,(t,p)} \leq M \cdot \ell_{v,i,(t,p)} \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

The cumulative running load evolves along the route as:

$$
Lc_{k,f,v,i,j'} \geq Lc_{k,f,v,i,j} + \Theta_{k,f,v,i,j'} - M\bigl(1 - x_{v,i,j,j'}\bigr) \qquad \forall\ (j,j') \in A,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

where

$$
\Theta_{k,f,v,i,j'} =
\begin{cases}
Ql_{k,f,v,i,(t,p)} & j' = (t,p) \in \hat{T} \\
-q_{c,k,f,v,i} & j' = c \in C \\
0 & j' = d \in D
\end{cases}
$$

and the depot anchor:

$$
Lc_{k,f,v,i,d} = 0 \qquad \forall\ d \in D,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

As the vehicle moves along its route, a compartment's running fuel level rises by whatever is loaded at a terminal stop (only when $\ell = 1$), falls by whatever is delivered at a customer stop, and is fixed at exactly zero at any depot.

**Purpose:** tracks what is physically in each compartment at every point in the route, and prevents a vehicle from loading fuel at a terminal slot that serves as the overnight end of its route.

---

### C5 — Per-Compartment Running Capacity

$$
0 \leq Lc_{k,f,v,i,j} \leq Qcap_{v,k} \cdot \delta_{f,k,v,i} \qquad \forall\ j \in \mathcal{N},\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

At every point along the route, a compartment's fuel level stays between empty and its physical capacity.

**Purpose:** prevents overfilling a compartment; the lower bound automatically prevents delivering more than what is currently inside it.

---

### C6 — Aggregate Vehicle Running Capacity per Fuel

$$
\sum_{k \in K_v} Lc_{k,f,v,i,j} \leq AC_{f,v} \qquad \forall\ j \in \mathcal{N},\ f \in F,\ v \in V,\ i \in I_v
$$

Across all of a vehicle's compatible compartments combined, the running total of a given fuel never exceeds the vehicle's aggregate capacity for it.

**Purpose:** a secondary cap on top of the per-compartment one, useful only if the vehicle's overall carrying capacity for a fuel is set independently of the simple compartment sum (e.g. a weight limit).

---

### C7 — Load–Delivery Balance

$$
\sum_{t \in T} \sum_{p=1}^{P} Ql_{k,f,v,i,(t,p)} = \sum_{c \in C} q_{c,k,f,v,i} \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

Total loaded across every terminal stop equals total delivered across every customer stop, for that compartment, for the day.

**Purpose:** ensures everything loaded into a compartment across the whole day is fully accounted for by deliveries.

---

### C8 — Visit–Delivery Linking

$$
q_{c,k,f,v,i} \leq M \cdot z_{v,i,c} \qquad \forall\ c \in C,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

A customer can only receive fuel if the vehicle actually visits them.

**Purpose:** blocks deliveries to customers who are not on the route.

---

### C9 — Flow Conservation: Customers

For $j \in C$ (customer nodes), $j' \in C \cup \hat{T}$ ($j' \neq j$):

$$
\sum_{j' \in C \cup \hat{T}} x_{v,i,j,j'} = \sum_{j' \in C \cup \hat{T}} x_{v,i,j',j} = z_{v,i,j} \qquad \forall\ j \in C,\ v \in V,\ i \in I_v
$$

> $j' \in C \cup \hat{T}$: a vehicle leaving or arriving at a customer comes from/goes to another customer or a terminal visit-slot only — customers are never endpoints and the route cannot proceed to a depot from a customer mid-route.

A visited customer has exactly one arc in and one arc out; an unvisited one has none.

**Purpose:** makes sure the route passes cleanly through each customer it serves, with no dangling or duplicated stops.

---

### C10 — Flow Conservation: Terminal Slots and Depots

**Terminal inflow** — for $(t,p) \in \hat{T}$, $j' \in C \cup \hat{T} \cup D$ (a terminal slot may be reached from a customer, another terminal slot, or a depot at route start):

$$
\sum_{j' \in C \cup \hat{T} \cup D} x_{v,i,j',(t,p)} = z_{v,i,(t,p)} - \xi^{o}_{v,i,t} \cdot [p{=}1] \qquad \forall\ (t,p) \in \hat{T},\ v \in V,\ i \in I_v
$$

**Terminal outflow** — for $(t,p) \in \hat{T}$, $j' \in C \cup \hat{T}$ only (a vehicle leaving a terminal never goes directly to a depot mid-route):

$$
\sum_{j' \in C \cup \hat{T}} x_{v,i,(t,p),j'} = z_{v,i,(t,p)} - \xi^{d}_{v,i,(t,p)} \qquad \forall\ (t,p) \in \hat{T},\ v \in V,\ i \in I_v
$$

**Depot outflow** — for $d \in D$, $j' \in \hat{T}$ only (a depot can only be a route's start, and the first move must be to a terminal to load before any delivery):

$$
\sum_{j' \in \hat{T}} x_{v,i,d,j'} = \xi^{o,D}_{v,i,d} \qquad \forall\ d \in D,\ v \in V,\ i \in I_v
$$

**Depot inflow** — for $d \in D$, $j' \in C \cup \hat{T}$ (a depot may only be reached from a customer or a terminal, never from another depot):

$$
\sum_{j' \in C \cup \hat{T}} x_{v,i,j',d} = \xi^{d,D}_{v,i,d} \qquad \forall\ d \in D,\ v \in V,\ i \in I_v
$$

**Single origin and single destination per dispatched vehicle:**

$$
\sum_{d \in D} \xi^{o,D}_{v,i,d} + \sum_{t \in T} \xi^{o}_{v,i,t} = w_{v,i} \qquad \forall\ v \in V,\ i \in I_v
$$

$$
\sum_{d \in D} \xi^{d,D}_{v,i,d} + \sum_{t \in T} \xi^{d}_{v,i,t} = w_{v,i} \qquad \forall\ v \in V,\ i \in I_v
$$

> $[p{=}1]$: the $\xi^o$ term applies only at slot $p = 1$ — slot 1 is the only slot that can ever be a route's start.

Every terminal visit-slot and every depot use has its arcs in and out balanced against whether it is used at all — except for whichever single node is the day's actual start (no arc in) and whichever single node is the day's actual end (no arc out). Exactly one start and one end is chosen per dispatched vehicle, and each can independently be a depot or a terminal.

**Purpose:** builds the route as one continuous path with a well-defined start and end, while letting that start and end be a depot or a terminal, and letting them differ from each other. The explicit arc-domain restrictions ($j' \in C \cup \hat{T}$ for terminal outflow; $j' \in \hat{T}$ for depot outflow; $j' \in C \cup \hat{T}$ for depot inflow) make the structural infeasibilities — no terminal-to-depot mid-route arcs, no depot-to-depot arcs, no depot-to-customer arcs — explicit in the constraint domains rather than relying solely on $A$.

---

### C11 — Terminal Eligibility

$$
\xi^{o}_{v,i,t} \leq E_t, \qquad \xi^{d}_{v,i,t} \leq E_t \qquad \forall\ t \in T,\ v \in V,\ i \in I_v
$$

A terminal can only be used as a route's start or end if it is flagged eligible.

**Purpose:** keeps certain terminals reserved for mid-route reloads only, never as a place a route begins or ends.

---

### C12 — Terminal Slot Ordering

$$
z_{v,i,(t,p)} \leq z_{v,i,(t,p-1)} \qquad \forall\ t \in T,\ p = 2,\dots,P,\ v \in V,\ i \in I_v
$$

A terminal's 2nd, 3rd, etc. visit-slot cannot be used unless the one before it was used too.

**Purpose:** stops the solver from representing one real visit in many meaninglessly different ways.

---

### C13 — Loading Gating

$$
Ql_{k,f,v,i,(t,p)} \leq M \cdot TF_{f,t}, \qquad Ql_{k,f,v,i,(t,p)} \leq M \cdot z_{v,i,(t,p)}, \qquad Ql_{k,f,v,i,(t,p)} \leq Qcap_{v,k} \cdot \delta_{f,k,v,i}
$$

$$
\forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

A vehicle cannot load a fuel the terminal does not stock, cannot load at a visit-slot it is not actually using, and cannot load a fuel into a compartment not assigned to carry it that day.

**Purpose:** three independent checks that keep every loading event physically realistic. Note that the $\ell$-based gate in C4 ($Ql \leq M \cdot \ell$) further prevents loading at the terminal slot that ends the route; together with the visit-gating here, all three conditions must hold simultaneously.

---

### C14 — Subtour Elimination (MTZ)

$$
u_{v,i,j} - u_{v,i,j'} + M \cdot x_{v,i,j,j'} \leq M - 1 \qquad \forall\ j \neq j' \in \mathcal{N},\ v \in V,\ i \in I_v
$$

Every customer and terminal visit-slot gets a position value; if an arc is used, the destination's value must be consistent with coming after the origin's. Depots are excluded — they are always a route's boundary, never part of an internal loop.

**Purpose:** prevents the solver from forming a closed loop of customers or terminal slots that is disconnected from the rest of the route.

---

### C15 — Time Windows

For any visited node $j \in \mathcal{N} \cup D$:

$$
EA_j - M(1 - z_{v,i,j}) \leq TA_{v,i,j} \leq LA_j + M(1 - z_{v,i,j}) \qquad \forall\ j \in \mathcal{N},\ v \in V,\ i \in I_v
$$

$$
EA_d \leq TA_{v,i,d} \leq LA_d \qquad \forall\ d \in D,\ v \in V,\ i \in I_v \quad \text{(active only when } \xi^{o,D}_{v,i,d} = 1 \text{ or } \xi^{d,D}_{v,i,d} = 1\text{)}
$$

> $j \in \mathcal{N} = C \cup \hat{T}$: covers customers and terminal visit-slots. Depots have their window enforced directly since every dispatched vehicle touches exactly one origin depot or one destination depot (or both). The big-$M$ relaxation for $j \in \mathcal{N}$ deactivates the window for unvisited nodes.

**Coverage check — start and end of route:**

- **Depot to first terminal** $(d \to (t,1))$: the depot window constrains $TA_{v,i,d}$ (departure point). The terminal window constrains $TA_{v,i,(t,1)}$. Arrival-time propagation (C16) links the two via travel time $\tau^v_{d,(t,1)}$, so the departure from the depot and the arrival at the first terminal are both window-checked.

- **Last node to terminal end-of-route** $(c \to (t,p))$ or $((t,p) \to (t',p'))$: arrival at the terminal end-slot $(t,p)$ is governed by the same terminal-slot window constraint above ($j = (t,p) \in \hat{T}$), so the window is enforced even when that slot is the route's final destination.

- **Customer to depot** $(c \to d)$: arrival at the destination depot is checked by the depot row. C16 links $TA_{v,i,d} \geq TA_{v,i,c} + s_{v,i,c} + \tau^v_{c,d}$, so travel from the last customer into the depot is clock-consistent and window-constrained.

Every leg of the trip — depot-to-first-terminal, customer-to-terminal, terminal-to-terminal, customer-to-depot — is therefore covered.

**Purpose:** keeps the route consistent with customer availability and terminal operating hours across the entire journey from origin to destination.

---

### C16 — Arrival-Time Propagation

Travel time on arc $(j,j')$ for vehicle type $v$:

$$
\tau^v_{j,j'} = \frac{Dist_{j,j'}}{Sp_{v,j,j'}} \qquad \forall\ (j,j') \in A,\ v \in V
$$

Dwell time at each node type:

$$
s_{v,i,c} = \gamma \cdot \sum_{k \in K_v} \sum_{f \in F} q_{c,k,f,v,i} \qquad \forall\ c \in C,\ v \in V,\ i \in I_v
$$

$$
s_{v,i,(t,p)} = \beta \cdot \sum_{k \in K_v} \sum_{f \in F} Ql_{k,f,v,i,(t,p)} \qquad \forall\ (t,p) \in \hat{T},\ v \in V,\ i \in I_v
$$

Propagation along the route for any arc $(j,j') \in A$, $j,j' \in \mathcal{N} \cup D$:

$$
TA_{v,i,j'} \geq TA_{v,i,j} + \bar{s}_{v,i,j} + \tau^v_{j,j'} - M\bigl(1 - x_{v,i,j,j'}\bigr)
$$

where

$$
\bar{s}_{v,i,j} =
\begin{cases}
s_{v,i,c} & j = c \in C \\
s_{v,i,(t,p)} & j = (t,p) \in \hat{T} \\
0 & j = d \in D
\end{cases}
$$

Arrival time at the next stop is whatever time the vehicle arrived at the current stop, plus however long it spends there, plus the travel time to the next node. Dwell time at a node depends on how much is loaded or delivered there, not a flat number.

**Purpose:** propagates a consistent clock along the whole route, accounting for speed that can vary by arc and vehicle type, and dwell time that can vary by quantity handled.

---

### C17 — Maximum Daily Route Duration (10 hours)

$$
TA_{v,i,n'} - TA_{v,i,n} - \bar{s}_{v,i,n} \leq T^{max} + M\bigl(2 - \Xi^{o}_{v,i,n} - \Xi^{d}_{v,i,n'}\bigr)
$$

$$
\forall\ n \in D \cup \{(t,1) : t \in T\},\ n' \in D \cup \hat{T},\ v \in V,\ i \in I_v,\quad T^{max} = 10 \text{ hours}
$$

where

$$
\Xi^{o}_{v,i,n} =
\begin{cases}
\xi^{o,D}_{v,i,n} & n \in D \\
\xi^{o}_{v,i,t} & n = (t,1)
\end{cases}
\qquad
\Xi^{d}_{v,i,n'} =
\begin{cases}
\xi^{d,D}_{v,i,n'} & n' \in D \\
\xi^{d}_{v,i,(t,p)} & n' = (t,p)
\end{cases}
$$

The gap between the moment the vehicle leaves its start and the moment it arrives at its end cannot exceed 10 hours.

**Purpose:** caps how long a vehicle's day can run, regardless of whether that day started and ended at a depot, a terminal, or one of each.

---

### C18 — Fleet Size per Type

$$
N_v = \sum_{i \in I_v} w_{v,i} \leq m_v \qquad \forall\ v \in V
$$

The number of vehicles of a type actually used cannot exceed how many of that type exist.

**Purpose:** caps fleet usage at what is physically available.

---

### C19 — Variable Domains

$$
x_{v,i,j,j'},\ z_{v,i,j},\ w_{v,i},\ \delta_{f,k,v,i},\ \ell_{v,i,(t,p)},\ \xi^{o}_{v,i,t},\ \xi^{o,D}_{v,i,d},\ \xi^{d}_{v,i,(t,p)},\ \xi^{d,D}_{v,i,d} \in \{0,1\}
$$

$$
Ql_{k,f,v,i,(t,p)},\ Lc_{k,f,v,i,j},\ s_{v,i,c},\ s_{v,i,(t,p)},\ q_{c,k,f,v,i},\ TA_{v,i,j} \geq 0
$$

States which variables are binary decisions and which are non-negative quantities.

**Purpose:** keeps the model a clean Mixed-Integer Linear Program.

---

# Linearity Check

The model is a **Mixed-Integer Linear Program**. Every conditional relationship — load propagation, time propagation, the duration cap — is written as a big-$M$ inequality rather than a direct product of two decision variables, so nothing here is quadratic. The two dwell-time definitions ($s_{v,i,c} = \gamma \sum q$, $s_{v,i,(t,p)} = \beta \sum Ql$) and the driver-cost term ($Cd_v \cdot \tau^v_{j,j'} \cdot x_{v,i,j,j'}$) are all a fixed constant multiplying a single decision variable, which stays linear. The new $\ell_{v,i,(t,p)}$ variable is binary and its linking constraints are linear. The model deliberately avoids ever multiplying two decision variables together.

Things that would **not** stay linear if added later: a running cost that depends on both distance *and* carried weight (a true product of two decision variables), or a penalty whose priority weight is itself decision-dependent.

---

# Modeling Notes

1. **Type vs. instance.** $N_v$ only makes sense if it counts *distinct trucks of type $v$*; routing variables cannot be indexed by type alone, since two trucks of the same type need independent routes. Hence $I_v$ and $i$.

2. **$\delta$ vs. $\Delta$.** "Is this compartment built for this fuel" ($\Delta$, fixed) and "is this compartment currently assigned this fuel" ($\delta$, a daily decision) are different things, kept as separate symbols.

3. **Why $Lc$ propagation only needs one direction.** A lower-bound-only chain on $Lc$, combined with the upper bound in C5, pins $Lc$ to its true value at any optimum — inflating $Lc$ only makes C5 harder to satisfy, never easier, so there is no incentive to misreport it.

4. **Why $Lc = 0$ at depots is a flat equality, not big-$M$ gated.** Unlike a terminal slot (which only sometimes is the route's start), a depot never carries fuel under any circumstance, so its $Lc$ is fixed at zero outright rather than conditioned on $\xi^{o,D}$.

5. **Origin is always slot 1; destination is tracked per-slot.** Slot 1 is the only slot that could ever have nothing before it (the slot-ordering rule means slot 2 always requires slot 1 to have already happened). The destination has no such restriction, since a route can legitimately end on a terminal's 2nd, 3rd, etc. visit.

6. **Why the duration cap (C17) does not need separate $Dep$/$Ret$ variables.** Referencing $TA$ directly, gated by whichever origin/destination indicator applies, avoids ever creating a $\xi \cdot TA$ product.

7. **"Exactly one origin/destination" already rules out using two terminals at once.** Because $\sum(\text{all origin options}) = w_{v,i}$ and $w_{v,i} \in \{0,1\}$, the terminal-only portion can never exceed 1 on its own. Same logic applies to the destination side.

8. **The aggregate demand identity is implied, not independent.** C1 already states the per-customer version; summing over every customer produces the aggregate version automatically.

9. **Slot-visit-order matching slot number is not required for correctness, only optionally for solver speed.** $Lc$ and $TA$ both propagate along actual arcs, not slot labels, so nothing breaks if the solver's internal numbering of "1st vs. 2nd visit" does not match real-world intuition. Adding $u_{v,i,(t,p)} > u_{v,i,(t,p-1)}$ as an extra symmetry-breaking constraint is optional, not required.

10. **Driver cost uses pure travel time, not total elapsed time.** $Cd_v$ is multiplied by $\sum \tau^v_{j,j'} x_{v,i,j,j'}$ — moving time only, excluding dwell time. This is a deliberate reading of "time travelled"; if driver pay should cover the whole day including dwell time, the driver-cost term would need to use the same elapsed-time expression as C17.

11. **C6 redundancy.** As currently defined, $AC_{f,v}$ is just the sum of the per-compartment caps already in C5, so C6 adds nothing unless $AC_{f,v}$ is set independently tighter (e.g. a weight-based de-rating).

12. **$TQd_f$** remains a definition for reporting/sanity-checks, not an independent constraint.

13. **Visit vs. load distinction ($z$ vs. $\ell$).** $z_{v,i,(t,p)} = 1$ means the vehicle is physically present at the terminal slot. $\ell_{v,i,(t,p)} = 1$ is the further condition that it actually takes on fuel there. The two can differ only when the slot is the route's overnight end — in all other cases the solver will set $\ell = z$ at optimum because loading is free (or beneficial) and no constraint forces $\ell = 0$ at non-terminal slots.

14. **Arc-domain explicitness in C9 and C10.** Writing $j' \in C \cup \hat{T}$ (customer flow), $j' \in C \cup \hat{T} \cup D$ (terminal inflow), $j' \in C \cup \hat{T}$ (terminal outflow), $j' \in \hat{T}$ (depot outflow), $j' \in C \cup \hat{T}$ (depot inflow) makes the structural infeasibilities visible in the constraint index sets and eliminates the need for those arc types to be absent from $A$ for correctness — though keeping them absent from $A$ as well speeds up the solver by reducing variable count.

---

# Not Yet Modeled

1. **Fuel-specific / safety: cross-contamination and compartment-adjacency rules.** Tanker layouts that forbid certain fuel pairs in adjacent compartments, or require a fixed unloading sequence by valve position, are not modeled. To be added only after the approach is confirmed.
