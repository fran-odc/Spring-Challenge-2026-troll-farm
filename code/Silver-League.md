Code 119th (out of 909)

<!-- 
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

    while True:
        turn += 1
        inv = list(map(int, inp().split())); inp()
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
        late_game = turn >= 150

        wt_id=-1; wt_sc=0; raider_id=-1; raider_sc=0
        for t in trolls:
            sc1=t['cap']*t['cp']
            if sc1>wt_sc: wt_sc=sc1; wt_id=t['id']
            if t['spd']>=2 and t['cp']+t['hp']>=1 and (n>=3 or t['spd']>=3):
                dist_o=md(t['x'],t['y'],sx,sy); dist_e=md(t['x'],t['y'],esx,esy)
                sc2=t['spd']*3+t['cp']*2+t['hp']+t['cap']
                if dist_e<=dist_o: sc2+=20
                if sc2>raider_sc: raider_sc=sc2; raider_id=t['id']
        if raider_sc<=3: raider_id=-1

        has_cc4=any(t['cap']>=4 for t in trolls)
        endgame=rem<80

        def can_train(ms,cc,hp2,cp2):
            return(inv_[0]>=n+ms*ms and inv_[1]>=n+cc*cc and
                   inv_[2]>=n+hp2*hp2 and inv_[4]>=n+cp2*cp2)

        if n<6 and rem>=15:
            trained=False
            if 85<=turn<=90 and not trained:
                dist_enemy=md(sx,sy,esx,esy)
                has_fast=any(t['spd']>=3 for t in trolls)
                need_speed=(not has_fast and raider_id==-1 and dist_enemy>max(W//3,7))
                if need_speed:
                    for cc,hp2,cp2 in [(4,1,2),(4,0,2),(4,2,0),(3,1,2),(3,0,2),
                                       (4,1,0),(2,1,2),(2,0,2),(2,2,0),(2,1,0),
                                       (3,1,0),(2,0,0),(1,1,0),(1,0,0)]:
                        if can_train(4,cc,hp2,cp2):
                            acts.append(f"TRAIN 4 {cc} {hp2} {cp2}")
                            inv_[0]-=n+16; inv_[1]-=n+cc*cc
                            inv_[2]-=n+hp2*hp2; inv_[4]-=n+cp2*cp2
                            trained=True; break
                else:
                    for ms,hp2,cp2 in [(2,2,2),(2,1,2),(2,0,2),(1,2,2),(1,1,2),
                                       (2,2,0),(2,1,0),(1,2,0),(1,1,0),(1,0,2),(1,0,0)]:
                        if can_train(ms,4,hp2,cp2):
                            acts.append(f"TRAIN {ms} 4 {hp2} {cp2}")
                            inv_[0]-=n+ms*ms; inv_[1]-=n+16
                            inv_[2]-=n+hp2*hp2; inv_[4]-=n+cp2*cp2
                            trained=True; break
            if not trained and n>=2 and 50<=rem<=280 and raider_id==-1:
                rc=[(3,4,1,2),(3,4,0,2),(3,3,0,2),(3,4,2,0),(3,3,2,0),
                    (2,4,1,2),(2,4,0,2),(2,3,0,2),(2,4,2,0),(2,3,2,0),
                    (3,4,1,0),(3,3,1,0),(2,4,1,0),(2,3,1,0),(2,2,1,0)]
                for ms,cc,hp2,cp2 in rc:
                    if ms>=2 and cp2+hp2>=1 and can_train(ms,cc,hp2,cp2):
                        acts.append(f"TRAIN {ms} {cc} {hp2} {cp2}")
                        inv_[0]-=n+ms*ms; inv_[1]-=n+cc*cc
                        inv_[2]-=n+hp2*hp2; inv_[4]-=n+cp2*cp2
                        trained=True; break
            if not trained:
                for ms,cc,hp2,cp2 in [(2,4,2,2),(2,4,1,2),(2,4,0,2),(1,4,2,2),(1,4,1,2),(1,4,0,2)]:
                    if can_train(ms,cc,hp2,cp2):
                        acts.append(f"TRAIN {ms} {cc} {hp2} {cp2}")
                        inv_[0]-=n+ms*ms; inv_[1]-=n+cc*cc
                        inv_[2]-=n+hp2*hp2; inv_[4]-=n+cp2*cp2
                        trained=True; break
            if not trained:
                for ms,cc,hp2,cp2 in [(2,3,2,2),(2,3,1,2),(2,3,2,1),(2,3,1,1),(2,2,2,1),(2,2,1,2),
                                       (1,3,1,2),(1,3,2,1),(1,3,1,1),(2,2,1,1),(2,2,1,0),(1,3,1,0),
                                       (1,2,1,1),(1,2,1,0),(1,2,0,1),(1,1,1,0),(1,1,0,1),(1,1,0,0)]:
                    if can_train(ms,cc,hp2,cp2):
                        acts.append(f"TRAIN {ms} {cc} {hp2} {cp2}")
                        inv_[0]-=n+ms*ms; inv_[1]-=n+cc*cc
                        inv_[2]-=n+hp2*hp2; inv_[4]-=n+cp2*cp2
                        break

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
            lp=(not has_cc4 and sp in wadj and rem>80 and inv_[1]<n+16)
            order=['LEMON','APPLE','BANANA','PLUM'] if lp else ['BANANA','APPLE','LEMON','PLUM']
            m={'PLUM':t['cplum'],'LEMON':t['clem'],'APPLE':t['capl'],'BANANA':t['cban']}
            for f in order:
                if m[f]>0: return f
            return None

        def proc(t):
            tx,ty=t['x'],t['y']; tid=t['id']
            spd=max(1,t['spd']); cp=t['cp']; hp=t['hp']
            tot=carry(t); fr=t['cap']-tot
            dp=depo(t); sd=frts(t)
            ash=ia(tx,ty,sx,sy); t2sh=tn(md(tx,ty,sx,sy),spd)
            is_raider=(raider_id==tid and rem>50)
            is_wood=(tid==wt_id and wt_sc>=6 and not is_raider)

            # ===LATE GAME RAIDER: from turn 150 chop wood near enemy tent===
            if is_raider and late_game:
                if(tot>0 and rem<=t2sh+3) or tot>=t['cap']:
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)
                if dp>0 and t2sh<=2: return gsh(t)
                # CHOP-only mode: target trees closest to enemy shack
                bv=-1.0; bp=None
                for tree in trees:
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    if cp<=0 or fr<=0: break
                    dm_=md(tree['x'],tree['y'],sx,sy)
                    de=md(tree['x'],tree['y'],esx,esy)
                    # Only trees on enemy side or near enemy shack
                    if de>dm_ and de>6: continue
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd); tb=tn(dm_,spd)
                    w=min(tree['sz'],fr)
                    if w<=0: continue
                    ct_=max(1,tn(tree['health'],cp))
                    if tt_+ct_+tb>rem-2: continue
                    # Strongly prefer trees closest to enemy shack
                    proximity=1.0/max(1,de)
                    v=(4.0*w/max(1,tt_+ct_+tb+1))*(1.0+proximity*5.0)
                    if v>bv: bv=v; bp=pos
                if bp is not None:
                    clm.add(bp)
                    if(tx,ty)==bp: act_pos.add(bp); return f"CHOP {tid}"
                    return f"MOVE {tid} {bp[0]} {bp[1]}"
                # Move toward enemy tent if no chop target found
                if esadj:
                    tgt=min(esadj,key=lambda p:md(tx,ty,p[0],p[1]))
                    if(tx,ty)!=tgt: clm.add(tgt); return f"MOVE {tid} {tgt[0]} {tgt[1]}"
                return None

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
                                if v>bv: bv=v; bp=pos; bact='C'
                    if hp>0 and fr>0 and tree['fruits']>0:
                        hf=min(tree['fruits'],fr)
                        if tt_+tb<=rem-1:
                            v=float(hf)/max(1,tt_+tb+1)
                            if is_e: v*=1.5
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
                if cp>0 and fr>0:
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
                        d=md(tx,ty,tree['x'],tree['y'])
                        tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                        v=float(min(tree['fruits'],fr))/max(1,d+tb+1)
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

            # ===LATE GAME NON-RAIDER: from turn 150 harvest LEMON/PLUM near own tent===
            if late_game and not is_raider and hp>0 and fr>0:
                # Check for LEMON/PLUM with fruits near own shack first
                cur=tat.get((tx,ty))
                if cur and (tx,ty) not in act_pos:
                    if cur['T'] in ('LEMON','PLUM') and cur['fruits']>0:
                        act_pos.add((tx,ty)); clm.add((tx,ty)); return f"HARVEST {tid}"
                # Find best LEMON/PLUM close to own shack
                bvlp=-1.0; bplp=None
                for tree in trees:
                    if tree['fruits']<=0: continue
                    if tree['T'] not in ('LEMON','PLUM'): continue
                    pos=(tree['x'],tree['y'])
                    if pos in clm or pos in act_pos: continue
                    if pos in tpos and pos!=(tx,ty): continue
                    dist_own=md(tree['x'],tree['y'],sx,sy)
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd)
                    hf=min(tree['fruits'],fr); ht2=max(1,tn(hf,hp))
                    tb=tn(dist_own,spd)
                    # Heavily reward closeness to own shack
                    shack_bonus=8.0/max(1,dist_own)
                    v=(float(hf)/max(1,tt_+ht2+tb+1))*shack_bonus
                    # Also check first step not blocked
                    if pos!=(tx,ty):
                        ddx=pos[0]-tx; ddy=pos[1]-ty
                        if abs(ddx)>=abs(ddy) and ddx!=0:
                            fs=(tx+(1 if ddx>0 else -1),ty)
                        elif ddy!=0:
                            fs=(tx,ty+(1 if ddy>0 else -1))
                        else:
                            fs=pos
                        if fs in tpos and fs!=(tx,ty): v*=0.3
                    if v>bvlp: bvlp=v; bplp=pos
                if bplp is not None:
                    clm.add(bplp)
                    if(tx,ty)==bplp: act_pos.add(bplp); return f"HARVEST {tid}"
                    return f"MOVE {tid} {bplp[0]} {bplp[1]}"
                # No LEMON/PLUM near own shack: drop if carrying, then harvest any fruit
                if tot>0 and t2sh<=2:
                    if ash: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"DROP {tid}"
                    return gsh(t)

            cur=tat.get((tx,ty))
            if cur and (tx,ty) not in act_pos:
                T2,sz,health,fruits=cur['T'],cur['sz'],cur['health'],cur['fruits']
                can_h=hp>0 and fruits>0 and fr>0
                can_c=cp>0 and fr>0 and (tx,ty) not in my_pl
                if can_h and can_c:
                    ct_=max(1,tn(health,cp)); cv=4.0*min(sz,fr)/max(1,ct_+t2sh+1)
                    if is_wood: cv*=1.5
                    hf=min(fruits,fr); hv=float(hf)/max(1,tn(hf,hp)+t2sh+1)
                    if T2=='BANANA': hv*=1.5
                    act_pos.add((tx,ty)); clm.add((tx,ty))
                    return f"{'CHOP' if cv>=hv else 'HARVEST'} {tid}"
                elif can_c: act_pos.add((tx,ty)); clm.add((tx,ty)); return f"CHOP {tid}"
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
                    if abs(ddx)>=abs(ddy) and ddx!=0:
                        fs=(tx+(1 if ddx>0 else -1),ty)
                    elif ddy!=0:
                        fs=(tx,ty+(1 if ddy>0 else -1))
                    else:
                        fs=pos
                    if fs in tpos and fs!=(tx,ty): v*=0.3
                if v>bv: bv=v; bp=pos; bt=btype

            if hp>0 and fr>0:
                for tree in trees:
                    if tree['fruits']<=0: continue
                    pos=(tree['x'],tree['y'])
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd)
                    tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                    hf=min(tree['fruits'],fr); ht2=max(1,tn(hf,hp))
                    v=float(hf)/max(1,tt_+ht2+tb+1)
                    if tree['T']=='BANANA': v*=1.5
                    # Late game: boost LEMON/PLUM near own shack significantly
                    if late_game and tree['T'] in ('LEMON','PLUM'):
                        dist_own=md(tree['x'],tree['y'],sx,sy)
                        v*=max(3.0, 10.0/max(1,dist_own))
                    elif not has_cc4:
                        if tree['T']=='LEMON' and inv_[1]<n+16: v*=1.3
                        elif tree['T']=='APPLE' and inv_[2]<n+4: v*=1.1
                    con(pos,v,'H')
            if cp>0 and fr>0:
                for tree in trees:
                    pos=(tree['x'],tree['y']); sz=tree['sz']
                    if pos in my_pl and sz<3: continue
                    if sz<2 and tree['T']!='BANANA': continue
                    # Late game: skip chopping LEMON/PLUM near own shack (harvest them instead)
                    if late_game and tree['T'] in ('LEMON','PLUM'):
                        dist_own=md(tree['x'],tree['y'],sx,sy)
                        if dist_own<=8 and tree['fruits']>0 and hp>0: continue
                    d=md(tx,ty,tree['x'],tree['y']); tt_=tn(d,spd)
                    tb=tn(md(tree['x'],tree['y'],sx,sy),spd)
                    w=min(sz,fr)
                    if w<=0: continue
                    ct_=max(1,tn(tree['health'],cp))
                    v=4.0*w/max(1,tt_+ct_+tb+1)
                    if is_wood: v*=1.5
                    con(pos,v,'C')
            if sd>0 and rem>15 and len(my_pl)<MAX_PLANTS:
                for sp in ps[:15]:
                    if sp in tpos and sp!=(tx,ty): continue
                    d=md(tx,ty,sp[0],sp[1]); tt_=tn(d,spd)
                    if sp in sadj_s: gr,base=2,22
                    elif sp in wadj: gr,base=4,13
                    else: gr,base=12,7
                    v=float(base)/max(1,tt_+gr+4)
                    con(sp,v,'P')
            if sd==0 and ash and fr>0 and rem>20 and len(my_pl)<MAX_PLANTS:
                for sp in ps[:6]:
                    if sp in clm or sp in act_pos: continue
                    dsp=md(tx,ty,sp[0],sp[1])
                    if dsp>2: continue
                    lp=(not has_cc4 and sp in wadj and rem>80 and inv_[1]<n+16)
                    pto=['LEMON','APPLE','BANANA','PLUM'] if lp else ['BANANA','APPLE','LEMON','PLUM']
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
            if t['id']==wt_id and wt_sc>=6: return(-8888,0,0)
            return(-depo(t),-frts(t),md(t['x'],t['y'],sx,sy))

        for t in sorted(trolls,key=sort_key):
            a=proc(t)
            if a: acts.append(a)

        print(';'.join(acts) if acts else 'WAIT')
        sys.stdout.flush()

main()

```
-->
