Code 22th (out of 485)

```
import sys

def main():
    read = sys.stdin.readline
    width, height = map(int, read().split())
    grid = []
    my_shack = None
    iron_list = []

    for i in range(height):
        row = read().rstrip()
        grid.append(row)
        for j, c in enumerate(row):
            if c == '0': my_shack = (j, i)
            elif c == '+': iron_list.append((j, i))

    sx, sy = my_shack

    MAX_HP = {
        'PLUM':   [0, 6, 8, 10, 12],
        'LEMON':  [0, 6, 8, 10, 12],
        'APPLE':  [0, 11, 14, 17, 20],
        'BANANA': [0, 3, 4, 5, 6],
    }

    def mdist(x1, y1, x2, y2): return abs(x1-x2) + abs(y1-y2)
    def is_adj(x1, y1, x2, y2): return abs(x1-x2)+abs(y1-y2) == 1
    def trns(d, spd): return (d+spd-1)//spd if d > 0 else 0
    def can_train(inv, n, m, c, h, ch):
        return (inv[0]>=n+m*m and inv[1]>=n+c*c and
                inv[2]>=n+h*h and inv[4]>=n+ch*ch)

    turn = 0
    while True:
        turn += 1
        my_inv = list(map(int, read().split()))
        read()

        tc = int(read())
        trees = []
        tree_at = {}
        for _ in range(tc):
            p = read().split()
            t = {'type': p[0], 'x': int(p[1]), 'y': int(p[2]),
                 'size': int(p[3]), 'health': int(p[4]),
                 'fruits': int(p[5]), 'cooldown': int(p[6])}
            trees.append(t)
            tree_at[(t['x'], t['y'])] = t

        nc = int(read())
        my_trolls = []
        for _ in range(nc):
            v = read().split()
            if int(v[1]) == 0:
                my_trolls.append({
                    'id': int(v[0]), 'x': int(v[2]), 'y': int(v[3]),
                    'spd': int(v[4]), 'cap': int(v[5]),
                    'hp': int(v[6]), 'cp': int(v[7]),
                    'cplum': int(v[8]), 'clem': int(v[9]),
                    'capl': int(v[10]), 'cban': int(v[11]),
                    'cirn': int(v[12]), 'cwod': int(v[13])
                })

        n = len(my_trolls)
        actions = []

        # Training: cheapest first when n < 3 to maximize troll count fast
        if n < 4 and turn < 270:
            if n < 3:
                builds = ([(1,1,1,0),(1,1,0,1),(1,1,1,1),(1,2,1,0),(1,2,0,1),
                           (1,2,1,1),(2,2,1,1),(2,2,1,2),(2,2,0,2),(1,2,0,2)]
                          if iron_list else
                          [(1,1,1,0),(1,2,1,0),(1,1,1,1),(2,2,1,0),(2,2,2,0)])
            else:
                builds = ([(2,2,1,2),(1,2,1,2),(2,2,1,1),(1,2,1,1),(2,2,0,2),
                           (1,2,0,2),(1,1,1,1),(1,1,0,1)]
                          if iron_list else
                          [(2,2,2,0),(2,2,1,0),(1,2,2,0),(1,2,1,0),(1,1,1,0)])
            for m, c, h, ch in builds:
                if can_train(my_inv, n, m, c, h, ch):
                    actions.append(f"TRAIN {m} {c} {h} {ch}")
                    break

        need_fruits = (n < 4 and
                      not can_train(my_inv, n, 1, 2, 1, 0) and
                      not (iron_list and can_train(my_inv, n, 1, 2, 0, 1)))

        # Local farm: count trees near shack
        trees_near_shack = sum(1 for t in trees if mdist(t['x'], t['y'], sx, sy) <= 4)
        want_plant = trees_near_shack < 6 and turn < 250

        for troll in my_trolls:
            tx, ty = troll['x'], troll['y']
            total = (troll['cplum'] + troll['clem'] + troll['capl'] +
                    troll['cban'] + troll['cirn'] + troll['cwod'])
            free = troll['cap'] - total
            tid = troll['id']
            spd = max(1, troll['spd'])
            cp = troll['cp']
            hp = troll['hp']
            adj_sh = is_adj(tx, ty, sx, sy)
            d_sh = mdist(tx, ty, sx, sy)
            remaining = 300 - turn
            action = None

            # Endgame: rush to shack
            if total > 0 and remaining <= trns(d_sh, spd) + 3:
                actions.append(f"DROP {tid}" if adj_sh else f"MOVE {tid} {sx} {sy}")
                continue

            # Full: return to shack
            if free == 0:
                actions.append(f"DROP {tid}" if adj_sh else f"MOVE {tid} {sx} {sy}")
                continue

            # Adjacent to shack with items: PLANT BANANA here if good spot, else DROP
            # KEY INSIGHT: planting near shack creates local farm (massive efficiency gain)
            if adj_sh and total > 0:
                can_plant_here = (
                    want_plant and
                    troll['cban'] > 0 and
                    (tx, ty) not in tree_at and
                    grid[ty][tx] == '.'
                )
                actions.append(f"PLANT {tid} BANANA" if can_plant_here else f"DROP {tid}")
                continue

            # Act on current cell tree
            cur = tree_at.get((tx, ty))
            if cur is not None and free > 0:
                is_local = mdist(cur['x'], cur['y'], sx, sy) <= 3  # protect our farm
                mh = MAX_HP.get(cur['type'], [0, 99, 99, 99, 99])
                sz = min(cur['size'], 4)
                is_dmg = sz > 0 and cur['health'] < mh[sz]

                if cur['fruits'] > 0 and hp > 0:
                    action = f"HARVEST {tid}"  # always harvest available fruits
                elif cp > 0 and is_dmg and not is_local:
                    action = f"CHOP {tid}"     # finish distant damaged tree
                elif cp > 0 and not is_local and cur['size'] >= 2:
                    action = f"CHOP {tid}"     # chop large distant trees

                if action:
                    actions.append(action)
                    continue

            # Find best target using FULL ROUND-TRIP cost
            best_v = -1.0
            best_pos = None
            best_kind = None

            for t in trees:
                d_to = mdist(tx, ty, t['x'], t['y'])
                d_back = mdist(t['x'], t['y'], sx, sy)
                tt_to = trns(d_to, spd)
                tt_back = trns(d_back, spd)
                is_local = mdist(t['x'], t['y'], sx, sy) <= 3

                # Harvest value
                if t['fruits'] > 0 and hp > 0 and free > 0:
                    ft = min(t['fruits'], free)
                    ha = (ft + hp - 1) // hp
                    trip = tt_to + ha + tt_back + 1
                    boost = 1.5 if need_fruits else 1.0
                    v = boost * ft / max(1, trip)
                    if v > best_v:
                        best_v = v; best_pos = (t['x'], t['y']); best_kind = 'harvest'

                # Chop value (only large, non-local trees)
                if cp > 0 and free > 0 and t['size'] >= 2 and not is_local:
                    chops = max(1, (t['health'] + cp - 1) // cp)
                    wood = min(t['size'], free)
                    if wood > 0:
                        trip = tt_to + chops + tt_back + 1
                        factor = 2.0 if need_fruits else 4.0
                        v = factor * wood / max(1, trip)
                        if v > best_v:
                            best_v = v; best_pos = (t['x'], t['y']); best_kind = 'chop'

            # Iron: ONLY mine if short of what's needed for next training
            # KEY FIX: stop at n+4, not 12! (was mining 65+ turns unnecessarily)
            iron_for_next = n + 4
            iron_shortfall = iron_for_next - my_inv[4]
            carrying_val = (troll['cplum'] + troll['clem'] + troll['capl'] +
                           troll['cban'] + troll['cwod'])
            if (n < 4 and cp > 0 and free > 0 and
                    iron_shortfall > 0 and carrying_val == 0):
                iv = 6.0 if my_inv[4] == 0 else 2.5
                for ix, iy in iron_list:
                    d = mdist(tx, ty, ix, iy)
                    if d == 0: continue
                    d_back_i = mdist(ix, iy, sx, sy)
                    ta = trns(max(d-1, 0), spd) if d > 1 else 0
                    trip = ta + 1 + trns(d_back_i, spd) + 1
                    v = iv * min(cp, iron_shortfall) / max(1, trip)
                    if v > best_v:
                        best_v = v; best_pos = (ix, iy); best_kind = 'mine'

            if best_pos is not None:
                bx, by = best_pos
                if best_kind == 'mine':
                    action = (f"MINE {tid}" if is_adj(tx, ty, bx, by)
                              else f"MOVE {tid} {bx} {by}")
                elif (tx, ty) == (bx, by):
                    t2 = tree_at.get((bx, by))
                    if t2 is not None:
                        if best_kind == 'chop' and cp > 0:
                            action = f"CHOP {tid}"
                        elif t2['fruits'] > 0 and hp > 0:
                            action = f"HARVEST {tid}"
                else:
                    action = f"MOVE {tid} {bx} {by}"
            elif total > 0:
                action = f"DROP {tid}" if adj_sh else f"MOVE {tid} {sx} {sy}"

            if action:
                actions.append(action)

        sys.stdout.write((';'.join(actions) if actions else "WAIT") + '\n')
        sys.stdout.flush()

main()
```

