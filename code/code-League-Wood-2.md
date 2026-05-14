```
import sys
import math

width, height = [int(i) for i in input().split()]
grid = []
my_shack = None
for i in range(height):
    line = input()
    grid.append(line)
    if '0' in line:
        my_shack = (line.index('0'), i)

while True:
    my_inventory = list(map(int, input().split()))
    opp_inventory = list(map(int, input().split()))
    
    trees = []
    trees_count = int(input())
    for i in range(trees_count):
        inputs = input().split()
        trees.append({
            'type': inputs[0],
            'x': int(inputs[1]),
            'y': int(inputs[2]),
            'size': int(inputs[3]),
            'health': int(inputs[4]),
            'fruits': int(inputs[5]),
            'cooldown': int(inputs[6])
        })
    
    my_trolls = []
    opp_trolls = []
    trolls_count = int(input())
    for i in range(trolls_count):
        data = list(map(int, input().split()))
        troll = {
            'id': data[0],
            'player': data[1],
            'x': data[2],
            'y': data[3],
            'movement_speed': data[4],
            'carry_capacity': data[5],
            'harvest_power': data[6],
            'chop_power': data[7],
            'carry_plum': data[8],
            'carry_lemon': data[9],
            'carry_apple': data[10],
            'carry_banana': data[11]
        }
        if troll['player'] == 0:
            my_trolls.append(troll)
        else:
            opp_trolls.append(troll)
    
    actions = []
    
    for troll in my_trolls:
        total_carry = troll['carry_plum'] + troll['carry_lemon'] + troll['carry_apple'] + troll['carry_banana']
        
        if total_carry > 0:
            if abs(troll['x'] - my_shack[0]) + abs(troll['y'] - my_shack[1]) == 1:
                actions.append(f"DROP {troll['id']}")
            else:
                actions.append(f"MOVE {troll['id']} {my_shack[0]} {my_shack[1]}")
        else:
            trees_with_fruits = [t for t in trees if t['fruits'] > 0]
            
            if trees_with_fruits:
                best_tree = None
                best_dist = float('inf')
                
                for tree in trees_with_fruits:
                    dist = abs(troll['x'] - tree['x']) + abs(troll['y'] - tree['y'])
                    if dist < best_dist:
                        best_dist = dist
                        best_tree = tree
                
                if best_tree:
                    if troll['x'] == best_tree['x'] and troll['y'] == best_tree['y']:
                        actions.append(f"HARVEST {troll['id']}")
                    else:
                        actions.append(f"MOVE {troll['id']} {best_tree['x']} {best_tree['y']}")
            else:
                trees_growing = sorted(trees, key=lambda t: t['cooldown'])
                if trees_growing:
                    target = trees_growing[0]
                    actions.append(f"MOVE {troll['id']} {target['x']} {target['y']}")
                else:
                    actions.append("WAIT")
    
    if actions:
        print(';'.join(actions))
    else:
        print("WAIT")
```
