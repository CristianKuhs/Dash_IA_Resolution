<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <meta name="description" content="Dashboard avançado para gerenciamento e registro de vendas com filtros e gráficos analíticos." />
  <meta http-equiv="Content-Security-Policy" content="default-src 'self' https://cdn.jsdelivr.net; script-src 'self' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com;">
  <title>Dashboard de Vendas — Refatorado</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    :root{
      --bg-dark:#0f172a; --bg-dark-2:#1e293b; --bg-light:#f1f5f9; --bg-light-2:#ffffff;
      --text-dark:#e2e8f0; --text-dark-sec:#93c5fd; --text-light:#1e293b; --text-light-sec:#2563eb;
      --primary:#3b82f6; --primary-dark:#2563eb; --danger:#ef4444; --success:#10b981;
      --border-dark:#334155; --border-light:#cbd5e1;
      --radius-sm:8px; --radius-md:10px; --radius-lg:16px;
      --spacing-sm:8px; --spacing-md:12px; --spacing-lg:20px;
      --shadow-sm:0 4px 10px rgba(0,0,0,0.4);
      --transition-base:0.3s ease;
      --min-touch:44px;
    }
    *{box-sizing:border-box;margin:0;padding:0}
    html,body{height:100%}
    body{
      font-family:Inter,system-ui,Segoe UI,Roboto,Helvetica,Arial,"Noto Sans",sans-serif;
      background:linear-gradient(135deg,var(--bg-dark),var(--bg-dark-2));
      color:var(--text-dark);
      padding:24px;
      transition:background var(--transition-base),color var(--transition-base);
    }
    body.light{background:linear-gradient(135deg,var(--bg-light),var(--bg-light-2)); color:var(--text-light)}
    header{display:flex;justify-content:space-between;align-items:center;padding:16px;border-radius:12px;background:rgba(30,41,59,0.8);backdrop-filter:blur(8px);box-shadow:var(--shadow-sm);margin-bottom:20px}
    header h1{font-size:1.5rem}
    .actions{display:flex;gap:8px}
    button{background:linear-gradient(135deg,var(--primary),var(--primary-dark));border:none;color:#fff;padding:10px 14px;border-radius:10px;cursor:pointer;min-height:var(--min-touch);min-width:var(--min-touch)}
    button:focus{outline:3px solid rgba(59,130,246,0.25)}
    .container{display:grid;grid-template-columns:1fr 420px;gap:20px;align-items:start}
    @media(max-width:980px){.container{grid-template-columns:1fr}}
    .card{background:rgba(30,41,59,0.85);padding:16px;border-radius:12px;box-shadow:var(--shadow-sm)}
    body.light .card{background:rgba(255,255,255,0.95);color:var(--text-light)}
    .summary{display:flex;gap:12px;justify-content:space-between;margin-bottom:12px}
    .summary__item{flex:1;text-align:center;padding:12px;border-radius:10px;background:rgba(255,255,255,0.02)}
    .form-row{display:flex;gap:8px;margin-bottom:8px;align-items:center}
    input,select{padding:10px;border-radius:8px;border:1px solid var(--border-dark);background:rgba(15,23,42,0.7);color:var(--text-dark);min-height:40px;width:100%}
    body.light input,body.light select{background:var(--bg-light);border:1px solid var(--border-light);color:var(--text-light)}
    table{width:100%;border-collapse:collapse;margin-top:8px;font-size:0.95rem}
    th,td{padding:10px;border-bottom:1px solid var(--border-dark);text-align:left}
    th{color:var(--text-dark-sec);font-weight:600}
    .charts{display:grid;grid-template-columns:1fr 260px;gap:12px}
    @media(max-width:980px){.charts{grid-template-columns:1fr}}
    .visually-hidden{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0}
    .button--danger{background:linear-gradient(135deg,var(--danger),#dc2626)}
    .toast{position:fixed;right:12px;bottom:18px;z-index:9999;padding:12px 16px;border-radius:8px;color:#fff;box-shadow:var(--shadow-sm)}
    .toast.success{background:var(--success)}
    .toast.error{background:var(--danger)}
  </style>
</head>
<body>
  <header aria-label="Cabeçalho">
    <h1>📊 Dashboard de Vendas</h1>
    <div class="actions">
      <button id="toggleTheme" aria-label="Alternar tema">🌙</button>
      <button id="exportBtn" aria-label="Exportar CSV">⬇ Exportar</button>
    </div>
  </header>

  <main class="container" id="app">
    <section>
      <div class="summary card" role="status" aria-live="polite">
        <div class="summary__item"><strong>Total de Vendas:</strong><div id="totalSales" aria-label="Total de vendas">R$ 0.00</div></div>
        <div class="summary__item"><strong>Usuários Cadastrados:</strong><div id="totalUsers">0</div></div>
      </div>
      <div class="card" aria-labelledby="tableTitle">
        <h2 id="tableTitle">Vendas Registradas</h2>
        <table id="salesTable" aria-label="Vendas">
          <thead><tr><th>Usuário</th><th>Valor (R$)</th><th>Data</th><th>Ações</th></tr></thead>
          <tbody></tbody>
        </table>
      </div>
      <div class="card charts" aria-label="Gráficos">
        <div><canvas id="salesChart" role="img" aria-label="Gráfico de vendas"></canvas></div>
        <div><canvas id="userPieChart" role="img" aria-label="Distribuição por usuário"></canvas></div>
      </div>
    </section>
    <aside>
      <div class="card" aria-labelledby="manageUsersTitle">
        <h3 id="manageUsersTitle">Gerenciar Usuários</h3>
        <label for="userName" class="visually-hidden">Nome do usuário</label>
        <div class="form-row">
          <input id="userName" placeholder="Nome do usuário" aria-describedby="userHelp" />
          <button id="addUserBtn" title="Adicionar usuário">Adicionar</button>
        </div>
        <ul id="userList" aria-label="Lista de usuários" style="margin-top:8px;list-style:none;padding:0"></ul>
      </div>
      <div class="card" aria-labelledby="registerSaleTitle" style="margin-top:12px">
        <h3 id="registerSaleTitle">Registrar Venda</h3>
        <div class="form-row">
          <select id="userSelect"><option value="">-- Selecione --</option></select>
        </div>
        <div class="form-row"><input id="saleAmount" type="number" placeholder="0.00" min="0.01" step="0.01" /></div>
        <div class="form-row"><input id="saleDate" type="date" /></div>
        <div class="form-row"><button id="addSaleBtn">Adicionar Venda</button></div>
      </div>
      <div class="card" aria-labelledby="filtersTitle" style="margin-top:12px">
        <h3 id="filtersTitle">Filtros</h3>
        <div class="form-row"><select id="filterUser"><option value="">Todos os Usuários</option></select></div>
        <div class="form-row"><input id="filterStart" type="date" /><input id="filterEnd" type="date" /></div>
      </div>
    </aside>
  </main>

  <script type="module">
  // Utility: small safe escape and CSV-safe value to prevent CSV injection
  const Utils = {
    uid() {
      return Date.now().toString(36) + Math.random().toString(36).slice(2,9);
    },
    escapeText(s = '') {
      return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
    },
    csvSafeCell(v = '') {
      let s = String(v);
      // Prevent CSV injection: prefix = +,-,=,@
      if (/^[=+\-@]/.test(s)) s = '\'' + s;
      // double quotes inside cell
      s = s.replace(/"/g, '""');
      return `"${s}"`;
    }
  };

  // StorageService: encapsula localStorage com namespace e validação
  class StorageService {
    constructor(namespace = 'dashboard') { this.ns = namespace; }
    key(k){ return `${this.ns}:${k}`; }
    set(k,v){ try { localStorage.setItem(this.key(k), JSON.stringify(v)); return true; } catch(e){ console.error('Storage.set',e); return false; } }
    get(k, fallback = null){ try { const v = localStorage.getItem(this.key(k)); return v ? JSON.parse(v) : fallback; } catch(e){ console.error('Storage.get', e); return fallback; } }
    remove(k){ try { localStorage.removeItem(this.key(k)); return true; } catch(e){ return false; } }
  }

  // Validator
  const Validator = {
    isValidName(name){ return typeof name === 'string' && (name = name.trim()) && name.length >= 2 && name.length <= 100 && !/[<>\"']/g.test(name); },
    isValidSale({user, value, date}) {
      return typeof user === 'string' && user.trim() &&
             typeof value === 'number' && value > 0 && value < 1_000_000 &&
             /^\d{4}-\d{2}-\d{2}$/.test(date);
    }
  };

  // Accessible modal (promise-based)
  class AccessibleModal {
    constructor({title='Confirmação', message='', confirmLabel='Confirmar', cancelLabel='Cancelar'}={}) {
      this.title = title; this.message = message; this.confirm = confirmLabel; this.cancel = cancelLabel;
      this.overlay = null;
    }
    show() {
      return new Promise(resolve => {
        this.overlay = document.createElement('div');
        Object.assign(this.overlay.style, {position:'fixed',inset:0,display:'flex',alignItems:'center',justifyContent:'center',background:'rgba(0,0,0,0.45)',zIndex:9999});
        this.overlay.addEventListener('click', (e)=>{ if(e.target===this.overlay) { this.close(); resolve(false); } });

        const dialog = document.createElement('div');
        dialog.setAttribute('role','dialog'); dialog.setAttribute('aria-modal','true');
        Object.assign(dialog.style,{background: getComputedStyle(document.body).backgroundColor, color: getComputedStyle(document.body).color, padding:'18px',borderRadius:'10px',maxWidth:'520px',width:'90%'});

        const h = document.createElement('h3'); h.textContent = this.title; h.style.marginBottom='8px';
        const p = document.createElement('p'); p.textContent = this.message; p.style.marginBottom='14px';
        const actions = document.createElement('div'); actions.style.display='flex'; actions.style.justifyContent='flex-end'; actions.style.gap='8px';

        const cancelBtn = document.createElement('button'); cancelBtn.textContent = this.cancel; cancelBtn.addEventListener('click',()=>{ this.close(); resolve(false); });
        const confirmBtn = document.createElement('button'); confirmBtn.textContent = this.confirm; confirmBtn.className='button--danger'; confirmBtn.addEventListener('click',()=>{ this.close(); resolve(true); });

        actions.appendChild(cancelBtn); actions.appendChild(confirmBtn);
        dialog.appendChild(h); dialog.appendChild(p); dialog.appendChild(actions);
        this.overlay.appendChild(dialog);
        document.body.appendChild(this.overlay);
        confirmBtn.focus();
      });
    }
    close(){ if(this.overlay){ this.overlay.remove(); this.overlay = null; } }
  }

  // Toasts (simple)
  const Toast = {
    create(msg, cls='success', duration=3000) {
      const t = document.createElement('div'); t.className = `toast ${cls}`; t.textContent = msg;
      document.body.appendChild(t);
      setTimeout(()=> { t.style.opacity = 0; setTimeout(()=>t.remove(),300); }, duration);
    },
    success(m){ this.create(m,'success'); },
    error(m){ this.create(m,'error'); }
  };

  // Domain services: UserService & SaleService
  class UserService {
    constructor(storage){ this.storage = storage; this.key = 'users'; this.users = this.storage.get(this.key, []); }
    all(){ return [...this.users]; }
    add(name){
      const s = String(name).trim();
      if(!Validator.isValidName(s)) return {ok:false, reason:'invalid'};
      if(this.users.includes(s)) return {ok:false, reason:'exists'};
      this.users.push(s); this.storage.set(this.key, this.users); return {ok:true};
    }
    removeAt(idx){ if(idx<0||idx>=this.users.length) return false; this.users.splice(idx,1); this.storage.set(this.key,this.users); return true; }
    count(){ return this.users.length; }
  }

  class SaleService {
    constructor(storage){ this.storage = storage; this.key='sales'; this.sales = this.storage.get(this.key, []); }
    all(){ return [...this.sales]; }
    add({user,value,date}){
      const sale = { id: Utils.uid(), user: String(user), value: Number(value), date: String(date) };
      if(!Validator.isValidSale(sale)) return {ok:false, reason:'invalid'};
      this.sales.push(sale); this.storage.set(this.key,this.sales); return {ok:true, sale};
    }
    deleteById(id){ const idx = this.sales.findIndex(s=>s.id===id); if(idx===-1) return false; this.sales.splice(idx,1); this.storage.set(this.key,this.sales); return true; }
    filter({user, minDate, maxDate} = {}) {
      return this.sales.filter(s=>{
        if(user && s.user !== user) return false;
        if(minDate && s.date < minDate) return false;
        if(maxDate && s.date > maxDate) return false;
        return true;
      });
    }
    getTotalsByUser(filtered=null){
      const set = filtered||this.sales;
      return set.reduce((acc,s)=>{ acc[s.user]=(acc[s.user]||0)+s.value; return acc; }, {});
    }
    total(filtered=null){ return (filtered||this.sales).reduce((sum,s)=>sum+s.value,0); }
  }

  // Chart manager: lazy load Chart.js and update
  const ChartManager = {
    async ensureLoaded(){
      if(window.Chart) return;
      await new Promise(resolve=>{
        const s = document.createElement('script');
        s.src = 'https://cdn.jsdelivr.net/npm/chart.js';
        s.onload = () => resolve();
        s.onerror = () => { console.warn('Chart.js load failed'); resolve(); };
        document.head.appendChild(s);
      });
    },
    async init(lineCanvas, pieCanvas){
      await this.ensureLoaded();
      if(!window.Chart) return;
      this.line = new Chart(lineCanvas.getContext('2d'), { type:'line', data:{labels:[],datasets:[{ label:'Vendas (R$)', data:[], borderColor:getComputedStyle(document.documentElement).getPropertyValue('--primary')||'#3b82f6', fill:true }] }, options:{responsive:true} });
      this.pie = new Chart(pieCanvas.getContext('2d'), { type:'pie', data:{labels:[],datasets:[{ data:[], backgroundColor:['#3b82f6','#10b981','#f59e0b','#ef4444','#8b5cf6'] }] }, options:{responsive:true} });
    },
    update(lineData, byUser){
      if(this.line){ this.line.data.labels = lineData.map((_,i)=>`Venda ${i+1}`); this.line.data.datasets[0].data = lineData; this.line.update(); }
      if(this.pie){ this.pie.data.labels = Object.keys(byUser); this.pie.data.datasets[0].data = Object.values(byUser); this.pie.update(); }
    }
  };

  // App: binds everything and handles UI safe render
  const App = (() => {
    const storage = new StorageService('dash_v1');
    const users = new UserService(storage);
    const sales = new SaleService(storage);

    const dom = {
      totalSales: document.getElementById('totalSales'),
      totalUsers: document.getElementById('totalUsers'),
      userList: document.getElementById('userList'),
      userName: document.getElementById('userName'),
      addUserBtn: document.getElementById('addUserBtn'),
      userSelect: document.getElementById('userSelect'),
      saleAmount: document.getElementById('saleAmount'),
      saleDate: document.getElementById('saleDate'),
      addSaleBtn: document.getElementById('addSaleBtn'),
      salesTableBody: document.querySelector('#salesTable tbody'),
      filterUser: document.getElementById('filterUser'),
      filterStart: document.getElementById('filterStart'),
      filterEnd: document.getElementById('filterEnd'),
      exportBtn: document.getElementById('exportBtn'),
      toggleTheme: document.getElementById('toggleTheme'),
      salesCanvas: document.getElementById('salesChart'),
      pieCanvas: document.getElementById('userPieChart'),
    };

    async function initCharts(){ await ChartManager.init(dom.salesCanvas, dom.pieCanvas); }

    function renderUsers(){
      // clear
      dom.userList.innerHTML=''; dom.userSelect.innerHTML=''; dom.filterUser.innerHTML='';
      const defaultOpt = new Option('-- Selecione --',''); dom.userSelect.add(defaultOpt.cloneNode(true));
      const defaultFilter = new Option('Todos os Usuários',''); dom.filterUser.add(defaultFilter.cloneNode(true));
      const ufrag = document.createDocumentFragment();

      users.all().forEach((u, idx)=>{
        const li = document.createElement('li');
        const name = document.createElement('span'); name.textContent = u;
        const btn = document.createElement('button'); btn.className='button--danger'; btn.textContent='Excluir';
        btn.style.marginLeft='8px';
        btn.addEventListener('click', async ()=>{
          const modal = new AccessibleModal({ title:'Excluir usuário', message:`Deseja remover ${u}? As vendas permanecerão.` });
          if(await modal.show()){
            users.removeAt(idx);
            renderUsers(); renderSales();
            Toast.success('Usuário removido');
          }
        });
        li.appendChild(name); li.appendChild(btn); ufrag.appendChild(li);

        const opt1 = new Option(u, idx); dom.userSelect.add(opt1.cloneNode(true));
        const opt2 = new Option(u, u); dom.filterUser.add(opt2.cloneNode(true));
      });

      dom.userList.appendChild(ufrag);
      dom.totalUsers.textContent = users.count();
    }

    function renderSales(){
      const filter = { user: dom.filterUser.value || undefined, minDate: dom.filterStart.value||undefined, maxDate: dom.filterEnd.value||undefined };
      const filtered = sales.filter(filter);
      const tbodyFrag = document.createDocumentFragment();
      let total = 0;

      filtered.forEach(sale=>{
        total += sale.value;
        const tr = document.createElement('tr');

        const tdUser = document.createElement('td'); tdUser.textContent = sale.user; tr.appendChild(tdUser);
        const tdVal = document.createElement('td'); tdVal.textContent = `R$ ${sale.value.toFixed(2)}`; tr.appendChild(tdVal);
        const tdDate = document.createElement('td'); tdDate.textContent = sale.date; tr.appendChild(tdDate);
        const tdAct = document.createElement('td'); 
        const delBtn = document.createElement('button'); delBtn.textContent='Excluir'; delBtn.className='button--danger';
        delBtn.addEventListener('click', async ()=> {
          const modal = new AccessibleModal({ title:'Confirmar exclusão', message:`Excluir venda R$ ${sale.value.toFixed(2)} de ${sale.user}?` });
          if(await modal.show()){
            sales.deleteById(sale.id);
            renderSales(); Toast.success('Venda excluída');
          }
        });
        tdAct.appendChild(delBtn); tr.appendChild(tdAct);

        tbodyFrag.appendChild(tr);
      });

      dom.salesTableBody.innerHTML=''; dom.salesTableBody.appendChild(tbodyFrag);
      dom.totalSales.textContent = `R$ ${sales.total(filtered).toFixed(2)}`;

      // charts
      const lineData = filtered.map(s=>s.value);
      const totals = sales.getTotalsByUser(filtered);
      ChartManager.update(lineData, totals);
    }

    // CSV export with escaping and prevention of CSV injection
    function exportCSV(){
      const all = sales.all();
      if(all.length===0){ Toast.error('Nenhuma venda para exportar'); return; }
      let csv = ['Usuário,Valor,Data'];
      for(const s of all){
        csv.push([ Utils.csvSafeCell(s.user), s.value.toFixed(2), s.date ].join(','));
      }
      const blob = new Blob([csv.join('\n')], { type:'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = `vendas_${new Date().toISOString().slice(0,10)}.csv`;
      document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
      Toast.success('Exportado com sucesso');
    }

    // Event bindings
    function bind(){
      dom.addUserBtn.addEventListener('click', ()=> {
        const v = dom.userName.value;
        const res = users.add(v);
        if(res.ok){ dom.userName.value=''; renderUsers(); Toast.success('Usuário adicionado'); dom.userName.focus(); }
        else if(res.reason==='exists') Toast.error('Usuário já existe');
        else Toast.error('Nome inválido');
      });
      dom.userName.addEventListener('keypress', (e)=> { if(e.key==='Enter') dom.addUserBtn.click(); });

      dom.addSaleBtn.addEventListener('click', ()=> {
        const userIdx = dom.userSelect.value; const user = dom.userSelect.options[dom.userSelect.selectedIndex]?.text || '';
        const amt = parseFloat(dom.saleAmount.value);
        const date = dom.saleDate.value || new Date().toISOString().slice(0,10);
        const res = sales.add({ user, value: amt, date });
        if(res.ok){ dom.saleAmount.value=''; dom.saleDate.value=''; renderSales(); Toast.success('Venda adicionada'); }
        else Toast.error('Venda inválida');
      });

      dom.filterUser.addEventListener('change', renderSales);
      dom.filterStart.addEventListener('change', renderSales);
      dom.filterEnd.addEventListener('change', renderSales);

      dom.exportBtn.addEventListener('click', exportCSV);
      dom.toggleTheme.addEventListener('click', ()=> { document.body.classList.toggle('light'); ChartManager.updateTheme?.( !document.body.classList.contains('light') ); });

      // keyboard accessibility for table actions delegated (optional)
      dom.salesTableBody.addEventListener('keydown', (e)=> { if(e.key==='Enter' && e.target.matches('button')) e.target.click(); });
    }

    return {
      async init(){
        await initCharts();
        renderUsers();
        renderSales();
        bind();
      },
      // exported for tests
      _services: { users, sales, storage }
    };
  })();

  // init
  document.addEventListener('DOMContentLoaded', () => { App.init().catch(err=>console.error(err)); });

  </script>
</body>
</html>
