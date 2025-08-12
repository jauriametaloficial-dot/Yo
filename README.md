# Yo
Cic
<!DOCTYPE html><html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Mini COC‑like (demo sin assets)</title>
  <style>
    :root{
      --bg:#0f1220; --panel:#151933; --accent:#7dd3fc; --accent2:#fca5a5; --ok:#86efac; --warn:#fde68a;
      --grid:#1e2448; --ink:#e5e7eb;
    }
    html,body{height:100%;margin:0;background:linear-gradient(180deg,#0b0e1a,#0f1220);color:var(--ink);font-family:system-ui,Segoe UI,Roboto,Ubuntu,Helvetica,Arial,sans-serif}
    .app{display:grid;grid-template-columns:320px 1fr;gap:10px;height:100%;padding:10px;box-sizing:border-box}
    .left{background:var(--panel);border:1px solid #252a56;border-radius:16px;display:flex;flex-direction:column;overflow:hidden}
    .left header{padding:12px 14px;border-bottom:1px solid #252a56;display:flex;gap:8px;align-items:center;justify-content:space-between}
    .kpi{display:flex;gap:8px;flex-wrap:wrap}
    .pill{display:flex;align-items:center;gap:6px;padding:6px 10px;border-radius:999px;background:#0f1430;border:1px solid #242a57;font-weight:600}
    .pill small{opacity:.8;font-weight:500}
    .section{padding:12px 14px;border-bottom:1px solid #252a56}
    .section h3{margin:0 0 8px 0;font-size:14px;letter-spacing:.3px;color:#c7d2fe}
    .grid-btns{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:8px}
    button{background:#1a1f42;color:var(--ink);border:1px solid #293063;border-radius:12px;padding:10px;font-weight:700;cursor:pointer}
    button:hover{filter:brightness(1.1)}
    button:disabled{opacity:.5;cursor:not-allowed}
    .mode{display:flex;gap:8px}
    .row{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
    .danger{background:#2a1220;border-color:#5d2337}
    .ok{background:#132a1f;border-color:#2f6b4f}
    canvas{width:100%;height:100%;image-rendering:pixelated;background:#0b0f22;border:1px solid #252a56;border-radius:16px}
    .right{position:relative}
    .toast{position:absolute;right:12px;top:12px;background:#101533;border:1px solid #283066;padding:8px 10px;border-radius:10px;font-size:12px;opacity:.95}
    .legend{display:flex;gap:8px;flex-wrap:wrap}
    .badge{display:inline-flex;align-items:center;gap:6px;padding:4px 8px;border-radius:999px;border:1px solid #2c3264;background:#10143a;font-size:12px}
    .dot{width:12px;height:12px;border-radius:3px;display:inline-block}
    .muted{opacity:.8}
    .footer{margin-top:auto;padding:10px 14px;border-top:1px solid #252a56;display:flex;gap:8px;flex-wrap:wrap}
    a.btn{display:inline-block;text-decoration:none}
  </style>
</head>
<body>
  <div class="app">
    <div class="left">
      <header>
        <strong>Mini COC‑like</strong>
        <div class="kpi">
          <div class="pill"><small>Oro</small> <span id="gold">0</span></div>
          <div class="pill"><small>Elixir</small> <span id="elixir">0</span></div>
          <div class="pill"><small>XP</small> <span id="xp">1</span></div>
        </div>
      </header><div class="section">
    <h3>Modo</h3>
    <div class="mode">
      <button id="modeBuild" class="ok">Construir</button>
      <button id="modeAttack">Atacar</button>
    </div>
  </div>

  <div class="section" id="buildPanel">
    <h3>Construcciones</h3>
    <div class="grid-btns" id="buildButtons"></div>
    <div class="legend" style="margin-top:10px">
      <span class="badge"><span class="dot" style="background:#f59e0b"></span> Ayuntamiento</span>
      <span class="badge"><span class="dot" style="background:#22c55e"></span> Mina (oro)</span>
      <span class="badge"><span class="dot" style="background:#06b6d4"></span> Extractor (elixir)</span>
      <span class="badge"><span class="dot" style="background:#ef4444"></span> Cañón</span>
      <span class="badge"><span class="dot" style="background:#a78bfa"></span> Cuartel</span>
    </div>
  </div>

  <div class="section" id="attackPanel" style="display:none">
    <h3>Despliegue</h3>
    <div class="row">
      <button id="spawn1">+1 Duende</button>
      <button id="spawn5">+5</button>
      <button id="endAttack" class="danger">Terminar ataque</button>
    </div>
    <p class="muted" style="margin-top:8px">Consejo: hacé clic/tap en los bordes del mapa para soltar unidades. Van al edificio más cercano.
    Los cañones defienden automáticamente.</p>
  </div>

  <div class="section">
    <h3>Gestión</h3>
    <div class="row">
      <button id="saveBtn">Guardar</button>
      <button id="loadBtn">Cargar</button>
      <button id="resetBtn" class="danger">Reiniciar</button>
    </div>
  </div>

  <div class="footer">
    <small class="muted">Demo educativa sin assets comerciales. Inspirada en mecánicas de base‑builder.</small>
  </div>
</div>

<div class="right">
  <canvas id="game" width="960" height="640"></canvas>
  <div class="toast" id="toast" style="display:none"></div>
</div>

  </div>  <script>
  // --- Parámetros de juego ---
  const GRID_W = 30;   // celdas
  const GRID_H = 20;
  const CELL = 32;     // px por celda

  // Colores por tipo
  const COLORS = {
    TOWN: "#f59e0b", // ayuntamiento
    MINE: "#22c55e",
    PUMP: "#06b6d4",
    CANNON: "#ef4444",
    BARRACKS: "#a78bfa",
    UNIT: "#10b981",
    BULLET: "#eab308",
    HPBAR: "#22c55e",
    HPBAD: "#ef4444"
  };

  // Catálogo de edificios
  const BUILDINGS = {
    TOWN: { name:"Ayuntamiento", w:3, h:3, hpMax:1200, cost:{gold:0, elixir:0}, icon:COLORS.TOWN },
    MINE: { name:"Mina de oro", w:2, h:2, hpMax:300, cost:{gold:50, elixir:0}, prod:"gold", rate:2, icon:COLORS.MINE},
    PUMP: { name:"Extractor de elixir", w:2, h:2, hpMax:300, cost:{gold:0, elixir:50}, prod:"elixir", rate:2, icon:COLORS.PUMP},
    CANNON: { name:"Cañón", w:2, h:2, hpMax:400, cost:{gold:80, elixir:20}, atk:20, range:6, cd:45, icon:COLORS.CANNON},
    BARRACKS: { name:"Cuartel", w:2, h:2, hpMax:350, cost:{gold:60, elixir:30}, icon:COLORS.BARRACKS}
  };

  // Unidades simples (duende recolector/atacante)
  const UNIT_TEMPLATE = { name:"Duende", hpMax:120, speed:0.06, atk:8, atkCd:35, target:"building" };

  // Estado del juego
  const state = {
    mode:"build", // build | attack
    gold:200,
    elixir:200,
    xp:1,
    time:0,
    buildings:[], // {type,x,y,w,h,hp}
    units:[], // {x,y,hp,cd}
    projectiles:[], // {x,y,dx,dy,spd,ttl,dmg}
    placing:null, // {type}
    attacking:false
  };

  // Utilidades
  const rnd = (a,b)=>Math.random()*(b-a)+a;
  const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
  function toast(msg, ms=1600){
    const t=document.getElementById('toast');
    t.textContent=msg; t.style.display='block';
    clearTimeout(toast._id);
    toast._id=setTimeout(()=>t.style.display='none', ms);
  }

  // Canvas
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  // Inicialización: ayuntamiento al centro
  function initNew(){
    state.buildings=[]; state.units=[]; state.projectiles=[]; state.attacking=false; state.mode='build';
    state.gold=200; state.elixir=200; state.xp=1; state.time=0; state.placing=null;
    const cx = Math.floor(GRID_W/2)-2; const cy = Math.floor(GRID_H/2)-2;
    addBuilding('TOWN', cx, cy);
  }

  // Guardar / cargar
  function save(){
    localStorage.setItem('mini-coc-save', JSON.stringify(state, (k,v)=>{
      if(['canvas'].includes(k)) return undefined; return v;
    }));
    toast('Progreso guardado');
  }
  function load(){
    const raw=localStorage.getItem('mini-coc-save');
    if(!raw){ toast('No hay partida guardada'); return; }
    const s=JSON.parse(raw);
    Object.assign(state, s);
    toast('Partida cargada');
  }

  // Construcción
  function canPlace(type, gx, gy){
    const t=BUILDINGS[type];
    if(!t) return false;
    if(gx<0||gy<0||gx+t.w>GRID_W||gy+t.h>GRID_H) return false;
    // Evitar solapamiento
    for(const b of state.buildings){
      if(rectsOverlap(gx,gy,t.w,t.h,b.x,b.y,b.w,b.h)) return false;
    }
    return true;
  }

  function pay(cost){
    if((state.gold>= (cost.gold||0)) && (state.elixir>= (cost.elixir||0))){
      state.gold-= (cost.gold||0); state.elixir-= (cost.elixir||0); return true;
    }
    return false;
  }

  function addBuilding(type,gx,gy){
    const t=BUILDINGS[type];
    const b={ id:crypto.randomUUID(), type, x:gx, y:gy, w:t.w, h:t.h, hp:t.hpMax, hpMax:t.hpMax, cd:0, stored:0 };
    state.buildings.push(b);
    return b;
  }

  // Detección simple
  function rectsOverlap(x,y,w,h,x2,y2,w2,h2){
    return !(x+w<=x2 || x2+w2<=x || y+h<=y2 || y2+h2<=y);
  }

  // Lógica por tick
  function tick(dt){
    state.time+=dt;

    // Producción pasiva
    for(const b of state.buildings){
      const t=BUILDINGS[b.type];
      if(t.prod){
        b.stored = (b.stored||0) + t.rate*dt/1000; // recursos por segundo
        // Auto‑cobro si supera 10
        if(b.stored>=10){
          const take = Math.floor(b.stored/10)*10; b.stored-=take;
          if(t.prod==='gold') state.gold+=take; else state.elixir+=take;
        }
      }
    }

    // Proyectiles (defensas)
    for(const p of state.projectiles){
      p.x += p.dx*p.spd*dt; p.y += p.dy*p.spd*dt; p.ttl -= dt;
    }
    state.projectiles = state.projectiles.filter(p=>p.ttl>0);

    // Cañones disparan
    const targets = state.units.filter(u=>u.hp>0);
    for(const b of state.buildings){
      const t=BUILDINGS[b.type]; if(!t.atk) continue;
      b.cd = Math.max(0, (b.cd||0)-dt);
      if(b.cd>0) continue;
      // Buscar unidad en rango
      let best=null, bestD=1e9;
      for(const u of targets){
        const dx = (u.x)-(b.x+b.w/2); const dy = (u.y)-(b.y+b.h/2);
        const d = Math.hypot(dx,dy);
        if(d < t.range && d<bestD){ best=u; bestD=d; }
      }
      if(best){
        b.cd = t.cd; // cooldown
        const cx=(b.x+b.w/2), cy=(b.y+b.h/2);
        const vx=(best.x-cx)/bestD, vy=(best.y-cy)/bestD;
        state.projectiles.push({x:cx,y:cy,dx:vx,dy:vy,spd:0.02,ttl:2000,dmg:t.atk});
      }
    }

    // Colisiones bala‑unidad
    for(const p of state.projectiles){
      for(const u of state.units){
        if(u.hp<=0) continue;
        const d=Math.hypot(u.x-p.x,u.y-p.y);
        if(d<0.3){ u.hp-=p.dmg; p.ttl=0; if(u.hp<=0) state.xp++; }
      }
    }

    // Unidades se mueven a edificio más cercano
    const aliveBuildings = state.buildings.filter(b=>b.hp>0);
    for(const u of state.units){
      if(u.hp<=0) continue;
      u.cd = Math.max(0,(u.cd||0)-dt);
      let target=null, best=1e9;
      for(const b of aliveBuildings){
        const dx=(b.x+b.w/2)-u.x, dy=(b.y+b.h/2)-u.y; const d=Math.hypot(dx,dy);
        if(d<best){best=d; target=b;}
      }
      if(target){
        // Si está cerca, ataca
        if(best<0.6){
          if(u.cd<=0){ u.cd = UNIT_TEMPLATE.atkCd; target.hp -= UNIT_TEMPLATE.atk; if(target.hp<=0){ state.xp+=3; }
          }
        } else {
          const dx=(target.x+target.w/2)-u.x; const dy=(target.y+target.h/2)-u.y; const d=Math.hypot(dx,dy)||1;
          u.x += (dx/d)*UNIT_TEMPLATE.speed*dt; u.y += (dy/d)*UNIT_TEMPLATE.speed*dt;
        }
      }
    }

    // Limpieza de muertos
    state.units = state.units.filter(u=>u.hp>0);
  }

  // Render
  function draw(){
    // Fondo
    ctx.fillStyle="#0a0e24"; ctx.fillRect(0,0,canvas.width,canvas.height);

    // Grid
    ctx.strokeStyle=getComputedStyle(document.documentElement).getPropertyValue('--grid');
    ctx.lineWidth=1; ctx.beginPath();
    for(let x=0;x<=GRID_W;x++){ ctx.moveTo(x*CELL+.5,0); ctx.lineTo(x*CELL+.5,GRID_H*CELL); }
    for(let y=0;y<=GRID_H;y++){ ctx.moveTo(0,y*CELL+.5); ctx.lineTo(GRID_W*CELL,y*CELL+.5); }
    ctx.stroke();

    // Edificios
    for(const b of state.buildings){
      const t=BUILDINGS[b.type];
      const x=b.x*CELL, y=b.y*CELL, w=b.w*CELL, h=b.h*CELL;
      // cuerpo
      ctx.fillStyle=t.icon; ctx.fillRect(x+2,y+2,w-4,h-4);
      // borde
      ctx.strokeStyle="#0b0b16"; ctx.lineWidth=2; ctx.strokeRect(x+1,y+1,w-2,h-2);
      // barra de vida
      const ratio= Math.max(0,b.hp/b.hpMax);
      ctx.fillStyle="#111428"; ctx.fillRect(x,y-8,w,6);
      ctx.fillStyle= ratio>0.5?COLORS.HPBAR:COLORS.HPBAD; ctx.fillRect(x,y-8,w*ratio,6);
      // producción almacenada
      if(BUILDINGS[b.type].prod){
        ctx.fillStyle="#ffffff88"; ctx.font='12px system-ui';
        ctx.fillText(`${BUILDINGS[b.type].prod==='gold'?'🪙':'💧'} ${Math.floor(b.stored||0)}`, x+4,y+h-6);
      }
    }

    // Proyectiles
    for(const p of state.projectiles){
      ctx.fillStyle=COLORS.BULLET;
      ctx.fillRect(p.x*CELL-2,p.y*CELL-2,4,4);
    }

    // Unidades
    for(const u of state.units){
      const x=u.x*CELL, y=u.y*CELL;
      ctx.fillStyle=COLORS.UNIT; ctx.beginPath(); ctx.arc(x,y,6,0,Math.PI*2); ctx.fill();
      // vida
      const r=Math.max(0,u.hp/UNIT_TEMPLATE.hpMax);
      ctx.fillStyle="#111428"; ctx.fillRect(x-10,y-14,20,4);
      ctx.fillStyle=r>0.5?COLORS.HPBAR:COLORS.HPBAD; ctx.fillRect(x-10,y-14,20*r,4);
    }

    // Fantasma de colocación
    if(state.mode==='build' && state.placing){
      const {gx,gy,ok}=state.placing;
      if(gx!=null){
        const t=BUILDINGS[state.placing.type];
        const x=gx*CELL,y=gy*CELL,w=t.w*CELL,h=t.h*CELL;
        ctx.fillStyle = ok?"#00ff3355":"#ff003355"; ctx.fillRect(x,y,w,h);
      }
    }

    // HUD auxiliar
    ctx.fillStyle="#ffffffaa"; ctx.font='12px system-ui';
    ctx.fillText(`Construcciones: ${state.buildings.length}  |  Unidades: ${state.units.length}` , 10, canvas.height-10);
  }

  // Input
  function getGridFromEvent(ev){
    const rect=canvas.getBoundingClientRect();
    const cx=(ev.touches?ev.touches[0].clientX:ev.clientX)-rect.left;
    const cy=(ev.touches?ev.touches[0].clientY:ev.clientY)-rect.top;
    const gx=Math.floor(cx/rect.width*canvas.width/CELL);
    const gy=Math.floor(cy/rect.height*canvas.height/CELL);
    return {gx,gy};
  }

  canvas.addEventListener('mousemove', e=>{
    if(state.mode!=='build' || !state.placing) return;
    const {gx,gy}=getGridFromEvent(e);
    state.placing.gx=gx; state.placing.gy=gy; state.placing.ok = canPlace(state.placing.type,gx,gy);
  });

  canvas.addEventListener('click', e=>{
    const {gx,gy}=getGridFromEvent(e);
    if(state.mode==='build'){
      if(!state.placing){ return; }
      const type=state.placing.type; const t=BUILDINGS[type];
      if(canPlace(type,gx,gy)){
        if(pay(t.cost)){
          addBuilding(type,gx,gy);
          toast(`${t.name} construida`);
        }else{
          toast('Recursos insuficientes');
        }
      } else {
        toast('No se puede colocar aquí');
      }
    } else if(state.mode==='attack'){
      // Despliegue solo en bordes
      if(gx===0||gy===0||gx===GRID_W-1||gy===GRID_H-1){
        spawnUnitsAt(gx+0.5,gy+0.5,1);
      } else {
        toast('Desplegá desde los bordes del mapa');
      }
    }
  }, {passive:true});

  // Touch soporte básico
  canvas.addEventListener('touchstart', e=>{ canvas.dispatchEvent(new MouseEvent('click', {clientX:e.touches[0].clientX, clientY:e.touches[0].clientY})); }, {passive:true});
  canvas.addEventListener('touchmove', e=>{ canvas.dispatchEvent(new MouseEvent('mousemove', {clientX:e.touches[0].clientX, clientY:e.touches[0].clientY})); }, {passive:true});

  // UI botones
  const buildButtons = document.getElementById('buildButtons');
  const buildList = ['MINE','PUMP','CANNON','BARRACKS'];
  function refreshBuildButtons(){
    buildButtons.innerHTML='';
    for(const key of buildList){
      const t=BUILDINGS[key];
      const btn=document.createElement('button');
      btn.innerHTML=`${t.name}<br><small>🪙${t.cost.gold||0} · 💧${t.cost.elixir||0}</small>`;
      btn.onclick=()=>{ state.placing={type:key}; toast(`Colocá: ${t.name}`); };
      buildButtons.appendChild(btn);
    }
  }

  function setMode(m){
    state.mode=m;
    document.getElementById('modeBuild').classList.toggle('ok', m==='build');
    document.getElementById('attackPanel').style.display = m==='attack'? 'block':'none';
    document.getElementById('buildPanel').style.display = m==='build'? 'block':'none';
    if(m==='build') state.attacking=false; else state.attacking=true;
  }

  document.getElementById('modeBuild').onclick=()=>setMode('build');
  document.getElementById('modeAttack').onclick=()=>setMode('attack');

  document.getElementById('saveBtn').onclick=save;
  document.getElementById('loadBtn').onclick=load;
  document.getElementById('resetBtn').onclick=()=>{ if(confirm('¿Reiniciar partida?')) initNew(); };

  document.getElementById('spawn1').onclick=()=>spawnEdge(1);
  document.getElementById('spawn5').onclick=()=>spawnEdge(5);
  document.getElementById('endAttack').onclick=()=>{ setMode('build'); toast('Ataque finalizado'); };

  function spawnEdge(n){
    // Spawnea en una posición aleatoria del borde
    for(let i=0;i<n;i++){
      const side=Math.floor(Math.random()*4);
      const gx = side===0?0: side===1?GRID_W-1: Math.floor(Math.random()*GRID_W);
      const gy = side===2?0: side===3?GRID_H-1: Math.floor(Math.random()*GRID_H);
      spawnUnitsAt(gx+0.5,gy+0.5,1);
    }
  }

  function spawnUnitsAt(x,y,count){
    for(let i=0;i<count;i++){
      state.units.push({ x:x+rnd(-0.2,0.2), y:y+rnd(-0.2,0.2), hp:UNIT_TEMPLATE.hpMax, cd:0 });
    }
  }

  // Loop
  let last=performance.now();
  function loop(now){
    const dt=clamp(now-last, 0, 50); last=now;
    tick(dt);
    draw();
    // KPIs
    document.getElementById('gold').textContent=Math.floor(state.gold);
    document.getElementById('elixir').textContent=Math.floor(state.elixir);
    document.getElementById('xp').textContent=Math.floor(state.xp);
    requestAnimationFrame(loop);
  }

  // Boot
  refreshBuildButtons();
  if(localStorage.getItem('mini-coc-save')){ load(); }
  else { initNew(); }
  requestAnimationFrame(loop);

  // Accesos rápidos
  window.addEventListener('keydown', (e)=>{
    if(e.key==='1'){ state.placing={type:'MINE'}; toast('Colocá: Mina'); }
    if(e.key==='2'){ state.placing={type:'PUMP'}; toast('Colocá: Extractor'); }
    if(e.key==='3'){ state.placing={type:'CANNON'}; toast('Colocá: Cañón'); }
    if(e.key==='4'){ state.placing={type:'BARRACKS'}; toast('Colocá: Cuartel'); }
    if(e.key==='b'){ setMode('build'); }
    if(e.key==='a'){ setMode('attack'); }
    if(e.key==='s'){ save(); }
  });
  </script></body>
</html>
