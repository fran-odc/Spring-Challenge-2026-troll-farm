# Spring Challenge 2026 - Troll Farm

[![MIT License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Profile views](https://views.igorkowalczyk.dev/api/badge/fran-odc?style=flat)
[![Python](https://img.shields.io/badge/Python-3.11-blueviolet.svg)](https://www.python.org/)
[![GitHub branch protection](https://img.shields.io/badge/Branch_Protected-main-green.svg)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches)

![🐍 Challenge](images/example-game.png)

*Screenshot from [here](https://www.youtube.com/watch?v=cBDjsuNzLQ0)*

## 🎯 Goal

Control a tribe of trolls and score more points than your opponent by collecting resources.

[📂 Official Contest Website](https://www.codingame.com/contests/spring-challenge-2026-troll-farm) 

[📂 Game Rules](docs/Game-Rules.md?plain=1)


## 🧠 Solution Approach

- Iterative improvement using insights from Claude Sonnet 4.6 and Lumo 1.3.

🪵 **Wood 2 League to Wood 1 League:**
- Established the basic loop and getting the troll behavior working reliably.
  
🥉 **Wood 1 League to Bronze League:**
- Added economic expansion and early planting.

🥈 **Bronze League to Silver League:**
- Added prioritization, coordination, and late-game sophistication.

🥇 **Silver League to Gold League:**
- Added deeper strategic planning, and more advanced decision-making.

## 🛠️ Tech Stack

Python 3.11
- No external dependencies
- Grid-based simulation
- Manhattan-distance targeting

## 🚀 How to Run & Test

**Development workflow:**
- Started from the official starter code
- Added logic incrementally
- Iterated directly through the challenge arena and replay system
  
**Testing:**
- Verified troll state transitions and coordination
- Observed replay behavior against opponents and adjusted targeting priorities
  
**Validation:** 
- Successfully climbed from one league to the next league
- Confirmed stable autonomous gameplay without invalid commands or timeouts
- Confirmed improved resource efficiency


## ⚖️ Design Trade-offs

🪵 **Wood 2 League to Wood 1 League:**
- **Simplicity over complexity**: nearest-tree targeting instead of predictive simulation
- **Reliable actions first**: prioritized valid commands over aggressive optimization
- **Lightweight logic**: distance-based selection and single-turn reasoning
- **No pathfinding**: relied on arena movement resolution instead of BFS/A*
- **Fast iteration**: replay analysis + arena testing workflow
  
🥉 **Wood 1 League to Bronze League:**
- **Economic scaling**: over per-turn optimization
- **Heuristic planting strategy**: instead of full resource forecasting

🥈 **Bronze League to Silver League:**
- **Controlled complexity**: added strategic systems while keeping the code single-file and contest-safe
- **Greedy coordination**: target reservation
- **Dynamic resource specialization**

🥇 **Silver League to Gold League:**
- **Strategic prioritization**: : adaptive decision-making
- **Multi-agent coordination**: improved troll synchronization for more scalable team behavior

## 🏆 Competition Results

🥉 **Bronze League**: [Code Winner](code/Bronze-League.md?plain=1) 

🥈 **Silver League**: [Code Winner](code/Silver-League.md?plain=1)

**Gold League**: [Top 25% (out of 2,022 final players 🥇 )](code/Gold-League.md?plain=1)

## Contact
LinkedIn : [Francesca Oliveira](https://www.linkedin.com/in/oliveirafrancesca/)

Email : fran.odc@pm.me

*Dernière mise à jour : Mai 2026*
