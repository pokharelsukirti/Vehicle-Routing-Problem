# Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)
*A Mixed-Integer Quadratically-Constrained Formulation*

**Problem setting.** A heterogeneous fleet of multi-compartment vehicles delivers several fuel types from a set of terminals to a set of customers over a single day. Each vehicle may reload at multiple terminal visits, must respect per-compartment fuel purity and capacity, must arrive within customer and terminal time windows, and must complete its route within a maximum daily duration. Routes may begin and end at a depot or at an eligible terminal. The model below is an exact Mixed-Integer Quadratically-Constrained Program (MIQCP): every nonlinear term is a bilinear product of a binary variable and a continuous (or binary) variable, linear in either factor when the other is fixed, so the formulation is exact rather than a relaxation.

---

## Table of Contents

1. [Notation](#1-notation)
   - 1.1 [Indices and Sets](#11-indices-and-sets)
   - 1.2 [Parameters](#12-parameters)
   - 1.3 [Derived Quantity: Travel Time](#13-derived-quantity-travel-time)
   - 1.4 [Decision Variables](#14-decision-variables)
2. [Objective Function](#2-objective-function)
3. [Constraints](#3-constraints)
   - C1 [Demand Satisfaction](#c1--demand-satisfaction)
   - C2 [Compartment–Fuel Compatibility](#c2--compartmentfuel-compatibility)
   - C3 [Single Fuel per Compartment per Day](#c3--single-fuel-per-compartment-per-day)
   - C4 [Cumulative Load Propagation](#c4--cumulative-load-propagation)
   - C5 [Per-Compartment Running Capacity](#c5--per-compartment-running-capacity)
   - C6 [Aggregate Vehicle Running Capacity per Fuel](#c6--aggregate-vehicle-running-capacity-per-fuel)
   - C7 [Load–Delivery Balance](#c7--loaddelivery-balance)
   - C8 [Visit–Delivery Linking](#c8--visitdelivery-linking)
   - C9 [Flow Conservation (All Node Types)](#c9--flow-conservation-all-node-types)
   - C10 [Terminal Eligibility](#c10--terminal-eligibility)
   - C11 [Terminal Slot Ordering](#c11--terminal-slot-ordering)
   - C12 [Loading Gating](#c12--loading-gating)
   - C13 [Subtour Elimination (MTZ)](#c13--subtour-elimination-mtz)
   - C14 [Time Windows](#c14--time-windows)
   - C15 [Arrival-Time Propagation](#c15--arrival-time-propagation)
   - C16 [Maximum Daily Route Duration](#c16--maximum-daily-route-duration)
   - C17 [Fleet Size per Type](#c17--fleet-size-per-type)
   - C18 [Variable Domains](#c18--variable-domains)
4. [Model Classification](#4-model-classification)
5. [Modeling Notes](#5-modeling-notes)
6. [Not Yet Modeled](#6-not-yet-modeled)

---

## 1. Notation

### 1.1 Indices and Sets

| Symbol | Description |
|--------|-------------|
| $v \in V$ | set of vehicle types |
| $k \in K_v$ | set of compartments belonging to vehicle type $v$ |
| $f \in F$ | set of fuel types |
| $c \in C$ | set of customers |
| $t \in T$ | set of terminals |
| $d \in D$ | set of depots |
| $p \in \{1,\dots,P\}$ | visit-slot index — the $p$-th time a given terminal is used by a given vehicle that day |
| $\hat{T} = \{(t,p) : t \in T,\ p = 1,\dots,P\}$ | set of terminal visit-slot nodes |
| $\mathcal{N} := C \cup \hat{T}$ | customer and terminal-visit-slot nodes |
| $j,\ j'$ | generic node indices, ranging over $\mathcal{N} \cup D$ unless stated otherwise |
| $i \in I_v = \{1,\dots,m_v\}$ | physical vehicle instances of type $v$ |
| $(j,j') \in A$ | feasible arcs — defined exhaustively by the index sets in C9; any pair not appearing there carries no variable |

> **Note:** $Dist_{(t,p),(t,p')} = 0$ for any $p,p'$ of the same terminal — different slots of one terminal are the same physical place.

### 1.2 Parameters

| Symbol | Description |
|--------|-------------|
| $Qcap_{v,k}$ | capacity of compartment $k \in K_v$ in a vehicle of type $v$ |
| $\Delta_{f,k,v}$ | 1 if compartment $k \in K_v$ of vehicle type $v$ is engineered to carry fuel $f$ |
| $AC_{f,v}$ | aggregate capacity for fuel $f$ in vehicle type $v$; $AC_{f,v} = \sum_{k \in K_v} Qcap_{v,k} \cdot \Delta_{f,k,v}$ |
| $Qd_{f,c}$ | quantity of fuel $f$ demanded by customer $c$ |
| $TQd_f$ | total demanded quantity of fuel $f$; $TQd_f = \sum_c Qd_{f,c}$ (reporting definition) |
| $EA_j,\ LA_j$ | earliest / latest acceptable arrival time at node $j \in \mathcal{N}$ (terminals and customers only — depots carry no time windows) |
| $m_v$ | maximum number of physical vehicles available of type $v$ |
| $Cm_v$ | maintenance cost per km for vehicle type $v$ |
| $Cf_v$ | fuel-consumption cost per km for vehicle type $v$ |
| $Cd_v$ | driver cost per unit travel time for vehicle type $v$ |
| $Sp_{v,j,j'}$ | average speed of vehicle type $v$ on arc $(j,j')$ |
| $Dist_{j,j'}$ | distance of arc $(j,j')$ |
| $\beta$ | loading-rate constant — time per unit quantity loaded at a terminal |
| $\gamma$ | unloading-rate constant — time per unit quantity delivered to a customer |
| $T^{max}$ | maximum permitted duration of a vehicle's entire daily route |
| $E_t \in \{0,1\}$ | 1 if terminal $t$ may be used as a route's start or end |
| $TF_{f,t} \in \{0,1\}$ | 1 if terminal $t$ stocks fuel $f$ |
| $Pr_{f,t} \geq 0$ | price of fuel $f$ at terminal $t$ |
| $FT_t \geq 0$ | fixed cost incurred if a route ends at terminal $t$ |
| $P$ | cap on visit-slots per terminal per vehicle per day |
| $M$ | large constant — the model needs it only in the terminal fuel-stocking gate of C12 ($Ql \le M\cdot TF_{f,t}$), because $TF_{f,t}$ is a fixed parameter rather than a decision variable, so no exact bilinear reformulation applies. Every other gate in the model is reformulated exactly and is $M$-free. |

### 1.3 Derived Quantity: Travel Time

$$\tau^{v}_{j,j'} = \frac{Dist_{j,j'}}{Sp_{v,j,j'}} \qquad \forall\ (j,j') \in A,\ v \in V$$

Travel time on each arc, used in the objective's driver-cost term and in the arrival-time propagation of C15.

### 1.4 Decision Variables

| Symbol | Description |
|--------|-------------|
| $Ql_{k,f,v,i,(t,p)} \geq 0$ | quantity of fuel $f$ loaded into compartment $k$ of vehicle-instance $i$ (type $v$) at visit-slot $(t,p)$ |
| $Lc_{k,f,v,i,j} \geq 0$ | running quantity of fuel $f$ in compartment $k$ of vehicle-instance $i$ (type $v$) immediately after node $j$ |
| $\delta_{f,k,v,i,(t,p)} \in \{0,1\}$ | 1 if fuel $f$ is loaded into compartment $k$ of vehicle-instance $i$ (type $v$) at visit-slot $(t,p)$. Because at most one fuel may ever be assigned to a compartment in a day (C3), $\delta$ being 1 at *any* slot also identifies that fuel as the compartment's day-level assignment (Note 2) |
| $N_v$ | number of vehicles of type $v$ used; $N_v = \sum_{i \in I_v} w_{v,i}$ |
| $TA_{v,i,j} \geq 0$ | arrival time of vehicle-instance $i$ (type $v$) at node $j \in \mathcal{N} \cup D$ |
| $w_{v,i} \in \{0,1\}$ | 1 if vehicle-instance $i$ of type $v$ is dispatched |
| $z_{v,i,j} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) visits node $j \in \mathcal{N}$ |
| $x_{v,i,j,j'} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) traverses arc $(j,j') \in A$ |
| $q_{c,k,f,v,i} \geq 0$ | quantity of fuel $f$ delivered to customer $c$ from compartment $k$ of vehicle-instance $i$ (type $v$) |
| $\xi^{o}_{v,i,t} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) starts its route at terminal $t$ — necessarily at slot $(t,1)$ |
| $\xi^{o,D}_{v,i,d} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) starts its route at depot $d$ |
| $\xi^{d}_{v,i,(t,p)} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) ends its route at visit-slot $(t,p)$ (overnight stay at the terminal — no further travel) |
| $\xi^{d,D}_{v,i,d} \in \{0,1\}$ | 1 if vehicle-instance $i$ (type $v$) ends its route at depot $d$ |
| $u_{v,i,j} \geq 0$ | MTZ subtour-elimination value, defined for $j \in \mathcal{N}$ |
| $s_{v,i,(t,p)} \geq 0$ | loading time at visit-slot $(t,p)$; $s_{v,i,(t,p)} = \beta \cdot \sum_{k \in K_v} \sum_f Ql_{k,f,v,i,(t,p)}$ |
| $s_{v,i,c} \geq 0$ | unloading time at customer $c$; $s_{v,i,c} = \gamma \cdot \sum_{k \in K_v} \sum_f q_{c,k,f,v,i}$ |
| $y_{v,i,n,n'} \geq 0$ | equals $\Xi^{o}_{v,i,n}\cdot\Xi^{d}_{v,i,n'}$ — 1 iff $n$ is the route's start and $n'$ is its end; pinned to $\{0,1\}$ by its defining inequalities in C16 |

> **Shorthand:** $\xi^{d}_{v,i,t} := \sum_{p=1}^{P} \xi^{d}_{v,i,(t,p)}$ — whether terminal $t$ is the route's overnight end at any slot.

---

## 2. Objective Function

$$\min Z = \underbrace{\sum_{v} \sum_{i} \sum_{(j,j') \in A} (Cm_v + Cf_v) \cdot Dist_{j,j'} \cdot x_{v,i,j,j'}}_{\text{driving cost}} + \underbrace{\sum_{v} \sum_{i} \sum_{t} \sum_{p} \sum_{k} \sum_{f} Pr_{f,t} \cdot Ql_{k,f,v,i,(t,p)}}_{\text{fuel purchase cost}}$$

$$+\ \underbrace{\sum_{v} \sum_{i} Cd_v \cdot \sum_{(j,j') \in A} \tau^v_{j,j'} \cdot x_{v,i,j,j'}}_{\text{driver cost}} \ +\ \underbrace{\sum_{v} \sum_{i} \sum_{t} FT_t \cdot \xi^{d}_{v,i,t}}_{\text{terminal-stay fixed cost}}$$

Four cost components: distance-based driving cost (maintenance plus fuel consumption of the vehicle itself), the price paid for fuel purchased at terminals, driver time on the road, and a fixed cost charged only when a route's overnight end is a terminal. $Cd_v$ multiplies travel time only, not dwell time; see Note 10 if driver pay should instead cover total elapsed time.

---

## 3. Constraints

### C1 — Demand Satisfaction

$$\sum_{v \in V} \sum_{i \in I_v} \sum_{k \in K_v} q_{c,k,f,v,i} = Qd_{f,c} \qquad \forall\ c \in C,\ f \in F$$

The total quantity of fuel $f$ received by customer $c$ across all vehicles, instances, and compartments equals exactly what they ordered, allowing split delivery.

### C2 — Compartment–Fuel Compatibility

$$\delta_{f,k,v,i,(t,p)} \leq \Delta_{f,k,v} \qquad \forall\ f \in F,\ k \in K_v,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P$$

Loading fuel $f$ into compartment $k$ at any slot is only permitted if the compartment is engineered to carry that fuel.

### C3 — Single Fuel per Compartment per Day

$$\delta_{f,k,v,i,(t,p)} + \delta_{f',k,v,i,(t',p')} \leq 1 \qquad \forall\ f \neq f' \in F,\ (t,p),(t',p') \in \hat T,\ k \in K_v,\ v \in V,\ i \in I_v$$

For every pair of visit-slots — including the same slot — and every pair of distinct fuels, at most one of the two slot/fuel assignments may be active. This single family of constraints does two jobs at once: taking $(t,p)=(t',p')$ forbids loading two different fuels into the same compartment during the same visit, while taking $(t,p)\ne(t',p')$ forbids switching fuels between visits across the day. Because the inequality only ever pairs *different* fuels, topping up the *same* fuel at any number of slots is left completely unrestricted, matching the no-mixing rule: a compartment may be refilled with its assigned fuel at every terminal stop, but may never carry a second fuel that day.

### C4 — Cumulative Load Propagation

Loading at a visit-slot is gated by the slot-level indicator $\delta_{f,k,v,i,(t,p)}$:

$$Ql_{k,f,v,i,(t,p)} \cdot \bigl(1 - \delta_{f,k,v,i,(t,p)}\bigr) = 0 \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P$$

Loading is further blocked at slots the vehicle does not visit, and at the route's final destination terminal:
\qquad \delta_{f,k,v,i,(t,p)} \leq 1 - \xi^{o}_{v,i,(t,p)} \qquad \forall\ k,f,v,i,t,1$$
\qquad \delta_{f,k,v,i,(t,p)} \leq 1 - \xi^{d}_{v,i,(t,p)} \qquad \forall\ k,f,v,i,t,p$$

The cumulative running load evolves *exactly* along the route:

$$x_{v,i,j,j'}\Bigl(Lc_{k,f,v,i,j'} - Lc_{k,f,v,i,j} - \Theta_{k,f,v,i,j'}\Bigr) = 0 \qquad \forall\ (j,j') \in A,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v$$

where,

$$\Theta_{k,f,v,i,j'} = \begin{cases} 
      Ql_{k,f,v,i,(t,p)} & \text{if } j' = (t,p) \in \hat{T} \\
      -q_{c,k,f,v,i}     & \text{if } j' = c \in C \\ 
      0                  & \text{if } j' = d \in D 
   \end{cases}$$

and the depot anchor:

$$Lc_{k,f,v,i,j} = 0 \quad \text{if } j \in D \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v$$

**Purpose.** Tracks the physical contents of each compartment along the route. When an arc is active, the running load rises by exactly the fuel loaded at a terminal, falls by exactly the fuel delivered to a customer, and resets to zero at any depot; when the arc is inactive the equation is vacuous ($0=0$). The propagation is written as an *equality*, not a one-sided bound: combined with the depot anchor and the nonnegativity of $Lc$ in C5, this is what stops a vehicle from delivering more fuel than is actually on board — if a requested delivery $q$ exceeded the content $Lc$ carried into that customer, the equality would force the post-delivery $Lc$ negative, violating C5. A one-sided ($\ge$) version of this gate would not catch that: $Lc$ could simply be set to its floor of zero regardless of how much was over-delivered, and the day-level total in C7 would not detect the sequencing violation either. Loading itself is permitted only when $\delta_{f,k,v,i,(t,p)} = 1$, which is itself suppressed at unvisited slots and at the route's overnight terminal end.

### C5 — Per-Compartment Running Capacity

$$0 \leq Lc_{k,f,v,i,j} \leq Qcap_{v,k} \qquad \forall\ j \in \mathcal{N},\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v$$

A plain physical capacity bound. Because C4's propagation is now an exact equality anchored at $Lc_d = 0$, a fuel that is never loaded into a given compartment automatically stays at $Lc=0$ for that fuel throughout the route — there is no need to condition the bound on a same-day loading indicator.

### C6 — Aggregate Vehicle Running Capacity per Fuel

$$\sum_{k \in K_v} Lc_{k,f,v,i,j} \leq AC_{f,v} \qquad \forall\ j \in \mathcal{N},\ f \in F,\ v \in V,\ i \in I_v$$

A vehicle-level cap on total running fuel, applicable when the aggregate capacity is set tighter than the sum of compartment caps (Note 11).

### C7 — Load–Delivery Balance

$$\sum_{t \in T} \sum_{p=1}^{P} Ql_{k,f,v,i,(t,p)} = \sum_{c \in C} q_{c,k,f,v,i} \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v$$

Everything loaded into a compartment across all terminal stops during the day is fully accounted for by deliveries. This is a day-level aggregate; the *sequencing* of loads and deliveries along the route is enforced separately, by C4.

### C8 — Visit–Delivery Linking

$$q_{c,k,f,v,i} \cdot \bigl(1 - z_{v,i,c}\bigr) = 0 \qquad \forall\ c \in C,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v$$

A vehicle can only deliver fuel to a customer it actually visits.

### C9 — Flow Conservation (All Node Types)

All arc existence is encoded in the index sets below; any $(j,j')$ pair not listed carries no variable and does not belong to $A$.

**Customer nodes ($j \in C$).**

$$\sum_{j' \in C \cup \hat{T}} x_{v,i,j',j} = z_{v,i,j} \qquad \text{(inflow)}$$

$$\sum_{j' \in C \cup \hat{T}\cup D} x_{v,i,j,j'} = z_{v,i,j} \qquad \text{(outflow)} \qquad \forall\ j \in C,\ v \in V,\ i \in I_v,\ j'\neq j$$

A visited customer has exactly one inbound arc, from another customer or a terminal slot — never from a depot, since every route's first move is to a terminal to load fuel. Its single outbound arc may continue to another customer or terminal slot, or, if this customer is the route's last stop, go directly to a depot.

**Terminal visit-slot nodes ($j = (t,p) \in \hat{T}$).**

$$\sum_{j' \in C \cup \hat{T} \cup D} x_{v,i,j',j} = z_{v,i,j} - \xi^{o}_{v,i,t} \cdot [p{=}1] \qquad \text{(inflow)}$$

$$\sum_{j' \in C \cup \hat{T} \cup D} x_{v,i,j,j'} = z_{v,i,j} - \xi^{d}_{v,i,(t,p)} \qquad \text{(outflow)} \qquad \forall\ j=(t,p) \in \hat{T},\ v \in V,\ i \in I_v$$

A terminal slot may be reached from a customer, another terminal slot, or a depot (route start); the correction at $p=1$ removes the missing inbound arc when that slot is the route's origin. A vehicle leaving a terminal slot may continue to a customer or another terminal slot, or, if this is the last stop before returning to base, go directly to a depot. If instead the slot is the route's overnight destination ($\xi^d=1$), the outbound count is $1-1=0$: no further movement of any kind — this is the only way a route ends without an arc into a depot.

**Depot nodes ($j \in D$).**

$$\sum_{j' \in \hat{T}} x_{v,i,j,j'} = \xi^{o,D}_{v,i,j} \qquad \text{(outflow)}$$

$$\sum_{j' \in C \cup \hat{T}} x_{v,i,j',j} = \xi^{d,D}_{v,i,j} \qquad \text{(inflow)} \qquad \forall\ j \in D,\ v \in V,\ i \in I_v$$

A depot is always a route's origin or destination, never a mid-route stop. Leaving a depot, the vehicle's first move is always to a terminal to load (no depot-to-customer or depot-to-depot arc exists). Entering a depot, the immediately preceding node may be a customer *or* a terminal — whichever was visited last.

**Single origin and single destination per dispatched vehicle.**

$$\sum_{d \in D} \xi^{o,D}_{v,i,d} + \sum_{t \in T} \xi^{o}_{v,i,t} = w_{v,i}, \qquad \sum_{d \in D} \xi^{d,D}_{v,i,d} + \sum_{t \in T} \xi^{d}_{v,i,t} = w_{v,i} \qquad \forall\ v \in V,\ i \in I_v$$

**Arc set $A$ — summary.**

| From $j$ | To $j'$ |
|----------|---------|
| $C$ | $C \cup \hat{T} \cup D$ |
| $\hat{T}$ | $C \cup \hat{T}$ |
| $D$ | $\hat{T}$ |

**Purpose.** Builds the route as a single continuous path with one start and one end. A depot is reachable only as the very last arc out of whichever node — customer or terminal slot — is visited last; it is never a mid-route stop, and every customer or terminal slot has at most one outbound arc, whether that arc continues the route or ends it at a depot. C13's subtour elimination operates on whatever arcs $A$ contains and requires no further arc-type exclusions (self-loops are in any case forced to zero by C13 itself, since $j'=j$ makes its right-hand side strictly negative).

### C10 — Terminal Eligibility

$$\xi^{o}_{v,i,t} \leq E_t, \qquad \xi^{d}_{v,i,t} \leq E_t \qquad \forall\ t \in T,\ v \in V,\ i \in I_v$$

A terminal may only be a route's start or end if it is flagged eligible; ineligible terminals are mid-route reload points only.

### C11 — Terminal Slot Ordering

$$z_{v,i,(t,p)} \leq z_{v,i,(t,p-1)} \qquad \forall\ t \in T,\ p = 2,\dots,P,\ v \in V,\ i \in I_v$$

A terminal's $p$-th visit-slot cannot be used unless slot $p-1$ was used, preventing multiple equivalent representations of the same visit count.

### C12 — Loading Gating

$$Ql_{k,f,v,i,(t,p)} \leq M \cdot TF_{f,t} \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P$$

$$Ql_{k,f,v,i,(t,p)} \cdot \bigl(1 - z_{v,i,(t,p)}\bigr) = 0$$

$$Ql_{k,f,v,i,(t,p)} \leq Qcap_{v,k} \cdot \delta_{f,k,v,i,(t,p)} \qquad \forall\ k,f,v,i,t,p$$

Three independent conditions must hold simultaneously: the terminal must stock the fuel (the only constraint in the model that still needs the constant $M$, since $TF_{f,t}$ is a fixed parameter); the slot must be visited; and the fuel must be assigned to the compartment at this slot. The $\delta$-gate in C4 further blocks loading at the route's overnight terminal end.

### C13 — Subtour Elimination (MTZ)

$$x_{v,i,j,j'}\bigl(u_{v,i,j'} - u_{v,i,j} - 1\bigr) \geq 0 \qquad \forall\ j \neq j' \in \mathcal{N},\ v \in V,\ i \in I_v$$

Every customer and terminal visit-slot is assigned a position value. When an arc is active, the destination's position must exceed the origin's. Depots are excluded, as they are always route boundaries, not intermediate stops.

**Purpose.** Prevents closed sub-loops disconnected from the rest of the route.

### C14 — Time Windows

$$z_{v,i,j}\bigl(TA_{v,i,j} - EA_j\bigr) \geq 0, \qquad z_{v,i,j}\bigl(LA_j - TA_{v,i,j}\bigr) \geq 0 \qquad \forall\ j \in \mathcal{N},\ v \in V,\ i \in I_v$$

When a node $j \in \mathcal{N} = C \cup \hat{T}$ is visited, arrival must fall within $[EA_j, LA_j]$; when unvisited, both inequalities read $0 \ge 0$ and impose no constraint. Depots carry no time windows.

**Purpose.** Keeps the route consistent with customer availability and terminal operating hours.

### C15 — Arrival-Time Propagation

Propagation along the route, using the travel time $\tau^v_{j,j'}$ defined in Section 1.3:

$$x_{v,i,j,j'}\Bigl(TA_{v,i,j'} - TA_{v,i,j} - \bar{s}_{v,i,j} - \tau^v_{j,j'}\Bigr) \geq 0 \qquad \forall\ (j,j') \in A,\ v \in V,\ i \in I_v$$

where

$$\bar{s}_{v,i,j} = \begin{cases} s_{v,i,c} & j = c \in C \\ s_{v,i,(t,p)} & j = (t,p) \in \hat{T} \\ 0 & j = d \in D \end{cases}$$

with $s_{v,i,c}$ and $s_{v,i,(t,p)}$ as defined in Section 1.4.

**Day-start grounding.** $TA_{v,i,n^{\text{start}}}$ — the arrival time at the route's first node — is the dispatch time, the moment from which the route clock runs:

- *Depot origin* ($\xi^{o,D}_{v,i,d}=1$): $TA_{v,i,d} \geq 0$ from the variable domain. The depot carries no time window. Propagation fires on the single outbound arc $d \to (t,1)$, establishing $TA_{v,i,(t,1)} \geq TA_{v,i,d} + \tau^v_{d,(t,1)}$; the terminal's own window in C14 then bounds $TA_{v,i,(t,1)}$ from above.
- *Terminal origin* ($\xi^{o}_{v,i,t}=1$): $z_{v,i,(t,1)}=1$ activates C14's window sandwich directly, bounding $TA_{v,i,(t,1)} \in [EA_{(t,1)}, LA_{(t,1)}]$. Propagation then fires on the first outbound arc from $(t,1)$.

In both cases the dispatch time is the arrival time at the start node itself; no separate variable is needed.

**Coverage — first and last arcs.**

- Depot start → first terminal ($d \to (t,1)$): dispatch time $TA_{v,i,d} \geq 0$; C15 propagates arrival at $(t,1)$; C14 enforces the terminal window.
- Terminal start → first destination ($(t,1) \to j'$): dispatch time grounded by C14's window on $(t,1)$; C15 propagates to $j'$.
- Last customer → depot end ($c \to d$): C15 propagates $TA_{v,i,d} \geq TA_{v,i,c} + s_{v,i,c} + \tau^v_{c,d}$; the depot carries no window.
- Last terminal → depot end ($(t,p) \to d$): C15 propagates $TA_{v,i,d} \geq TA_{v,i,(t,p)} + s_{v,i,(t,p)} + \tau^v_{(t,p),d}$; this is the case in which the day's final reload is followed by a return to base rather than an overnight stay.
- Last node → terminal end ($j \to (t,p)$, overnight stay): C15 propagates arrival; C14 enforces the terminal window even at the overnight stop.

**Purpose.** Propagates a consistent clock along the whole route, accounting for arc-specific speed and quantity-dependent dwell time.

### C16 — Maximum Daily Route Duration

The two binary start/end indicators are combined into a single auxiliary variable to keep the gate at degree 2:

$$\Xi^{o}_{v,i,n} = \begin{cases} \xi^{o,D}_{v,i,n} & n \in D \\ \xi^{o}_{v,i,t} & n = (t,1) \end{cases} \qquad \Xi^{d}_{v,i,n'} = \begin{cases} \xi^{d,D}_{v,i,n'} & n' \in D \\ \xi^{d}_{v,i,(t,p)} & n' = (t,p) \end{cases}$$

**Auxiliary variable** $y_{v,i,n,n'} \geq 0$ — 1 iff $n$ is the route's start and $n'$ is its end:

$$y_{v,i,n,n'} \leq \Xi^{o}_{v,i,n}, \qquad y_{v,i,n,n'} \leq \Xi^{d}_{v,i,n'}, \qquad y_{v,i,n,n'} \geq \Xi^{o}_{v,i,n} + \Xi^{d}_{v,i,n'} - 1 \qquad \forall\ n,n',v,i$$

**Duration gate:**

$$y_{v,i,n,n'}\Bigl(TA_{v,i,n'} - TA_{v,i,n} - \bar{s}_{v,i,n} - T^{max}\Bigr) \leq 0 \qquad \forall\ n \in D\cup\{(t,1):t\in T\},\ n' \in D\cup\hat{T},\ v \in V,\ i \in I_v$$

When $y_{v,i,n,n'}=1$, the gate enforces $TA_{n'} - TA_n - \bar{s}_n \leq T^{max}$. When $y_{v,i,n,n'}=0$, it reads $0 \leq 0$ and is trivially satisfied.

**Purpose.** Caps the elapsed time between the dispatch moment at the route's start and the arrival at its end, regardless of whether those endpoints are depots or terminals.

### C17 — Fleet Size per Type

$$N_v = \sum_{i \in I_v} w_{v,i} \leq m_v \qquad \forall\ v \in V$$

### C18 — Variable Domains

$$x_{v,i,j,j'},\ z_{v,i,j},\ w_{v,i},\ \delta_{f,k,v,i,(t,p)},\ \xi^{o}_{v,i,t},\ \xi^{o,D}_{v,i,d},\ \xi^{d}_{v,i,(t,p)},\ \xi^{d,D}_{v,i,d} \in \{0,1\}$$

$$Ql_{k,f,v,i,(t,p)},\ Lc_{k,f,v,i,j},\ s_{v,i,c},\ s_{v,i,(t,p)},\ q_{c,k,f,v,i},\ TA_{v,i,j},\ u_{v,i,j},\ y_{v,i,n,n'} \geq 0$$

$y_{v,i,n,n'}$ is declared continuous and non-negative; its three defining inequalities in C16 pin it to $\{0,1\}$ at every feasible point without requiring an explicit integrality constraint (Note 13). $u_{v,i,j}$ is likewise continuous; C13's exact bilinear form needs no upper bound on it to remain valid (unlike a big-$M$ linearization of MTZ, which would).

---

## 4. Model Classification

The model is a Mixed-Integer Quadratically-Constrained Program (MIQCP). Every nonlinear term is a bilinear product of a binary variable and a continuous variable (or, for the AND-linearization in C16, of two binaries): the loading gate ( $Ql\cdot(1-\delta)$ ), the $Lc$ propagation ( $x\cdot(Lc_{j'}-Lc_j-\Theta_{j'})$ ), the visit–delivery link ( $q\cdot(1-z)$ ), the visit gate in C12 ( $Ql\cdot(1-z)$ ), MTZ ( $x\cdot(u_{j'}-u_j-1)$ ), time windows ( $z\cdot(TA-EA)$ and $z\cdot(LA-TA)$ ), arrival-time propagation ( $x\cdot(TA_{j'}-TA_j-\bar{s}-\tau)$ ), and the duration gate ( $y\cdot(TA_{n'}-TA_n-\bar{s}-T^{max})$ ). Each is linear in either factor when the other is held fixed, so these are exact reformulations of logical implications, not approximations. The one exception is C12's terminal-stocking gate, $Ql \le M\cdot TF_{f,t}$ : since $TF_{f,t}$ is a fixed parameter, the term is linear to begin with and no bilinear product — and hence no reformulation — is needed there.

---

## 5. Modeling Notes

1. $N_v$ counts distinct vehicles of type $v$; routing variables require the instance index $i$ since two vehicles of the same type need independent routes.
2. $\delta_{f,k,v,i,(t,p)}$ doubles as the day-level fuel-assignment indicator for the compartment: whichever fuel has $\delta=1$ at any slot is that compartment's fuel for the day, and C3 ensures this is at most one fuel across the entire day.
3. Because C4's load-propagation gate is an equality anchored at $Lc_d=0$, $Lc$ is pinned *exactly* along the route rather than merely bounded — this exactness is what makes C5's nonnegativity bound effective at catching over-delivery.
4. A depot never carries fuel under any circumstance, so $Lc_{k,f,v,i,d}=0$ is a flat equality rather than a conditional gate.
5. Slot 1 is the only slot that can ever lack an inbound arc; the slot-ordering rule (C11) means slot 2 always requires slot 1 to have occurred first. The route's destination has no such restriction — it may end on any slot.
6. To keep C16's gate at degree 2, the two binary start/end indicators are first combined into $y$ via an exact linear AND-linearization, and only $y$ is multiplied against the duration expression.
7. $\sum(\text{all origin options}) = w_{v,i} \in \{0,1\}$ rules out any route having more than one origin or more than one destination.
8. The aggregate demand $TQd_f = \sum_c Qd_{f,c}$ is implied by summing C1 over customers and is a reporting definition only.
9. $Lc$ and $TA$ propagate along actual arcs, so slot-visit ordering in $u$ is not required for correctness; adding $u_{v,i,(t,p)} > u_{v,i,(t,p-1)}$ is an optional symmetry-breaking device.
10. $Cd_v$ is multiplied by travel time only, excluding dwell time. If driver pay should cover total elapsed time instead, the driver-cost term should use the same expression as C16.
11. C6 is redundant unless $AC_{f,v}$ is set tighter than the sum of the per-compartment caps in C5.
12. C3's pairwise no-mixing constraints imply the per-slot single-fuel condition as a special case: taking $(t,p)=(t',p')$ shows that loading two different fuels into the same compartment at the same slot would violate $\delta_f + \delta_{f'} \le 1$ directly.
13. The AND-linearization pins $y_{v,i,n,n'}$ to $\min(\Xi^o_n,\Xi^d_{n'}) \in \{0,1\}$ for any binary $\Xi^o,\Xi^d$, with no fractional values possible — so no separate integrality declaration is needed.
14. A route's final stop before returning to a depot may be either a customer or a terminal visit-slot (C9); this is distinct from ending the day by staying overnight at a terminal ($\xi^d=1$), which involves no further movement at all.

---

## 6. Not Yet Modeled

1. **Cross-contamination and compartment-adjacency rules.** Tanker layouts that forbid certain fuel pairs in adjacent compartments, or require a fixed unloading sequence by valve position, are not modeled.
