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
9. **Depot** — A depot never stocks fuel and is never an intermediate stop; a vehicle uses a depot at most once as its start and at most once as its end, per day. Consequently, arcs from a terminal (or customer) back to a depot mid-route are infeasible — a depot can only be reached as the route's final destination. Using a depot as an endpoint carries no fixed cost. Ending a route at a terminal instead does carry one ($FT_t$).

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
| $(j,j') \in A,\ A\subseteq(\mathcal N\cup D)\times(\mathcal N\cup D)$ | feasible arcs. Depot-to-depot, depot-to-customer, and terminal-to-depot arcs are excluded. A vehicle can't deliver before loading; it never travels directly between two depots; and once it departs from a terminal it either continues to a customer or another terminal — returning to a depot mid-route is impossible since a depot is only ever a route's start or end, never an intermediate stop (Presumption 9). |

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
