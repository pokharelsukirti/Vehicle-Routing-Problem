## Optimization Opportunities

The current model is deliberately scoped to a single operational day with deterministic demand. Several extensions — identified through comparison with related literature and operational requirements — are noted for future incorporation.

### Currently Modelled

- Single-day planning horizon with full route optimisation
- Heterogeneous multi-compartment fleet with fuel compatibility
- Multi-terminal, multi-customer network with flexible endpoints
- Split delivery, mid-route reloads, time windows, duration cap

### Not Yet Modelled (Future Work)

- **Multi-day planning horizon:** demand served in batches over a week; and priority based routing
- **Quadratic relationship:** explore fuel consumption cost as a quadratic function of weight of the truck
- **Time-of-day speed variation:** morning, afternoon, and evening traffic conditions produce different arc travel times
- **Dual capacity constraints:** simultaneous volume and weight limits per compartment (relevant when fuel density varies)
- **Multiple Objective:** instead of a single cost objective, explore the possibilities for a multiple objective
- **Cross-contamination and compartment-adjacency safety rules:** certain fuel pairs may not be carried in adjacent compartments

> **Priority **
>
> The multi-day extension is the single highest-priority future addition — it transforms the model from a daily planning tool into a weekly operational planning system, enabling batched demand fulfilment, terminal inventory management, and overnight vehicle positioning to be optimised jointly.

### Needs Revisting 

- **Regarding Time Windows:**  introduce soft constraint for time windows, i.e penalties for late deliveries, include break times and other slack variables
