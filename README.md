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

## Vehicle Types
To be added later.

# Notation

## Indices and Sets

| Symbol | Description |
|---|---|
| v<sub>1</sub>, ⋯, V | set of vehicles in the transportation fleet. |
| k<sub>1</sub>, ⋯, K | set of compartments within a vehicle. |
| f<sub>1</sub>, ⋯, F | set of fuel types. |
| c<sub>1</sub>, ⋯, C | set of customers. |
| t<sub>1</sub>, ⋯, T | set of terminals. |
| j | generic index for a node, where a node is either a terminal or a customer location. |

## Parameters

| Symbol | Description |
|---|---|
| Qcap<sub>v,k</sub> | capacity of compartment k in vehicle v. |
| AC<sub>f,v</sub> | alternative capacity available for fuel f in vehicle v. |
| Qd<sub>f,c</sub> | quantity of fuel f demanded by customer c. |
| TQd<sub>f</sub> | total demanded quantity of fuel f across all customers. |
| EA<sub>j</sub> | earliest acceptable arrival time at node j. |
| LA<sub>j</sub> | latest acceptable arrival time at node j. |

## Variables

| Symbol | Description |
|---|---|
| Ql<sub>k,f,v</sub> | quantity of fuel f loaded into compartment k of vehicle v. |
| δ<sub>f,k,v</sub> | binary variable; equals 1 if fuel f is filled in compartment k of vehicle v, 0 otherwise. |
| N<sub>v</sub> | number of vehicles of type v used in a trip (instances: nv<sub>1</sub>, nv<sub>2</sub>, nv<sub>3</sub>, nv<sub>4</sub>, nv<sub>5</sub> for v<sub>1</sub>, ⋯, v<sub>5</sub>). |
| TA<sub>v,j</sub> | arrival time of vehicle v at node j. |
## Constraints

To be added later... 

---

*This README serves as a reference document for the presumptions, parameters, and constraints of the HCVRP fuel distribution model.*
