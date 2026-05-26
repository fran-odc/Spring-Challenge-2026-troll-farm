# 🗺️ Map

Grid-based map with:

- 🌱 GRASS → traversable.
- 🌊 WATER
- 🪨 ROCK
- ⛏️ IRON
- 🛖 SHACKS

# 🌳 Trees

- Types:
  - 🍑 PLUM
  - 🍋 LEMON
  - 🍎 APPLE
  - 🍌 BANANA

- Trees have their own time for:
  - ✅ growing ⏳
  - ✅ producing fruits 🍓
  - ✅ growing faster near water 🌊


# 👹 Trolls & Training

✅ 1 action per troll each turn

✅ For each attribute of a troll, they do need a specific ressource:

| Resource | Training |
| -------- | ---------------- |
| 🍑 PLUM | 🚶 movementSpeed |
| 🍋 LEMON | 🎒 carryCapacity |
| 🍎 APPLE | 🌾 harvestPower |
| ⛏️ IRON | 🪓 chopPower |

✅ Cost = existing trolls + attribute²

  
# 🌾 Resource Actions

✅ HARVEST → collect fruits from trees on the same cell

✅ PLANT → plant a tree using 1 carried fruit seed

✅ CHOP → destroy trees to gain 🪵 WOOD equal to tree size

✅ MINE → mine adjacent ⛏️ IRON cells (infinite resources)
  

# 🎮 Commands

| Commands | Stdout |
| --------- | ------ |
| **MOVE** | `id x y` |
| **HARVEST** | `id` |
| **PLANT** | `id TYPE` |
| **CHOP** | `id` |
| **MINE** | `id` |
| **DROP** | `id` |
| **PICK** | `id TYPE` |
| **TRAIN** | `ms cc hp cp` |
| **Pause** | `WAIT` |
| **Debug** | `MSG text` |
| **Separator** | `;` |


# ⛔  Game End

❌ 300 turns reached

❌ One player guarantees victory by inactivity

❌ No trees remain for 10 consecutive turns


# 🏆 Victory/Defeat

✅ **VICTORY**: Highest score after 300 turns wins.

❌ **DEFEAT**: Timeout \| invalid commands


# 🔧 Debugging

🔸 Viewer options available

🔸 Simultaneous actions are important

🔸 Shared harvesting/chopping may duplicate final resource

🔸 Optimize for ≤ 50 ms per turn
