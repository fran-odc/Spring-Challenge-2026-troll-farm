Code 136th (out of 909)

<!-- 
```
import sys

def main():
    read=sys.stdin.readline
    W,H=map(int,read().split())
    grid,shack,iron,water=[],None,[],set()
    for y in range(H):
        row=read().rstrip(); grid.append(row)
        for x,c in enumerate(row):
            if c=='0': shack=(x,y)
            elif c=='+': iron.append((x,y))
            elif c=='~': water.add((x,y))
    sx,sy=shack
    sadj=[(sx+dx,sy+dy) for dx,dy in[(1,0),(-1,0),(0,1),(0,-1)]
          if 0<=sx+dx<W and 0<=sy+dy<H and grid[sy+dy][sx+dx]=='.']
    wadj=set()
    for wx,wy in water:
        for dx,dy in[(0,1),(0,-1),(1,0),(-1,0)]:
            nx,ny=wx+dx,wy+dy
            if 0<=nx<W and 0<=ny<H and grid[ny][nx]=='.': wadj.add((nx,ny))
    md=lambda a,b,c,d:abs(a-c)+abs(b-d)
    ia=lambda a,b,c,d:md(a,b,c,d)==1
    tn=lambda d,s:(d+s-1)//s if d>0 else 0
    def ct(inv,n,ms,cc,hp,cp):
        return inv[0]>=n+ms*ms and inv[1]>=n+cc*cc and inv[2]>=n+hp*hp and inv[4]>=n+cp*cp
    nonseed=lambda t:t['cplum']+t['clem']+t['capl']+t['cirn']+t['cwod']
    cy=lambda t:t['cplum']+t['clem']+t['capl']+t['cban']+t['cirn']+t['cwod']
    fc=lambda t:t['cap']-cy(t)
    turn=0
    while True:
        turn+=1
        inv=list(map(int,read().split())); read()
        tc=int(read()); trees,tat=[],{}
        for _ in range(tc):
            p=read().split()
            t={'T':p[0],'x':int(p[1]),'y':int(p[2]),'sz':int(p[3]),'health':int(p[4]),'fruits':int(p[5]),'cd':int(p[6])}
            trees.append(t); tat[(t['x'],t['y'])]=t
        nc=int(read()); trolls=[]
        for _ in range(nc):
            v=read().split()
            if int(v[1])==0:
                trolls.append({'id':int(v[0]),'x':int(v[2]),'y':int(v[3]),'spd':int(v[4]),'cap':int(v[5]),'hp':int(v[6]),'cp':int(v[7]),'cplum':int(v[8]),'clem':int(v[9]),'capl':int(v[10]),'cban':int(v[11]),'cirn':int(v[12]),'cwod':int(v[13])})
        n=len(trolls); rem=300-turn; inv_=inv[:]
        tp={(t['x'],t['y']) for t in trees}; tpos={(t['x'],t['y']) for t in trolls}; acts=[]
        if n<6 and rem>=20:
            bs=([(2,3,1,2),(1,3,1,2),(2,2,1,2),(1,2,1,2),(2,3,1,1),(1,3,1,1),(2,2,1,1),(1,2,1,1),(2,2,0,2),(1,2,0,2),(2,2,1,0),(1,2,1,0),(1,2,0,1),(1,2,0,0),(1,1,1,0),(1,1,0,1),(1,1,0,0)]
                if iron else [(2,3,2,0),(1,3,2,0),(2,2,2,0),(1,2,2,0),(2,2,1,0),(1,2,1,0),(1,2,0,0),(1,1,1,0),(1,1,0,0)])
            for ms,cc,hp,cp in bs:
                if ct(inv_,n,ms,cc,hp,cp):
                    acts.append(f"TRAIN {ms} {cc} {hp} {cp}")
                    inv_[0]-=n+ms*ms; inv_[1]-=n+cc*cc; inv_[2]-=n+hp*hp; inv_[4]-=n+cp*cp; break
        ps,pset=[],set()
        for p in sorted(wadj,key=lambda q:md(q[0],q[1],sx,sy)):
            if p not in tp: ps.append(p); pset.add(p)
        for dy2 in range(-7,8):
            for dx2 in range(-7,8):
                nx2,ny2=sx+dx2,sy+dy2
                if(0<=nx2<W and 0<=ny2<H and grid[ny2][nx2]=='.' and (nx2,ny2) not in tp and (nx2,ny2) not in pset):
                    ps.append((nx2,ny2)); pset.add((nx2,ny2))
        ineed=max(0,(n+1)*4-inv_[4]) if n<5 and iron else 0
        clm=set(); nxt=set()
        def dst(tx,ty):
            best,bd=None,999
            for p in sadj:
                if(p not in tpos or (tx,ty)==p) and p not in nxt and p not in clm:
                    d=md(tx,ty,p[0],p[1])
                    if d<bd: bd=d; best=p
            return best or(sadj[0] if sadj else (sx,sy))
        def proc(t):
            tx,ty=t['x'],t['y']; tid=t['id']; spd=max(1,t['spd']); cp=t['cp']; hp=t['hp']
            tot=cy(t); fr=fc(t); cban=t['cban']; nsv=nonseed(t)
            ash=ia(tx,ty,sx,sy); dsh=md(tx,ty,sx,sy); t2sh=tn(dsh,spd)
            def dep():
                if ash: return f"DROP {tid}"
                d=dst(tx,ty); nxt.add(d); return f"MOVE {tid} {d[0]} {d[1]}"
            if nsv>0 and rem<=t2sh+2: return dep()
            if nsv>0 and fr==0: return dep()
            if cban>0 and nsv==0 and rem<=3: return dep()
            cur=tat.get((tx,ty))
            if ash:
                if nsv>0:
                    if cban>0 and (tx,ty) in wadj and (tx,ty) not in tp and (tx,ty) not in clm:
                        tp.add((tx,ty)); clm.add((tx,ty)); return f"PLANT {tid} BANANA"
                    return f"DROP {tid}"
                elif cban>0:
                    # ONLY bananas: don't drop! Go plant.
                    if (tx,ty) in wadj and (tx,ty) not in tp and (tx,ty) not in clm:
                        tp.add((tx,ty)); clm.add((tx,ty)); return f"PLANT {tid} BANANA"
                    # Fall through to target scoring
                else:
                    if inv_[3]>=2 and fr>0 and ps and rem>25:
                        inv_[3]-=1; return f"PICK {tid} BANANA"
            if cur and (tx,ty) not in clm:
                cch=cp>0 and fr>0 and(cur['T']=='BANANA' or cur['sz']>=2)
                chr_=cur['fruits']>0 and hp>0 and fr>0
                if cch and chr_:
                    ct2=(cur['health']+cp-1)//cp; w=min(cur['sz'],fr)
                    cv=4.0*w/max(1,ct2+t2sh+1)
                    ft2=min(cur['fruits'],fr); ht2=(ft2+hp-1)//hp
                    hv=ft2/max(1,ht2+t2sh+1)
                    clm.add((tx,ty)); return f"{'CHOP' if cv>hv else 'HARVEST'} {tid}"
                elif cch: clm.add((tx,ty)); return f"CHOP {tid}"
                elif chr_: clm.add((tx,ty)); return f"HARVEST {tid}"
            if cban>0 and cur is None and (tx,ty) in wadj and (tx,ty) not in clm:
                tp.add((tx,ty)); clm.add((tx,ty)); return f"PLANT {tid} BANANA"
            if cp>0 and fr>0 and ineed>0:
                for ix,iy in iron:
                    if ia(tx,ty,ix,iy): return f"MINE {tid}"
            bv,bp,bt=-1.0,None,None
            def consider(pos,v,tp_):
                nonlocal bv,bp,bt
                if pos in tpos and pos!=(tx,ty): return
                if pos in clm: return
                if v>bv: bv=v; bp=pos; bt=tp_
            bm=2.5 if inv_[3]<2 else(1.5 if inv_[3]<5 else 1.0)
            if hp>0 and fr>0:
                for t2 in trees:
                    if t2['fruits']<=0: continue
                    pos=(t2['x'],t2['y']); d=md(tx,ty,t2['x'],t2['y'])
                    tt=tn(d,spd); tb=tn(md(t2['x'],t2['y'],sx,sy),spd)
                    ft2=min(t2['fruits'],fr); ht2=(ft2+hp-1)//hp
                    v=ft2/max(1,tt+ht2+tb+1)
                    if t2['T']=='BANANA': v*=bm
                    consider(pos,v,'H')
            if cp>0 and fr>0:
                for t2 in trees:
                    if t2['T']!='BANANA' and t2['sz']<2: continue
                    pos=(t2['x'],t2['y']); d=md(tx,ty,t2['x'],t2['y'])
                    tt=tn(d,spd); tb=tn(md(t2['x'],t2['y'],sx,sy),spd)
                    ct2=(t2['health']+cp-1)//cp; w=min(t2['sz'],fr)
                    if w<=0: continue
                    v=4.0*w/max(1,tt+ct2+tb+1)
                    consider(pos,v,'C')
            if cban>0 and ps:
                for spt in ps[:5]:
                    if spt in clm or(spt in tpos and spt!=(tx,ty)): continue
                    d=md(tx,ty,spt[0],spt[1]); tt=tn(d,spd)
                    gr=12 if spt in wadj else 18
                    v=15.0/max(1,tt+gr+5)
                    consider(spt,v,'P'); break
            if cban==0 and inv_[3]>=2 and ps and rem>30:
                for bsa in sorted(sadj,key=lambda q:md(tx,ty,q[0],q[1])):
                    if bsa in tpos and bsa!=(tx,ty): continue
                    if bsa in nxt or bsa in clm: continue
                    for spt in ps[:3]:
                        if spt in clm: continue
                        dsa=md(tx,ty,bsa[0],bsa[1]); dsp=md(bsa[0],bsa[1],spt[0],spt[1])
                        tt=tn(dsa,spd)+1+tn(dsp,spd); gr=12 if spt in wadj else 18
                        v=13.0/max(1,tt+gr+5)
                        if v>bv: bv=v; bp=bsa; bt='gpk'
                    break
            if cp>0 and fr>0 and iron and ineed>0:
                for ix,iy in iron:
                    for adx,ady in[(0,1),(0,-1),(1,0),(-1,0)]:
                        nx3,ny3=ix+adx,iy+ady
                        if 0<=nx3<W and 0<=ny3<H and grid[ny3][nx3]=='.':
                            d=md(tx,ty,nx3,ny3); tt=tn(d,spd)
                            v=min(cp,ineed)*0.3/max(1,tt+1)
                            consider((nx3,ny3),v,'M')
            if not bp:
                if nsv>0: return dep()
                return None
            clm.add(bp); bx,by=bp
            if (tx,ty)==bp:
                c2=tat.get(bp)
                if bt=='C' and c2 and cp>0: return f"CHOP {tid}"
                if bt=='H' and c2 and c2['fruits']>0: return f"HARVEST {tid}"
                if bt=='P' and cban>0 and bp not in tp: tp.add(bp); return f"PLANT {tid} BANANA"
                if bt=='gpk' and ash and inv_[3]>0: inv_[3]-=1; return f"PICK {tid} BANANA"
                if bt=='M':
                    for ix,iy in iron:
                        if ia(tx,ty,ix,iy): return f"MINE {tid}"
                return None
            else:
                if bt=='M':
                    for ix,iy in iron:
                        if ia(tx,ty,ix,iy): return f"MINE {tid}"
                return f"MOVE {tid} {bx} {by}"
        for t in sorted(trolls,key=lambda t2:(-nonseed(t2),-cy(t2),md(t2['x'],t2['y'],sx,sy))):
            a=proc(t)
            if a: acts.append(a)
        print(';'.join(acts) if acts else 'WAIT')
        sys.stdout.flush()

main()

```
-->
