Frobozz Magic Operations Planner is a constraint-based engine that replaces guesswork with physics.

Classic MRP explodes demand and assumes you can make it. Frobozz OP applies Liebig’s Law (limiting component of the BOM) and Kirchhoff conservation (assures component qty is conserved through all levels of production, nothing created or lost) to your actual inventory, open POs, and WOs to show what’s to produce over a 90 day range — plus the exact components that hold up production.

What-if testing computes global results of cutting production orders, adding production order for missing WIP components, and setting raw componets to available. 

Produced excel reports to show component pegging to mirroe the optimized state, the reunning totals, run out reports, discontinues items unique component run outs, excess (orphaned) components created by cuts, every bottle neck and what it holds back, and the before and after state of all What-If states. 

Built for planners who need to triage shortages, find bottlenecks, and test changes before committing them in JD Edwards

I built this to expand upon and improve the standard run out reports, with the idea that Finished Goods that share component parts can be redistributed, 
cutting the over produced finished good using the released components to increase the production of a shorted finished good that uses those shared components. 
this developed gradually into a system that uses Lagrangian physics, QUBO, Annealing, Kirchhoff and Liebigs, and even a version of Pontryagin's maximum principle that I borrowed from the Artemis II fuel conservation system. To create a full digital twin over lay of the existing MRP to allocate the actual known supply to the demand over the whole BOM chain or existing Production orders, attempting to produce the highest fill rats. (with run weight to shift between filling for qty, value, or profit margin) 
