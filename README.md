
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Gestión de Gimnasio</title>
  <style>
    body { background:#121212; color:#bd9a00; font-family:Arial,sans-serif; margin:0; }
    header { background:#1f1f1f; padding:1rem; text-align:center; font-size:1.5rem; }
    main { display:flex; }
    nav { width:200px; background:#1a1a1a; min-height:100vh; padding:1rem; }
    nav button { display:block; width:100%; margin:.5rem 0; padding:.5rem; background:#333; border:none; color:#eee; cursor:pointer; border-radius:5px; }
    section { flex:1; padding:1rem; display:none; }
    section.active { display:block; }
    table { width:100%; border-collapse:collapse; margin-top:1rem; }
    th,td { border:1px solid #444; padding:.5rem; text-align:left; }
    input,select { padding:.3rem; margin:.2rem 0; border-radius:4px; border:1px solid #444; background:#222; color:#eee; }
    .history-filters { margin:1rem 0; }
  </style>
</head>
<body>
  <header>🏋️ Sistema de Gimnasio</header>
  <main>
    <nav>
      <button onclick="show('members')">Miembros</button>
      <button onclick="show('attendance')">Registro de Asistencia / Pagos del día</button>
      <button onclick="show('history')">Historial</button>
    </nav>
    <section id="members" class="active">
      <h2>Miembros</h2>
      <form onsubmit="addMember(event)">
        <input id="memberName" placeholder="Nombre" required>
        <input id="memberDays" type="number" placeholder="Días" required>
        <input id="memberAmount" type="number" placeholder="Pago $" required>
        <button>Registrar</button>
      </form>
      <div class="history-filters">
        <label>Filtrar por fecha inicio:</label>
        <input type="date" id="filterMemberDate" onchange="renderMembers(this.value)">
      </div>
      <table>
        <thead><tr><th>Nombre</th><th>Inicio</th><th>Fin</th><th>Pago</th><th>Acciones</th></tr></thead>
        <tbody id="membersTbody"></tbody>
      </table>
    </section>
    <section id="attendance">
      <h2>Registro de Visitas / Zumba</h2>
      <form onsubmit="addVisit(event)">
        <input id="visitName" placeholder="Nombre visitante" required>
        <input id="visitAmount" type="number" placeholder="Pago $" required>
        <select id="visitType">
          <option value="Visita">Visita</option>
          <option value="Zumba">Zumba</option>
        </select>
        <button>Registrar</button>
      </form>
      <table>
        <thead><tr><th>Nombre</th><th>Tipo</th><th>Pago</th><th>Fecha</th></tr></thead>
        <tbody id="visitsTbody"></tbody>
      </table>
    </section>
    <section id="history">
      <h2>Historial</h2>
      <div class="history-filters">
        <label>Tipo:</label>
        <select id="filterType" onchange="renderHistory()">
          <option value="all">Todos</option>
          <option value="Membresía">Membresías</option>
          <option value="Visita">Visitas</option>
          <option value="Zumba">Zumba</option>
          <option value="Asistencia">Asistencias</option>
        </select>
        <label>Fecha:</label>
        <input type="date" id="filterDate" onchange="renderHistory(this.value)">
      </div>
      <table>
        <thead><tr><th>Fecha</th><th>Tipo</th><th>Persona</th><th>Monto</th><th>Detalle</th></tr></thead>
        <tbody id="historyTbody"></tbody>
      </table>
      <p id="totalHistory">Ganancia total: $0</p>
    </section>
  </main>

  <!-- ==== MODAL DE ASISTENCIA ==== -->
  <div id="attendanceModal" style="
    position:fixed; top:0; left:0; width:100%; height:100%;
    background:rgba(0,0,0,0.7); display:none; align-items:center; justify-content:center;
  ">
    <div style="background:#1e1e1e; padding:2rem; border-radius:10px; text-align:center; max-width:400px; box-shadow:0 0 15px #000;">
      <h2 id="modalTitle">👋 Bienvenido</h2>
      <p id="modalMessage" style="margin:1rem 0; font-size:1.1rem; color:#ccc;"></p>
      <button onclick="closeModal()" style="padding:0.5rem 1rem; border:none; border-radius:5px; background:#444; color:#eee; cursor:pointer;">Cerrar</button>
    </div>
  </div>

<script>
const el=id=>document.getElementById(id);
let members=[],visits=[],history=[];

// ==== NAV ====
function show(id){
  document.querySelectorAll("section").forEach(s=>s.classList.remove("active"));
  el(id).classList.add("active");
}

// ==== UTIL ====
function fmt(d){ return d.toLocaleString(); }
function parseDate(s){ return s?new Date(s+"T00:00:00") : null; }

// ==== HISTORIAL ====
function logHistory(type,person,amount,detail){
  history.push({date:+new Date(),type,person,amount,detail});
  renderHistory();
}
function renderHistory(dateFilter=null){
  let list=history;
  const type=el("filterType").value;
  if(type!=="all") list=list.filter(h=>h.type===type);
  if(dateFilter){
    const d=parseDate(dateFilter);
    if(d){
      const start=new Date(d); start.setHours(0,0,0,0);
      const end=new Date(d); end.setHours(23,59,59,999);
      list=list.filter(h=>{
        const hd=new Date(h.date);
        return hd>=start && hd<=end;
      });
    }
  }
  el("historyTbody").innerHTML=list.map(h=>`
    <tr>
      <td>${fmt(new Date(h.date))}</td>
      <td>${h.type}</td>
      <td>${h.person}</td>
      <td>$${h.amount.toFixed(2)}</td>
      <td>${h.detail||''}</td>
    </tr>`).join('');
  const total=list.reduce((s,h)=>s+h.amount,0);
  el("totalHistory").textContent=`Ganancia total: $${total.toFixed(2)}`;
}

// ==== MIEMBROS ====
function addMember(e){
  e.preventDefault();
  const name=el("memberName").value;
  const days=+el("memberDays").value;
  const amount=+el("memberAmount").value;
  const start=new Date(); start.setHours(0,0,0,0);
  const end=new Date(start); end.setDate(end.getDate()+days);
  const m={id:Date.now(),name,days,amount,start:+start,end:+end};
  members.push(m);
  logHistory("Membresía",name,amount,`Membresía de ${days} días`);
  renderMembers();
  e.target.reset();
}
function renderMembers(dateFilter=null){
  let list=members;
  if(dateFilter){
    const d=parseDate(dateFilter);
    if(d){
      const start=new Date(d); start.setHours(0,0,0,0);
      const end=new Date(d); end.setHours(23,59,59,999);
      list=list.filter(m=>{
        const md=new Date(m.start);
        return md>=start && md<=end;
      });
    }
  }
  el("membersTbody").innerHTML=list.map(m=>`
    <tr>
      <td>${m.name}</td>
      <td>${fmt(new Date(m.start))}</td>
      <td>${fmt(new Date(m.end))}</td>
      <td>$${m.amount.toFixed(2)}</td>
      <td>
        <button onclick="renewMember('${m.id}')">Renovar</button>
        <button onclick="markAttendance('${m.id}')">Asistencia</button>
      </td>
    </tr>`).join('');
}
function renewMember(id){
  const m=members.find(x=>x.id==id);
  if(!m) return;
  const extra=prompt("¿Cuántos días desea renovar?");
  const pay=prompt("Pago recibido por renovación $");
  if(extra&&pay){
    const addDays=+extra, amount=+pay;
    m.end=new Date(m.end).setDate(new Date(m.end).getDate()+addDays);
    logHistory("Membresía",m.name,amount,`Renovación de ${addDays} días`);
    renderMembers();
  }
}

// ==== VISITAS / ZUMBA ====
function addVisit(e){
  e.preventDefault();
  const name=el("visitName").value;
  const amount=+el("visitAmount").value;
  const type=el("visitType").value;
  visits.push({id:Date.now(),name,amount,type,date:+new Date()});
  logHistory(type,name,amount,"Pago por "+type);
  renderVisits();
  e.target.reset();
}
function renderVisits(){
  el("visitsTbody").innerHTML=visits.map(v=>`
    <tr>
      <td>${v.name}</td>
      <td>${v.type}</td>
      <td>$${v.amount.toFixed(2)}</td>
      <td>${fmt(new Date(v.date))}</td>
    </tr>`).join('');
}

// ==== ASISTENCIA ====
window.markAttendance=id=>{
  const m=members.find(x=>x.id==id);
  if(!m) return;
  const today=new Date(); today.setHours(0,0,0,0);
  const end=new Date(m.end);
  const diff=Math.ceil((end-today)/(1000*60*60*24));
  logHistory("Asistencia",m.name,0,"Registro de asistencia");
  el("modalTitle").textContent=`👋 Bienvenido ${m.name}`;
  el("modalMessage").textContent=`Te quedan ${diff} días de membresía.`;
  el("attendanceModal").style.display="flex";
};
function closeModal(){ el("attendanceModal").style.display="none"; }

// ==== INIT ====
renderMembers();
renderVisits();
renderHistory();
</script>
</body>
</html>
