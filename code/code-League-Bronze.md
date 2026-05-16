Code 330th (out of 809)

```
import sys
from collections import defaultdict


width, height = [int(i) for i in input().split()]
grid = []
my_shack = None
opp_shack = None
water_positions = []
iron_positions = []


for i in range(height):
   line = input()
   grid.append(line)
   if '0' in line:
       my_shack = (line.index('0'), i)
   if '1' in line:
       opp_shack = (line.index('1'), i)
   for j, char in enumerate(line):
       if char == '~':
           water_positions.append((j, i))
       elif char == '+':
           iron_positions.append((j, i))


def manhattan_distance(x1, y1, x2, y2):
   return abs(x1 - x2) + abs(y1 - y2)


def is_adjacent(x1, y1, x2, y2):
   return manhattan_distance(x1, y1, x2, y2) == 1


def is_near_water(x, y):
   return any(abs(x - wx) + abs(y - wy) == 1 for wx, wy in water_positions)


def can_afford_training(inventory, num_trolls, move, carry, harvest, chop):
   costs = [
       num_trolls + move * move,   # PLUM
       num_trolls + carry * carry,  # LEMON
       num_trolls + harvest * harvest,  # APPLE
       num_trolls + chop * chop     # IRON
   ]
   return all(inventory[i] >= costs[i] for i in range(4))


def is_opponent_near(x, y, opp_trolls):
   """Check if an opponent troll is adjacent to (x, y)"""
   return any(is_adjacent(x, y, t['x'], t['y']) for t in opp_trolls)


def is_opponent_targeting(x, y, opp_trolls):
   """Check if an opponent troll is moving toward (x, y)"""
   for t in opp_trolls:
       if manhattan_distance(t['x'], t['y'], x, y) <= 2:
           return True
   return False


turn = 0
planted_this_turn = set()
training_phase = True


while True:
   turn += 1
   my_inventory = list(map(int, input().split()))  # plum, lemon, apple, banana, iron, wood
   opp_inventory = list(map(int, input().split()))


   trees = []
   tree_positions = set()
   trees_count = int(input())
   for _ in range(trees_count):
       inputs = input().split()
       tree = {
           'type': inputs[0],
           'x': int(inputs[1]),
           'y': int(inputs[2]),
           'size': int(inputs[3]),
           'health': int(inputs[4]),
           'fruits': int(inputs[5]),
           'cooldown': int(inputs[6])
       }
       trees.append(tree)
       tree_positions.add((tree['x'], tree['y']))


   my_trolls = []
   opp_trolls = []
   trolls_count = int(input())
   for _ in range(trolls_count):
       data = input().split()
       troll = {
           'id': int(data[0]),
           'player': int(data[1]),
           'x': int(data[2]),
           'y': int(data[3]),
           'movement_speed': int(data[4]),
           'carry_capacity': int(data[5]),
           'harvest_power': int(data[6]),
           'chop_power': int(data[7]),
           'carry_plum': int(data[8]),
           'carry_lemon': int(data[9]),
           'carry_apple': int(data[10]),
           'carry_banana': int(data[11]),
           'carry_iron': int(data[12]),
           'carry_wood': int(data[13])
       }
       if troll['player'] == 0:
           my_trolls.append(troll)
       else:
           opp_trolls.append(troll)


   actions = []
   num_trolls = len(my_trolls)
   occupied_positions = {(t['x'], t['y']) for t in my_trolls + opp_trolls} | tree_positions


   # --- TRAINING LOGIC (More aggressive) ---
   if num_trolls == 1 and can_afford_training(my_inventory, num_trolls, 2, 2, 1, 3):
       actions.append("TRAIN 2 2 1 3")  # Chopper
       training_phase = False
   elif num_trolls == 2 and turn > 50 and can_afford_training(my_inventory, num_trolls, 2, 3, 2, 0):
       actions.append("TRAIN 2 3 2 0")  # Harvester
   elif num_trolls >= 3 and turn > 100 and can_afford_training(my_inventory, num_trolls, 2, 2, 1, 2):
       actions.append("TRAIN 2 2 1 2")  # Balanced


   # --- TROLL ACTIONS ---
   planted_this_turn = set()


   for troll in my_trolls:
       total_carry = sum([
           troll['carry_plum'], troll['carry_lemon'],
           troll['carry_apple'], troll['carry_banana'],
           troll['carry_wood'], troll['carry_iron']
       ])


       # If carrying, return to shack
       if total_carry > 0:
           if is_adjacent(troll['x'], troll['y'], my_shack[0], my_shack[1]):
               actions.append(f"DROP {troll['id']}")
           else:
               actions.append(f"MOVE {troll['id']} {my_shack[0]} {my_shack[1]}")
           continue


       # --- ROLE ASSIGNMENT ---
       if troll['chop_power'] >= 3:
           role = 'chopper'
       elif troll['harvest_power'] >= 2:
           role = 'harvester'
       else:
           role = 'resource_gatherer' if training_phase else 'blocker' if turn > 100 else 'flexible'


       # --- BLOCKER ROLE (New: Disrupt opponent) ---
       if role == 'blocker':
           # 1. Block opponent's access to water for planting
           water_candidates = []
           for wx, wy in water_positions:
               for dx, dy in [(0,1), (1,0), (0,-1), (-1,0)]:
                   nx, ny = wx + dx, wy + dy
                   if (0 <= nx < width and 0 <= ny < height and
                       grid[ny][nx] == '.' and
                       (nx, ny) not in occupied_positions and
                       is_opponent_targeting(nx, ny, opp_trolls)):
                       water_candidates.append((nx, ny))
           if water_candidates:
               target = min(water_candidates, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
               if (troll['x'], troll['y']) == target:
                   actions.append("WAIT")  # Already blocking
               else:
                   actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
               continue


           # 2. Block opponent's access to iron
           iron_candidates = [p for p in iron_positions if is_opponent_targeting(p[0], p[1], opp_trolls)]
           if iron_candidates:
               target = min(iron_candidates, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
               if is_adjacent(troll['x'], troll['y'], target[0], target[1]):
                   actions.append(f"MINE {troll['id']}")
               else:
                   actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
               continue


           # 3. Block opponent's shack (prevent resource drops)
           if opp_shack and is_opponent_near(opp_shack[0], opp_shack[1], opp_trolls):
               # Move adjacent to opponent's shack
               for dx, dy in [(0,1), (1,0), (0,-1), (-1,0)]:
                   nx, ny = opp_shack[0] + dx, opp_shack[1] + dy
                   if (0 <= nx < width and 0 <= ny < height and
                       grid[ny][nx] == '.' and
                       (nx, ny) not in occupied_positions):
                       actions.append(f"MOVE {troll['id']} {nx} {ny}")
                       break
               else:
                   actions.append("WAIT")
               continue


           # 4. Fallback: Block opponent's trolls
           if opp_trolls:
               # Find opponent trolls carrying resources
               opp_carriers = [t for t in opp_trolls if sum([t['carry_plum'], t['carry_lemon'], t['carry_apple'], t['carry_banana'], t['carry_wood'], t['carry_iron']]) > 0]
               if opp_carriers:
                   # Move to block their path to their shack
                   target = opp_carriers[0]
                   # Try to move between them and their shack
                   path_x = (target['x'] + opp_shack[0]) // 2
                   path_y = (target['y'] + opp_shack[1]) // 2
                   if (0 <= path_x < width and 0 <= path_y < height and
                       grid[path_y][path_x] == '.' and
                       (path_x, path_y) not in occupied_positions):
                       actions.append(f"MOVE {troll['id']} {path_x} {path_y}")
                   else:
                       actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                   continue


           # If no blocking opportunity, fall back to chopping
           role = 'chopper'


       # --- CHOPPER: Prioritize size-4 trees, then plant BANANA near water ---
       if role == 'chopper':
           # 1. Chop opponent's BANANA trees near water
           opp_banana_trees = [t for t in trees if t['type'] == 'BANANA' and is_near_water(t['x'], t['y'])]
           if opp_banana_trees:
               target = min(opp_banana_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
               if (troll['x'], troll['y']) == (target['x'], target['y']):
                   actions.append(f"CHOP {troll['id']}")
               else:
                   actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
               continue


           # 2. Chop size-4 trees
           large_trees = [t for t in trees if t['size'] == 4]
           if large_trees:
               target = min(large_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
               if (troll['x'], troll['y']) == (target['x'], target['y']):
                   actions.append(f"CHOP {troll['id']}")
               else:
                   actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
               continue


           # 3. Plant BANANA near water if carrying
           if troll['carry_banana'] > 0:
               candidates = []
               for y in range(height):
                   for x in range(width):
                       if (grid[y][x] == '.' and is_near_water(x, y) and
                           (x, y) not in occupied_positions and (x, y) not in planted_this_turn):
                           candidates.append((x, y))
               if candidates:
                   target = min(candidates, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                   if (troll['x'], troll['y']) == target:
                       actions.append(f"PLANT {troll['id']} BANANA")
                       planted_this_turn.add((troll['x'], troll['y']))
                       occupied_positions.add((troll['x'], troll['y']))
                   else:
                       actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                   continue


           # 4. Chop any mature tree
           mature_trees = [t for t in trees if t['size'] >= 3]
           if mature_trees:
               target = min(mature_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
               if (troll['x'], troll['y']) == (target['x'], target['y']):
                   actions.append(f"CHOP {troll['id']}")
               else:
                   actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
               continue


           # 5. Mine iron as fallback
           if iron_positions:
               target = min(iron_positions, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
               if is_adjacent(troll['x'], troll['y'], target[0], target[1]):
                   actions.append(f"MINE {troll['id']}")
               else:
                   actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
           else:
               actions.append("WAIT")


       # --- HARVESTER: Prioritize fruits needed for training ---
       elif role == 'harvester':
           trees_with_fruits = [t for t in trees if t['fruits'] > 0]
           if trees_with_fruits:
               # Prioritize PLUM, LEMON, APPLE for training
               needed_types = []
               if my_inventory[0] < 5:  # PLUM
                   needed_types.append('PLUM')
               if my_inventory[1] < 5:  # LEMON
                   needed_types.append('LEMON')
               if my_inventory[2] < 2:  # APPLE
                   needed_types.append('APPLE')
               if my_inventory[3] < 3:  # BANANA
                   needed_types.append('BANANA')


               priority_trees = [t for t in trees_with_fruits if t['type'] in needed_types]
               target = min(priority_trees if priority_trees else trees_with_fruits,
                            key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
               if (troll['x'], troll['y']) == (target['x'], target['y']):
                   actions.append(f"HARVEST {troll['id']}")
               else:
                   actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
           else:
               # Help plant BANANA if carrying
               if troll['carry_banana'] > 0:
                   candidates = []
                   for y in range(height):
                       for x in range(width):
                           if (grid[y][x] == '.' and is_near_water(x, y) and
                               (x, y) not in occupied_positions and (x, y) not in planted_this_turn):
                               candidates.append((x, y))
                   if candidates:
                       target = min(candidates, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                       if (troll['x'], troll['y']) == target:
                           actions.append(f"PLANT {troll['id']} BANANA")
                           planted_this_turn.add((troll['x'], troll['y']))
                           occupied_positions.add((troll['x'], troll['y']))
                       else:
                           actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                   else:
                       actions.append("WAIT")
               else:
                   actions.append("WAIT")


       # --- RESOURCE GATHERER (Early game) ---
       elif role == 'resource_gatherer':
           # 1. Get iron first
           if my_inventory[4] < 10:  # Need iron for chopper
               if iron_positions:
                   target = min(iron_positions, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                   if is_adjacent(troll['x'], troll['y'], target[0], target[1]):
                       actions.append(f"MINE {troll['id']}")
                   else:
                       actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
               else:
                   actions.append("WAIT")
           # 2. Then get fruits for training
           else:
               trees_with_fruits = [t for t in trees if t['fruits'] > 0]
               if trees_with_fruits:
                   needed_types = []
                   if my_inventory[0] < 5:  # PLUM
                       needed_types.append('PLUM')
                   if my_inventory[1] < 5:  # LEMON
                       needed_types.append('LEMON')
                   if my_inventory[2] < 2:  # APPLE
                       needed_types.append('APPLE')
                   priority_trees = [t for t in trees_with_fruits if t['type'] in needed_types]
                   target = min(priority_trees if priority_trees else trees_with_fruits,
                                key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                   if (troll['x'], troll['y']) == (target['x'], target['y']):
                       actions.append(f"HARVEST {troll['id']}")
                   else:
                       actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
               else:
                   actions.append("WAIT")


       # --- FLEXIBLE (Late game) ---
       else:
           # 1. If near water and carrying BANANA, plant
           if troll['carry_banana'] > 0 and is_near_water(troll['x'], troll['y']):
               if (troll['x'], troll['y']) not in occupied_positions and (troll['x'], troll['y']) not in planted_this_turn:
                   actions.append(f"PLANT {troll['id']} BANANA")
                   planted_this_turn.add((troll['x'], troll['y']))
                   occupied_positions.add((troll['x'], troll['y']))
               else:
                   actions.append("WAIT")
           # 2. Otherwise, harvest or chop
           else:
               trees_with_fruits = [t for t in trees if t['fruits'] > 0]
               if trees_with_fruits:
                   target = min(trees_with_fruits, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                   if (troll['x'], troll['y']) == (target['x'], target['y']):
                       actions.append(f"HARVEST {troll['id']}")
                   else:
                       actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
               else:
                   # Chop any tree
                   if trees:
                       target = min(trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                       if (troll['x'], troll['y']) == (target['x'], target['y']):
                           actions.append(f"CHOP {troll['id']}")
                       else:
                           actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                   else:
                       actions.append("WAIT")


   print(';'.join(actions) if actions else "WAIT")
```
