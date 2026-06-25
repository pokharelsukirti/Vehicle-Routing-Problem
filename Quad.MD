# Heterogeneous Capacitated Vehicle Routing Problem (HCVRP)

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
| $P$ | cap on visit-slots per terminal per vehicle per day — a modeling device (e.g. $P = \lvert C\rvert$), not a real operating limit |
| $M$ | large constant (big-$M$) — **retained only in C13's terminal fuel-stocking gate** ($Ql \le M\cdot TF_{f,t}$), since that gate's condition is a fixed parameter rather than a decision variable, so no bilinear substitute applies; eliminated everywhere else in the model |

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
| $y_{v,i,n,n'} \geq 0$ | **(new)** 1 if vehicle-instance $i$ (type $v$) both starts its route at $n$ *and* ends it at $n'$ that day; equals $\Xi^{o}_{v,i,n}\cdot\Xi^{d}_{v,i,n'}$, pinned to $\{0,1\}$ automatically by its defining inequalities in C17 (see Modeling Note 15). Introduced solely to keep the C17 duration gate at degree 2 instead of degree 3. |

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

*(Unchanged — the objective never contained $M$.)*

---

# Constraints

### C1 — Demand Satisfaction

$$
\sum_{v \in V} \sum_{i \in I_v} \sum_{k \in K_v} q_{c,k,f,v,i} = Qd_{f,c} \qquad \forall\ c \in C,\ \forall\ f \in F
$$

The fuel a customer receives, added up across every vehicle, every instance of that vehicle, and every compartment that visits them, equals exactly what they ordered.

**Purpose:** guarantees every customer's order is fully met, without requiring any single vehicle or compartment to cover the whole thing — this is what makes split delivery possible.

*(Unchanged — no $M$ in this constraint.)*

---

### C2 — Compartment–Fuel Compatibility

$$
\delta_{f,k,v,i} \leq \Delta_{f,k,v} \qquad \forall\ f \in F,\ k \in K_v,\ v \in V,\ i \in I_v
$$

A compartment can only be assigned a fuel it was physically built to carry.

**Purpose:** stops the model from loading a fuel into a compartment that is not engineered for it.

*(Unchanged — no $M$ in this constraint.)*

---

### C3 — Single Fuel per Compartment per Day

$$
\sum_{f \in F} \delta_{f,k,v,i} \leq 1 \qquad \forall\ k \in K_v,\ v \in V,\ i \in I_v
$$

Each compartment is assigned at most one fuel for the whole day.

**Purpose:** enforces the no-mixing rule — a compartment can be topped up with more of its fuel, but never switched to a different one mid-day.

*(Unchanged — no $M$ in this constraint.)*

---

### C4 — Cumulative Load Propagation *(MODIFIED — two big-$M$ pieces converted to quadratic)*

The binary variable $\ell_{v,i,(t,p)} \in \{0,1\}$ indicates whether vehicle-instance $i$ (type $v$) actually loads fuel at visit-slot $(t,p)$. A vehicle that ends its route at a terminal visits that slot ($z_{v,i,(t,p)} = 1$) but must not load fuel it will not deliver ($\ell_{v,i,(t,p)} = 0$). This is enforced by:

$$
\ell_{v,i,(t,p)} \leq z_{v,i,(t,p)} \qquad \forall\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

$$
\ell_{v,i,(t,p)} \leq 1 - \xi^{d}_{v,i,(t,p)} \qquad \forall\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

The first constraint ensures loading can only occur at visited slots. The second prevents loading at a slot that is the route's final destination — if a vehicle parks overnight at a terminal it may not take on fuel it will never deliver. *(These two pieces never used $M$ and are unchanged.)*

The load propagation then uses $\ell$ to gate all loading quantities, now via an exact product instead of a big-$M$ bound:

$$
Ql_{k,f,v,i,(t,p)} \cdot \bigl(1 - \ell_{v,i,(t,p)}\bigr) = 0 \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

**Exactness check:** if $\ell_{v,i,(t,p)}=0$, the product forces $Ql_{k,f,v,i,(t,p)}=0$ — same as $Ql\le M\cdot0$. If $\ell_{v,i,(t,p)}=1$, the equation reads $Ql\cdot0=0$, true for any $Ql\ge0$, leaving the loaded quantity free to be set by C5/C13 exactly as before. This reproduces $Ql\le M\ell$ for every value of $M$, without needing $M$ at all.

The cumulative running load evolves along the route as, with the arc-activation product replacing the big-$M$ deactivation term:

$$
x_{v,i,j,j'}\Bigl(Lc_{k,f,v,i,j'} - Lc_{k,f,v,i,j} - \Theta_{k,f,v,i,j'}\Bigr) \geq 0 \qquad \forall\ (j,j') \in A,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
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

*(The $\Theta$ definition and the depot anchor never used $M$ and are unchanged.)*

**Exactness check:** when $x_{v,i,j,j'}=1$, the inequality reads $Lc_{j'}\ge Lc_j+\Theta_{j'}$ — identical to the big-$M$ version's active case. When $x_{v,i,j,j'}=0$, it reads $0\ge0$, trivially true regardless of $Lc_j, Lc_{j'}$ — exactly the same "no constraint on an unused arc" behavior the big-$M$ term was designed to produce, for any $M$.

As the vehicle moves along its route, a compartment's running fuel level rises by whatever is loaded at a terminal stop (only when $\ell = 1$), falls by whatever is delivered at a customer stop, and is fixed at exactly zero at any depot.

**Purpose:** tracks what is physically in each compartment at every point in the route, and prevents a vehicle from loading fuel at a terminal slot that serves as the overnight end of its route — now enforced by exact bilinear products rather than big-$M$ bounds.

---

### C5 — Per-Compartment Running Capacity

$$
0 \leq Lc_{k,f,v,i,j} \leq Qcap_{v,k} \cdot \delta_{f,k,v,i} \qquad \forall\ j \in \mathcal{N},\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

At every point along the route, a compartment's fuel level stays between empty and its physical capacity.

**Purpose:** prevents overfilling a compartment; the lower bound automatically prevents delivering more than what is currently inside it.

*(Unchanged — no $M$ in this constraint.)*

---

### C6 — Aggregate Vehicle Running Capacity per Fuel

$$
\sum_{k \in K_v} Lc_{k,f,v,i,j} \leq AC_{f,v} \qquad \forall\ j \in \mathcal{N},\ f \in F,\ v \in V,\ i \in I_v
$$

Across all of a vehicle's compatible compartments combined, the running total of a given fuel never exceeds the vehicle's aggregate capacity for it.

**Purpose:** a secondary cap on top of the per-compartment one, useful only if the vehicle's overall carrying capacity for a fuel is set independently of the simple compartment sum (e.g. a weight limit).

*(Unchanged — no $M$ in this constraint.)*

---

### C7 — Load–Delivery Balance

$$
\sum_{t \in T} \sum_{p=1}^{P} Ql_{k,f,v,i,(t,p)} = \sum_{c \in C} q_{c,k,f,v,i} \qquad \forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

Total loaded across every terminal stop equals total delivered across every customer stop, for that compartment, for the day.

**Purpose:** ensures everything loaded into a compartment across the whole day is fully accounted for by deliveries.

*(Unchanged — no $M$ in this constraint.)*

---

### C8 — Visit–Delivery Linking *(MODIFIED — converted to quadratic)*

$$
q_{c,k,f,v,i} \cdot \bigl(1 - z_{v,i,c}\bigr) = 0 \qquad \forall\ c \in C,\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v
$$

A customer can only receive fuel if the vehicle actually visits them.

**Exactness check:** if $z_{v,i,c}=0$ (not visited), the product forces $q_{c,k,f,v,i}=0$ — same as $q\le M\cdot0$. If $z_{v,i,c}=1$, the equation reads $q\cdot0=0$, true for any $q\ge0$. This reproduces $q\le Mz$ for every value of $M$, without needing $M$ at all.

**Purpose:** blocks deliveries to customers who are not on the route — now via an exact bilinear product rather than a big-$M$ bound.

---

### # C9 — Flow Conservation (All Node Types)
Any $(j, j')$ combination not listed in any sub-case below does not belong to $A$ and carries no variable.

---

#### Case 1 — Customer nodes:

$$
\sum_{j'} x_{v,i,j,j'}
\;=\;
\sum_{j'} x_{v,i,j',j}
\;=\;
z_{v,i,j}
\qquad \forall\; j \in C,j' \in C \cup \hat{T}\; v \in V,\; i \in I_v
$$

A customer can be reached from a terminal and other customer and A visited customer has exactly one arc in and one arc out; an unvisited one has none.

---

#### Case 2 — Terminal visit-slot nodes: 

##### Inflow

$$
\sum_{j'} x_{v,i,j',j}
\;=\;
z_{v,i,j} - \xi^{o}_{v,i,t} \cdot [p{=}1]
\qquad \forall\; j=(t,p) \in \hat{T},\; j' \in C \cup \hat{T}\cup D, \; v \in V,\; i \in I_v
$$
A terminal slot may be reached from a customer, another terminal slot, or a depot at route start.

##### Outflow

$$
\sum_{j'} x_{v,i,j,j'}
\;=\;
z_{v,i,j} - \xi^{d}_{v,i,(t,p)}
\qquad \forall\; j=(t,p) \in \hat{T}, \; j' \in C \cup \hat{T}, \; v \in V,\; i \in I_v
$$

A vehicle leaving a terminal mid-route never goes directly to a depot
> When a terminal slot is the route's final destination ($\xi^d_{v,i,(t,p)}=1$),
> the vehicle is present ($z=1$) but no outbound arc exists, giving right-hand side $1 - 1 = 0$ and similarly no inbound arc exist for inflow.

---

#### Case 3 — Depot nodes:

##### Outflow

$$
\sum_{j'} x_{v,i,j,j'}
\;=\;
\xi^{o,D}_{v,i,j}
\qquad \forall\; j \in D,\;j' \in \hat{T},\; v \in V,\; i \in I_v
$$
A depot is always a route's start; the vehicle's first move must be to a terminal to load before any delivery;no depot-to-customer or depot-to-depot arc exists.

##### Inflow

$$
\sum_{j} x_{v,i,j',j}
\;=\;
\xi^{d,D}_{v,i,j}
\qquad \forall\; j \in D,\; j' \in C \; v \in V,\; i \in I_v
$$
A depot may only be entered from a customer.

---

#### Single Origin and Single Destination per Dispatched Vehicle

$$
\sum_{d \in D} \xi^{o,D}_{v,i,d} + \sum_{t \in T} \xi^{o}_{v,i,t} = w_{v,i}
\qquad \forall\; v \in V,\; i \in I_v
$$

$$
\sum_{d \in D} \xi^{d,D}_{v,i,d} + \sum_{t \in T} \xi^{d}_{v,i,t} = w_{v,i}
\qquad \forall\; v \in V,\; i \in I_v
$$

---

### C11 — Terminal Eligibility

$$
\xi^{o}_{v,i,t} \leq E_t, \qquad \xi^{d}_{v,i,t} \leq E_t \qquad \forall\ t \in T,\ v \in V,\ i \in I_v
$$

A terminal can only be used as a route's start or end if it is flagged eligible.

**Purpose:** keeps certain terminals reserved for mid-route reloads only, never as a place a route begins or ends.

*(Unchanged — no $M$ in this constraint.)*

---

### C12 — Terminal Slot Ordering

$$
z_{v,i,(t,p)} \leq z_{v,i,(t,p-1)} \qquad \forall\ t \in T,\ p = 2,\dots,P,\ v \in V,\ i \in I_v
$$

A terminal's 2nd, 3rd, etc. visit-slot cannot be used unless the one before it was used too.

**Purpose:** stops the solver from representing one real visit in many meaninglessly different ways.

*(Unchanged — no $M$ in this constraint.)*

---

### C13 — Loading Gating *(MODIFIED — one of three pieces converted to quadratic; one kept as big-$M$; one already $M$-free)*

$$
Ql_{k,f,v,i,(t,p)} \leq M \cdot TF_{f,t}, \qquad Ql_{k,f,v,i,(t,p)} \cdot \bigl(1 - z_{v,i,(t,p)}\bigr) = 0, \qquad Ql_{k,f,v,i,(t,p)} \leq Qcap_{v,k} \cdot \delta_{f,k,v,i}
$$

$$
\forall\ k \in K_v,\ f \in F,\ v \in V,\ i \in I_v,\ t \in T,\ p = 1,\dots,P
$$

A vehicle cannot load a fuel the terminal does not stock, cannot load at a visit-slot it is not actually using, and cannot load a fuel into a compartment not assigned to carry it that day.

**Why the first piece keeps big-$M$:** $TF_{f,t}$ is a fixed input *parameter* (whether terminal $t$ stocks fuel $f$), not a decision variable — there is no second decision variable to multiply $Ql$ against, so no meaningful bilinear substitute exists. (A tighter bound without going quadratic is possible — replacing $M$ with the constant $Qcap_{v,k}$ — but that is a linear tightening, not a quadratic reformulation, so the piece is left exactly as originally written.)

**Exactness check (second piece):** if $z_{v,i,(t,p)}=0$, the product forces $Ql_{k,f,v,i,(t,p)}=0$ — same as $Ql\le M\cdot0$. If $z_{v,i,(t,p)}=1$, the equation reads $Ql\cdot0=0$, true for any $Ql\ge0$. This reproduces $Ql\le Mz$ for every value of $M$, without needing $M$ at all.

*(The third piece never used $M$ — it was already a tight bound on $Qcap_{v,k}$ — and is unchanged.)*

**Purpose:** three independent checks that keep every loading event physically realistic — one still a (parameter-gated) linear big-$M$ bound, one now an exact bilinear product, and one already a tight linear bound with no $M$ to begin with. Note that the $\ell$-based gate in C4 ($Ql\cdot(1-\ell)=0$) further prevents loading at the terminal slot that ends the route; together with the visit-gating here, all three conditions must hold simultaneously.

---

### C14 — Subtour Elimination (MTZ) *(MODIFIED — converted to quadratic)*

$$
x_{v,i,j,j'}\bigl(u_{v,i,j'} - u_{v,i,j} - 1\bigr) \geq 0 \qquad \forall\ j \neq j' \in \mathcal{N},\ v \in V,\ i \in I_v
$$

Every customer and terminal visit-slot gets a position value; if an arc is used, the destination's value must be consistent with coming after the origin's. Depots are excluded — they are always a route's boundary, never part of an internal loop.

**Exactness check:** if $x_{v,i,j,j'}=1$, the inequality reads $u_{j'}-u_j-1\ge0$, i.e. $u_{j'}\ge u_j+1$ — identical to the active case of $u_j-u_{j'}+Mx\le M-1$. If $x_{v,i,j,j'}=0$, it reads $0\ge0$, trivially true for any $u_j,u_{j'}$ — the same "no constraint on an unused arc" behavior the big-$M$ version relied on $M$ being large enough to produce.

**Purpose:** prevents the solver from forming a closed loop of customers or terminal slots that is disconnected from the rest of the route — now enforced exactly, without needing $M$ to be "large enough."

---

### C15 — Time Windows *(MODIFIED — node-window sandwich converted to quadratic; depot row never had $M$ and is unchanged)*

For any visited node $j \in \mathcal{N}$:

$$
z_{v,i,j}\bigl(TA_{v,i,j} - EA_j\bigr) \geq 0 \qquad\text{and}\qquad z_{v,i,j}\bigl(LA_j - TA_{v,i,j}\bigr) \geq 0 \qquad \forall\ j \in \mathcal{N},\ v \in V,\ i \in I_v
$$

$$
EA_d \leq TA_{v,i,d} \leq LA_d \qquad \forall\ d \in D,\ v \in V,\ i \in I_v \quad \text{(active only when } \xi^{o,D}_{v,i,d} = 1 \text{ or } \xi^{d,D}_{v,i,d} = 1\text{)}
$$

> $j \in \mathcal{N} = C \cup \hat{T}$: covers customers and terminal visit-slots. Depots have their window enforced directly since every dispatched vehicle touches exactly one origin depot or one destination depot (or both). This depot row contains no $M$ in the original document and is reproduced unchanged.

**Exactness check:** if $z_{v,i,j}=1$, the two inequalities read $TA_{v,i,j}\ge EA_j$ and $TA_{v,i,j}\le LA_j$ — the window is enforced exactly, identical to the active case of the big-$M$ sandwich. If $z_{v,i,j}=0$, both read $0\ge0$, trivially true for any $TA_{v,i,j}$ — the window is fully deactivated for an unvisited node, the same outcome the big-$M$ relaxation produced for every value of $M$.

**Coverage check — start and end of route:** *(unchanged reasoning — it refers to C16, which is converted below in the same exact, case-matching way, so the coverage argument still holds.)*

- **Depot to first terminal** $(d \to (t,1))$: the depot window constrains $TA_{v,i,d}$ (departure point). The terminal window constrains $TA_{v,i,(t,1)}$. Arrival-time propagation (C16) links the two via travel time $\tau^v_{d,(t,1)}$, so the departure from the depot and the arrival at the first terminal are both window-checked.

- **Last node to terminal end-of-route** $(c \to (t,p))$ or $((t,p) \to (t',p'))$: arrival at the terminal end-slot $(t,p)$ is governed by the same terminal-slot window constraint above ($j = (t,p) \in \hat{T}$), so the window is enforced even when that slot is the route's final destination.

- **Customer to depot** $(c \to d)$: arrival at the destination depot is checked by the depot row. C16 links $TA_{v,i,d} \geq TA_{v,i,c} + s_{v,i,c} + \tau^v_{c,d}$, so travel from the last customer into the depot is clock-consistent and window-constrained.

Every leg of the trip — depot-to-first-terminal, customer-to-terminal, terminal-to-terminal, customer-to-depot — is therefore covered.

**Purpose:** keeps the route consistent with customer availability and terminal operating hours across the entire journey from origin to destination.

---

### C16 — Arrival-Time Propagation *(MODIFIED — propagation inequality converted to quadratic)*

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

*(The travel-time and dwell-time definitions never used $M$ and are unchanged.)*

Propagation along the route for any arc $(j,j') \in A$, $j,j' \in \mathcal{N} \cup D$:

$$
x_{v,i,j,j'}\Bigl(TA_{v,i,j'} - TA_{v,i,j} - \bar{s}_{v,i,j} - \tau^v_{j,j'}\Bigr) \geq 0
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

*(The $\bar s$ definition never used $M$ and is unchanged.)*

Arrival time at the next stop is whatever time the vehicle arrived at the current stop, plus however long it spends there, plus the travel time to the next node — whenever the arc connecting them is actually used.

**Exactness check:** if $x_{v,i,j,j'}=1$, the inequality reads $TA_{j'}\ge TA_j+\bar s_j+\tau^v_{j,j'}$ — identical to the active case of the big-$M$ version. If $x_{v,i,j,j'}=0$, it reads $0\ge0$, trivially true for any $TA_j, TA_{j'}$ — the same "no constraint on an unused arc" behavior, for every value of $M$.

**Purpose:** propagates a consistent clock along the whole route, accounting for speed that can vary by arc and vehicle type, and dwell time that can vary by quantity handled — now enforced exactly rather than via a big-$M$ relaxation.

---

### C17 — Maximum Daily Route Duration (10 hours) *(MODIFIED — converted to quadratic via a new auxiliary variable)*

This is the one constraint where directly mirroring the C14/C16-style trick would not stay quadratic: the gate here is a *conjunction of two separate binary indicators* ($\Xi^o_{v,i,n}$ and $\Xi^d_{v,i,n'}$), not a single one. Multiplying both binaries *and* the duration expression together in one constraint would be degree 3, not degree 2. To keep every individual constraint genuinely quadratic, the two binaries are first combined into one auxiliary variable using an exact **linear** AND-linearization (no $M$, no quadratic term needed there), and only that single auxiliary variable is then multiplied against the duration expression.

**New auxiliary variable:**

$$
y_{v,i,n,n'} \geq 0 \qquad \forall\ n \in D\cup\{(t,1):t\in T\},\ n' \in D\cup\hat{T},\ v \in V,\ i \in I_v
$$

representing whether $n$ is vehicle-instance $i$'s actual route start *and* $n'$ is its actual route end, that day.

**Exact linear AND-linearization (no big-$M$):**

$$
y_{v,i,n,n'} \leq \Xi^{o}_{v,i,n}, \qquad y_{v,i,n,n'} \leq \Xi^{d}_{v,i,n'}, \qquad y_{v,i,n,n'} \geq \Xi^{o}_{v,i,n} + \Xi^{d}_{v,i,n'} - 1
$$

$$
\forall\ n \in D\cup\{(t,1):t\in T\},\ n' \in D\cup\hat{T},\ v \in V,\ i \in I_v
$$

**Quadratic duration gate:**

$$
y_{v,i,n,n'}\Bigl(TA_{v,i,n'} - TA_{v,i,n} - \bar{s}_{v,i,n} - T^{max}\Bigr) \leq 0
$$

$$
\forall\ n \in D\cup\{(t,1):t\in T\},\ n' \in D\cup\hat{T},\ v \in V,\ i \in I_v,\quad T^{max} = 10 \text{ hours}
$$

where, exactly as in the original document,

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

**Exactness check:** the three AND-linearization inequalities pin $y_{v,i,n,n'}$ to exactly $\min(\Xi^o_{v,i,n}, \Xi^d_{v,i,n'}) \in \{0,1\}$ for any binary $\Xi^o,\Xi^d$ — see Modeling Note 15. Then: if $y_{v,i,n,n'}=1$ (both "$n$ is the start" and "$n'$ is the end" hold), the gate reads $TA_{n'}-TA_n-\bar s_n \le T^{max}$ — identical to the active case of the original big-$M$ version. If $y_{v,i,n,n'}=0$, it reads $0\le0$, trivially true for any $TA_n, TA_{n'}$ — the same "inactive unless this pair is the actual start/end" behavior, for every value of $M$.

**Purpose:** caps how long a vehicle's day can run, regardless of whether that day started and ended at a depot, a terminal, or one of each — now enforced exactly, with the conjunction of start- and end-indicators resolved by an auxiliary variable rather than a big-$M$ slack.

---

### C18 — Fleet Size per Type

$$
N_v = \sum_{i \in I_v} w_{v,i} \leq m_v \qquad \forall\ v \in V
$$

The number of vehicles of a type actually used cannot exceed how many of that type exist.

**Purpose:** caps fleet usage at what is physically available.

*(Unchanged — no $M$ in this constraint.)*

---

### C19 — Variable Domains *(MODIFIED — domain added for the new auxiliary variable only)*

$$
x_{v,i,j,j'},\ z_{v,i,j},\ w_{v,i},\ \delta_{f,k,v,i},\ \ell_{v,i,(t,p)},\ \xi^{o}_{v,i,t},\ \xi^{o,D}_{v,i,d},\ \xi^{d}_{v,i,(t,p)},\ \xi^{d,D}_{v,i,d} \in \{0,1\}
$$

$$
Ql_{k,f,v,i,(t,p)},\ Lc_{k,f,v,i,j},\ s_{v,i,c},\ s_{v,i,(t,p)},\ q_{c,k,f,v,i},\ TA_{v,i,j},\ y_{v,i,n,n'} \geq 0
$$

States which variables are binary decisions and which are non-negative quantities. The new auxiliary variable $y_{v,i,n,n'}$, introduced only by the C17 quadratic reformulation, is declared continuous and non-negative rather than binary — its three defining inequalities pin it to $\{0,1\}$ at every feasible point regardless (see Modeling Note 15), so no extra integrality is required.

**Purpose:** keeps the model a clean Mixed-Integer Quadratically-Constrained Program.

---

# Linearity / Quadratic Check

The model is now a **Mixed-Integer Quadratically-Constrained Program (MIQCP)**, not a pure MILP. Every former big-$M$ conditional relationship whose gating object was a *decision variable* — the loading gate, the $Lc$ propagation, the visit–delivery link, the visit gate inside C13, MTZ subtour elimination, the time-window sandwich, arrival-time propagation, and the duration cap — is now written as an exact bilinear (degree-2) constraint: either a product equated to zero ($y\cdot(1-b)=0$) or a product signed against zero ($b\cdot(\text{expr})\ge0$ or $\le0$).

Every one of these products is **binary × continuous** — never continuous × continuous — which keeps the nonlinearity to the mildest possible kind: each constraint is linear in either variable held fixed, with the binary variable acting purely as a degree-2 on/off multiplier. This is what makes each reformulation *exact* rather than a relaxation or approximation: a binary variable has only two possible values, so "the product is zero / the inequality collapses to $0\ge0$" is a complete case analysis, not an approximation valid only for large $M$.

The one exception, kept in its original linear big-$M$ form, is C13's terminal-stocking gate ($Ql \le M\cdot TF_{f,t}$): $TF_{f,t}$ is a fixed input parameter, not a decision variable, so there is no second decision variable to multiply against — converting it would not introduce a "product of two decision variables" at all, just relabel an already-linear bound. It is therefore left untouched, exactly as the task requires.

The two dwell-time definitions ($s_{v,i,c} = \gamma\sum q$, $s_{v,i,(t,p)} = \beta\sum Ql$) and the driver-cost term ($Cd_v\cdot\tau^v_{j,j'}\cdot x_{v,i,j,j'}$) remain exactly as before — a fixed constant multiplying a single decision variable — unaffected by this conversion, since they never contained $M$.

The new auxiliary variable $y_{v,i,n,n'}$ (introduced solely to keep C17's gate at degree 2 instead of degree 3) is linked to $\Xi^o, \Xi^d$ by purely **linear** inequalities — no quadratic terms and no big-$M$ appear in its definition — and only the final duration gate involving $y$ is quadratic.

Things that would **not** stay at this same "mild" quadratic level if added later: any constraint multiplying two *continuous* decision variables together (e.g. a running cost depending on both distance and a continuously-variable carried weight) is a true bilinear term without one side pinned to $\{0,1\}$, and is materially harder to bound/relax than the binary × continuous products used throughout this reformulation; likewise a penalty whose priority weight is itself a continuous decision variable.

---

# Modeling Notes

1. **Type vs. instance.** $N_v$ only makes sense if it counts *distinct trucks of type $v$*; routing variables cannot be indexed by type alone, since two trucks of the same type need independent routes. Hence $I_v$ and $i$.

2. **$\delta$ vs. $\Delta$.** "Is this compartment built for this fuel" ($\Delta$, fixed) and "is this compartment currently assigned this fuel" ($\delta$, a daily decision) are different things, kept as separate symbols.

3. **Why $Lc$ propagation only needs one direction.** A lower-bound-only chain on $Lc$, combined with the upper bound in C5, pins $Lc$ to its true value at any optimum — inflating $Lc$ only makes C5 harder to satisfy, never easier, so there is no incentive to misreport it. This argument is unaffected by the C4 quadratic conversion: the new $Lc$ constraint is still a lower-bound-only chain, just gated by a product instead of a big-$M$ term.

4. **Why $Lc = 0$ at depots is a flat equality, not big-$M$ gated.** Unlike a terminal slot (which only sometimes is the route's start), a depot never carries fuel under any circumstance, so its $Lc$ is fixed at zero outright rather than conditioned on $\xi^{o,D}$.

5. **Origin is always slot 1; destination is tracked per-slot.** Slot 1 is the only slot that could ever have nothing before it (the slot-ordering rule means slot 2 always requires slot 1 to have already happened). The destination has no such restriction, since a route can legitimately end on a terminal's 2nd, 3rd, etc. visit.

6. **Why the duration cap (C17) does not need separate $Dep$/$Ret$ variables — and why it now needs $y$.** Referencing $TA$ directly, gated by whichever origin/destination indicator applies, avoids the need for separate per-leg bookkeeping variables. In this reformulation the gate is itself a $\xi\cdot TA$-style product — by design, since removing $M$ here specifically requires that product. To keep the product at degree 2 rather than degree 3 (which a direct $\Xi^o_n\cdot\Xi^d_{n'}\cdot TA$ substitution would create, since two binaries and one continuous variable would all be multiplied at once), the two binary indicators are first combined into a single auxiliary variable $y_{v,i,n,n'}$ via an exact linear AND-linearization, and only then is $y$ multiplied against the duration expression.

7. **"Exactly one origin/destination" already rules out using two terminals at once.** Because $\sum(\text{all origin options}) = w_{v,i}$ and $w_{v,i} \in \{0,1\}$, the terminal-only portion can never exceed 1 on its own. Same logic applies to the destination side.

8. **The aggregate demand identity is implied, not independent.** C1 already states the per-customer version; summing over every customer produces the aggregate version automatically.

9. **Slot-visit-order matching slot number is not required for correctness, only optionally for solver speed.** $Lc$ and $TA$ both propagate along actual arcs, not slot labels, so nothing breaks if the solver's internal numbering of "1st vs. 2nd visit" does not match real-world intuition. Adding $u_{v,i,(t,p)} > u_{v,i,(t,p-1)}$ as an extra symmetry-breaking constraint is optional, not required.

10. **Driver cost uses pure travel time, not total elapsed time.** $Cd_v$ is multiplied by $\sum \tau^v_{j,j'} x_{v,i,j,j'}$ — moving time only, excluding dwell time. This is a deliberate reading of "time travelled"; if driver pay should cover the whole day including dwell time, the driver-cost term would need to use the same elapsed-time expression as C17.

11. **C6 redundancy.** As currently defined, $AC_{f,v}$ is just the sum of the per-compartment caps already in C5, so C6 adds nothing unless $AC_{f,v}$ is set independently tighter (e.g. a weight-based de-rating).

12. **$TQd_f$** remains a definition for reporting/sanity-checks, not an independent constraint.

13. **Visit vs. load distinction ($z$ vs. $\ell$).** $z_{v,i,(t,p)} = 1$ means the vehicle is physically present at the terminal slot. $\ell_{v,i,(t,p)} = 1$ is the further condition that it actually takes on fuel there. The two can differ only when the slot is the route's overnight end — in all other cases the solver will set $\ell = z$ at optimum because loading is free (or beneficial) and no constraint forces $\ell = 0$ at non-terminal slots.

14. **Arc-domain explicitness in C9 and C10.** Writing $j' \in C \cup \hat{T}$ (customer flow), $j' \in C \cup \hat{T} \cup D$ (terminal inflow), $j' \in C \cup \hat{T}$ (terminal outflow), $j' \in \hat{T}$ (depot outflow), $j' \in C \cup \hat{T}$ (depot inflow) makes the structural infeasibilities visible in the constraint index sets and eliminates the need for those arc types to be absent from $A$ for correctness — though keeping them absent from $A$ as well speeds up the solver by reducing variable count.

15. **(New) The AND-linearization for $y_{v,i,n,n'}$ needs no big-$M$ and no extra integrality.** Because $\Xi^o_{v,i,n}, \Xi^d_{v,i,n'}\in\{0,1\}$ are already binary, the three inequalities $y\le\Xi^o_n$, $y\le\Xi^d_{n'}$, $y\ge\Xi^o_n+\Xi^d_{n'}-1$ pin $y$ to exactly $\min(\Xi^o_n,\Xi^d_{n'})\in\{0,1\}$ at every feasible point — never a fractional value — regardless of whether $y$ itself is declared binary or simply $y\ge0$. This mirrors Modeling Note 3's logic: the value is forced to its correct value by the surrounding inequalities, so no separate integrality requirement is needed.

---

# Not Yet Modeled

1. **Fuel-specific / safety: cross-contamination and compartment-adjacency rules.** Tanker layouts that forbid certain fuel pairs in adjacent compartments, or require a fixed unloading sequence by valve position, are not modeled. To be added only after the approach is confirmed.
