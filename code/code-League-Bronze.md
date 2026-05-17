Code 165th (out of 485)

```
import sys

def main():
    read = sys.stdin.readline
    width, height = map(int, read().split())
    my_shack = None
    iron_list = []
    for i in range(height):
        row = read().rstrip()
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
    def mdist(x1, y1, x2, y2): return abs(x1-x2)+abs(y1-y2)
    def is_adj(x1, y1, x2, y2): return abs(x1-x2)+abs(y1-y2) == 1
    def can_train(inv, n, m, c, h, ch):
        return (inv[0] >= n+m*m and inv[1] >= n+c*c and
                inv[2] >= n+h*h and inv[4] >= n+ch*ch)
    turn = 0
    while True:
        turn += 1
        my_inv = list(map(int, read().split()))
        read()
        tc = int(read())
        trees = []; tree_at = {}
        for _ in range(tc):
            p = read().split()
            t = {'type': p[0], 'x': int(p[1]), 'y': int(p[2]),
                 'size': int(p[3]), 'health': int(p[4]),
                 'fruits': int(p[5]), 'cooldown': int(p[6])}
            trees.append(t); tree_at[(t['x'], t['y'])] = t
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
                    'cirn': int(v[12]), 'cwod': int(v[13]),
                })
        n = len(my_trolls)
        actions = []
        # Training: best affordable build first
        if n < 4 and turn < 260:
            builds = ([(2,2,1,2),(1,2,1,2),(2,2,1,1),(1,2,1,1),
                       (2,2,0,2),(1,2,0,2),(2,2,0,1),(1,2,0,1),
                       (1,1,1,1),(1,1,0,1)] if iron_list else
                      [(2,2,2,0),(1,2,2,0),(2,2,1,0),(1,2,1,0),(1,1,1,0)])
            for m, c, h, ch in builds:
                if can_train(my_inv, n, m, c, h, ch):
                    actions.append(f"TRAIN {m} {c} {h} {ch}")
                    break
        need_fruits = (n < 4 and
                       not can_train(my_inv, n, 1, 2, 1, 0) and
                       not (iron_list and can_train(my_inv, n, 1, 2, 0, 1)))
        for troll in my_trolls:
            tx, ty = troll['x'], troll['y']
            total = (troll['cplum'] + troll['clem'] + troll['capl'] +
                     troll['cban'] + troll['cirn'] + troll['cwod'])
            free = troll['cap'] - total
            tid = troll['id']
            spd = max(1, troll['spd'])
            cp = troll['cp']; hp = troll['hp']
            adj_sh = is_adj(tx, ty, sx, sy)
            # Endgame: return immediately
            if turn >= 293 and total > 0:
                actions.append(f"DROP {tid}" if adj_sh else f"MOVE {tid} {sx} {sy}")
                continue
            # ONLY return when FULL (key fix!)
            if free == 0:
                actions.append(f"DROP {tid}" if adj_sh else f"MOVE {tid} {sx} {sy}")
                continue
            action = None
            # Act on current cell tree
            cur = tree_at.get((tx, ty))
            if cur is not None and free > 0:
                mh = MAX_HP.get(cur['type'], [0, 99, 99, 99, 99])
                sz = min(cur['size'], 4)
                is_damaged = sz > 0 and cur['health'] < mh[sz]
                if cp > 0 and is_damaged:
                    action = f"CHOP {tid}"
                elif cur['fruits'] > 0 and hp > 0:
                    action = (f"HARVEST {tid}" if (need_fruits or cp == 0)
                              else f"CHOP {tid}")
                elif cp > 0:
                    action = f"CHOP {tid}"
                if action:
                    actions.append(action); continue
            # Value-based target selection
            best_v = -1.0; best_pos = None; best_kind = None
            for t in trees:
                d = mdist(tx, ty, t['x'], t['y'])
                tt = (d + spd - 1) // spd if d > 0 else 0
                if t['fruits'] > 0 and hp > 0 and free > 0:
                    fg = min(t['fruits'], free, hp)
                    v = (1.5 if need_fruits else 1.0) * fg / max(1, tt + 1)
                    if v > best_v:
                        best_v = v; best_pos = (t['x'], t['y']); best_kind = 'harvest'
                if cp > 0 and free > 0:
                    chops = max(1, (t['health'] + cp - 1) // cp)
                    wood = min(t['size'], free)
                    if wood > 0:
                        v = (2.0 if need_fruits else 4.0) * wood / max(1, tt + chops)
                        if v > best_v:
                            best_v = v; best_pos = (t['x'], t['y']); best_kind = 'chop'
            # Iron mining for training
            if n < 4 and cp > 0 and free > 0:
                iv = 10.0 if my_inv[4] < 5 else (4.0 if my_inv[4] < 20 else 0.0)
                if iv > 0:
                    for ix, iy in iron_list:
                        d = mdist(tx, ty, ix, iy)
                        if d == 0: continue
                        ta = (d - 1 + spd - 1) // spd if d > 1 else 0
                        v = iv / max(1, ta + 1)
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

