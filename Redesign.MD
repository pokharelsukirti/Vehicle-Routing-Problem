##  Key Redesigning

1. **Terminal Removal**
   * Remove **Terminal** as the explicit start and end points of a route. 
   * *Note:* This requires corresponding updates to **flow conservation constraints**, variable definitions, and pathing logic.

2. **Capacity & Gateway Constraints**
   * Gate the total quantity loaded using the visit binary variable $z$.
   * Implement a strict **no terminal-start** and **no terminal-end** framework.

3. **Travel Duration Simplification**
   * Simplify the calculation of total travel duration from start to end ($c_{17}$) by using the decision variable:
     $$t(j, j', v, i)$$

4. **Time Propagation & Driver Breaks**
   * Introduce a dedicated tracking variable for **Leave Time** at node $j$ ($LT_j$), defined as:
     $$LT_j = \text{Arrival Time at Node } j + \text{Load/Unload Time} + \text{Break Time} + \text{Long Break}$$
   * The **Long Break** is triggered via a binary variable automatically after **5 hours** of elapsed time from the route's initial start point.

5. **Switch Loading**
   * Explore and integrate explicit **switch loading constraints** into the routing logic.

6. **Soft Time Windows**
   * Transition strict time windows into **soft constraints**, incorporating specific penalty functions for late deliveries.

---

##  New Ideas & Extensions

* **Priority-Based Fulfillment:** Explore heuristics or weights to guarantee the fulfillment of high-priority orders first.
* **Fuel-Switching Multi-Objective Optimization:** Integrate a multi-objective function that evaluates and balances emission/cost trade-offs based on fuel switching.
