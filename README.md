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
   Vehicles in the fleet differ from one another in maintenance cost, speed, and fuel consumption per kilometre. No two vehicle types are treated as identical in operational cost structure.

3. **Depot Return Policy**
   Every vehicle must return to the depot at the end of each day's trip, regardless of route completion status or remaining capacity.

4. **Load-then-Deliver Workflow**
   Vehicles first load fuel at a terminal and then travel to customer locations for delivery. No en-route loading is permitted.

5. **Vehicle Size Scaling**
   Larger vehicles incur higher fuel consumption per kilometre and higher driver costs. Operational costs scale proportionally with vehicle size.

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

## Vehicle Parameters

Each vehicle in the fleet is characterised by the following attributes:

- **Maintenance Cost** — Periodic cost incurred to keep the vehicle operational
- **Speed** — Travel speed, which affects route duration and time window compliance
- **Fuel Consumption per km** — Distance-based consumption rate; increases with vehicle size
- **Driver Cost** — Labour cost associated with operating the vehicle; increases with vehicle size
- **Compartment Configuration** — Determines which fuel type(s) the vehicle can carry

---

## Constraints

The HCVRP model operates under the following hard constraints:

### 1. Demand Satisfaction
All customer orders must be fulfilled in full. Partial deliveries are not permitted unless explicitly modelled.

### 2. Vehicle Selection
Vehicles are selected based on:
- **Order quantity** — The vehicle must have sufficient capacity to carry the requested volume.
- **Order type** — The vehicle must be appropriate for the category of fuel being delivered.

### 3. Fuel–Compartment Compatibility
Each fuel type must be loaded into and delivered from a compatible compartment. Incompatible fuel-compartment pairings are strictly prohibited to ensure safety and regulatory compliance.

### 4. Time Window Compliance
Each customer delivery must occur within a specified time window. Routes must be planned to ensure vehicles arrive no earlier than the opening time and no later than the closing time of each customer's window.

---

## Summary

| Category | Detail |
|----------|--------|
| Fleet Type | Heterogeneous (mixed vehicle sizes and types) |
| Fuel Types | Petrol, Diesel, Kerosine |
| Depot Return | Required daily |
| Load Source | Terminal only |
| Key Constraints | Demand, Vehicle Selection, Compatibility, Time Windows |

---

*This README serves as a reference document for the presumptions, parameters, and constraints of the HCVRP fuel distribution model.*
