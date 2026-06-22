# Vehicle-Routing-Problem
# Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)
## Fuel Distribution Model
---
## Overview
The **Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)** is an operations research model used to optimize the routing of a fleet of vehicles with varying capacities and characteristics for fuel delivery. This document lays out the presumptions, notation, objective, and constraints that define the model.

---
## Presumptions

1. **Terminal Stock** — A terminal maintains sufficient stock only of the fuels it actually carries. Not every terminal stocks every fuel; which fuels a terminal carries is fixed and given by $TF_{f,t}$.
2. **Vehicle Heterogeneity** — Vehicles are grouped into types. Within a single type, maintenance cost, fuel-consumption cost, driver cost rate, average speed (which can also vary by which arc is being driven), and number of compartments are identical across every vehicle of that type. These attributes are exactly what differs from one vehicle type to another.
3. **Route Origin — Depot or Terminal** — A vehicle's day normally starts at a depot. If the vehicle ended the previous day's route at a terminal instead of returning to depot, it may start the new day from that terminal instead.
4. **Route Destination — Flexible, Terminal or Depot** — A route ends at a terminal or back at a depot, and the end doesn't have to be the same location as the start. When a terminal is chosen as either endpoint, it must be one flagged eligible ($E_t=1$). Which endpoint is used — and which eligible terminal, if any — is whatever minimizes total cost (driving distance, fuel purchased, driver time, and, if ending at a terminal, that terminal's fixed stay cost).
5. **Mid-Route Reloads** — A vehicle may revisit terminals any number of times (up to a generous cap $P$) interleaved freely with customer stops, in any order. Terminal eligibility only restricts which terminals can be a route's start or end — any terminal can be used for a mid-route reload regardless of $E_t$.
6. **Split Delivery** — A customer's demand for a fuel may be met by more than one compartment or vehicle, provided every contributing compartment is fuel-compatible. This works both ways: one compartment may serve several customers, and one customer's order may be filled by several different compartments, possibly on different vehicles.
7. **Compartment Fuel-Identity Lock, With Top-Up** — A compartment's assigned fuel is fixed for the entire day; it cannot switch fuels mid-day. It can, however, be topped up — a later terminal stop may add more of that same fuel to a partially-delivered compartment.
8. **Customers** — A customer is never revisited by the same vehicle on the same day, and never serves as a route's start or end. Only depots and terminals can be route endpoints.
9. **Depot** — A depot never stocks fuel and is never revisited mid-route; a vehicle uses a depot at most once as its start and at most once as its end, per day. Using a depot, at either end, carries no fixed cost. Ending a route at a terminal instead does carry one ($FT_t$).

---
## Vehicle Types
Each vehicle type $v$ carries its own: compartment set $K_v$ (so the *number* of compartments differs by type), per-compartment capacities $Qcap_{v,k}$, maintenance cost $Cm_v$, fuel-consumption cost $Cf_v$, driver cost rate $Cd_v$, speed $Sp_{v,jj'}$ (per arc), and fleet size $m_v$.

---
# Notation

## Indices and Sets

| Symbol | Description |
|---|---|
| $v_1,\cdots,V$ | set of vehicle types |
| $k\in K_v$ | set of compartments belonging to vehicle type $v$ — count and per-compartment specs vary by type |
| $f_1,\cdots,F$ | set of fuel types |
| $c_1,\cdots,C$ | set of customers |
| $t_1,\cdots,T$ | set of terminals |
| $d_1,\cdots,D$ | set of depots |
| $p\in\{1,\dots,P\}$ | visit-slot index — the $p$-th time a given terminal is used by a given vehicle that day |
| $\hat T=\{(t,p): t\in T,\ p=1,\dots,P\}$ | the set of terminal visit-slot nodes |
| $\mathcal N := C\cup\hat T$ | customer and terminal-visit-slot nodes — the nodes that get a "visited" indicator $z$ and a subtour-elimination value $u$ |
| $j,j'$ | generic node indices, ranging over $\mathcal N\cup D$ unless stated otherwise |
| $i \in I_v = \{1,\dots,m_v\}$ | physical vehicle instances of type $v$ |
| $(j,j') \in A,\ A\subseteq(\mathcal N\cup D)\times(\mathcal N\cup D)$ | feasible arcs. Depot-to-depot and depot-to-customer arcs are excluded — a vehicle can't deliver before loading, and never travels directly between two depots. |

> $Dist_{(t,p),(t,p')}=0$ for any $p,p'$ of the same terminal — different slots of one terminal are the same physical place.

## Parameters

| Symbol | Description |
|---|---|
| $Qcap_{v,k}$ | capacity of compartment $k\in K_v$ in a vehicle of type $v$ |
| $\Delta_{f,k,v}$ | 1 if compartment $k\in K_v$ of vehicle type $v$ is engineered to carry fuel $f$ |
| $AC_{f,v}$ | aggregate capacity for fuel $f$ in vehicle type $v$; $AC_{f,v}=\sum_{k\in K_v} Qcap_{v,k}\cdot\Delta_{f,k,v}$ |
| $Qd_{f,c}$ | quantity of fuel $f$ demanded by customer $c$ |
| $TQd_f$ | total demanded quantity of fuel $f$; $TQd_f=\sum_c Qd_{f,c}$ (a reporting definition, not an independent input) |
| $EA_j,\ LA_j$ | earliest / latest acceptable arrival time at node $j$ |
| $m_v$ | maximum number of physical vehicles available of type $v$ |
| $Cm_v$ | maintenance cost per km for vehicle type $v$ |
| $Cf_v$ | fuel-consumption cost per km for vehicle type $v$ (driving fuel, not delivered fuel) |
| $Cd_v$ | driver cost per unit travel time for vehicle type $v$ |
| $Sp_{v,jj'}$ | average speed of vehicle type $v$ on arc $(j,j')$ |
| $Dist_{jj'}$ | distance of arc $(j,j')$ |
| $\beta$ | loading-rate constant — time per unit quantity loaded at a terminal |
| $\gamma$ | unloading-rate constant — time per unit quantity delivered to a customer |
| $T^{max}$ | maximum permitted duration of a vehicle's entire daily route, 10 hours |
| $E_t\in\{0,1\}$ | 1 if terminal $t$ may be used as a route's start or end |
| $TF_{f,t}\in\{0,1\}$ | 1 if terminal $t$ stocks fuel $f$ |
| $Pr_{f,t}\geq0$ | price of fuel $f$ at terminal $t$ |
| $FT_t\geq0$ | fixed cost incurred if a route ends at terminal $t$ |
| $P$ | cap on visit-slots per terminal per vehicle per day — a modeling device (e.g. $P=|C|$), not a real operating limit |
| $M$ | large constant (big-M) |

## Variables

| Symbol | Description |
|---|---|
| $Ql_{k,f,v,i,(t,p)}$ | quantity of fuel $f$ loaded into compartment $k$ of vehicle-instance $i$ (type $v$) at visit-slot $(t,p)$ |
| $Lc_{k,f,v,i,j}\geq0$ | running quantity of fuel $f$ in compartment $k$ of vehicle-instance $i$ (type $v$) immediately after node $j$ |
| $\delta_{f,k,v,i}\in\{0,1\}$ | 1 if compartment $k$ of vehicle-instance $i$ (type $v$) is assigned fuel $f$ for the day |
| $N_v$ | number of vehicles of type $v$ used; $N_v=\sum_{i\in I_v} w_{v,i}$ |
| $TA_{v,i,j}$ | arrival time of vehicle-instance $i$ (type $v$) at node $j\in\mathcal N\cup D$ |
| $w_{v,i}\in\{0,1\}$ | 1 if vehicle-instance $i$ of type $v$ is dispatched on its (single) daily route |
| $z_{v,i,j}\in\{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) visits node $j\in\mathcal N$ |
| $x_{v,i,jj'}\in\{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) traverses arc $(j,j')\in A$ |
| $q_{c,k,f,v,i}\geq0$ | quantity of fuel $f$ delivered to customer $c$ from compartment $k$ of vehicle-instance $i$ (type $v$) |
| $\xi^{o}_{v,i,t}\in\{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) starts its route at terminal $t$ — necessarily at slot $(t,1)$ |
| $\xi^{o,D}_{v,i,d}\in\{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) starts its route at depot $d$ |
| $\xi^{d}_{v,i,(t,p)}\in\{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) ends its route at visit-slot $(t,p)$ |
| $\xi^{d,D}_{v,i,d}\in\{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) ends its route at depot $d$ |
| $u_{v,i,j}$ | MTZ subtour-elimination value, defined for $j\in\mathcal N$ |
| $s_{v,i,(t,p)}\geq0$ | loading time vehicle-instance $i$ (type $v$) spends at visit-slot $(t,p)$; $s_{v,i,(t,p)}=\beta\cdot\sum_{k\in K_v}\sum_f Ql_{k,f,v,i,(t,p)}$ |
| $s_{v,i,c}\geq0$ | unloading time vehicle-instance $i$ (type $v$) spends at customer $c$; $s_{v,i,c}=\gamma\cdot\sum_{k\in K_v}\sum_f q_{c,k,f,v,i}$ |

> Shorthand used below: $\xi^{d}_{v,i,t}:=\sum_{p=1}^{P}\xi^{d}_{v,i,(t,p)}$ — whether terminal $t$ is the end of the route at all, regardless of which slot.

---
# Objective Function

$$
\min Z \;=\; \underbrace{\sum_{v\in V}\sum_{i\in I_v}\sum_{(j,j')\in A} (Cm_v+Cf_v)\cdot Dist_{jj'}\cdot x_{v,i,jj'}}_{\text{driving cost}} \;+\; \underbrace{\sum_{v\in V}\sum_{i\in I_v}\sum_{t\in T}\sum_{p=1}^{P}\sum_{k\in K_v}\sum_{f\in F} Pr_{f,t}\cdot Ql_{k,f,v,i,(t,p)}}_{\text{fuel purchase cost}} \;+\; \underbrace{\sum_{v\in V}\sum_{i\in I_v} Cd_v\cdot\sum_{(j,j')\in A}\tau^v_{jj'}\cdot x_{v,i,jj'}}_{\text{driver cost}} \;+\; \underbrace{\sum_{v\in V}\sum_{i\in I_v}\sum_{t\in T} FT_t\cdot\xi^{d}_{v,i,t}}_{\text{terminal-stay fixed cost}}
$$

Total cost is driving cost, plus the fuel actually purchased, plus the driver's time on the move, plus a fixed cost only when a route happens to end at a terminal rather than back at a depot. There's no separate per-vehicle dispatch fee — running a vehicle at all is already captured by the other three terms.

---
# Constraints

### C1 — Demand Satisfaction
$$
\sum_{v\in V}\sum_{i\in I_v}\sum_{k\in K_v} q_{c,k,f,v,i} \;=\; Qd_{f,c} \qquad \forall\, c\in C,\ \forall\, f\in F
$$
The fuel a customer receives, added up across every vehicle, every instance of that vehicle, and every compartment that visits them, equals exactly what they ordered.
**Purpose:** guarantees every customer's order is fully met, without requiring any single vehicle or compartment to cover the whole thing — this is what makes split delivery possible.

### C2 — Compartment–Fuel Compatibility
$$
\delta_{f,k,v,i} \leq \Delta_{f,k,v} \qquad \forall\, f\in F,\ \forall\, k\in K_v,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
A compartment can only be assigned a fuel it was physically built to carry.
**Purpose:** stops the model from loading a fuel into a compartment that isn't engineered for it.

### C3 — Single Fuel per Compartment per Day
$$
\sum_{f\in F} \delta_{f,k,v,i} \leq 1 \qquad \forall\, k\in K_v,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
Each compartment is assigned at most one fuel for the whole day.
**Purpose:** enforces the no-mixing rule — a compartment can be topped up with more of its fuel, but never switched to a different one mid-day.

### C4 — Cumulative Load Propagation
$$
Lc_{k,f,v,i,j'} \;\geq\; Lc_{k,f,v,i,j} + \Theta_{k,f,v,i,j'} - M\bigl(1-x_{v,i,jj'}\bigr) \qquad \forall\,(j,j')\in A,\ \forall\,k\in K_v,\ \forall\,f\in F,\ \forall\,v\in V,\ \forall\,i\in I_v
$$
$$
\text{where } \Theta_{k,f,v,i,j'} = \begin{cases} Ql_{k,f,v,i,(t,p)} & j'=(t,p)\in\hat T \\ -\,q_{c,k,f,v,i} & j'=c\in C \\ 0 & j'=d\in D \end{cases}
$$
$$
Lc_{k,f,v,i,d} = 0 \qquad \forall\, d\in D,\ \forall\, k\in K_v,\ \forall\, f\in F,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
As the vehicle moves along its route, a compartment's running fuel level rises by whatever is loaded at a terminal stop, falls by whatever is delivered at a customer stop, and is fixed at exactly zero at any depot (depots never carry fuel).
**Purpose:** tracks what's physically in each compartment at every point in the route, which is what makes partial deliveries and topping up possible to represent correctly.

### C5 — Per-Compartment Running Capacity
$$
0 \;\leq\; Lc_{k,f,v,i,j} \;\leq\; Qcap_{v,k}\cdot\delta_{f,k,v,i} \qquad \forall\, j\in\mathcal N,\ \forall\, k\in K_v,\ \forall\, f\in F,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
At every point along the route, a compartment's fuel level stays between empty and its physical capacity.
**Purpose:** prevents overfilling a compartment, and the lower bound automatically prevents delivering more than what's currently inside it.

### C6 — Aggregate Vehicle Running Capacity per Fuel
$$
\sum_{k\in K_v} Lc_{k,f,v,i,j} \;\leq\; AC_{f,v} \qquad \forall\, j\in\mathcal N,\ \forall\, f\in F,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
Across all of a vehicle's compatible compartments combined, the running total of a given fuel never exceeds the vehicle's aggregate capacity for it.
**Purpose:** a secondary cap on top of the per-compartment one, useful only if the vehicle's overall carrying capacity for a fuel is set independently of the simple compartment sum (e.g. a weight limit).

### C7 — Load–Delivery Balance
$$
\sum_{t\in T}\sum_{p=1}^{P} Ql_{k,f,v,i,(t,p)} \;=\; \sum_{c\in C} q_{c,k,f,v,i} \qquad \forall\, k\in K_v,\ \forall\, f\in F,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
Total loaded across every terminal stop equals total delivered across every customer stop, for that compartment, for the day.
**Purpose:** ensures everything loaded into a compartment across the whole day is fully accounted for by deliveries.

### C8 — Visit–Delivery Linking
$$
q_{c,k,f,v,i} \;\leq\; M\cdot z_{v,i,c} \qquad \forall\, c\in C,\ \forall\, k\in K_v,\ \forall\, f\in F,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
A customer can only receive fuel if the vehicle actually visits them.
**Purpose:** blocks deliveries to customers who aren't on the route.

### C9 — Flow Conservation: Customers
$$
\sum_{j'} x_{v,i,c,j'} \;=\; \sum_{j'} x_{v,i,j',c} \;=\; z_{v,i,c} \qquad \forall\, c\in C,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
A visited customer has exactly one arc in and one arc out; an unvisited one has none.
**Purpose:** makes sure the route passes cleanly through each customer it serves, with no dangling or duplicated stops.

### C10 — Flow Conservation & Route Endpoints
$$
\sum_{j'} x_{v,i,j',(t,p)} \;=\; z_{v,i,(t,p)} - \xi^{o}_{v,i,t}\cdot[p{=}1] \qquad \forall\, t\in T,\ p=1,\dots,P,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
\sum_{j'} x_{v,i,(t,p),j'} \;=\; z_{v,i,(t,p)} - \xi^{d}_{v,i,(t,p)} \qquad \forall\, t\in T,\ p=1,\dots,P,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
\sum_{j'} x_{v,i,d,j'} \;=\; \xi^{o,D}_{v,i,d}, \qquad \sum_{j'} x_{v,i,j',d} \;=\; \xi^{d,D}_{v,i,d} \qquad \forall\, d\in D,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
\sum_{d\in D}\xi^{o,D}_{v,i,d} + \sum_{t\in T}\xi^{o}_{v,i,t} \;=\; w_{v,i}, \qquad \sum_{d\in D}\xi^{d,D}_{v,i,d} + \sum_{t\in T}\xi^{d}_{v,i,t} \;=\; w_{v,i} \qquad \forall\, v\in V,\ \forall\, i\in I_v
$$
($[p{=}1]$ above means the $\xi^o$ term only applies at slot 1 — slot 1 is the only slot that can ever be a route's start.) Every terminal visit-slot and every depot use has its arcs in and out balanced against whether it's used at all — except for whichever single node is the day's actual start (no arc in) and whichever single node is the day's actual end (no arc out). Exactly one start and one end is chosen per dispatched vehicle, and each can independently be a depot or a terminal.
**Purpose:** builds the route as one continuous path with a well-defined start and end, while letting that start and end be a depot or a terminal, and letting them differ from each other.

### C11 — Terminal Eligibility
$$
\xi^{o}_{v,i,t} \;\leq\; E_t, \qquad \xi^{d}_{v,i,t} \;\leq\; E_t \qquad \forall\, t\in T,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
A terminal can only be used as a route's start or end if it's flagged eligible.
**Purpose:** keeps certain terminals reserved for mid-route reloads only, never as a place a route begins or ends.

### C12 — Terminal Slot Ordering
$$
z_{v,i,(t,p)} \;\leq\; z_{v,i,(t,p-1)} \qquad \forall\, t\in T,\ p=2,\dots,P,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
A terminal's 2nd, 3rd, etc. visit-slot can't be used unless the one before it was used too.
**Purpose:** stops the solver from representing one real visit in many meaninglessly different ways.

### C13 — Loading Gating
$$
Ql_{k,f,v,i,(t,p)} \;\leq\; M\cdot TF_{f,t}, \qquad Ql_{k,f,v,i,(t,p)} \;\leq\; M\cdot z_{v,i,(t,p)}, \qquad Ql_{k,f,v,i,(t,p)} \;\leq\; Qcap_{v,k}\cdot \delta_{f,k,v,i}
$$
$$
\forall\, k\in K_v,\ f\in F,\ v\in V,\ i\in I_v,\ t\in T,\ p=1,\dots,P
$$
A vehicle can't load a fuel the terminal doesn't stock, can't load at a visit-slot it isn't actually using, and can't load a fuel into a compartment not assigned to carry it that day.
**Purpose:** three independent checks that keep every loading event physically realistic.

### C14 — Subtour Elimination (MTZ)
$$
u_{v,i,j} - u_{v,i,j'} + |\mathcal N|\cdot x_{v,i,j,j'} \leq |\mathcal N|-1 \qquad \forall\, j\neq j' \in \mathcal N,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
Every customer and terminal visit-slot gets a position value; if an arc is used, the destination's value must be consistent with coming after the origin's.
**Purpose:** prevents the solver from forming a closed loop of customers/terminal-slots that's disconnected from the rest of the route. Depots are excluded from this — they're always a route's boundary, never part of an internal loop.

### C15 — Time Windows
$$
EA_c - M\bigl(1-z_{v,i,c}\bigr) \;\leq\; TA_{v,i,c} \;\leq\; LA_c + M\bigl(1-z_{v,i,c}\bigr) \qquad \forall\, c\in C,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
EA_t - M\bigl(1-z_{v,i,(t,p)}\bigr) \;\leq\; TA_{v,i,(t,p)} \;\leq\; LA_t + M\bigl(1-z_{v,i,(t,p)}\bigr) \qquad \forall\, t\in T,\ p=1,\dots,P,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
Every visited node — customer or terminal slot — must be arrived at within that node's allowed time band.
**Purpose:** keeps the route consistent with customer availability and terminal opening hours. A terminal's window only checks arrival, not how long the vehicle then stays.

### C16 — Arrival-Time Propagation
$$
\tau^v_{jj'} \;=\; \dfrac{Dist_{jj'}}{Sp_{v,jj'}} \qquad \forall\,(j,j')\in A,\ \forall\, v\in V
$$
$$
s_{v,i,c} \;=\; \gamma\cdot\sum_{k\in K_v}\sum_{f\in F} q_{c,k,f,v,i} \qquad \forall\, c\in C,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
s_{v,i,(t,p)} \;=\; \beta\cdot\sum_{k\in K_v}\sum_{f\in F} Ql_{k,f,v,i,(t,p)} \qquad \forall\, t\in T,\ p=1,\dots,P,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
TA_{v,i,j'} \;\geq\; TA_{v,i,j} + \bar s_{v,i,j} + \tau^v_{jj'} - M\bigl(1-x_{v,i,jj'}\bigr) \qquad \forall\,(j,j')\in A,\ j,j'\in\mathcal N\cup D,\ \forall\, v\in V,\ \forall\, i\in I_v
$$
$$
\text{where } \bar s_{v,i,j} = \begin{cases} s_{v,i,c} & j=c\in C \\ s_{v,i,(t,p)} & j=(t,p)\in\hat T \\ 0 & j=d\in D \end{cases}
$$
Arrival time at the next stop is whatever time the vehicle arrived at the current stop, plus however long it spends there, plus the travel time to get to the next one. How long it spends at a stop now depends on how much it loads or delivers there, not a flat number.
**Purpose:** propagates a consistent clock along the whole route, accounting for speed that can vary by arc and dwell time that can vary by quantity handled.

### C17 — Maximum Daily Driving Duration (10 hours)
$$
TA_{v,i,n'} - TA_{v,i,n} - \bar s_{v,i,n} \;\leq\; T^{max} + M\bigl(2-\Xi^{o}_{v,i,n}-\Xi^{d}_{v,i,n'}\bigr)
$$
$$
\forall\, n\in D\cup\{(t,1):t\in T\},\ \forall\, n'\in D\cup\hat T,\ \forall\, v\in V,\ \forall\, i\in I_v,\quad T^{max}=10\text{ hours}
$$
$$
\text{where } \Xi^{o}_{v,i,n}=\xi^{o,D}_{v,i,n}\text{ if }n\in D,\ \xi^{o}_{v,i,t}\text{ if }n=(t,1);\qquad \Xi^{d}_{v,i,n'}=\xi^{d,D}_{v,i,n'}\text{ if }n'\in D,\ \xi^{d}_{v,i,(t,p)}\text{ if }n'=(t,p)
$$
The gap between the moment the vehicle actually leaves its start and the moment it actually arrives at its end can't exceed 10 hours.
**Purpose:** caps how long a vehicle's day can run, regardless of whether that day started and ended at a depot, a terminal, or one of each.

### C18 — Fleet Size per Type
$$
N_v = \sum_{i\in I_v} w_{v,i} \leq m_v \qquad \forall\, v\in V
$$
The number of vehicles of a type actually used can't exceed how many of that type exist.
**Purpose:** caps fleet usage at what's physically available.

### C19 — Variable Domains
$$
x_{v,i,jj'},\ z_{v,i,j},\ w_{v,i},\ \delta_{f,k,v,i},\ \xi^{o}_{v,i,t},\ \xi^{o,D}_{v,i,d},\ \xi^{d}_{v,i,(t,p)},\ \xi^{d,D}_{v,i,d} \in \{0,1\}
$$
$$
Ql_{k,f,v,i,(t,p)},\ Lc_{k,f,v,i,j},\ s_{v,i,c},\ s_{v,i,(t,p)},\ q_{c,k,f,v,i},\ TA_{v,i,j} \geq 0
$$
States which variables are binary decisions and which are non-negative quantities.
**Purpose:** keeps the model a clean Mixed-Integer Linear Program.

---
# Linearity Check

The model is a **Mixed-Integer Linear Program**. Every conditional relationship — load propagation, time propagation, the duration cap — is written as a big-M inequality rather than a direct product of two decision variables, so nothing here is quadratic. The two dwell-time definitions ($s_{v,i,c}=\gamma\sum q$, $s_{v,i,(t,p)}=\beta\sum Ql$) and the driver-cost term ($Cd_v\cdot\tau^v_{jj'}\cdot x_{v,i,jj'}$) are all a fixed constant multiplying a single decision variable, which stays linear. The model deliberately avoids ever multiplying two decision variables together (e.g. a binary indicator by a continuous time variable) — that's why $Dep_{v,i}/Ret_{v,i}$-style helper variables aren't used anywhere; constraints reference $TA$ directly, gated by big-M.

Things that would **not** stay linear if added later: a running cost that depends on both distance *and* carried weight (a true product of two decision variables), or a penalty whose priority weight is itself decision-dependent.

---
# Modeling Notes

1. **Type vs. instance.** $N_v$ only makes sense if it counts *distinct trucks of type v*; routing variables can't be indexed by type alone, since two trucks of the same type need independent routes. Hence $I_v$ and $i$.
2. **$\delta$ vs. $\Delta$.** "Is this compartment built for this fuel" ($\Delta$, fixed) and "is this compartment currently assigned this fuel" ($\delta$, a daily decision) are different things, kept as separate symbols.
3. **Why $Lc$ propagation only needs one direction.** A lower-bound-only chain on $Lc$, combined with the upper bound in C5, pins $Lc$ to its true value at any optimum — inflating $Lc$ only makes C5 harder to satisfy, never easier, so there's no incentive to misreport it.
4. **Why $Lc=0$ at depots is a flat equality, not big-M gated.** Unlike a terminal slot (which only sometimes is the route's start), a depot never carries fuel under any circumstance, so its $Lc$ is fixed at zero outright rather than conditioned on $\xi^{o,D}$.
5. **Origin is always slot 1; destination is tracked per-slot.** This isn't arbitrary — slot 1 is the only slot that could ever have nothing before it (the slot-ordering rule means slot 2 of a terminal always requires slot 1 of that same terminal to have already happened). The destination has no such restriction, since a route can legitimately end on a terminal's 2nd, 3rd, etc. visit.
6. **Why the duration cap (C17) doesn't need separate $Dep$/$Ret$ variables.** Referencing $TA$ directly, gated by whichever origin/destination indicator applies, avoids ever creating a $\xi\cdot TA$ product.
7. **"Exactly one origin/destination" already rules out using two terminals at once.** Because $\sum(\text{all origin options, depot and terminal combined})=w_{v,i}$ and $w_{v,i}\in\{0,1\}$, the terminal-only portion of that sum can never exceed 1 on its own — a separate "$\le1$" constraint restricted to terminals would just restate something the equality already guarantees. Same logic applies to the destination side.
8. **The aggregate demand identity ($\sum_{c,v,i,k} q_{c,k,f,v,i}=TQd_f$) is implied, not independent.** C1 already states the per-customer version; summing it over every customer produces the aggregate version automatically. It's true the same way $TQd_f$ itself is just a reporting definition.
9. **Slot-visit-order matching slot number is not required for correctness, only optionally for solver speed.** $Lc$ and $TA$ both propagate along actual arcs, not slot labels, so nothing breaks if the solver's internal numbering of "1st vs 2nd visit" doesn't match real-world intuition. Adding $u_{v,i,(t,p)} > u_{v,i,(t,p-1)}$ as an extra symmetry-breaking constraint is optional, not required, and was left out to keep the model minimal.
10. **Driver cost uses pure travel time, not total elapsed time.** $Cd_v$ is multiplied by $\sum\tau^v_{jj'}x_{v,i,jj'}$ — moving time only, excluding however long the vehicle dwells at any stop. This is a deliberate reading of "time travelled"; if driver pay should instead cover the whole day including dwell time, the driver-cost term would need to use the same elapsed-time expression as C17 instead.
11. **C6 redundancy.** As currently defined, $AC_{f,v}$ is just the sum of the per-compartment caps already in C5, so C6 adds nothing unless $AC_{f,v}$ is set independently tighter (e.g. a weight-based de-rating).
12. **$TQd_f$** remains a definition for reporting/sanity-checks, not an independent constraint. 

---
# Not Yet Modeled

1. **Fuel-specific / safety: cross-contamination and compartment-adjacency rules.** Tanker layouts that forbid certain fuel pairs in adjacent compartments, or require a fixed unloading sequence by valve position, are not modeled. To be added only after the approach is confirmed.
