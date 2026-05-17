Code 174th (out of 485)

```
import sys

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
        num_trolls + move * move,
        num_trolls + carry * carry,
        num_trolls + harvest * harvest,
        num_trolls + chop * chop
    ]
    return all(inventory[i] >= costs[i] for i in range(4))

turn = 0

while True:
    turn += 1
    my_inventory = list(map(int, input().split()))
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
    
    # ENTRAÎNEMENT ULTRA-AGRESSIF - La clé de la victoire !
    # Troll 2: Harvester (vitesse=2, capacité=3, récolte=2, coupe=0)
    if num_trolls == 1 and can_afford_training(my_inventory, num_trolls, 2, 3, 2, 0):
        actions.append("TRAIN 2 3 2 0")
    # Troll 3: Chopper (vitesse=2, capacité=2, récolte=0, coupe=3)  
    elif num_trolls == 2 and can_afford_training(my_inventory, num_trolls, 2, 2, 0, 3):
        actions.append("TRAIN 2 2 0 3")
    # Troll 4: Polyvalent (vitesse=2, capacité=2, récolte=1, coupe=2)
    elif num_trolls == 3 and turn > 40 and can_afford_training(my_inventory, num_trolls, 2, 2, 1, 2):
        actions.append("TRAIN 2 2 1 2")
    # Trolls supplémentaires si ressources abondantes
    elif num_trolls >= 4 and turn > 80 and turn % 50 == 0 and can_afford_training(my_inventory, num_trolls, 2, 2, 1, 1):
        actions.append("TRAIN 2 2 1 1")

    planted_this_turn = set()
    occupied_positions = {(t['x'], t['y']) for t in my_trolls + opp_trolls}
    
    for troll in my_trolls:
        total_carry = sum([
            troll['carry_plum'], troll['carry_lemon'],
            troll['carry_apple'], troll['carry_banana'],
            troll['carry_wood'], troll['carry_iron']
        ])

        # Retour cabane si on porte quelque chose
        if total_carry > 0:
            if is_adjacent(troll['x'], troll['y'], my_shack[0], my_shack[1]):
                actions.append(f"DROP {troll['id']}")
            else:
                actions.append(f"MOVE {troll['id']} {my_shack[0]} {my_shack[1]}")
            continue

        # RÔLES basés sur les attributs
        if troll['chop_power'] >= 3:
            role = 'chopper'
        elif troll['harvest_power'] >= 2:
            role = 'harvester'
        elif troll['id'] == 0 and num_trolls <= 2:
            role = 'bootstrap'  # Premier troll en phase de démarrage
        else:
            role = 'versatile'

        # ═══════════════════════════════════════════════════════════
        # BOOTSTRAP - Premier troll collecte ressources d'entraînement
        # ═══════════════════════════════════════════════════════════
        if role == 'bootstrap':
            # Objectif: Collecter rapidement pour entraîner trolls 2 et 3
            # Besoins troll 2 (2,3,2,0): 5 PLUM, 10 LEMON, 5 APPLE, 1 IRON
            # Besoins troll 3 (2,2,0,3): 8 PLUM, 8 LEMON, 2 APPLE, 13 IRON
            
            # Phase 1: Obtenir fer (priorité absolue!)
            if my_inventory[4] < 3 and iron_positions:
                target = min(iron_positions, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                if is_adjacent(troll['x'], troll['y'], target[0], target[1]):
                    actions.append(f"MINE {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                continue
            
            # Phase 2: Collecter fruits manquants pour entraînement
            needed_fruits = []
            if my_inventory[0] < 8:  # PLUM
                needed_fruits.append('PLUM')
            if my_inventory[1] < 12:  # LEMON (besoin de 10 + marge)
                needed_fruits.append('LEMON')
            if my_inventory[2] < 6:  # APPLE
                needed_fruits.append('APPLE')
            if my_inventory[3] < 5:  # BANANA (pour planter plus tard)
                needed_fruits.append('BANANA')
            
            if needed_fruits:
                # Chercher arbres avec fruits nécessaires
                target_trees = [t for t in trees if t['fruits'] > 0 and t['type'] in needed_fruits]
                if target_trees:
                    target = min(target_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                    if (troll['x'], troll['y']) == (target['x'], target['y']):
                        actions.append(f"HARVEST {troll['id']}")
                    else:
                        actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                else:
                    # Récolter n'importe quel fruit disponible
                    any_fruit_trees = [t for t in trees if t['fruits'] > 0]
                    if any_fruit_trees:
                        target = min(any_fruit_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                        if (troll['x'], troll['y']) == (target['x'], target['y']):
                            actions.append(f"HARVEST {troll['id']}")
                        else:
                            actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                    else:
                        actions.append("WAIT")
            else:
                # Compléter le fer si nécessaire
                if my_inventory[4] < 15 and iron_positions:
                    target = min(iron_positions, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                    if is_adjacent(troll['x'], troll['y'], target[0], target[1]):
                        actions.append(f"MINE {troll['id']}")
                    else:
                        actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                else:
                    actions.append("WAIT")

        # ═══════════════════════════════════════════════════════════
        # CHOPPER - BOIS = 4 POINTS !!! PRIORITÉ MAXIMALE
        # ═══════════════════════════════════════════════════════════
        elif role == 'chopper':
            # 1. Couper arbres taille 4 (4 bois = 16 points!)
            size4_trees = [t for t in trees if t['size'] == 4]
            if size4_trees:
                # Priorité BANANA près eau (repoussent vite)
                banana_water = [t for t in size4_trees if t['type'] == 'BANANA' and is_near_water(t['x'], t['y'])]
                target = min(banana_water if banana_water else size4_trees,
                           key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                
                if (troll['x'], troll['y']) == (target['x'], target['y']):
                    actions.append(f"CHOP {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
            
            # 2. Couper arbres taille 3 (3 bois = 12 points)
            elif [t for t in trees if t['size'] == 3]:
                size3_trees = [t for t in trees if t['size'] == 3]
                target = min(size3_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                if (troll['x'], troll['y']) == (target['x'], target['y']):
                    actions.append(f"CHOP {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
            
            # 3. Planter BANANA si on en porte
            elif troll['carry_banana'] > 0:
                spots = []
                for y in range(height):
                    for x in range(width):
                        if (grid[y][x] == '.' and is_near_water(x, y) and
                            (x, y) not in tree_positions and (x, y) not in planted_this_turn and
                            (x, y) not in occupied_positions):
                            spots.append((x, y))
                
                if spots:
                    target = min(spots, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                    if (troll['x'], troll['y']) == target:
                        actions.append(f"PLANT {troll['id']} BANANA")
                        planted_this_turn.add(target)
                        tree_positions.add(target)
                    else:
                        actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                else:
                    actions.append("WAIT")
            
            # 4. Récolter BANANA pour planter
            else:
                banana_trees = [t for t in trees if t['type'] == 'BANANA' and t['fruits'] > 0]
                if banana_trees:
                    target = min(banana_trees, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                    if (troll['x'], troll['y']) == (target['x'], target['y']):
                        actions.append(f"HARVEST {troll['id']}")
                    else:
                        actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                # 5. Couper n'importe quel arbre
                elif trees:
                    target = min(trees, key=lambda t: (-t['size'], manhattan_distance(troll['x'], troll['y'], t['x'], t['y'])))
                    if (troll['x'], troll['y']) == (target['x'], target['y']):
                        actions.append(f"CHOP {troll['id']}")
                    else:
                        actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                else:
                    actions.append("WAIT")

        # ═══════════════════════════════════════════════════════════
        # HARVESTER - Collecter fruits efficacement
        # ═══════════════════════════════════════════════════════════
        elif role == 'harvester':
            trees_with_fruits = [t for t in trees if t['fruits'] > 0]
            if trees_with_fruits:
                # Priorité fruits pour entraînement
                needed = []
                if my_inventory[0] < 20: needed.append('PLUM')
                if my_inventory[1] < 20: needed.append('LEMON')
                if my_inventory[2] < 15: needed.append('APPLE')
                if my_inventory[3] < 15: needed.append('BANANA')
                
                priority = [t for t in trees_with_fruits if t['type'] in needed]
                target = min(priority if priority else trees_with_fruits,
                           key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                
                if (troll['x'], troll['y']) == (target['x'], target['y']):
                    actions.append(f"HARVEST {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
            
            # Planter BANANA si on en porte
            elif troll['carry_banana'] > 0:
                spots = []
                for y in range(height):
                    for x in range(width):
                        if (grid[y][x] == '.' and is_near_water(x, y) and
                            (x, y) not in tree_positions and (x, y) not in planted_this_turn and
                            (x, y) not in occupied_positions):
                            spots.append((x, y))
                
                if spots:
                    target = min(spots, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                    if (troll['x'], troll['y']) == target:
                        actions.append(f"PLANT {troll['id']} BANANA")
                        planted_this_turn.add(target)
                        tree_positions.add(target)
                    else:
                        actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                else:
                    actions.append("WAIT")
            else:
                actions.append("WAIT")

        # ═══════════════════════════════════════════════════════════
        # VERSATILE - S'adapter aux besoins
        # ═══════════════════════════════════════════════════════════
        else:
            # 1. Récolter fruits d'abord
            trees_with_fruits = [t for t in trees if t['fruits'] > 0]
            if trees_with_fruits:
                target = min(trees_with_fruits, key=lambda t: manhattan_distance(troll['x'], troll['y'], t['x'], t['y']))
                if (troll['x'], troll['y']) == (target['x'], target['y']):
                    actions.append(f"HARVEST {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
            
            # 2. Couper arbres matures pour bois
            elif [t for t in trees if t['size'] >= 3]:
                mature = [t for t in trees if t['size'] >= 3]
                target = min(mature, key=lambda t: (-t['size'], manhattan_distance(troll['x'], troll['y'], t['x'], t['y'])))
                if (troll['x'], troll['y']) == (target['x'], target['y']):
                    actions.append(f"CHOP {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
            
            # 3. Miner fer si besoin
            elif my_inventory[4] < 20 and iron_positions:
                target = min(iron_positions, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                if is_adjacent(troll['x'], troll['y'], target[0], target[1]):
                    actions.append(f"MINE {troll['id']}")
                else:
                    actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
            
            # 4. Planter BANANA si on en a
            elif troll['carry_banana'] > 0:
                spots = []
                for y in range(height):
                    for x in range(width):
                        if (grid[y][x] == '.' and is_near_water(x, y) and
                            (x, y) not in tree_positions and (x, y) not in planted_this_turn and
                            (x, y) not in occupied_positions):
                            spots.append((x, y))
                
                if spots:
                    target = min(spots, key=lambda p: manhattan_distance(troll['x'], troll['y'], p[0], p[1]))
                    if (troll['x'], troll['y']) == target:
                        actions.append(f"PLANT {troll['id']} BANANA")
                        planted_this_turn.add(target)
                        tree_positions.add(target)
                    else:
                        actions.append(f"MOVE {troll['id']} {target[0]} {target[1]}")
                else:
                    actions.append("WAIT")
            else:
                actions.append("WAIT")

    print(';'.join(actions) if actions else "WAIT")
```

