# HCVRP Fuel Distribution Model
### Heterogeneous Capacitated Vehicle Routing Problem
#### Model Overview & Motivation

*A Mixed-Integer Quadratically Constrained Programming Framework for Optimal Fuel Distribution Logistics*

*Warning:* This repository is a work in progress. The model is in active development, and you may encounter inaccuracies as I work towards an improved system.

---

## 1. Industry Context and Motivation

The distribution of liquid petroleum products — petrol, diesel, aviation fuel, and heating oil — represents one of the most logistically demanding sectors in modern supply chains. Unlike standard freight, fuel delivery imposes a unique convergence of physical constraints, safety regulations, multi-product complexity, and time-sensitive customer requirements that make generic routing models inadequate.

Fuel distribution companies must simultaneously optimise across four deeply interlinked operational dimensions: transportation logistics, inventory management at supply terminals, supply chain coordination across the network, and fleet management across a heterogeneous vehicle pool. Fuel distribution is further complicated by a set of operational constraints. Customers and terminals must be visited within strict time windows reflecting customer availability and terminal operating hours respectfully. Driver hours are regulated. Vehicle compartments carry strict fuel compatibility constraints, not every compartment can carry every fuel type, and cross-contamination between fuel grades is a safety and quality violation.

Environmental and energy efficiency objectives are increasingly incorporated alongside economic goals, reflecting sustainability priorities in the sector. Route length minimisation, fuel consumption reduction, and the matching of vehicle size to delivery volume all contribute to emissions reduction and regulatory compliance, adding a further dimension to the optimisation problem beyond pure cost.

**Why Optimisation Matters in Fuel Distribution**

- In the petroleum distribution industry, transport activities typically account for between 40% and 60% of total operating costs.
- Fuel is a high-volume, low-margin business, even a 2% cost saving across thousands of daily deliveries is significant. 
- A modest 5% improvement in route efficiency translates directly into significant annual savings.
- Commercial planning systems such as AMCS Fuel Planner demonstrate that route optimisation software can reduce operational costs by 5–15%, increase delivered volume per driven kilometre by a similar margin, and cut planning time by 25–50%.

  At its structural core, this problem is a Multi-Pickup, Multi-Delivery Problem with a Heterogeneous Vehicle Fleet and Time Windows (MPDP-HF-TW). Each    vehicle may visit multiple fuel terminals to load (pickups) and multiple customers to unload (deliveries), in any feasible interleaved order, using a    fleet of trucks that differ in compartment count, compartment capacity, fuel compatibility, speed, and operating cost. This is among the most complex    vehicle routing variants in the operations research literature. 
---

## 2. Model Key Features

The table below summarises the defining features of the HCVRP model. Each feature reflects a real operational constraint or decision that must be represented faithfully to produce actionable routing plans for petroleum distribution.

| Feature | Detail |
|---------|--------|
| **Heterogeneous fleet** | Vehicles differ in compartment count, per-compartment capacities, cost rates (maintenance, fuel-consumption, driver), and arc-specific speed |
| **Multi-product, multi-compartment** | Each compartment carries exactly one fuel type per day; mid-day top-up of the same fuel is allowed |
| **Multi-pickup, multi-delivery** | Each vehicle may visit multiple terminals (pickups) and multiple customers (deliveries), interleaved in any feasible order — a true MPDP structure |
| **Flexible endpoints** | Start and end nodes are independently chosen from depots or eligible terminals; they need not match — enabling overnight parking at terminals |
| **Mid-route reloads** | Any terminal may be visited multiple times within a day (visit-slots capped at P per vehicle per terminal per day) |
| **Split delivery** | A customer's demand for a fuel may be split across multiple vehicles and/or compartments |
| **Visit vs. load distinction** | Binary z records whether a terminal slot was visited; binary l separately records whether fuel was actually loaded — a vehicle parked overnight visits but does not load |
| **Route-end no-loading** | A vehicle whose route ends at a terminal cannot load fuel it will not deliver — enforced via the l variable gating all loading quantities |
| **Terminal eligibility** | Only terminals with E = 1 may serve as route start or end; any terminal may be used for mid-route reloads regardless of eligibility flag |
| **Compartment fuel-identity lock** | Once a compartment is assigned a fuel for the day, it cannot switch fuels; it may only be topped up with more of the same fuel |
| **Arrival time windows** | Customer and terminal arrival windows enforced via big-M constraints; windows apply to every visited node including depots at route boundaries |
| **Arrival-time propagation** | A consistent clock is maintained along the entire route, with travel time varying by vehicle type and arc, and dwell time varying by quantity handled |
| **Quantity-dependent dwell time** | Time spent at a terminal or customer depends on how much is loaded or unloaded — not a flat constant — making the schedule physically accurate |
| **Maximum daily duration** | Total elapsed time from route start to route end is capped at T_max = 10 hours |
| **MTZ subtour elimination** | Miller-Tucker-Zemlin constraints applied over the visit-slot replicated node set N = C ∪ T̂, using big-M, guaranteeing route connectivity |
| **No mid-route depot returns** | Depots are strictly route boundaries — a vehicle cannot return to a depot as an intermediate stop; only as route start or end |
| **Compartment running-load tracking** | Continuous variable Lc tracks fuel level in each compartment at every node, enabling partial deliveries and top-ups to be represented exactly |
| **Driver cost on travel time only** | Driver pay covers moving time only (not dwell time), matching a typical per-hour-driven pay structure — a deliberate modelling choice that can be altered |
| **Terminal-stay fixed cost** | An overnight stop at a terminal incurs a fixed fee FT; ending at a depot does not |
| **Fleet size enforcement** | Number of vehicles dispatched per type cannot exceed the available physical fleet m_v |

---
## 3. Problem Description

This fuel distribution model addresses the daily operational planning problem faced by a petroleum distributor operating across a network of supply terminals, customer sites, and vehicle depots. The planning horizon is a single operational day. Each dispatched vehicle completes one route, departing from an origin node (a depot or terminal) and arriving at a destination node (a depot or eligible terminal), with any number of terminal reload stops and customer delivery stops interleaved along the way.

### 3.1 Network Structure

The distribution network consists of three distinct node types, each playing a different operational role:

| Node Type | Role in Model | Key Constraints |
|-----------|--------------|-----------------|
| **Depots** | Route origin/destination only; never intermediate stops | No fuel stocked; Lc = 0 always; no fixed stay cost |
| **Terminals** | Fuel supply points; mid-route reload stops; may be eligible as route endpoints | Stock specific fuels (TF); eligibility flag (E); fixed overnight cost (FT); visit-slots up to P per vehicle |
| **Customers** | Fuel demand points; mid-route delivery stops only | Demand per fuel type; time windows; visited at most once per vehicle per day; split delivery allowed |

### 3.2 Vehicle Fleet

The fleet is heterogeneous — vehicles are grouped into types, and within each type all operational parameters are identical. Across types they differ in the number of compartments, per-compartment capacities, fuel compatibility of each compartment, maintenance cost per kilometre, fuel consumption cost per kilometre, driver cost rate, and arc-specific average speed. Each vehicle type has a fixed fleet size limiting the number of instances that can be dispatched in a day.

Every vehicle carries multiple compartments. Each compartment is physically engineered to carry a subset of fuel types (a fixed technical parameter). IN this problem each day, a compartment is assigned at most one fuel type for the entire day, i.e it cannot switch fuels mid-route. However, it may be topped up at a subsequent terminal stop with more of the same fuel. This compartment-level granularity is what allows a single tanker truck to simultaneously carry petrol, diesel, and aviation fuel in separate compartments on a single route, serving different customers with different products.

### 3.3 The Core Optimisation Problem

Given the network, fleet, and customer demands, the model simultaneously determines:

- Which vehicles to dispatch and which to leave idle
- Where each vehicle's route starts and ends (depot or eligible terminal)
- The complete ordered sequence of terminal reload stops and customer delivery stops for each vehicle
- Which fuel is assigned to each compartment of each vehicle for the day
- How much fuel is loaded at each terminal visit, into which compartment
- How much fuel is delivered to each customer from which compartment of which vehicle
- The arrival time at every node on every route

All decisions are made simultaneously to minimise total cost comprising driving costs (maintenance and fuel consumption), purchased fuel costs (which vary by terminal and fuel type), driver time costs, and fixed terminal overnight stay costs.

---

## 4. Objective Function

The model minimises total daily operating cost across the entire fleet. The objective has four components, each capturing a distinct cost driver in petroleum logistics:

| Cost Component | What It Captures | Why It Matters |
|----------------|-----------------|----------------|
| **Driving Cost** | (Maintenance + fuel consumption) per km driven, summed over all arcs and vehicles | Directly penalises unnecessary distance — the primary lever for route optimisation |
| **Fuel Purchase Cost** | Price of delivered fuel multiplied by quantity loaded, at each terminal visited | Fuel prices vary by terminal — the model selects the cheapest terminal to load from, balancing price against route detour cost |
| **Driver Cost** | Driver wage rate multiplied by time spent travelling (moving time only) | Captures labour cost; separating it from dwell time matches industry pay structures |
| **Terminal-Stay Fixed Cost** | Fixed fee incurred when a route ends at a terminal overnight, rather than returning to depot | Reflects real parking and administration costs at terminals; the model chooses endpoints to balance this against route length |

*There is no separate per-vehicle dispatch fee — running a vehicle at all is already captured by the driving, driver, and fuel purchase components. The model naturally selects the fewest vehicles necessary to serve demand at minimum total cost.*

---

## 5. Formulation Approach
The simplest version of vehicle routing, just one vehicle, one product, no time windows, is already the Travelling Salesman Problem (TSP), which is NP-hard. The problem is also NP-hard because it combines TSP-style sequencing, bin-packing-style assignment, and scheduling with time windows into a single tightly coupled model where none can be solved independently of the others. The size of the search space grows faster than any polynomial in the number of customers, vehicles, and terminals, making it impossible to guarantee an optimal solution in polynomial time unless P = NP. For small to medium instances (up to approximately 25–30 customers, 3–5 terminals, 6–8 vehicles), modern solvers can find proven optimal solutions within acceptable time limits. 

---

