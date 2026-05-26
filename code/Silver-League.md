```
import sys

def main():
    inp = sys.stdin.readline
    W, H = map(int, inp().split())
    grid = []; shack = None; eshack = None; iron = []; water = set()
    for y in range(H):
        row = inp().rstrip(); grid.append(row)
        for x, c in enumerate(row):
            if c == '0': shack = (x, y)
            elif c == '1': eshack = (x, y)
            elif c == '+': iron.append((x, y))
            elif c == '~': water.add((x, y))
    sx, sy = shack
    esx, esy = eshack if eshack else (W-1-sx, H-1-sy)
    sadj = []
    for dx, dy in [(1,0),(-1,0),(0,1),(0,-1)]:
        nx, ny = sx+dx, sy+dy
        if 0<=nx<W and 0<=ny<H and grid[ny][nx]=='.': sadj.append((nx,ny))
    sadj_s = set(sadj)
    wadj = set()
    for wx, wy in water:
        for dx, dy in [(1,0),(-1,0),(0,1),(0,-1)]:
            nx, ny = wx+dx, wy+dy
            if 0<=nx<W and 0<=ny<H and grid[ny][nx]=='.': wadj.add((nx,ny))
    esadj = []
    for dx, dy in [(1,0),(-1,0),(0,1),(0,-1)]:
        nx, ny = esx+dx, esy+dy
        if 0<=nx<W and 0<=ny<H and grid[ny][nx]=='.': esadj.append((nx,ny))
    useful_iron = [(ix,iy) for ix,iy in iron
                   if abs(ix-sx)+abs(iy-sy) < abs(ix-esx)+abs(iy-esy)]

    shacks_dist = abs(sx-esx)+abs(sy-esy)
    shacks_close   = shacks_dist <= 12
    shacks_distant = shacks_dist > 16
    # FIX: track if iron deposits exist near our shack
    no_iron_deposits = not useful_iron

    def md(a,b,c,d): return abs(a-c)+abs(b-d)
    def tn(d,s): return (d+s-1)//s if d>0 else 0
    def ia(a,b,c,d): return abs(a-c)+abs(b-d)==1
    def carry(t): return t['cplum']+t['clem']+t['capl']+t['cban']+t['cirn']+t['cwod']
    def depo(t): return t['cirn']+t['cwod']
    def frts(t): return t['cplum']+t['clem']+t['capl']+t['cban']
    def get_seed(t):
        for f,k in [('BANANA','cban'),('APPLE','capl'),('LEMON','clem'),('PLUM','cplum')]:
            if t[k]>0: return f
        return None

    MAX_PL_BASE = max(4, len(sadj)+2)
    turn = 0; my_pl = set()

    # Training candidates: cc>=1 AND hp+cp>0 always
    MANDATORY_CANDS = [c for c in [
        (3,4,2,2),(2,4,2,2),(3,4,1,2),(2,4,1,2),(3,4,0,2),(2,4,0,2),
        (3,4,2,0),(2,4,2,0),(3,4,1,0),(2,4,1,0),
        (1,4,2,2),(1,4,1,2),(1,4,0,2),(1,4,2,0),(1,4,1,0),
        (2,3,2,2),(1,3,2,2),(2,3,1,2),(1,3,1,2),(2,3,0,2),(1,3,0,2),
        (2,3,2,0),(1,3,2,0),(2,3,1,0),(1,3,1,0),
        (2,2,2,2),(1,2,2,2),(2,2,1,2),(1,2,1,2),(2,2,0,2),(1,2,0,2),
        (2,2,2,0),(1,2,2,0),(2,2,1,0),(1,2,1,0),
        (2,1,1,1),(1,1,1,1),(2,1,1,0),(1,1,1,0),(2,1,0,1),(1,1,0,1),
    ] if c[1] >= 1 and c[2]+c[3] > 0]

    # FIX: cp=0 only candidates for iron-scarce situations
    MANDATORY_CANDS_CP0 = [c for c in [
        (3,4,2,0),(2,4,2,0),(1,4,2,0),(3,4,1,0),(2,4,1,0),(1,4,1,0),
        (3,3,2,0),(2,3,2,0),(1,3,2,0),(3,3,1,0),(2,3,1,0),(1,3,1,0),
        (2,2,2,0),(1,2,2,0),(2,2,1,0),(1,2,1,0),(1,1,1,0),
    ] if c[1] >= 1 and c[2] > 0]

    while True:
        turn += 1
        inv = list(map(int, inp().split()))
        opp_inv = list(map(int, inp().split()))
        tc = int(inp()); trees = []; tat = {}
        for _ in range(tc):
            p = inp().split()
            t = {'T':p[0],'x':int(p[1]),'y':int(p[2]),'sz':int(p[3]),
                 'health':int(p[4]),'fruits':int(p[5]),'cd':int(p[6])}
            trees.append(t); tat[(t['x'],t['y'])] = t
        nc = int(inp()); trolls = []
        for _ in range(nc):
            v = inp().split()
            if int(v[1])==0:
                trolls.append({'id':int(v[0]),'x':int(v[2]),'y':int(v[3]),
                               'spd':int(v[4]),'cap':int(v[5]),'hp':int(v[6]),'cp':int(v[7]),
                               'cplum':int(v[8]),'clem':int(v[9]),'capl':int(v[10]),
                               'cban':int(v[11]),'cirn':int(v[12]),'cwod':int(v[13])})
        n = len(trolls); rem = 300-turn
        tp = {(t['x'],t['y']) for t in trees}
        tpos = {(t['x'],t['y']) for t in trolls}
        acts = []; my_pl &= tp; inv_ = inv[:]
        MAX_PLANTS = max(MAX_PL_BASE, n*2+2)
        late_game  = turn >= 150
        early_lemon= turn < 150
        sprint_end = turn >= 285
        ultra_late = turn >= 240

        our_score = inv[0]+inv[1]+inv[2]+inv[3]+inv[5]*4
        opp_score = opp_inv[0]+opp_inv[1]+opp_inv[2]+opp_inv[3]+opp_inv[5]*4

        plum_trees_near = sum(1 for tr in trees if tr['T']=='PLUM' and md(tr['x'],tr['y'],sx,sy)<=12)
        plum_short  = inv_[0] < n+4
        lemon_short = inv_[1] < n+9
        apple_short = inv_[2] < n+4

        # FIX: detect iron scarcity — if no deposits and low iron, conserve by using cp=0
        iron_scarce = no_iron_deposits and inv_[4] <= n + 3

        wt_id=-1; wt_sc=0; raider_id=-1; raider_sc=0
        wood_id=-1; wood_sc=0
        for t in trolls:
            sc1=t['cap']*t['cp']
            if sc1>wt_sc: wt_sc=sc1; wt_id=t['id']
            if t['cp']>=2 and t['cap']>=1:
                sc_w=t['cp']*3+t['cap']*2
                if sc_w>wood_sc: wood_sc=sc_w; wood_id=t['id']
            if t['spd']>=2 and t['cp']+t['hp']>=1 and (n>=3 or t['spd']>=3) and t['cap']>=1:
                dist_o=md(t['x'],t['y'],sx,sy); dist_e=md(t['x'],t['y'],esx,esy)
                sc2=t['spd']*3+t['cp']*2+t['hp']+t['cap']
                if dist_e<=dist_o: sc2+=20
                if sc2>raider_sc: raider_sc=sc2; raider_id=t['id']
        if raider_sc<=3: raider_id=-1

        if shacks_distant and raider_id != -1:
            rt = next((t for t in trolls if t['id']==raider_id), None)
            if rt and md(rt['x'],rt['y'],esx,esy) >= md(rt['x'],rt['y'],sx,sy):
                raider_id = -1

        # FIX: counter_attack uses max(1, n-1) trolls — ALWAYS keep at least 1 home for LEMON harvest
        counter_attack = False
        ca_troll_ids = []
        score_gap = opp_score - our_score
        if score_gap > 10 and rem > 30:
            counter_attack = True
            eligible = [(t['cp']*4 + t['cap'] + t['spd'], t['id'])
                        for t in trolls
                        if t['id'] != raider_id
                        and t['cap'] > 0 and t['cp'] > 0]
            eligible.sort(reverse=True)
            # FIX: never use ALL trolls — keep at least 1 home for harvesting
            max_ca_trolls = max(1, n - 1)  # Always keep 1 troll harvesting locally
            ca_troll_ids = [s[1] for s in eligible[:max_ca_trolls]]
            if not ca_troll_ids: counter_attack = False

        aggressor_id = -1; aggressor_sc = 0
        if shacks_close and n >= 2 and not sprint_end and not counter_attack:
            for t in trolls:
                if t['id'] == raider_id: continue
                if t['id'] in ca_troll_ids: continue
                if t['hp']+t['cp'] == 0 or t['cap'] == 0: continue
                sc = t['hp']*3 + t['cap']*2 + t['spd']
                if sc > aggressor_sc: aggressor_sc = sc; aggressor_id = t['id']
            if aggressor_sc < 2: aggressor_id = -1

        fruit_rusher_id = -1
        if sprint_end:
            best_hr = -1
            for t in trolls:
                if t['id'] == raider_id: continue
                if t['id'] in ca_troll_ids: continue
                if t['hp'] == 0 or t['cap'] == 0: continue
                sc = t['hp']*2 + t['spd'] + t['cap']
                if sc > best_hr: best_hr = sc; fruit_rusher_id = t['id']

        has_cc4=any(t['cap']>=4 for t in trolls)
        endgame=rem<80

        def can_train(ms,cc,hp2,cp2):
            if cc == 0: return False
            if hp2+cp2 == 0: return False
            # FIX: when iron scarce (no deposits), only allow cp=0 to conserve iron
            if iron_scarce and cp2 > 0: return False
            return(inv_[0]>=n+ms*ms and inv_[1]>=n+cc*cc and
                   inv_[2]>=n+hp2*hp2 and inv_[4]>=n+cp2*cp2)

        def do_train(ms,cc,hp2,cp2):
            acts.append(f"TRAIN {ms} {cc} {hp2} {cp2}")
            inv_[0]-=n+ms*ms; inv_[1]-=n+cc*cc
            inv_[2]-=n+hp2*hp2; inv_[4]-=n+cp2*cp2

        def best_troll_candidates():
            has_cp2 = any(t['cp'] >= 2 for t in trolls)
            has_cap4 = any(t['cap'] >= 4 for t in trolls)
            has_spd3 = any(t['spd'] >= 3 for t in trolls)
            iron_avail = bool(useful_iron) or inv_[4] > max(3, n+4)
            dist_e = md(sx, sy, esx, esy)

            # FIX: if no iron deposits, ALWAYS use cp=0 to allow multiple future trainings
            # This is the key fix: on a 6-iron budget with no deposits:
            #   cp=0 allows 3 trainings (n=1:1fe, n=2:2fe, n=3:3fe = 6 total)
            #   cp=2 at n=1 wastes 5fe leaving only 1fe → can NEVER train again
            if no_iron_deposits:
                cands = [(3,4,2,0),(2,4,2,0),(1,4,2,0),(3,4,1,0),(2,4,1,0),(1,4,1,0),
                         (3,3,2,0),(2,3,2,0),(1,3,2,0),(3,3,1,0),(2,3,1,0),(1,3,1,0),
                         (2,2,2,0),(1,2,2,0),(2,2,1,0),(1,2,1,0),(1,1,1,0)]
                # Speed raider is still useful even with cp=0
                if dist_e > 10 and not has_spd3 and n >= 2:
                    cands = [(3,4,2,0),(3,3,2,0),(3,4,1,0),(3,3,1,0)] + cands
                return [c for c in cands if c[1]>=1 and c[2]>0]

            # Normal: iron deposits available, can use cp>0
            cands = []
            if iron_avail and not has_cp2:
                cands += [(2,4,0,3),(1,4,0,3),(2,4,1,2),(1,4,1,2),(2,4,0,2),(1,4,0,2),
                          (2,3,0,3),(1,3,0,3),(2,3,0,2),(1,3,0,2),(2,2,0,2),(1,2,0,2)]
            if dist_e > 10 and not has_spd3 and n >= 2:
                cands += [(3,4,1,2),(3,4,0,2),(3,3,0,2),(3,4,1,0),(3,3,1,0)]
            if not has_cap4:
                cands += [(2,4,2,0),(1,4,2,0),(2,4,1,0),(1,4,1,0)]
            cands += [(2,3,2,1),(1,3,2,1),(2,3,1,1),(1,3,1,1),(2,2,1,1),(1,2,1,1),(1,1,1,1)]
            return [c for c in cands if c[1]>=1 and c[2]+c[3]>0]

        # MANDATORY TRAINING: try every turn until desired count reached
        desired_n = min(6, 2 + turn // 40)
        must_train = n < desired_n and n < 6 and rem >= 20
        trained_mandatory = False

        if must_train:
            # FIX: when iron scarce, use cp=0 candidates; otherwise normal
            base_cands = MANDATORY_CANDS_CP0 if iron_scarce else (best_troll_candidates() + MANDATORY_CANDS)
            all_train_cands = base_cands + [(1,2,1,0),(1,1,1,0)]
            for ms,cc,hp2,cp2 in all_train_cands:
                if can_train(ms,cc,hp2,cp2):
                    do_train(ms,cc,hp2,cp2)
                    trained_mandatory = True; break

        # Regular training (opportunistic)
        if not trained_mandatory and n < 6 and rem >= 15:
            trained = False
            if not trained and n>=2 and 50<=rem<=280 and raider_id==-1:
                rc=[(3,4,1,2),(3,4,0,2),(3,3,0,2),(3,4,2,0),(3,3,2,0),
                    (2,4,1,2),(2,4,0,2),(2,3,0,2),(2,4,2,0),(2,3,2,0),
                    (3,4,1,0),(3,3,1,0),(2,4,1,0),(2,3,1,0),(2,2,1,0)]
                for ms,cc,hp2,cp2 in rc:
                    if ms>=2 and hp2+cp2>=1 and can_train(ms,cc,hp2,cp2):
                        do_train(ms,cc,hp2,cp2); trained=True; break
            if not trained:
                for ms,cc,hp2,cp2 in [(2,4,2,2),(2,4,1,2),(2,4,0,2),(1,4,2,2),(1,4,1,2),(1,4,0,2)]:
                    if can_train(ms,cc,hp2,cp2):
                        do_train(ms,cc,hp2,cp2); trained=True; break
            if not trained:
                for ms,cc,hp2,cp2 in [(2,3,2,2),(2,3,1,2),(2,3,2,1),(2,3,1,1),(2,2,2,1),(2,2,1,2),
                                       (1,3,1,2),(1,3,2,1),(1,3,1,1),(2,2,1,1),(2,2,1,0),(1,3,1,0),
                                       (1,2,1,1),(1,2,1,0),(1,2,0,1),(1,1,1,0),(1,1,0,1),(1,1,1,1)]:
                    if can_train(ms,cc,hp2,cp2):
                        do_train(ms,cc,hp2,cp2); break

        ps=[]; seen=set()
        for p in sadj:
            if p not in tp: ps.append(p); seen.add(p)
        for p in sorted(wadj,key=lambda q:md(q[0],q[1],sx,sy)):
            if p not in tp and p not in seen: ps.append(p); seen.add(p)
        for dy2 in range(-7,8):
            for dx2 in range(-7,8):
                nx2,ny2=sx+dx2,sy+dy2
                if(0<=nx2<W and 0<=ny2<H and grid[ny2][nx2]=='.' and
                   (nx2,ny2) not in tp and (nx2,ny2) not in seen):
                    ps.append((nx2,ny2)); seen.add((nx2,ny2))
        if shacks_distant:
            ps = [p for p in ps if md(p[0],p[1],sx,sy) <= 5]

        ineed=max(0,n+4-inv_[4]) if useful_iron and n<5 else 0
        clm=set(); act_pos=set(); nxt=set(); gpk_pick={}

        def gsh(t):
            tx,ty,tid=t['x'],t['y'],t['id']
            if ia(tx,ty,sx,sy): return f"DROP {tid}"
            best=None; bd=9999
            for p in sadj:
                if p in nxt or (p in clm and p!=(tx,ty)) or (p in tpos and p!=(tx,ty)): continue
                d=md(tx,ty,p[0],p[1])
                if d<bd: bd=d; best=p
            if best is None:
                for p in sadj:
                    if p in nxt or (p in tpos and p!=(tx,ty)): continue
                    d=md(tx,ty,p[0],p[1])
                    if d<bd: bd=d; best=p
            if best is None:
                for p in sadj:
                    if p in tpos and p!=(tx,ty): continue
                    d=md(tx,ty,p[0],p[1])
                    if d<bd: bd=d; best=p
            if best is None and sadj:
                best=min(sadj,key=lambda p:md(tx,ty,p[0],p[1]))
            if best: nxt.add(best); clm.add(best); return f"MOVE {tid} {best[0]} {best[1]}"
            return "WAIT"

        def seed_for_spot(t,sp):
            if early_lemon and plum_trees_near==0 and t['cplum']>0: return 'PLUM'
            if early_lemon and inv_[0]<=n+2 and plum_trees_near<=1 and t['cplum']>0: return 'PLUM'
            if shacks_distant:
                counts={'BANANA':inv_[3],'APPLE':inv_[2],'LEMON':inv_[1],'PLUM':inv_[0]}
                m={'PLUM':t['cplum'],'LEMON':t['clem'],'APPLE':t['capl'],'BANANA':t['cban']}
                order=sorted([f for f in counts if m.get(f,0)>0], key=lambda f:counts[f])
                for f in order:
                    if m.get(f,0)>0: return f
                return None
            if early_lemon: order=['LEMON','BANANA','APPLE','PLUM']
            elif (not has_cc4 and sp in wadj and rem>80 and inv_[1]<n+16): order=['LEMON','APPLE','BANANA','PLUM']
            else: order=['BANANA','APPLE','LEMON','PLUM']
            m={'PLUM':t['cplum'],'LEMON':t['clem'],'APPLE':t['capl'],'BANANA':t['cban']}
            for f in order:
                if m[f]>0: return f
            return None

        def diversity_boost(ttype, t_obj):
            if not shacks_distant: return 1.0
            counts={'PLUM':t_obj['cplum']+inv_[0],'LEMON':t_obj['clem']+inv_[1],
                    'APPLE':t_obj['capl']+inv_[2],'BANANA':t_obj['cban']+inv_[3]}
            total=sum(counts.values())
            if total==0: return 2.0
            ratio=counts.get(ttype,0)/max(1,total)
            return max(0.8, 2.5-ratio*5.0)

        def training_resource_boost(ttype):
            if ttype=='PLUM' and plum_short: return 3.0
            if ttype=='LEMON' and lemon_short: return 2.5
            if ttype=='APPLE' and apple_short: return 2.0
            return 1.0

        def shack_proximity_boost(tree_x, tree_y):
            """Strongly prefer trees near own shack — prevents wasted navigation."""
            dist = md(tree_x, tree_y, sx, sy)
            return max(0.3, 4.0 / max(1, dist / 3))

        def proc(t):
            tx,ty=t['x'],t['y']; tid=t['id']
            spd=max(1,t['spd']); cp=t['cp']; hp=t['hp']
            tot=carry(t); fr=t['cap']-tot
            dp=depo(t); sd=frts(t)
            ash=ia(tx,ty,sx,sy); t2sh=tn(md(tx,ty,sx,sy),spd)

            if t['cap'] == 0: return None

            is_raider=(raider_id==tid and rem>50)
            is_ca=(tid in ca_troll_ids and counter_attack)
            is_aggressor=(aggressor_id==tid and shacks_close and rem>40 and not is_ca)
            is_fruit_rusher=(fruit_rusher_id==tid and sprint_end)
            is_wood_collector=(wood_id==tid and not is_raider and not is_ca
                               and not is_aggressor and not is_fruit_rusher
                               and cp>=2 and rem>40)
            is_wood=(tid==wt_id and wt_sc>=6 and not is_raider and not is_ca
                     and not is_aggressor and not is_fruit_rusher and not is_wood_collector)
            is_planter_only=(hp==0 and cp==0 and not is_raider and not is_ca
                             and not is_aggressor and not is_fruit_rusher)

            if is_planter_only:
                if dp>0 and ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                if dp>0: return gsh(t)
                if sd>0 and len(my_pl)<MAX_PLANTS:
                    for sp in ps:
                        if sp in clm or sp in act_pos: continue
                        if sp in tpos and sp!=(tx,ty): continue
                        if md(sp[0],sp[1],sx,sy)>8: continue
                        clm.add(sp)
                        if(tx,ty)==sp:
                            st2=seed_for_spot(t,sp) or get_seed(t)
                            if st2: my_pl.add(sp); tp.add(sp); act_pos.add(sp); return f"PLANT {tid} {st2}"
                            return None
                        return f"MOVE {tid} {sp[0]} {sp[1]}"
                if ash and fr>0 and len(my_pl)<MAX_PLANTS:
                    fm={'PLUM':0,'LEMON':1,'APPLE':2,'BANANA':3}
                    for pf in ['BANANA','APPLE','LEMON','PLUM']:
                        if inv_[fm[pf]]>0:
                            inv_[fm[pf]]-=1; act_pos.add((tx,ty))
                            return f"PICK {tid} {pf}"
                if sd==0 and fr>0 and len(my_pl)<MAX_PLANTS:
                    if not ash: return gsh(t)
                return None

            if is_fruit_rusher:
                if tot>0 and ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                if fr==0: return gsh(t)
                if tot>0 and rem<=t2sh+2: return gsh(t)
                cur=tat.get((tx,ty))
                if cur and cur['fruits']>0 and (tx,ty) not in act_pos and hp>0:
                    act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                bvfr=-1.0; bpfr=None
                for tree in trees:
                    if tree['fruits']<=0: continue
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    d=md(tx,ty,tree['x'],tree['y'])
                    hf=min(tree['fruits'],fr) if hp>0 else 0
                    if hf<=0: continue
                    v=float(hf)/max(1,d+1)
                    if tree['T'] in ('LEMON','PLUM'): v*=1.5
                    if tree['T']=='BANANA': v*=1.3
                    if v>bvfr: bvfr=v; bpfr=pos
                if bpfr:
                    clm.add(bpfr)
                    if(tx,ty)==bpfr: act_pos.add(bpfr); return f"HARVEST {tid}"
                    return f"MOVE {tid} {bpfr[0]} {bpfr[1]}"
                if tot>0: return gsh(t)
                return None

            if is_wood_collector:
                if tot>=t['cap'] or (tot>0 and rem<=t2sh+2):
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=1: return gsh(t)
                cur=tat.get((tx,ty))
                if cur and cp>0 and fr>0 and (tx,ty) not in act_pos:
                    act_pos.add((tx,ty)); clm.add((tx,ty)); return f"CHOP {tid}"
                bv=-1.0; bp=None
                for tree in trees:
                    if cp<=0 or fr<=0: break
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    dm_=md(tree['x'],tree['y'],sx,sy)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    w=min(tree['sz'],fr)
                    if w<=0: continue
                    ct_=max(1,tn(tree['health'],cp))
                    if tt_+ct_+tb>rem-2: continue
                    v=4.0*w/max(1,tt_+ct_+tb+1)
                    v*=shack_proximity_boost(tree['x'],tree['y'])
                    if v>bv: bv=v; bp=pos
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"CHOP {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                return None

            if is_ca:
                if tot>=t['cap'] or (tot>0 and rem<=t2sh+3):
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=2: return gsh(t)
                bv=-1.0; bp=None; bact='C'
                for tree in trees:
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    de=md(tree['x'],tree['y'],esx,esy)
                    dm_=md(tree['x'],tree['y'],sx,sy)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    if cp>0 and fr>0 and tree['sz']>=1:
                        w=min(tree['sz'],fr)
                        if w>0:
                            ct_=max(1,tn(tree['health'],cp))
                            if tt_+ct_+tb<=rem-2:
                                v=4.0*w/max(1,tt_+ct_+tb+1)
                                v*=(6.0/max(1,de))
                                if v>bv: bv=v; bp=pos; bact='C'
                    if hp>0 and fr>0 and tree['fruits']>0:
                        hf=min(tree['fruits'],fr)
                        if tt_+tb<=rem-1:
                            v=float(hf)/max(1,tt_+tb+1)
                            v*=(3.0/max(1,de))
                            if v>bv: bv=v; bp=pos; bact='H'
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"{'CHOP' if bact=='C' else 'HARVEST'} {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                if esadj:
                    tgt=min(esadj,key=lambda p:md(tx,ty,p[0],p[1]))
                    if(tx,ty)!=tgt: clm.add(tgt); return f"MOVE {tid} {tgt[0]} {tgt[1]}"
                return None

            if is_aggressor:
                if tot>=t['cap'] or (tot>0 and rem<=t2sh+3):
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=2: return gsh(t)
                bv=-1.0; bp=None
                for tree in trees:
                    if tree['fruits']<=0 or hp<=0 or fr<=0: continue
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    de=md(tree['x'],tree['y'],esx,esy); dm_=md(tree['x'],tree['y'],sx,sy)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    if tt_+tb>rem-1: continue
                    hf=min(tree['fruits'],fr)
                    v=float(hf)/max(1,tt_+tb+1)*(1.0+5.0/max(1,de))
                    if v>bv: bv=v; bp=pos
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"HARVEST {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                if esadj:
                    tgt=min(esadj,key=lambda p:md(tx,ty,p[0],p[1]))
                    if(tx,ty)!=tgt: clm.add(tgt); return f"MOVE {tid} {tgt[0]} {tgt[1]}"

            if is_raider and ultra_late:
                if tot>=t['cap'] or (tot>0 and rem<=t2sh+2):
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=1: return gsh(t)
                bv=-1.0; bp=None
                for tree in trees:
                    if cp<=0 or fr<=0: break
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    dm_=md(tree['x'],tree['y'],sx,sy); de=md(tree['x'],tree['y'],esx,esy)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    w=min(tree['sz'],fr)
                    if w<=0: continue
                    ct_=max(1,tn(tree['health'],cp))
                    if tt_+ct_+tb>rem-1: continue
                    is_e=(de<=dm_)
                    v=4.0*w/max(1,tt_+ct_+tb+1)
                    if is_e: v*=3.0
                    v*=(1.0/max(1,d))
                    if v>bv: bv=v; bp=pos
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"CHOP {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                if esadj:
                    tgt=min(esadj,key=lambda p:md(tx,ty,p[0],p[1]))
                    if(tx,ty)!=tgt: clm.add(tgt); return f"MOVE {tid} {tgt[0]} {tgt[1]}"
                return None

            if is_raider and late_game:
                if(tot>0 and rem<=t2sh+3) or tot>=t['cap']:
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=2: return gsh(t)
                bv=-1.0; bp=None
                for tree in trees:
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    if cp<=0 or fr<=0: break
                    dm_=md(tree['x'],tree['y'],sx,sy); de=md(tree['x'],tree['y'],esx,esy)
                    if de>dm_ and de>6: continue
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    w=min(tree['sz'],fr)
                    if w<=0: continue
                    ct_=max(1,tn(tree['health'],cp))
                    if tt_+ct_+tb>rem-2: continue
                    proximity=1.0/max(1,de)
                    v=(4.0*w/max(1,tt_+ct_+tb+1))*(1.0+proximity*5.0)
                    if v>bv: bv=v; bp=pos
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"CHOP {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                if esadj:
                    tgt=min(esadj,key=lambda p:md(tx,ty,p[0],p[1]))
                    if(tx,ty)!=tgt: clm.add(tgt); return f"MOVE {tid} {tgt[0]} {tgt[1]}"

            if is_raider:
                if(tot>0 and rem<=t2sh+3) or tot>=t['cap']:
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=2: return gsh(t)
                bv=-1.0; bp=None; bact='C'
                for tree in trees:
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    dm_=md(tree['x'],tree['y'],sx,sy); de=md(tree['x'],tree['y'],esx,esy)
                    is_e=(de<=dm_)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    if cp>0 and fr>0:
                        w=min(tree['sz'],fr)
                        if w>0:
                            ct_=max(1,tn(tree['health'],cp))
                            if tt_+ct_+tb<=rem-2:
                                v=4.0*w/max(1,tt_+ct_+tb+1)
                                if is_e: v*=2.0
                                if early_lemon and tree['T']=='LEMON' and tree['fruits']>0 and hp>0: v*=0.1
                                if v>bv: bv=v; bp=pos; bact='C'
                    if hp>0 and fr>0 and tree['fruits']>0:
                        hf=min(tree['fruits'],fr)
                        if tt_+tb<=rem-1:
                            v=float(hf)/max(1,tt_+tb+1)
                            if is_e: v*=1.5
                            if early_lemon and tree['T']=='LEMON': v*=6.0
                            if v>bv: bv=v; bp=pos; bact='H'
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"{'CHOP' if bact=='C' else 'HARVEST'} {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                if esadj:
                    tgt=min(esadj,key=lambda p:md(tx,ty,p[0],p[1]))
                    if(tx,ty)!=tgt: clm.add(tgt); return f"MOVE {tid} {tgt[0]} {tgt[1]}"

            if endgame:
                if tot>0 and ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                if tot>=t['cap']: return gsh(t)
                if dp>0 and t2sh<=3: return gsh(t)
                if shacks_distant and hp>0 and fr>0:
                    cur=tat.get((tx,ty))
                    if cur and cur['fruits']>0 and (tx,ty) not in act_pos:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                if cp>0 and fr>0 and not shacks_distant:
                    cur=tat.get((tx,ty))
                    if cur and (tx,ty) not in act_pos:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"CHOP {tid}"
                    bv2=-1; bp2=None
                    for tree in trees:
                        pos=(tree['x'],tree['y'])
                        if pos in clm or pos in act_pos: continue
                        if pos in tpos and pos!=(tx,ty): continue
                        d=md(tx,ty,tree['x'],tree['y'])
                        tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                        w=min(tree['sz'],fr); ct_=max(1,tn(tree['health'],cp))
                        v=4.0*w/max(1,d+ct_+tb+1)
                        if v>bv2: bv2=v; bp2=pos
                    if bp2:
                        clm.add(bp2)
                        if(tx,ty)==bp2: act_pos.add(bp2); return f"CHOP {tid}"
                        return f"MOVE {tid} {bp2[0]} {bp2[1]}"
                if tot>0: return gsh(t)
                if hp>0 and fr>0:
                    cur=tat.get((tx,ty))
                    if cur and cur['fruits']>0 and (tx,ty) not in act_pos:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                    bv2=-1; bp2=None
                    for tree in trees:
                        if tree['fruits']<=0: continue
                        pos=(tree['x'],tree['y'])
                        if pos in clm or pos in act_pos: continue
                        if pos in tpos and pos!=(tx,ty): continue
                        if shacks_distant and md(tree['x'],tree['y'],sx,sy)>10: continue
                        d=md(tx,ty,tree['x'],tree['y'])
                        tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                        v=float(min(tree['fruits'],fr))/max(1,d+tb+1)
                        v*=diversity_boost(tree['T'],t)
                        if v>bv2: bv2=v; bp2=pos
                    if bp2:
                        clm.add(bp2)
                        if(tx,ty)==bp2: act_pos.add(bp2); return f"HARVEST {tid}"
                        return f"MOVE {tid} {bp2[0]} {bp2[1]}"
                return None

            if tot>0 and rem<=t2sh+2: return gsh(t)
            if dp>0 and ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
            if dp>0 and fr==0: return gsh(t)
            if sd>0 and fr==0 and len(my_pl)>=MAX_PLANTS:
                if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                return gsh(t)

            if late_game and not is_raider and not is_ca and not is_aggressor and not is_fruit_rusher and not is_wood_collector and hp>0 and fr>0:
                cur=tat.get((tx,ty))
                if cur and (tx,ty) not in act_pos:
                    if cur['T'] in ('LEMON','PLUM') and cur['fruits']>0:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                bvlp=-1.0; bplp=None
                for tree in trees:
                    if tree['fruits']<=0: continue
                    if tree['T'] not in ('LEMON','PLUM'): continue
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    if shacks_distant and md(tree['x'],tree['y'],sx,sy)>10: continue
                    dist_own=md(tree['x'],tree['y'],sx,sy)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd)
                    hf=min(tree['fruits'],fr); ht2=max(1,tn(hf,hp))
                    tb=tn(dist_own,spd)
                    v=(float(hf)/max(1,tt_+ht2+tb+1))*(8.0/max(1,dist_own))
                    if pos!=(tx,ty):
                        ddx=pos[0]-tx; ddy=pos[1]-ty
                        if abs(ddx)>=abs(ddy) and ddx!=0: fs=(tx+(1 if ddx>0 else -1),ty)
                        elif ddy!=0: fs=(tx,ty+(1 if ddy>0 else -1))
                        else: fs=pos
                        if fs in tpos and fs!=(tx,ty): v*=0.3
                    if v>bvlp: bvlp=v; bplp=pos
                if bplp is not None:
                    clm.add(bplp)
                    if(tx,ty)==bplp: act_pos.add(bplp); return f"HARVEST {tid}"
                    return f"MOVE {tid} {bplp[0]} {bplp[1]}"
                if tot>0 and t2sh<=2:
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)

            cur=tat.get((tx,ty))
            if cur and (tx,ty) not in act_pos:
                T2,sz,health,fruits=cur['T'],cur['sz'],cur['health'],cur['fruits']
                can_h=hp>0 and fruits>0 and fr>0
                can_c=cp>0 and fr>0 and (tx,ty) not in my_pl
                if can_h and can_c:
                    if early_lemon and T2=='LEMON' and fruits>0:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                    if early_lemon and T2=='BANANA' and fruits>0:
                        lemon_available=any(tr['T']=='LEMON' and tr['fruits']>0 for tr in trees
                                           if (tr['x'],tr['y'])!=(tx,ty))
                        if not lemon_available:
                            act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                    ct_=max(1,tn(health,cp)); cv=4.0*min(sz,fr)/max(1,ct_+t2sh+1)
                    if is_wood or is_wood_collector: cv*=2.0
                    if shacks_distant: cv*=0.15
                    hf=min(fruits,fr); hv=float(hf)/max(1,tn(hf,hp)+t2sh+1)
                    if early_lemon:
                        if T2=='LEMON': hv*=6.0
                        elif T2=='BANANA': hv*=1.8
                        elif T2=='PLUM': hv*=1.2
                    elif shacks_distant: hv*=diversity_boost(T2,t)
                    elif T2=='BANANA': hv*=1.5
                    act_pos.add((tx,ty)); clm.add((tx,ty))
                    return f"{'CHOP' if cv>=hv else 'HARVEST'} {tid}"
                elif can_c:
                    if early_lemon and T2=='LEMON' and fruits>0 and hp>0:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                    if shacks_distant and can_h:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                    if not shacks_distant:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"CHOP {tid}"
                    act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                elif can_h: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
            if((tx,ty) not in tp and (tx,ty) not in clm and (tx,ty) not in act_pos
               and len(my_pl)<MAX_PLANTS and ((tx,ty) in wadj or (tx,ty) in sadj_s)):
                st=seed_for_spot(t,(tx,ty))
                if st:
                    my_pl.add((tx,ty)); tp.add((tx,ty))
                    act_pos.add((tx,ty)); clm.add((tx,ty))
                    return f"PLANT {tid} {st}"
            if cp>0 and ineed>0 and fr>0 and sd==0:
                for ix,iy in useful_iron:
                    if ia(tx,ty,ix,iy): act_pos.add((tx,ty)); return f"MINE {tid}"

            bv=-1.0; bp=None; bt=None
            def con(pos,v,btype):
                nonlocal bv,bp,bt
                if pos in clm or pos in act_pos: return
                if pos in tpos and pos!=(tx,ty): return
                if pos!=(tx,ty):
                    ddx=pos[0]-tx; ddy=pos[1]-ty
                    if abs(ddx)>=abs(ddy) and ddx!=0: fs=(tx+(1 if ddx>0 else -1),ty)
                    elif ddy!=0: fs=(tx,ty+(1 if ddy>0 else -1))
                    else: fs=pos
                    if fs in tpos and fs!=(tx,ty): v*=0.3
                if v>bv: bv=v; bp=pos; bt=btype

            if hp>0 and fr>0:
                for tree in trees:
                    if tree['fruits']<=0: continue
                    pos=(tree['x'],tree['y'])
                    if shacks_distant and md(tree['x'],tree['y'],sx,sy)>10: continue
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd)
                    tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                    hf=min(tree['fruits'],fr); ht2=max(1,tn(hf,hp))
                    v=float(hf)/max(1,tt_+ht2+tb+1)
                    if early_lemon:
                        if tree['T']=='LEMON': v*=6.0
                        elif tree['T']=='BANANA': v*=1.8
                        elif tree['T']=='PLUM': v*=1.2
                        elif tree['T']=='APPLE': v*=1.1
                    elif shacks_distant: v*=1.5*diversity_boost(tree['T'],t)
                    elif late_game and tree['T'] in ('LEMON','PLUM'):
                        dist_own=md(tree['x'],tree['y'],sx,sy)
                        v*=max(3.0, 10.0/max(1,dist_own))
                    else:
                        if tree['T']=='BANANA': v*=1.5
                        if not has_cc4:
                            if tree['T']=='LEMON' and inv_[1]<n+16: v*=1.3
                            elif tree['T']=='APPLE' and inv_[2]<n+4: v*=1.1
                    if not endgame and not late_game:
                        v*=training_resource_boost(tree['T'])
                    # Boost nearby trees to prefer local harvest over distant wandering
                    v*=shack_proximity_boost(tree['x'],tree['y'])
                    con(pos,v,'H')
            if cp>0 and fr>0:
                for tree in trees:
                    pos=(tree['x'],tree['y']); sz=tree['sz']
                    if pos in my_pl and sz<3: continue
                    if sz<2 and tree['T']!='BANANA': continue
                    if early_lemon and tree['T']=='LEMON' and tree['fruits']>0 and hp>0: continue
                    if late_game and tree['T'] in ('LEMON','PLUM'):
                        dist_own=md(tree['x'],tree['y'],sx,sy)
                        if dist_own<=8 and tree['fruits']>0 and hp>0: continue
                    if shacks_distant: continue
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd)
                    tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                    w=min(sz,fr)
                    if w<=0: continue
                    ct_=max(1,tn(tree['health'],cp))
                    v=4.0*w/max(1,tt_+ct_+tb+1)
                    if is_wood: v*=1.5
                    if not is_wood_collector:
                        v*=shack_proximity_boost(tree['x'],tree['y'])
                    con(pos,v,'C')
            if sd>0 and rem>15 and len(my_pl)<MAX_PLANTS:
                for sp in ps[:15]:
                    if sp in tpos and sp!=(tx,ty): continue
                    d=md(tx,ty,sp[0],sp[1]); tt_=tn(d,spd)
                    if sp in sadj_s: gr,base=2,22
                    elif sp in wadj: gr,base=4,13
                    else: gr,base=12,7
                    v=float(base)/max(1,tt_+gr+4)
                    if shacks_distant:
                        dist_sp=md(sp[0],sp[1],sx,sy)
                        v*=max(1.0, 5.0/max(1,dist_sp))
                    con(sp,v,'P')
            if sd==0 and ash and fr>0 and rem>20 and len(my_pl)<MAX_PLANTS:
                for sp in ps[:6]:
                    if sp in clm or sp in act_pos: continue
                    dsp=md(tx,ty,sp[0],sp[1])
                    if dsp>2: continue
                    if early_lemon and plum_trees_near==0 and inv_[0]>0:
                        pto=['PLUM','LEMON','BANANA','APPLE']
                    elif early_lemon and inv_[0]<=n+2 and plum_trees_near<=1 and inv_[0]>0:
                        pto=['PLUM','LEMON','BANANA','APPLE']
                    elif shacks_distant:
                        counts={'BANANA':inv_[3],'APPLE':inv_[2],'LEMON':inv_[1],'PLUM':inv_[0]}
                        pto=sorted(['BANANA','APPLE','LEMON','PLUM'], key=lambda f:counts[f])
                    elif early_lemon:
                        pto=['LEMON','BANANA','APPLE','PLUM']
                    else:
                        lp2=(not has_cc4 and sp in wadj and rem>80 and inv_[1]<n+16)
                        pto=['LEMON','APPLE','BANANA','PLUM'] if lp2 else ['BANANA','APPLE','LEMON','PLUM']
                    fm={'PLUM':0,'LEMON':1,'APPLE':2,'BANANA':3}
                    ft,pidx=None,-1
                    for pf in pto:
                        if inv_[fm[pf]]>0: ft,pidx=pf,fm[pf]; break
                    if ft is None: continue
                    if sp in sadj_s: gr,base=2,22
                    elif sp in wadj: gr,base=4,13
                    else: gr,base=12,7
                    v=float(base)*0.8/max(1,(1+tn(dsp,spd))+gr+4)
                    gpk_pick[tid]=(ft,pidx); con((tx,ty),v,'gpk'); break
            if cp>0 and fr>0 and ineed>0 and useful_iron and sd==0:
                for ix,iy in useful_iron:
                    for ddx,ddy in [(0,1),(0,-1),(1,0),(-1,0)]:
                        ap=(ix+ddx,iy+ddy)
                        if not(0<=ap[0]<W and 0<=ap[1]<H) or grid[ap[1]][ap[0]]!='.': continue
                        d=md(tx,ty,ap[0],ap[1]); tt_=tn(d,spd)
                        v=float(min(cp,ineed))*0.4/max(1,tt_+1)
                        con(ap,v,'M')
            if bp is None:
                if sd>0 and len(my_pl)<MAX_PLANTS:
                    for sp in ps:
                        if sp in clm or sp in act_pos: continue
                        if sp in tpos and sp!=(tx,ty): continue
                        clm.add(sp)
                        if(tx,ty)==sp:
                            st2=seed_for_spot(t,sp) or get_seed(t)
                            if st2: my_pl.add(sp); tp.add(sp); act_pos.add(sp); return f"PLANT {tid} {st2}"
                            return None
                        return f"MOVE {tid} {sp[0]} {sp[1]}"
                if tot>0:
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                return None
            clm.add(bp); bx,by=bp
            if(tx,ty)==bp:
                cur2=tat.get(bp)
                if bt=='C' and cur2 and cp>0: act_pos.add(bp); return f"CHOP {tid}"
                if bt=='H' and cur2 and cur2['fruits']>0: act_pos.add(bp); return f"HARVEST {tid}"
                if bt=='P' and bp not in tp:
                    st2=seed_for_spot(t,bp) or get_seed(t)
                    if st2: my_pl.add(bp); tp.add(bp); act_pos.add(bp); return f"PLANT {tid} {st2}"
                if bt=='gpk' and ash and tid in gpk_pick:
                    ft,pidx=gpk_pick[tid]
                    if inv_[pidx]>0: inv_[pidx]-=1; act_pos.add(bp); return f"PICK {tid} {ft}"
                if bt=='M':
                    for ix,iy in useful_iron:
                        if ia(tx,ty,ix,iy): act_pos.add(bp); return f"MINE {tid}"
                return None
            else:
                if bt=='M':
                    for ix,iy in useful_iron:
                        if ia(tx,ty,ix,iy): act_pos.add((tx,ty)); return f"MINE {tid}"
                return f"MOVE {tid} {bx} {by}"

        def sort_key(t):
            if t['id']==raider_id and rem>50: return(-9999,0,0)
            if t['id'] in ca_troll_ids and counter_attack: return(-9997,0,0)
            if t['id']==wood_id and wood_sc>0: return(-9996,0,0)
            if t['id']==aggressor_id and shacks_close: return(-9995,0,0)
            if t['id']==fruit_rusher_id and sprint_end: return(-8999,0,0)
            if t['id']==wt_id and wt_sc>=6: return(-8888,0,0)
            return(-depo(t),-frts(t),md(t['x'],t['y'],sx,sy))

        for t in sorted(trolls,key=sort_key):
            a=proc(t)
            if a: acts.append(a)

        delta = our_score - opp_score
        sign = '+' if delta >= 0 else ''
        acts.append(f"MSG T{turn} {sign}{delta} {n}t {inv_[4]}fe")

        print(';'.join(acts) if acts else 'WAIT')
        sys.stdout.flush()

main()

```
