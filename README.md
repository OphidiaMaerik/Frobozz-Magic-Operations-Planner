The Python source Code will be available soon, Once I have decoupled fully from private data sets.

Frobozz OP-1 — What It Is
Frobozz is a decision system that sits on top of JDE. It doesn't replace MRP, it fixes what MRP can't do.
JDE tells you what you need to make everything. Frobozz tells you what you should make with what you actually have, to fill the most real customer demand.
How It Works For A Planner
It’s 3 apps that work together:
1. The Control Panel — Before The Run
A simple pop-up before the run where you pick:
•	Division: DG pasta, Non-Pasta, or All
•	Objective Mode: Fill Rate (max cases/orders), High Value (max revenue), or Profit-Minded (max margin)
•	Lagrangian Weights: 6 sliders — kinetic mass, shortage penalty, inventory cost, component over-use, change cost, WIP conservation — plus a Value Weight slider. Lead planners can lock this behind a password. Defaults to stock-neutral, so most users just hit Run.
It writes a transient JSON config. The optimizer reads it. No settings are permanently changed.
2. The Optimizer — The Brain
Pulls live JDE from Snowflake: OH, POs, WOs, BOMs, routings, sales orders, forecasts, safety stock, lot expirations. Converts everything to Stocking UOM correctly via F41002.
Then it builds a time-phased flow network:
•	Component nodes = supply (OH + PO arrivals + WO completions)
•	FG and WIP WO nodes = consumers/producers
•	BOM edges = physical recipes
It enforces two laws:
•	Whole-BOM Feasibility (Liebig's Law): You can't make 80% of a case. The scarcest ingredient caps the WO.
•	Time + Identity: A PO arriving Friday can't fill an order due Wednesday. Raw is fungible, WIP is identity-conserved — a WIP finishing next week can't help today.
It seeds demand pressure from the real order book: SO > Safety Stock > Forecast > Topping off a WO. That pressure propagates down the entire BOM chain, so when sugar is contested, it flows to the chain that unlocks more real sales orders. Allocation is proportional to that pressure, not first-come-first-served. Components needed by many short FGs get a higher shadow price.
It outputs optimizer_graph.json — every node, edge, feasibility %, binding component, batch min/max, and daily inventory curves.
3. The Flow Map — What You Actually Use
GraphAPI.py is a Flask server. It never loads the full graph into the browser. It serves subgraph slices on demand.
You open http://localhost:5000 and get FLOW.MAP — an interactive dark-mode canvas:
•	Left sidebar: all SHORT, PARTIAL, ghost (missing WO), and late PO WOs. Search and filter by week window.
•	Canvas: Click any WO and see its chain — upstream components and downstream orders. Edges are colored by feasibility, nodes by status. Hover for tooltip with qty, UOM, need vs got, and cost.
•	Right detail panel: Exact math for that node, why it's capped, planner notes (persisted across runs by WO# so they survive rebuilds), and action buttons.
•	Top bar: Live stats — how many short, partial, etc.
What makes it interactive, not just a viewer:
•	Force-Available: Mark any component as "pretend it's unlimited" — it can never be binding again. Blue edge, instant re-solve.
•	Fire-Ghost: A WIP that has 0% feasibility because it has no WO? Click to create a WO for it on the fly, stepping by batch size. The API sends create_wo to the optimizer and computes real feasibility from real supply.
•	Cut / Grow WO: Dial any FG WO up or down. Each action is a real forward re-solve through field_a, not an estimate.
•	Running Total Panel: Top-right, always visible. Shows cumulative impact of all your what-ifs applied at once: Cases gained/lost, dollars gained/lost, raw stranded, WIP stranded, unused WIP. Graph shows this cut, Running Total shows all cuts.
•	Who Can I Steal From?: Click any SHORT FG WO. It ranks every OTHER FG WO you could cut to cover its short. For each donor it shows: how much of the short it covers, what the donor gives up (cs + $), what excess raw + already-made WIP the cut strands, and net. Each row is a proven re-solve of [cut donor] + [prioritize this short].
•	Two power reports that also prove with re-solves:
o	⚖️ Cut Trade Matrix: Rows = WO you'd CUT (per batch and in full), Columns = how many cases each SHORT FG gains. Plus cascade stranded.
o	🎯 SO: What To Cut: Every short sales order: what caps it, whether any cut can help, and the price.
All of this exports to Excel: Fill Rates, Run-Out (Daily), Bottleneck Ingredients, WO Pegging with lot-level OH/PO#/WIP-WO sourcing, Stranded Components, What-If Delta, Cut Effects, Cut Matrix, SO Cuts.
Why It's Better
vs. JDE MRP:
MRP is infinite. It says "make everything you're short on" and then creates exception messages. It doesn't know how to choose when you have to choose. It doesn't know time matters. Frobozz is finite, time-phased, and choice-aware. It makes the choice that maximizes real order fill and shows you the binding component for every WO it couldn't fill.
vs. Basic Run-Out Reporting:
Run-Out is OH / daily usage = days left. It's one item at a time, no BOM, no timing, no alternative. It can't tell you that cutting an overage frees the exact WIP you need for a shortage, or that a component is valuable because 20 FGs need it. Frobozz sees the whole network at once and shows the ripple effect of any action.
vs. Kinaxis Maestro:
Kinaxis is a generic concurrent planning platform. You pay to license it, then you pay again to teach it how your JDE actually works — Julian dates, two UOM systems, plant-specific sourcing, phantom WIPs, lot expirations. And you still get a black-box answer.
Frobozz is JDE-native by design, and it's explainable and interactive. You don't get a static plan. You get a live model you can interrogate: click to see why, force it, cut it, create it, see the global total move, add a note for the next shift, and export the proof. A planner can defend the decision to finance because the math — pressure, edge draws, remaining_before, stranded cost — is visible.
In one line:
JDE tells you what you need. Run-Out tells you when you run out. Kinaxis tells you what a generic model thinks.
Frobozz gives you a live map of your plant that shows what to build today to fill the most real demand, lets you test any what-if instantly with honest re-solves, and tracks the total business impact.

