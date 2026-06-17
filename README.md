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
   Vehicles in the fleet differ from one another in maintenance cost, speed, and fuel consumption per kilometre. They are categorized into 5 category based on these parameters

3. **Depot Return Policy**
   Every vehicle must return to the depot at the end of each day's trip.

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

## Constraints

To be added later... 

---

*This README serves as a reference document for the presumptions, parameters, and constraints of the HCVRP fuel distribution model.*
