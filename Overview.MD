1. Industry Context and Motivation

The distribution of liquid petroleum products — petrol, diesel, aviation fuel, and heating oil — represents one of the most logistically demanding sectors in modern supply chains. Unlike standard freight, fuel delivery imposes a unique convergence of physical constraints, safety regulations, multi-product complexity, and time-sensitive customer requirements that make generic routing models inadequate.

Fuel distribution companies must simultaneously optimise across four deeply interlinked operational dimensions: transportation logistics, inventory management at supply terminals, supply chain coordination across the network, and fleet management across a heterogeneous vehicle pool. These functions cannot be optimised independently. For example, the quantity of a specific fuel demanded by a customer directly determines which vehicle is eligible to serve that customer — only a truck with a compatible compartment of sufficient capacity can fulfil the order. A routing decision is simultaneously a fleet assignment decision, a loading plan, and a schedule.


Why Optimisation Matters in Fuel Distribution


In the petroleum distribution industry, transport activities typically account for between 40% and 60% of total operating costs.
Fuel is a high-volume, low-margin business — even a 2% cost saving across thousands of daily deliveries is significant.
A modest 5% improvement in route efficiency translates directly into significant annual savings.
Commercial planning systems such as AMCS Fuel Planner demonstrate that route optimisation software can reduce operational costs by 5–15%, increase delivered volume per driven kilometre by a similar margin, and cut planning time by 25–50% (AMCS, 2023).




Fuel distribution is further complicated by a set of operational constraints that are either absent or far less severe in generic logistics. Customers must be visited within strict time windows reflecting both customer availability and terminal operating hours. Supply terminals have depot offtake limits — the total volume that can be loaded per day is physically bounded by terminal capacity and throughput rates. Driver hours are regulated. Vehicle compartments carry strict fuel compatibility constraints — not every compartment can carry every fuel type, and cross-contamination between fuel grades is a safety and quality violation.

Environmental and energy efficiency objectives are increasingly incorporated alongside economic goals, reflecting sustainability priorities in the sector. Route length minimisation, fuel consumption reduction, and the matching of vehicle size to delivery volume all contribute to emissions reduction and regulatory compliance, adding a further dimension to the optimisation problem beyond pure cost.

At its structural core, this problem is a Multi-Pickup, Multi-Delivery Problem with a Heterogeneous Vehicle Fleet and Time Windows (MPDP-HF-TW). Each vehicle may visit multiple fuel terminals to load (pickups) and multiple customers to unload (deliveries), in any feasible interleaved order, using a fleet of trucks that differ in compartment count, compartment capacity, fuel compatibility, speed, and operating cost. This is among the most complex vehicle routing variants in the operations research literature — NP-hard in the general case and practically intractable for large instances without sophisticated solution methods.


2. Problem Description

The HCVRP fuel distribution model addresses the daily operational planning problem faced by a petroleum distributor operating across a network of supply terminals, customer sites, and vehicle depots. The planning horizon is a single operational day. Each dispatched vehicle completes one route, departing from an origin node (a depot or terminal) and arriving at a destination node (a depot or eligible terminal), with any number of terminal reload stops and customer delivery stops interleaved along the way.

2.1 Network Structure

The distribution network consists of three distinct node types, each playing a different operational role:

Node TypeRole in ModelKey ConstraintsDepotsRoute origin/destination only; never intermediate stopsNo fuel stocked; Lc = 0 always; no fixed stay costTerminalsFuel supply points; mid-route reload stops; may be eligible as route endpointsStock specific fuels (TF); eligibility flag (E); fixed overnight cost (FT); visit-slots up to P per vehicleCustomersFuel demand points; mid-route delivery stops onlyDemand per fuel type; time windows; visited at most once per vehicle per day; split delivery allowed

2.2 Vehicle Fleet

The fleet is heterogeneous — vehicles are grouped into types, and within each type all operational parameters are identical. Across types they differ in the number of compartments, per-compartment capacities, fuel compatibility of each compartment, maintenance cost per kilometre, fuel consumption cost per kilometre, driver cost rate, and arc-specific average speed. Each vehicle type has a fixed fleet size limiting the number of instances that can be dispatched in a day.

Every vehicle carries multiple compartments. Each compartment is physically engineered to carry a subset of fuel types (a fixed technical parameter). Each day, a compartment is assigned at most one fuel type for the entire day — it cannot switch fuels mid-route. However, it may be topped up at a subsequent terminal stop with more of the same fuel. This compartment-level granularity is what allows a single tanker truck to simultaneously carry petrol, diesel, and aviation fuel in separate compartments on a single route, serving different customers with different products.

2.3 The Core Optimisation Problem

Given the network, fleet, and customer demands, the model simultaneously determines:


Which vehicles to dispatch and which to leave idle
Where each vehicle's route starts and ends (depot or eligible terminal)
The complete ordered sequence of terminal reload stops and customer delivery stops for each vehicle
Which fuel is assigned to each compartment of each vehicle for the day
How much fuel is loaded at each terminal visit, into which compartment
How much fuel is delivered to each customer from which compartment of which vehicle
The arrival time at every node on every route


All decisions are made simultaneously to minimise total cost comprising driving costs (maintenance and fuel consumption), purchased fuel costs (which vary by terminal and fuel type), driver time costs, and fixed terminal overnight stay costs.


3. Model Key Features

The table below summarises the defining features of the HCVRP model. Each feature reflects a real operational constraint or decision that must be represented faithfully to produce actionable routing plans for petroleum distribution.

FeatureDetailHeterogeneous fleetVehicles differ in compartment count, per-compartment capacities, cost rates (maintenance, fuel-consumption, driver), and arc-specific speedMulti-product, multi-compartmentEach compartment carries exactly one fuel type per day; mid-day top-up of the same fuel is allowedMulti-pickup, multi-deliveryEach vehicle may visit multiple terminals (pickups) and multiple customers (deliveries), interleaved in any feasible order — a true MPDP structureFlexible endpointsStart and end nodes are independently chosen from depots or eligible terminals; they need not match — enabling overnight parking at terminalsMid-route reloadsAny terminal may be visited multiple times within a day (visit-slots capped at P per vehicle per terminal per day)Split deliveryA customer's demand for a fuel may be split across multiple vehicles and/or compartmentsVisit vs. load distinctionBinary z records whether a terminal slot was visited; binary l separately records whether fuel was actually loaded — a vehicle parked overnight visits but does not loadRoute-end no-loadingA vehicle whose route ends at a terminal cannot load fuel it will not deliver — enforced via the l variable gating all loading quantitiesTerminal eligibilityOnly terminals with E = 1 may serve as route start or end; any terminal may be used for mid-route reloads regardless of eligibility flagCompartment fuel-identity lockOnce a compartment is assigned a fuel for the day, it cannot switch fuels; it may only be topped up with more of the same fuelArrival time windowsCustomer and terminal arrival windows enforced via big-M constraints; windows apply to every visited node including depots at route boundariesArrival-time propagationA consistent clock is maintained along the entire route, with travel time varying by vehicle type and arc, and dwell time varying by quantity handledQuantity-dependent dwell timeTime spent at a terminal or customer depends on how much is loaded or unloaded — not a flat constant — making the schedule physically accurateMaximum daily durationTotal elapsed time from route start to route end is capped at T_max = 10 hoursMTZ subtour eliminationMiller-Tucker-Zemlin constraints applied over the visit-slot replicated node set N = C ∪ T̂, using big-M, guaranteeing route connectivityNo mid-route depot returnsDepots are strictly route boundaries — a vehicle cannot return to a depot as an intermediate stop; only as route start or endCompartment running-load trackingContinuous variable Lc tracks fuel level in each compartment at every node, enabling partial deliveries and top-ups to be represented exactlyDriver cost on travel time onlyDriver pay covers moving time only (not dwell time), matching a typical per-hour-driven pay structure — a deliberate modelling choice that can be alteredTerminal-stay fixed costAn overnight stop at a terminal incurs a fixed fee FT; ending at a depot does notFleet size enforcementNumber of vehicles dispatched per type cannot exceed the available physical fleet m_vLinear (MILP) formulationAll conditional logic is linearised via big-M; no product of two decision variables appears anywhere — the model is a clean Mixed-Integer Linear Program


4. Objective Function

The model minimises total daily operating cost across the entire fleet. The objective has four components, each capturing a distinct cost driver in petroleum logistics:

Cost ComponentWhat It CapturesWhy It MattersDriving Cost(Maintenance + fuel consumption) per km driven, summed over all arcs and vehiclesDirectly penalises unnecessary distance — the primary lever for route optimisationFuel Purchase CostPrice of delivered fuel multiplied by quantity loaded, at each terminal visitedFuel prices vary by terminal — the model selects the cheapest terminal to load from, balancing price against route detour costDriver CostDriver wage rate multiplied by time spent travelling (moving time only)Captures labour cost; separating it from dwell time matches industry pay structuresTerminal-Stay Fixed CostFixed fee incurred when a route ends at a terminal overnight, rather than returning to depotReflects real parking and administration costs at terminals; the model chooses endpoints to balance this against route length

There is no separate per-vehicle dispatch fee — running a vehicle at all is already captured by the driving, driver, and fuel purchase components. The model naturally selects the fewest vehicles necessary to serve demand at minimum total cost.


5. Formulation Approach

5.1 Mixed-Integer Linear Program (MILP)

The model is formulated as a Mixed-Integer Linear Program. All binary routing, assignment, and visit decisions are represented as 0-1 integer variables. All quantity, timing, and load tracking decisions are continuous variables. Every conditional relationship — load propagation, time propagation, the duration cap, the route-end no-loading rule — is written as a big-M linear inequality rather than a product of two decision variables, preserving linearity throughout.


MILP is deliberately chosen over non-linear formulations (such as QCMINLP used in some related literature) because:


Modern MILP solvers (Gurobi, CPLEX, HiGHS) handle the formulation natively with no approximation.
The LP relaxation provides a provable lower bound — enabling optimality gap certification.
Big-M linearisation avoids the numerical difficulties and local-optima problems of non-linear solvers.
The formulation is compatible with hybrid matheuristics (GA + LP Decoding, LNS-MIP, Benders Decomposition) that exploit the MILP structure for large instances.




5.2 Scalability and Solution Strategy

The HCVRP is NP-hard. For small to medium instances (up to approximately 25–30 customers, 3–5 terminals, 6–8 vehicles), modern exact MILP solvers can find proven optimal solutions within acceptable time limits. For larger operational instances, the MILP structure supports sophisticated hybrid methods:


GA + Exact LP Decoding: Genetic algorithm handles combinatorial routing decisions; an LP exactly optimises loading quantities for each candidate route
Large Neighbourhood Search + MIP: Heuristic destroys and MIP repairs subsets of decisions, combining speed with quality
Benders Decomposition: Separates routing (integer master) from loading (linear subproblem) for provably bounded solutions


The MILP formulation is therefore not just a theoretical model — it is the practical foundation for both exact solution on small instances and matheuristic solution on large ones.


6. Constraint Summary

The model comprises 19 constraint groups. Each enforces a specific physical, operational, or structural requirement of the fuel distribution system.

#ConstraintPurposeC1Demand SatisfactionTotal fuel delivered to each customer equals their order — across all vehicles, instances, and compartments. Enables split delivery.C2Compartment-Fuel CompatibilityA compartment can only be assigned a fuel it was physically built to carry.C3Single Fuel per Compartment per DayEach compartment carries at most one fuel type for the entire day — no mixing.C4Cumulative Load PropagationTracks running fuel level in each compartment at every node. Gates loading via the load/visit distinction variable l.C5Per-Compartment Running CapacityFuel level in each compartment stays between empty and physical capacity at all times.C6Aggregate Vehicle Running CapacityTotal fuel of a type across all compartments does not exceed the vehicle aggregate cap (useful for weight-based limits).C7Load-Delivery BalanceTotal loaded into a compartment across the day equals total delivered from it — no fuel lost or created.C8Visit-Delivery LinkingA customer can only receive fuel if the vehicle actually visits them.C9Flow Conservation: CustomersA visited customer has exactly one arc in and one arc out; unvisited has none.C10Flow Conservation: Terminals and DepotsRoute flow balance at all terminal slots and depots, with explicit arc-domain restrictions encoding structural infeasibilities.C11Terminal EligibilityOnly eligible terminals may be route start or end; any terminal may be used for mid-route reloads.C12Terminal Slot OrderingVisit-slots used in sequence — slot p cannot be used unless slot p-1 was used first. Eliminates solver symmetry.C13Loading GatingThree independent checks: terminal stocks the fuel, slot is actually visited, compartment is assigned that fuel.C14Subtour Elimination (MTZ)Prevents disconnected loops of customers and terminal slots using Miller-Tucker-Zemlin position values with big-M.C15Time WindowsArrival at every visited node must fall within the allowed time band — customers, terminals, and depots.C16Arrival-Time PropagationClock propagates along the route: arrival = previous arrival + dwell time (quantity-dependent) + travel time.C17Maximum Daily DurationTotal elapsed time from route start to route end cannot exceed 10 hours.C18Fleet SizeVehicles dispatched per type cannot exceed physical fleet availability.C19Variable DomainsBinary and non-negative variable declarations maintaining MILP structure.


7. Scope and Future Extensions

The current model is deliberately scoped to a single operational day with deterministic demand. Several extensions — identified through comparison with related literature and operational requirements — are noted for future incorporation.

Currently Modelled


Single-day planning horizon with full route optimisation
Heterogeneous multi-compartment fleet with fuel compatibility
Multi-terminal, multi-customer network with flexible endpoints
Split delivery, mid-route reloads, time windows, duration cap
Visit vs. load distinction and route-end no-loading rule
Full MILP formulation with MTZ subtour elimination


Not Yet Modelled (Future Work)


Multi-day planning horizon: demand served in batches over a week; vehicle state carries over between days
Terminal inventory depletion: terminal stock levels decrease as vehicles load and must be replenished by supply shipments
Time-of-day speed variation: morning, afternoon, and evening traffic conditions produce different arc travel times
Dual capacity constraints: simultaneous volume and weight limits per compartment (relevant when fuel density varies)
Cross-contamination and compartment-adjacency safety rules: certain fuel pairs may not be carried in adjacent compartments
Cargo loading sequence optimisation: the order in which compartments are loaded determines unloading order at customers
Post-optimal sensitivity analysis: systematic evaluation of solution robustness to fuel price volatility, capacity changes, and demand fluctuations
