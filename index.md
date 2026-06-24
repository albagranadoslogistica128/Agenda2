
<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: var(--font-sans); }
.app { padding: 1.5rem 1rem; max-width: 600px; margin: 0 auto; }
.header { margin-bottom: 1.5rem; }
.fecha-hoy { font-size: 13px; color: var(--color-text-secondary); margin-bottom: 4px; }
.titulo { font-size: 22px; font-weight: 500; color: var(--color-text-primary); }
.add-section { background: var(--color-background-primary); border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-lg); padding: 1rem 1.25rem; margin-bottom: 1.5rem; }
.add-row { display: flex; gap: 8px; margin-bottom: 8px; }
.add-row input[type="text"] { flex: 1; }
.add-row input[type="date"] { width: 160px; }
.add-row input[type="time"] { width: 110px; }
.btn-add { background: var(--color-background-primary); border: 0.5px solid var(--color-border-secondary); border-radius: var(--border-radius-md); padding: 0 16px; height: 36px; cursor: pointer; font-size: 14px; font-weight: 500; color: var(--color-text-primary); display: flex; align-items: center; gap: 6px; white-space: nowrap; }
.btn-add:hover { background: var(--color-background-secondary); }
.filters { display: flex; gap: 8px; margin-bottom: 1rem; flex-wrap: wrap; }
.filter-btn { background: transparent; border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-md); padding: 4px 12px; font-size: 13px; cursor: pointer; color: var(--color-text-secondary); }
.filter-btn.active { background: var(--color-background-secondary); border-color: var(--color-border-secondary); color: var(--color-text-primary); font-weight: 500; }
.grupo { margin-bottom: 1.25rem; }
.grupo-titulo { font-size: 12px; font-weight: 500; color: var(--color-text-secondary); text-transform: uppercase; letter-spacing: 0.06em; padding: 0 4px; margin-bottom: 8px; }
.tarea { display: flex; align-items: flex-start; gap: 10px; background: var(--color-background-primary); border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-md); padding: 10px 12px; margin-bottom: 6px; cursor: default; }
.tarea.completada .tarea-texto { text-decoration: line-through; color: var(--color-text-tertiary); }
.tarea-check { width: 18px; height: 18px; border-radius: 50%; border: 1.5px solid var(--color-border-secondary); display: flex; align-items: center; justify-content: center; cursor: pointer; flex-shrink: 0; margin-top: 1px; transition: background 0.15s; }
.tarea.completada .tarea-check { background: var(--color-background-success); border-color: var(--color-border-success); }
.tarea-body { flex: 1; min-width: 0; }
.tarea-texto { font-size: 14px; color: var(--color-text-primary); line-height: 1.4; }
.tarea-meta { font-size: 12px; color: var(--color-text-secondary); margin-top: 2px; display: flex; gap: 8px; }
.btn-del { background: transparent; border: none; color: var(--color-text-tertiary); cursor: pointer; padding: 0 4px; font-size: 16px; line-height: 1; flex-shrink: 0; }
.btn-del:hover { color: var(--color-text-danger); }
.empty { text-align: center; color: var(--color-text-tertiary); font-size: 14px; padding: 2rem 0; }
</style>

<div class="app">
  <h2 class="sr-only">Agenda de tareas personal</h2>
  <div class="header">
    <div class="fecha-hoy" id="fecha-hoy"></div>
    <div class="titulo"><i class="ti ti-calendar" aria-hidden="true" style="font-size:20px;vertical-align:-2px;margin-right:8px;"></i>Mi Agenda</div>
  </div>

  <div class="add-section">
    <div class="add-row">
      <input type="text" id="nueva-tarea" placeholder="Nueva tarea..." maxlength="120" />
    </div>
    <div class="add-row">
      <input type="date" id="nueva-fecha" />
      <input type="time" id="nueva-hora" />
      <button class="btn-add" onclick="agregarTarea()"><i class="ti ti-plus" aria-hidden="true"></i>Añadir</button>
    </div>
  </div>

  <div class="filters">
    <button class="filter-btn active" onclick="setFiltro('todas', this)">Todas</button>
    <button class="filter-btn" onclick="setFiltro('pendientes', this)">Pendientes</button>
    <button class="filter-btn" onclick="setFiltro('completadas', this)">Completadas</button>
  </div>

  <div id="lista-tareas"></div>
</div>

<script>
const MESES = ['enero','febrero','marzo','abril','mayo','junio','julio','agosto','septiembre','octubre','noviembre','diciembre'];
const DIAS = ['domingo','lunes','martes','miércoles','jueves','viernes','sábado'];

let tareas = JSON.parse(localStorage.getItem('agenda_tareas') || '[]');
let filtro = 'todas';

function guardar() {
  try { localStorage.setItem('agenda_tareas', JSON.stringify(tareas)); } catch(e) {}
}

function fechaLabel(fechaStr) {
  if (!fechaStr) return '';
  const hoy = new Date(); hoy.setHours(0,0,0,0);
  const manana = new Date(hoy); manana.setDate(hoy.getDate()+1);
  const [y,m,d] = fechaStr.split('-').map(Number);
  const f = new Date(y,m-1,d);
  if (f.getTime() === hoy.getTime()) return 'Hoy';
  if (f.getTime() === manana.getTime()) return 'Mañana';
  return `${DIAS[f.getDay()]} ${d} ${MESES[m-1]}`;
}

function setFiltro(f, btn) {
  filtro = f;
  document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  render();
}

function toggleTarea(id) {
  const t = tareas.find(t => t.id === id);
  if (t) { t.completada = !t.completada; guardar(); render(); }
}

function eliminarTarea(id) {
  tareas = tareas.filter(t => t.id !== id);
  guardar(); render();
}

function agregarTarea() {
  const texto = document.getElementById('nueva-tarea').value.trim();
  if (!texto) return;
  const fecha = document.getElementById('nueva-fecha').value;
  const hora = document.getElementById('nueva-hora').value;
  tareas.push({ id: Date.now(), texto, fecha, hora, completada: false });
  guardar();
  document.getElementById('nueva-tarea').value = '';
  render();
}

document.getElementById('nueva-tarea').addEventListener('keydown', e => {
  if (e.key === 'Enter') agregarTarea();
});

function render() {
  const lista = document.getElementById('lista-tareas');
  let visibles = tareas.filter(t =>
    filtro === 'todas' ? true : filtro === 'pendientes' ? !t.completada : t.completada
  );

  if (!visibles.length) {
    lista.innerHTML = '<div class="empty"><i class="ti ti-clipboard" aria-hidden="true" style="font-size:32px;display:block;margin:0 auto 8px;"></i>No hay tareas aquí</div>';
    return;
  }

  const conFecha = visibles.filter(t => t.fecha).sort((a,b) => a.fecha.localeCompare(b.fecha) || (a.hora||'').localeCompare(b.hora||''));
  const sinFecha = visibles.filter(t => !t.fecha);

  let html = '';

  if (conFecha.length) {
    const grupos = {};
    conFecha.forEach(t => { (grupos[t.fecha] = grupos[t.fecha]||[]).push(t); });
    Object.entries(grupos).forEach(([fecha, items]) => {
      html += `<div class="grupo"><div class="grupo-titulo">${fechaLabel(fecha)}</div>`;
      items.forEach(t => { html += tarjetaHTML(t); });
      html += '</div>';
    });
  }

  if (sinFecha.length) {
    html += `<div class="grupo"><div class="grupo-titulo">Sin fecha</div>`;
    sinFecha.forEach(t => { html += tarjetaHTML(t); });
    html += '</div>';
  }

  lista.innerHTML = html;
}

function tarjetaHTML(t) {
  const check = t.completada ? `<i class="ti ti-check" style="font-size:11px;color:var(--color-text-success);" aria-hidden="true"></i>` : '';
  const meta = [t.fecha ? fechaLabel(t.fecha) : '', t.hora || ''].filter(Boolean).join(' · ');
  return `<div class="tarea ${t.completada?'completada':''}">
    <div class="tarea-check" onclick="toggleTarea(${t.id})" role="button" aria-label="${t.completada?'Marcar como pendiente':'Marcar como completada'}">${check}</div>
    <div class="tarea-body">
      <div class="tarea-texto">${t.texto}</div>
      ${meta ? `<div class="tarea-meta"><i class="ti ti-clock" aria-hidden="true" style="font-size:12px;vertical-align:-1px;"></i>${meta}</div>` : ''}
    </div>
    <button class="btn-del" onclick="eliminarTarea(${t.id})" aria-label="Eliminar tarea"><i class="ti ti-trash" aria-hidden="true"></i></button>
  </div>`;
}

const ahora = new Date();
document.getElementById('fecha-hoy').textContent = `${DIAS[ahora.getDay()]} ${ahora.getDate()} de ${MESES[ahora.getMonth()]} de ${ahora.getFullYear()}`;
const hoyStr = ahora.toISOString().split('T')[0];
document.getElementById('nueva-fecha').value = hoyStr;

render();
</script>
