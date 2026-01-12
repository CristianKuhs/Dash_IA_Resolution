<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <meta name="description" content="Dashboard de Vendas — registro, filtros e gráficos." />
  <title>Dashboard de Vendas — Refatorado</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    /* =======================================================
       Theme / Layout (clean, responsive, accessible)
       ======================================================= */
    :root{
      --bg-1:#0f172a; --bg-2:#1e293b; --card:#0b1220aa;
      --accent:#3b82f6; --success:#10b981; --danger:#ef4444;
      --muted:#93c5fd; --text-light:#e6eef9; --text-dark:#062027;
      --radius:12px; --pad-sm:8px; --pad-md:14px; --pad-lg:20px;
      --shadow:0 10px 30px rgba(2,6,23,0.6);
      font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial;
    }
    *{box-sizing:border-box;margin:0;padding:0}
    html,body{height:100%}
    body{
      min-height:100vh;background:linear-gradient(135deg,var(--bg-1),var(--bg-2));
      color:var(--text-light);padding:28px;transition:background .25s,color .25s;
      -webkit-font-smoothing:antialiased;
    }
    body.light{
      background:linear-gradient(135deg,#f8fafc,#ffffff);color:var(--text-dark);
    }
    .wrapper{max-width:1100px;margin:0 auto;display:grid;gap:18px}
    header{display:flex;align-items:center;justify-content:space-between;background:var(--card);padding:14px;border-radius:12px;box-shadow:var(--shadow);backdrop-filter:blur(6px)}
    body.light header{background:#ffffffcc}
    header h1{font-size:1.05rem;display:flex;gap:10px;align-items:center}
    .controls{display:flex;gap:10px;align-items:center}
    .btn{cursor:pointer;padding:8px 12px;border-radius:10px;border:0;font-weight:600;color:white;background:linear-gradient(135deg,var(--accent),#2563eb);transition:transform .12s}
    .btn.secondary{background:transparent;border:1px solid rgba(255,255,255,0.06);color:inherit}
    .btn.danger{background:linear-gradient(135deg,var(--danger),#dc2626)}
    .btn:active{transform:translateY(1px)}
    .summary{display:flex;gap:12px;justify-content:space-between;background:var(--card);padding:14px;border-radius:12px}
    body.light .summary{background:#fff}
    .card{flex:1;padding:10px;border-radius:8px}
    .card strong{display:block;color:var(--muted);font-weight:600}
    .card .value{font-size:1.25rem;margin-top:6px;font-weight:700}
    .grid{display:grid;grid-template-columns:1fr 360px;gap:16px}
    @media (max-width:980px){ .grid{grid-template-columns:1fr} }
    section.panel{background:var(--card);padding:14px;border-radius:12px;box-shadow:var(--shadow)}
    body.light section.panel{background:#fff}
    .form-row{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-top:10px}
    @media (max-width:520px){ .form-row{grid-template-columns:1fr} }
    input,select,button{font-family:inherit}
    input,select{padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.03);background:transparent;color:inherit}
    body.light input,body.light select{background:#fff;border:1px solid #e6eef9;color:var(--text-dark)}
    table{width:100%;border-collapse:collapse;margin-top:10px}
    thead th{padding:10px;text-align:left;color:var(--muted);font-weight:600;border-bottom:1px solid rgba(255,255,255,0.04)}
    tbody td{padding:10px;border-bottom:1px solid rgba(255,255,255,0.03)}
    .charts{display:grid;grid-template-columns:1fr 300px;gap:12px;margin-top:12px}
    @media (max-width:980px){ .charts{grid-template-columns:1fr} }
    canvas{width:100%;height:220px;border-radius:8px}
    .list{margin-top:12px;max-height:340px;overflow:auto}
    li{display:flex;justify-content:space-between;padding:8px;border-bottom:1px solid rgba(255,255,255,0.03)}
    .muted{color:var(--muted)}
    .visually-hidden{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0}
  </style>
</head>
<body>
  <div class="wrapper" id="app">
    <header>
      <h1>📊 Dashboard de Vendas — Refatorado</h1>
      <div class="controls" role="group" aria-label="Controles principais">
        <button id="themeToggle" class="btn secondary" aria-pressed="false">Alternar tema</button>
        <button id="exportCsv" class="btn">Exportar CSV</button>
      </div>
    </header>
    <div class="summary" aria-live="polite" role="status">
      <div class="card">
        <strong>Total de Vendas</strong>
        <div id="totalSales" class="value">R$ 0.00</div>
        <div id="salesCount" class="muted">0 transações</div>
      </div>
      <div class="card">
        <strong>Usuários Cadastrados</strong>
        <div id="totalUsers" class="value">0</div>
        <div class="muted">Gerencie usuários ao lado</div>
      </div>
    </div>
    <div class="grid" role="main">
      <main>
        <section class="panel" aria-labelledby="registerTitle">
          <h2 id="registerTitle">Registrar Venda</h2>
          <div class="form-row" aria-hidden="false">
            <label>
              Usuário
              <select id="userSelect" aria-label="Selecionar usuário"><option value="">-- selecione --</option></select>
            </label>
            <label>
              Valor (R$)
              <input id="saleAmount" type="number" min="0.01" step="0.01" placeholder="0.00" inputmode="decimal" />
            </label>
          </div>
          <div class="form-row">
            <label>
              Data
              <input id="saleDate" type="date" />
            </label>
            <div style="display:flex;align-items:end;gap:8px">
              <button id="addSaleBtn" class="btn">Adicionar Venda</button>
              <button id="clearFilterBtn" class="btn secondary">Limpar filtros</button>
            </div>
          </div>
          <div style="margin-top:12px">
            <h3 class="muted">Filtros</h3>
            <div class="form-row" style="margin-top:6px">
              <label>
                Filtrar por Usuário
                <select id="filterUser"><option value="">Todos</option></select>
              </label>
              <div style="display:grid;grid-template-columns:1fr 1fr;gap:8px">
                <input id="filterStart" type="date" aria-label="Data inicial" />
                <input id="filterEnd" type="date" aria-label="Data final" />
              </div>
            </div>
          </div>
          <div style="margin-top:12px">
            <h3 class="muted">Vendas Registradas</h3>
            <div style="overflow:auto;max-height:320px">
              <table aria-live="polite" id="salesTable">
                <thead><tr><th>Usuário</th><th>Valor</th><th>Data</th><th>Ações</th></tr></thead>
                <tbody></tbody>
              </table>
            </div>
          </div>
        </section>
        <section class="panel" aria-labelledby="chartsTitle" style="margin-top:12px">
          <h2 id="chartsTitle">Gráficos</h2>
          <div class="charts">
            <canvas id="salesChart" aria-label="Gráfico de vendas"></canvas>
            <canvas id="userPie" aria-label="Distribuição por usuário"></canvas>
          </div>
        </section>
      </main>
      <aside>
        <section class="panel" aria-labelledby="usersTitle">
          <h2 id="usersTitle">Usuários</h2>
          <div style="margin-top:12px">
            <label>
              Nome
              <input id="userName" type="text" placeholder="Nome do usuário" />
            </label>
            <div style="display:flex;gap:8px;margin-top:10px">
              <button id="addUserBtn" class="btn">Adicionar</button>
              <button id="sampleBtn" class="btn secondary">Importar Exemplo</button>
            </div>
          </div>
          <ul id="userList" class="list" aria-live="polite"></ul>
        </section>
      </aside>
    </div>
  </div>

  <script>
    /* =======================================================
       Dashboard de Vendas - Código refatorado moderno
       - Arquivo único (HTML + CSS + JS)
       - Modular, com validação, sanitização e tratamento de erros
       - Lazy-load Chart.js, atualizações mínimas de DOM
       ======================================================= */

    /**
     * Utils: funções puras e seguras
     */
    const Utils = {
      sanitize(v){ return String(v ?? '').trim(); },
      isValidName(name){ const n = Utils.sanitize(name); return n.length >=2 && n.length <=100; },
      toISODate(d){ try { if(!d) return new Date().toISOString().slice(0,10); const dt = new Date(d); if(isNaN(dt)) return new Date().toISOString().slice(0,10); return dt.toISOString().slice(0,10); } catch(e){ return new Date().toISOString().slice(0,10);} },
      parseCurrency(v){ if(v===undefined||v===null) return NaN; const n = Number(String(v).replace(',', '.')); return Number.isFinite(n) ? n : NaN; },
      formatCurrency(n){ return 'R$ ' + Number(n || 0).toFixed(2); }
    };

    /**
     * Storage: encapsula localStorage com chaves versionadas e tratamento de erros
     */
    const Storage = {
      keyUsers: 'dash_v2_users',
      keySales: 'dash_v2_sales',
      load(){
        try{
          const users = JSON.parse(localStorage.getItem(this.keyUsers) || '[]');
          const sales = JSON.parse(localStorage.getItem(this.keySales) || '[]');
          return {
            users: Array.isArray(users) ? users.filter(u=>typeof u==='string' && u.trim()) : [],
            sales: Array.isArray(sales) ? sales.filter(s=> s && typeof s.user==='string' && typeof s.value==='number' && s.date) : []
          };
        }catch(err){
          console.error('Storage.load error', err);
          return {users:[], sales:[]};
        }
      },
      saveUsers(users){
        try{ localStorage.setItem(this.keyUsers, JSON.stringify(users)); }catch(e){ console.error(e); }
      },
      saveSales(sales){
        try{ localStorage.setItem(this.keySales, JSON.stringify(sales)); }catch(e){ console.error(e); }
      }
    };

    /**
     * App state and DOM cache
     */
    const State = { users:[], sales:[], charts:{line:null,pie:null} };
    const DOM = {};

    /**
     * Minimal toast (non-blocking)
     * @param {string} message
     */
    function toast(message){
      try{
        const el = document.createElement('div');
        el.textContent = message;
        el.style.cssText = 'position:fixed;right:16px;bottom:16px;background:#071025;color:white;padding:10px 14px;border-radius:10px;box-shadow:0 10px 30px rgba(0,0,0,0.6);z-index:9999';
        document.body.appendChild(el);
        setTimeout(()=>el.remove(),2300);
      }catch(e){ console.log(message); }
    }

    /**
     * Accessible confirm modal fallback using native confirm (keeps code small).
     * Could be replaced by a custom modal component for full A11Y.
     * @param {string} message
     * @returns {Promise<boolean>}
     */
    function confirmAsync(message){
      return Promise.resolve(confirm(message));
    }

    /**
     * Render users list and options
     */
    function renderUsers(){
      try{
        DOM.userList.innerHTML = '';
        DOM.userSelect.innerHTML = '<option value="">-- selecione --</option>';
        DOM.filterUser.innerHTML = '<option value="">Todos</option>';

        State.users.forEach((u, idx) => {
          // list item
          const li = document.createElement('li');
          const nameSpan = document.createElement('span');
          nameSpan.textContent = u;
          const btn = document.createElement('button');
          btn.textContent = 'Excluir';
          btn.className = 'btn secondary';
          btn.addEventListener('click', async ()=>{
            const ok = await confirmAsync(`Remover usuário "${u}"? As vendas permanecerão.`);
            if(!ok) return;
            State.users.splice(idx,1);
            Storage.saveUsers(State.users);
            renderUsers(); renderSales();
            toast('Usuário removido');
          });
          li.appendChild(nameSpan);
          li.appendChild(btn);
          DOM.userList.appendChild(li);

          // select options
          const opt = document.createElement('option'); opt.value = String(idx); opt.textContent = u;
          DOM.userSelect.appendChild(opt);
          const optFilter = document.createElement('option'); optFilter.value = u; optFilter.textContent = u;
          DOM.filterUser.appendChild(optFilter);
        });

        DOM.totalUsers.textContent = String(State.users.length);
      }catch(err){ console.error('renderUsers error', err); }
    }

    /**
     * Returns list of sales filtered by UI filters.
     * @returns {Array<{user:string,value:number,date:string}>}
     */
    function getFilteredSales(){
      const userFilter = DOM.filterUser.value;
      const start = DOM.filterStart.value || '';
      const end = DOM.filterEnd.value || '';
      return State.sales.filter(s=>{
        if(userFilter && s.user !== userFilter) return false;
        if(start && s.date < start) return false;
        if(end && s.date > end) return false;
        return true;
      });
    }

    /**
     * Render sales table and update summary
     */
    function renderSales(){
      try{
        const list = getFilteredSales();
        DOM.salesBody.innerHTML = '';
        let total = 0;
        list.forEach(sale=>{
          total += sale.value;
          const tr = document.createElement('tr');
          const tdUser = document.createElement('td'); tdUser.textContent = sale.user;
          const tdValue = document.createElement('td'); tdValue.textContent = Utils.formatCurrency(sale.value);
          const tdDate = document.createElement('td'); tdDate.textContent = sale.date;
          const tdActions = document.createElement('td');
          const del = document.createElement('button'); del.textContent='Excluir'; del.className='btn danger';
          del.addEventListener('click', async ()=>{
            const ok = await confirmAsync(`Excluir venda ${Utils.formatCurrency(sale.value)} de ${sale.user}?`);
            if(!ok) return;
            const idx = State.sales.indexOf(sale);
            if(idx>=0){ State.sales.splice(idx,1); Storage.saveSales(State.sales); renderSales(); toast('Venda excluída'); }
          });
          tdActions.appendChild(del);
          tr.append(tdUser,tdValue,tdDate,tdActions);
          DOM.salesBody.appendChild(tr);
        });
        DOM.totalSales.textContent = Utils.formatCurrency(total);
        DOM.salesCount.textContent = `${list.length} transações`;
        updateCharts(list);
      }catch(err){ console.error('renderSales error', err); }
    }

    /**
     * Charts: lazy-load Chart.js and initialize
     */
    async function loadChartJs(){
      if(window.Chart) return;
      return new Promise((resolve,reject)=>{
        const s = document.createElement('script');
        s.src = 'https://cdn.jsdelivr.net/npm/chart.js';
        s.onload = resolve; s.onerror = reject; document.head.appendChild(s);
      });
    }

    async function initCharts(){
      try{ await loadChartJs(); }catch(e){ console.warn('Chart.js failed to load', e); return; }

      const ctxLine = DOM.salesChart.getContext('2d');
      const ctxPie = DOM.userPie.getContext('2d');

      State.charts.line = new Chart(ctxLine, {
        type:'line',
        data:{labels:[],datasets:[{label:'Vendas (R$)',data:[],borderColor:getComputedStyle(document.documentElement).getPropertyValue('--accent')||'#3b82f6',tension:0.3,fill:true,backgroundColor:'rgba(59,130,246,0.12)'}]},
        options:{responsive:true,plugins:{legend:{labels:{color:getComputedStyle(document.documentElement).getPropertyValue('--muted')||'#fff'}}},scales:{x:{ticks:{color:getComputedStyle(document.documentElement).getPropertyValue('--muted')||'#fff'}},y:{ticks:{color:getComputedStyle(document.documentElement).getPropertyValue('--muted')||'#fff'}}}
      });

      State.charts.pie = new Chart(ctxPie, {
        type:'pie',
        data:{labels:[],datasets:[{data:[],backgroundColor:['#3b82f6','#10b981','#f59e0b','#ef4444','#8b5cf6']}]},
        options:{responsive:true,plugins:{legend:{labels:{color:getComputedStyle(document.documentElement).getPropertyValue('--muted')||'#fff'}}}}
      });

      updateCharts(getFilteredSales());
    }

    function updateCharts(list){
      if(!State.charts.line || !State.charts.pie) return;
      State.charts.line.data.labels = list.map((_,i)=>`#${i+1}`);
      State.charts.line.data.datasets[0].data = list.map(s=>s.value);
      State.charts.line.update();

      const totals = {};
      list.forEach(s => totals[s.user] = (totals[s.user] || 0) + s.value);
      State.charts.pie.data.labels = Object.keys(totals);
      State.charts.pie.data.datasets[0].data = Object.values(totals);
      State.charts.pie.update();
    }

    /**
     * Actions: add user, add sale, export CSV
     */
    function addUser(){
      try{
        const name = Utils.sanitize(DOM.userName.value);
        if(!Utils.isValidName(name)){ toast('Nome inválido (2-100 caracteres)'); return; }
        if(State.users.includes(name)){ toast('Usuário já existe'); return; }
        State.users.push(name); Storage.saveUsers(State.users); renderUsers(); toast('Usuário adicionado'); DOM.userName.value=''; DOM.userName.focus();
      }catch(err){ console.error('addUser', err); }
    }

    function addSale(){
      try{
        const userIndex = Number(DOM.userSelect.value);
        if(Number.isNaN(userIndex) || userIndex < 0 || userIndex >= State.users.length){ toast('Selecione um usuário válido'); return; }
        const amount = Utils.parseCurrency(DOM.saleAmount.value);
        if(Number.isNaN(amount) || amount <= 0){ toast('Informe um valor maior que zero'); return; }
        const date = Utils.toISODate(DOM.saleDate.value);
        const sale = { user: State.users[userIndex], value: Number(amount), date };
        State.sales.push(sale);
        Storage.saveSales(State.sales);
        renderSales();
        toast('Venda registrada');
        DOM.saleAmount.value=''; DOM.saleDate.value=''; DOM.userSelect.focus();
      }catch(err){ console.error('addSale', err); }
    }

    function exportCSV(){
      try{
        if(!State.sales.length){ toast('Nenhuma venda para exportar'); return; }
        const rows = ['Usuário,Valor,Data', ...State.sales.map(s=>`"${s.user}",${s.value.toFixed(2)},${s.date}`)];
        const csv = rows.join('\n');
        const blob = new Blob([csv], { type:'text/csv;charset=utf-8;' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a'); a.href = url; a.download = `vendas_${new Date().toISOString().slice(0,10)}.csv`;
        document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
        toast('CSV gerado');
      }catch(err){ console.error('exportCSV', err); }
    }

    /**
     * Bind UI events (cached DOM elements)
     */
    function bindEvents(){
      DOM.addUserBtn.addEventListener('click', addUser);
      DOM.sampleBtn.addEventListener('click', ()=> {
        const samples = ['Ana','Bruno','Carla'];
        samples.forEach(s => { if(!State.users.includes(s)) State.users.push(s); });
        State.sales.push({user:'Ana',value:120.5,date:Utils.toISODate()}, {user:'Bruno',value:54.25,date:Utils.toISODate()});
        Storage.saveUsers(State.users); Storage.saveSales(State.sales); renderUsers(); renderSales(); initCharts(); toast('Dados de exemplo importados');
      });

      DOM.addSaleBtn.addEventListener('click', addSale);
      DOM.filterUser.addEventListener('change', renderSales);
      DOM.filterStart.addEventListener('change', renderSales);
      DOM.filterEnd.addEventListener('change', renderSales);
      DOM.clearFilterBtn.addEventListener('click', ()=>{ DOM.filterUser.value=''; DOM.filterStart.value=''; DOM.filterEnd.value=''; renderSales(); });

      DOM.exportCsv.addEventListener('click', exportCSV);
      DOM.themeToggle.addEventListener('click', ()=>{
        const isLight = document.body.classList.toggle('light');
        DOM.themeToggle.setAttribute('aria-pressed', String(isLight));
      });

      DOM.userName.addEventListener('keypress', e => { if(e.key === 'Enter') addUser(); });
      DOM.saleAmount.addEventListener('keypress', e => { if(e.key === 'Enter') addSale(); });
    }

    /**
     * Initialize app: cache DOM, load storage, render UI and charts
     */
    async function init(){
      DOM.userName = document.getElementById('userName');
      DOM.addUserBtn = document.getElementById('addUserBtn');
      DOM.sampleBtn = document.getElementById('sampleBtn');
      DOM.userList = document.getElementById('userList');
      DOM.userSelect = document.getElementById('userSelect');
      DOM.addSaleBtn = document.getElementById('addSaleBtn');
      DOM.saleAmount = document.getElementById('saleAmount');
      DOM.saleDate = document.getElementById('saleDate');
      DOM.salesBody = document.querySelector('#salesTable tbody');
      DOM.totalSales = document.getElementById('totalSales');
      DOM.totalUsers = document.getElementById('totalUsers');
      DOM.salesCount = document.getElementById('salesCount');
      DOM.filterUser = document.getElementById('filterUser');
      DOM.filterStart = document.getElementById('filterStart');
      DOM.filterEnd = document.getElementById('filterEnd');
      DOM.clearFilterBtn = document.getElementById('clearFilterBtn');
      DOM.exportCsv = document.getElementById('exportCsv');
      DOM.themeToggle = document.getElementById('themeToggle');
      DOM.salesChart = document.getElementById('salesChart');
      DOM.userPie = document.getElementById('userPie');

      const data = Storage.load();
      State.users = data.users; State.sales = data.sales;

      renderUsers(); renderSales(); bindEvents(); await initCharts();
    }

    document.addEventListener('DOMContentLoaded', init);
  </script>
</body>
</html>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Dashboard avançado para gerenciamento e registro de vendas com filtros e gráficos analíticos.">
  <meta name="theme-color" content="#1e293b">
  <title>Dashboard de Vendas - Gerenciamento de Vendas</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    :root {
    :root {
      --color-bg-dark: #0f172a;
      --color-bg-dark-2: #1e293b;
      --color-bg-light: #f1f5f9;
      --color-bg-light-2: #ffffff;
      --color-text-dark: #e2e8f0;
      --color-text-dark-sec: #93c5fd;
      --color-text-light: #1e293b;
      --color-text-light-sec: #2563eb;
      --color-primary: #3b82f6;
      --color-primary-dark: #2563eb;
      --color-primary-darker: #1d4ed8;
      --color-danger: #ef4444;
      --color-danger-dark: #dc2626;
      --color-danger-darker: #b91c1c;
      --color-success: #10b981;
      --color-success-dark: #059669;
      --color-warning: #f59e0b;
      --color-warning-dark: #d97706;
      --color-border-dark: #334155;
      --color-border-light: #cbd5e1;
      --color-input-bg-dark: rgba(15, 23, 42, 0.7);
      --color-input-bg-light: rgba(241, 245, 259, 0.9);
      --radius-sm: 8px;
      --radius-md: 10px;
      --radius-lg: 16px;
      --spacing-xs: 6px;
      --spacing-sm: 8px;
      --spacing-md: 12px;
      --spacing-lg: 15px;
      --spacing-xl: 20px;
      --spacing-2xl: 25px;
      --shadow-sm: 0 4px 10px rgba(0, 0, 0, 0.4);
      --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.4);
      --shadow-light: 0 4px 10px rgba(0, 0, 0, 0.1);
      --transition-fast: 0.2s ease;
      --transition-base: 0.3s ease;
      --transition-slow: 0.4s ease;
    }
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      background: linear-gradient(135deg, var(--color-bg-dark), var(--color-bg-dark-2));
      color: var(--color-text-dark);
      padding: var(--spacing-xl);
      transition: background var(--transition-slow), color var(--transition-slow);
    }
    body.light {
      background: linear-gradient(135deg, var(--color-bg-light), var(--color-bg-light-2));
      color: var(--color-text-light);
    } transition: background 0.4s, color 0.4s;
    }
    header {
      display: flex;
      transition: background var(--transition-fast);
      align-items: center;
      padding: var(--spacing-lg) var(--spacing-xl);
      border-radius: var(--radius-lg);
      background: rgba(30, 41, 59, 0.8);
      backdrop-filter: blur(10px);
      box-shadow: var(--shadow-sm);
        table {
          font-size: 0.85rem;
          overflow-x: hidden; /* Added to remove overflow-x */
        }
      margin-bottom: var(--spacing-2xl);
      transition: background var(--transition-slow), box-shadow var(--transition-slow);
    }
    body.light header {
      background: rgba(255, 255, 255, 0.8);
      box-shadow: var(--shadow-light);
    }
    header h1 {
      font-size: 2rem;
      font-weight: 700;
    }
    button {
      background: linear-gradient(135deg, var(--color-primary), var(--color-primary-dark));
      border: none;
      color: #fff;
      padding: 10px 18px;
      border-radius: var(--radius-md);
      cursor: pointer;
      font-size: 0.95rem;
      font-weight: 600;
      transition: background var(--transition-base), box-shadow var(--transition-fast);
      min-height: 44px;
      min-width: 44px;
      display: inline-flex;
      align-items: center;
      justify-content: center;
    }
    button:hover {
      background: linear-gradient(135deg, var(--color-primary-dark), var(--color-primary-darker));
      box-shadow: 0 2px 8px rgba(59, 130, 246, 0.4);
    }
    @media (max-width: 640px) {
      table, thead, tbody, th, td, tr {
        display: block;
      }
      thead tr {
        position: absolute;
        top: -9999px;
        left: -9999px;
      }
      tr {
        margin-bottom: var(--spacing-xl);
        border: 1px solid var(--color-border-dark);
        border-radius: var(--radius-md);
        padding: var(--spacing-md);
      }
      body.light tr {
        border: 1px solid var(--color-border-light);
      }
      td {
        padding-left: 50%;
        position: relative;
        margin-bottom: var(--spacing-sm);
      }
      td:before {
        content: attr(data-label);
        position: absolute;
        left: var(--spacing-md);
        font-weight: 600;
        text-transform: capitalize;
        color: var(--color-text-dark-sec);
      }
      body.light td:before {
        color: var(--color-text-light-sec);
      }
    }
    button:focus-visible {
      outline: 2px solid var(--color-primary);
      outline-offset: 2px;
    }
    button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
    button.button--danger {
      background: linear-gradient(135deg, var(--color-danger), var(--color-danger-dark));
      font-size: 0.8rem;
      padding: var(--spacing-xs) 14px;
    }
    .summary {
      display: flex;
      justify-content: space-between;
      background: rgba(30, 41, 59, 0.85);
      backdrop-filter: blur(10px);
      padding: var(--spacing-xl);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-md);
      text-align: center;
      transition: background var(--transition-slow);
    }
    body.light .summary {
      background: rgba(255, 255, 255, 0.85);
      box-shadow: var(--shadow-light);
    }
    .summary__item {
      flex: 1;
      font-size: 1.1rem;
      font-weight: 600;
    } box-shadow: 0 2px 8px rgba(239, 68, 68, 0.4);
    }
    section {
      background: rgba(30, 41, 59, 0.85);
      backdrop-filter: blur(10px);
      padding: var(--spacing-xl);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-sm);
      transition: background var(--transition-slow);
    }
    body.light section {
      background: rgba(255, 255, 255, 0.85);
    }
    section__title {
      font-size: 1.4rem;
      margin-bottom: var(--spacing-lg);
      color: #60a5fa;
    }
    body.light section__title {
      color: var(--color-text-light-sec);
    }
    label {
      display: block;
      margin-bottom: var(--spacing-sm);
      font-weight: 500;
      font-size: 0.95rem;
    }
    .form__group {
      margin-bottom: var(--spacing-md);
    }
    input, select {
      width: 100%;
      padding: 10px;
      margin: var(--spacing-sm) 0;
      border: 1px solid var(--color-border-dark);
      border-radius: var(--radius-md);
      font-size: 1rem;
      background: var(--color-input-bg-dark);
      color: var(--color-text-dark);
      transition: border var(--transition-base), background var(--transition-base);
      min-height: 44px;
    }
    input:focus-visible, select:focus-visible {
      border-color: var(--color-primary);
      outline: 2px solid var(--color-primary);
      outline-offset: -1px;
    }
    input:invalid:not(:placeholder-shown) {
      border-color: var(--color-danger);
    }
    input:valid:not(:placeholder-shown),
    input[type="date"],
    input[type="number"] {
      border-color: var(--color-success);
    }
    body.light input, body.light select {
      background: var(--color-input-bg-light);
      color: var(--color-text-light);
      border: 1px solid var(--color-border-light);
    }
    .input__error {
      display: block;
      color: var(--color-danger);
      font-size: 0.85rem;
      margin-top: var(--spacing-xs);
    }abel {
      display: block;
      margin-bottom: var(--spacing-sm);
      font-weight: 500;
      font-size: 0.95rem;
    }
    input, select {
      width: 100%;
      padding: 10px;
      margin: var(--spacing-sm) 0;
      border: 1px solid var(--color-border-dark);
      border-radius: var(--radius-md);
      font-size: 1rem;
      background: var(--color-input-bg-dark);
      color: var(--color-text-dark);
      transition: border 0.3s, background 0.3s;
    }
    input:focus-visible, select:focus-visible {
      border-color: var(--color-primary);
      outline: 2px solid var(--color-primary);
      outline-offset: -1px;
    }
    body.light input, body.light select {
      background: var(--color-input-bg-light);
      color: var(--color-text-light);
      border: 1px solid var(--color-border-light);
    }
    ul {
      list-style: none;
      padding: 0;
      margin: var(--spacing-md) 0 0;
    }
    li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: var(--spacing-sm) var(--spacing-md);
      margin-bottom: var(--spacing-xs);
      border-radius: var(--radius-sm);
      background: rgba(15, 23, 42, 0.8);
      font-size: 0.95rem;
      transition: background 0.2s;
    }
    li:hover {
      background: rgba(30, 41, 59, 0.9);
    }
    body.light li {
      background: rgba(241, 245, 259, 0.9);
    }
    body.light li:hover {
      background: rgba(226, 232, 240, 0.95);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: var(--spacing-md);
      font-size: 0.95rem;
      overflow-x: auto;
    }
    thead {
      background: rgba(30, 41, 59, 0.9);
    }
    th, td {
      padding: var(--spacing-md);
      text-align: left;
      border-bottom: 1px solid var(--color-border-dark);
    }
    th {
      color: var(--color-text-dark-sec);
      font-weight: 600;
    }
    body.light th {
      background: rgba(226, 232, 240, 0.9);
      color: var(--color-text-light);
    }
    .charts {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: var(--spacing-xl);
    }
    canvas {
      background: rgba(15, 23, 42, 0.85);
      padding: 10px;
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-sm);
    }
    body.light canvas {
      background: rgba(255, 255, 255, 0.85);
    }
    .visually-hidden {
      position: absolute;
      width: 1px;
      height: 1px;
      padding: 0;
      margin: -1px;
      overflow: hidden;
      clip: rect(0, 0, 0, 0);
      white-space: nowrap;
      border-width: 0;
    }
    @media (max-width: 768px) {
      header {
        flex-direction: column;
        align-items: flex-start;
        gap: var(--spacing-md);
      }
      .summary {
        flex-direction: column;
        gap: var(--spacing-md);
      }
      table {
        font-size: 0.85rem;
      }
      th, td {
        padding: var(--spacing-sm);
      }
    }
  </style>
</head>
<body>
  <header>
    <h1>📊 Dashboard de Vendas</h1>
    <div>
      <button id="toggleTheme" aria-label="Alternar tema claro/escuro">🌙 Tema</button>
      <button id="exportButton" aria-label="Exportar dados de vendas em CSV">⬇ Exportar CSV</button>
    </div>
  </header>

  <main>
    <div class="summary" role="status" aria-live="polite">
      <div class="summary__item">
        <strong>Total de Vendas:</strong> R$ <span id="totalSales" aria-label="Total de vendas">0.00</span>
      </div>
      <div class="summary__item">
        <strong>Usuários Cadastrados:</strong> <span id="totalUsers" aria-label="Quantidade de usuários">0</span>
      </div>
    </div>
    <section aria-labelledby="manageUsersTitle">
      <h2 id="manageUsersTitle">Gerenciar Usuários</h2>
      <div>
        <label for="userName">Nome do usuário</label>
        <input type="text" id="userName" name="userName" placeholder="Digite o nome" aria-describedby="userNameHelp">
        <span id="userNameHelp" class="visually-hidden">Digite um nome válido para adicionar novo usuário</span>
      </div>
      <button id="addUserBtn">Adicionar Usuário</button>
      <ul id="userList" role="list" aria-label="Lista de usuários cadastrados"></ul>
    </section>
    <section aria-labelledby="registerSaleTitle">
      <h2 id="registerSaleTitle">Registrar Venda</h2>
      <fieldset>
        <legend class="visually-hidden">Formulário de registro de venda</legend>
        <div>
          <label for="userSelect">Selecione um usuário</label>
          <select id="userSelect" name="userSelect" aria-describedby="userSelectHelp">
            <option value="">-- Selecione um usuário --</option>
          </select>
          <span id="userSelectHelp" class="visually-hidden">Escolha um usuário para registrar a venda</span>
        </div>
        <div>
          <label for="saleAmount">Valor da venda (R$)</label>
          <input type="number" id="saleAmount" name="saleAmount" placeholder="0.00" min="0.01" step="0.01" aria-describedby="saleAmountHelp">
          <span id="saleAmountHelp" class="visually-hidden">Digite um valor positivo para a venda</span>
        </div>
        <div>
          <label for="saleDate">Data da venda</label>
          <input type="date" id="saleDate" name="saleDate" aria-describedby="saleDateHelp">
          <span id="saleDateHelp" class="visually-hidden">Selecione a data da venda ou deixe em branco para usar a data de hoje</span>
        </div>
      </fieldset>
      <button id="addSaleBtn">Adicionar Venda</button>
    </section>
    <section aria-labelledby="filterSalesTitle">
      <h2 id="filterSalesTitle">Filtrar Vendas</h2>
      <fieldset>
        <legend class="visually-hidden">Filtros de vendas</legend>
        <div>
          <label for="filterUser">Usuário</label>
          <select id="filterUser" name="filterUser" aria-describedby="filterUserHelp">
            <option value="">Todos os Usuários</option>
          </select>
          <span id="filterUserHelp" class="visually-hidden">Filtre por um usuário específico ou veja todos</span>
        </div>
        <div>
          <label for="filterStart">Data inicial</label>
          <input type="date" id="filterStart" name="filterStart" aria-describedby="filterStartHelp">
          <span id="filterStartHelp" class="visually-hidden">Selecione a data inicial do período</span>
        </div>
        <div>
          <label for="filterEnd">Data final</label>
          <input type="date" id="filterEnd" name="filterEnd" aria-describedby="filterEndHelp">
          <span id="filterEndHelp" class="visually-hidden">Selecione a data final do período</span>
        </div>
      </fieldset>
    </section>
    <section aria-labelledby="salesTableTitle">
      <h2 id="salesTableTitle">Vendas Registradas</h2>
      <table id="salesTable" role="grid" aria-label="Tabela de vendas registradas">
        <thead>
          <tr>
            <th scope="col">Usuário</th>
            <th scope="col">Valor (R$)</th>
            <th scope="col">Data</th>
            <th scope="col">Ações</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </section>
    <div class="charts" role="region" aria-label="Gráficos analíticos">
      <div>
        <canvas id="salesChart" role="img" aria-label="Gráfico de linha com vendas por transação"></canvas>
      </div>
      <div>
        <canvas id="userPieChart" role="img" aria-label="Gráfico de pizza com distribuição de vendas por usuário"></canvas>
      </div>
    </div>
  </main>

  <script defer>
    /* ===== MODAL COMPONENT ===== */
    class AccessibleModal {
      constructor(options = {}) {
        this.title = options.title || 'Confirmação';
        this.message = options.message || '';
        this.onConfirm = options.onConfirm || (() => {});
        this.onCancel = options.onCancel || (() => {});
        this.modal = null;
      }

      show() {
        this.modal = document.createElement('div');
        this.modal.setAttribute('role', 'alertdialog');
        this.modal.setAttribute('aria-modal', 'true');
        this.modal.setAttribute('aria-labelledby', 'modal-title');
        this.modal.setAttribute('aria-describedby', 'modal-message');
        this.modal.style.cssText = `
          position: fixed; top: 0; left: 0; right: 0; bottom: 0;
          background: rgba(0,0,0,0.5); display: flex; align-items: center;
          justify-content: center; z-index: 1000; padding: var(--spacing-lg);
        `;

        const content = document.createElement('div');
        content.style.cssText = `
          background: var(--color-bg-dark-2); color: var(--color-text-dark);
          padding: var(--spacing-xl); border-radius: var(--radius-lg);
          max-width: 400px; box-shadow: 0 10px 40px rgba(0,0,0,0.5);
        `;
        document.body.classList.contains('light') && (content.style.background = 'var(--color-bg-light-2)', content.style.color = 'var(--color-text-light)');

        content.innerHTML = `
          <h3 id="modal-title" style="margin-bottom: var(--spacing-md); font-size: 1.2rem;">${this.title}</h3>
          <p id="modal-message" style="margin-bottom: var(--spacing-xl); line-height: 1.6;">${this.message}</p>
          <div style="display: flex; gap: var(--spacing-md); justify-content: flex-end;">
            <button id="modal-cancel" class="button" style="background: linear-gradient(135deg, var(--color-border-dark), var(--color-border-dark));">Cancelar</button>
            <button id="modal-confirm" class="button button--danger">Confirmar</button>
          </div>
        `;

        this.modal.appendChild(content);
        document.body.appendChild(this.modal);

        const confirmBtn = this.modal.querySelector('#modal-confirm');
        const cancelBtn = this.modal.querySelector('#modal-cancel');

        confirmBtn.addEventListener('click', () => this.handleConfirm());
        cancelBtn.addEventListener('click', () => this.handleCancel());
        this.modal.addEventListener('click', (e) => e.target === this.modal && this.handleCancel());

        confirmBtn.focus();
      }

      handleConfirm() {
        this.onConfirm();
        this.close();
      }

      handleCancel() {
        this.onCancel();
        this.close();
      }

      close() {
        if (this.modal) {
          this.modal.remove();
          this.modal = null;
        }
      }
    }

    /* ===== TOAST NOTIFICATION SYSTEM ===== */
    class Toast {
      static create(message, type = 'info', duration = 3000) {
        const toast = document.createElement('div');
        toast.setAttribute('role', 'status');
        toast.setAttribute('aria-live', 'polite');
        toast.setAttribute('aria-atomic', 'true');

        const bgColor = {
          success: 'var(--color-success)',
          error: 'var(--color-danger)',
          warning: 'var(--color-warning)',
          info: 'var(--color-primary)'
        }[type] || 'var(--color-primary)';

        toast.style.cssText = `
          position: fixed; bottom: var(--spacing-xl); right: var(--spacing-xl);
          background: ${bgColor}; color: white; padding: var(--spacing-md) var(--spacing-lg);
          border-radius: var(--radius-md); box-shadow: var(--shadow-md);
          animation: slideIn 0.3s ease; z-index: 999; max-width: 300px;
          word-wrap: break-word;
        `;
        toast.textContent = message;

        if (!document.getElementById('toast-style')) {
          const style = document.createElement('style');
          style.id = 'toast-style';
          style.textContent = `
            @keyframes slideIn {
              from { transform: translateX(400px); opacity: 0; }
              to { transform: translateX(0); opacity: 1; }
            }
            @keyframes slideOut {
              from { transform: translateX(0); opacity: 1; }
              to { transform: translateX(400px); opacity: 0; }
            }
          `;
          document.head.appendChild(style);
        }

        document.body.appendChild(toast);

        setTimeout(() => {
          toast.style.animation = 'slideOut 0.3s ease forwards';
          setTimeout(() => toast.remove(), 300);
        }, duration);
      }

      static success(message) { this.create(message, 'success'); }
      static error(message) { this.create(message, 'error'); }
      static warning(message) { this.create(message, 'warning'); }
      static info(message) { this.create(message, 'info'); }
    }

    /* ===== MAIN APP ===== */
    const DashboardApp = (() => {
      const DOM = {
        userNameInput: () => document.getElementById('userName'),
        userSelect: document.getElementById('userSelect'),
        filterUser: document.getElementById('filterUser'),
        saleAmountInput: () => document.getElementById('saleAmount'),
        saleDateInput: () => document.getElementById('saleDate'),
        filterStart: () => document.getElementById('filterStart'),
        filterEnd: () => document.getElementById('filterEnd'),
        salesTable: document.getElementById('salesTable').querySelector('tbody'),
        totalSalesEl: document.getElementById('totalSales'),
        totalUsersEl: document.getElementById('totalUsers'),
        userList: document.getElementById('userList'),
        addUserBtn: document.getElementById('addUserBtn'),
        addSaleBtn: document.getElementById('addSaleBtn'),
        exportBtn: document.getElementById('exportButton'),
        toggleThemeBtn: document.getElementById('toggleTheme'),
      };

      let users = [];
      let sales = [];
      let salesChart = null;
      let userPieChart = null;

      const Storage = {
        load() {
          try {
            users = JSON.parse(localStorage.getItem('users')) || [];
            sales = JSON.parse(localStorage.getItem('sales')) || [];
            this.validate();
          } catch (e) {
            console.error('Erro ao carregar dados:', e);
            users = [];
            sales = [];
          }
        },
        save() {
          try {
            localStorage.setItem('users', JSON.stringify(users));
            localStorage.setItem('sales', JSON.stringify(sales));
          } catch (e) {
            console.error('Erro ao salvar dados:', e);
          }
        },
        validate() {
          users = Array.isArray(users) ? users.filter(u => typeof u === 'string' && u.trim()) : [];
          sales = Array.isArray(sales) ? sales.filter(s => s.user && typeof s.value === 'number' && s.date) : [];
        }
      };

      const Sanitize = {
        text(str) {
          const div = document.createElement('div');
          div.textContent = String(str || '').trim();
          return div.textContent;
        }
      };

      const Users = {
        add(name) {
          const sanitized = Sanitize.text(name);
          if (!Sanitize.isValidName(sanitized)) {
            Toast.error('Nome inválido. Use 2-100 caracteres.');
            return false;
          }
          if (users.includes(sanitized)) {
            Toast.warning('Este usuário já existe.');
            return false;
          }
          users.push(sanitized);
          Storage.save();
          this.render();
          Toast.success(`Usuário "${sanitized}" adicionado!`);
          return true;
        },
        delete(index) {
          if (index < 0 || index >= users.length) return;
          if (!confirm(`Deseja excluir "${users[index]}"? As vendas dele permanecerão.`)) return;
          users.splice(index, 1);
          Storage.save();
          this.render();
        },
        render() {
          DOM.userList.innerHTML = '';
          DOM.userSelect.innerHTML = '';
          DOM.filterUser.innerHTML = '<option value="">Todos os Usuários</option>';

          users.forEach((user, index) => {
            const opt1 = document.createElement('option');
            opt1.value = index;
            opt1.textContent = user;
            DOM.userSelect.appendChild(opt1);

            const opt2 = document.createElement('option');
            opt2.value = user;
            opt2.textContent = user;
            DOM.filterUser.appendChild(opt2);

            const li = document.createElement('li');
            li.setAttribute('role', 'listitem');
            const span = document.createElement('span');
            span.textContent = user;
            const btn = document.createElement('button');
            btn.className = 'action';
            btn.textContent = 'Excluir';
            btn.setAttribute('aria-label', `Excluir usuário ${user}`);
            btn.addEventListener('click', () => Users.delete(index));

            li.appendChild(span);
            li.appendChild(btn);
            DOM.userList.appendChild(li);
          });

          DOM.totalUsersEl.textContent = users.length;
        }
      };

      const Sales = {
        add(userIndex, amount, date) {
          const idx = parseInt(userIndex, 10);
          if (isNaN(idx) || idx < 0 || idx >= users.length) {
            alert('Selecione um usuário válido.');
            return false;
          }
          const val = parseFloat(amount);
          if (isNaN(val) || val <= 0) {
            alert('Digite um valor válido maior que zero.');
            return false;
          }
          const saleDate = date && date.trim() ? date : new Date().toISOString().split('T')[0];
          sales.push({ user: users[idx], value: val, date: saleDate });
          Storage.save();
          this.render();
          return true;
        },
        delete(index) {
          if (index < 0 || index >= sales.length) return;
          const sale = sales[index];
          const modal = new AccessibleModal({
            title: 'Confirmar exclusão de venda',
            message: `Deseja realmente excluir a venda de R$ ${sale.value.toFixed(2)} do usuário ${sale.user}?`,
            onConfirm: () => {
              sales.splice(index, 1);
              Storage.save();
              this.render();
              Toast.success('Venda excluída com sucesso.');
            },
            onCancel: () => Toast.info('Exclusão cancelada.')
          });
          modal.show();
        },
        filter() {
          const filterUserVal = DOM.filterUser.value;
          const filterStartVal = DOM.filterStart().value;
          const filterEndVal = DOM.filterEnd().value;

          return sales.filter(s => {
            if (filterUserVal && s.user !== filterUserVal) return false;
            if (filterStartVal && s.date < filterStartVal) return false;
            if (filterEndVal && s.date > filterEndVal) return false;
            return true;
          });
        },
        render() {
          DOM.salesTable.innerHTML = '';
          const filtered = this.filter();
          let total = 0;

          filtered.forEach((sale, index) => {
            total += sale.value;
            const row = document.createElement('tr');
            row.setAttribute('role', 'row');

            const userCell = document.createElement('td');
            userCell.textContent = sale.user;
            userCell.setAttribute('data-label', 'Usuário');
            userCell.setAttribute('role', 'gridcell');

            const valueCell = document.createElement('td');
            valueCell.textContent = `R$ ${sale.value.toFixed(2)}`;
            valueCell.setAttribute('data-label', 'Valor (R$)');
            valueCell.setAttribute('role', 'gridcell');

            const dateCell = document.createElement('td');
            dateCell.textContent = sale.date;
            dateCell.setAttribute('data-label', 'Data');
            dateCell.setAttribute('role', 'gridcell');

            const actionCell = document.createElement('td');
            actionCell.setAttribute('role', 'gridcell');
            actionCell.setAttribute('data-label', 'Ações');
            const btn = document.createElement('button');
            btn.className = 'button button--danger';
            btn.textContent = 'Excluir';
            btn.setAttribute('aria-label', `Excluir venda de ${sale.value.toFixed(2)} reais`);
            btn.addEventListener('click', () => Sales.delete(index));

            actionCell.appendChild(btn);
            row.appendChild(userCell);
            row.appendChild(valueCell);
            row.appendChild(dateCell);
            row.appendChild(actionCell);
            DOM.salesTable.appendChild(row);
          });

          DOM.totalSalesEl.textContent = total.toFixed(2);
          Charts.update(filtered);
        }
      };

      const Charts = {
        async loadChartJS() {
          return new Promise((resolve) => {
            if (window.Chart) return resolve();
            const script = document.createElement('script');
            script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
            script.onload = () => resolve();
            document.head.appendChild(script);
          });
        },
        init() {
          this.loadChartJS().then(() => this._initCharts());
        },
        _initCharts() {
          const ctxLineEl = document.getElementById('salesChart');
          if (!ctxLineEl) return;
          const ctxLine = ctxLineEl.getContext('2d');
          salesChart = new Chart(ctxLine, {
            type: 'line',
            data: {
              labels: [],
              datasets: [{
                label: 'Vendas (R$)',
                data: [],
                borderColor: getComputedStyle(document.documentElement).getPropertyValue('--color-primary').trim() || '#3b82f6',
                fill: true,
                backgroundColor: 'rgba(59,130,246,0.12)',
                tension: 0.4
              }]
            },
            options: {
              responsive: true,
              plugins: { legend: { labels: { color: getComputedStyle(document.documentElement).getPropertyValue('--color-text-dark').trim() || '#fff' } } },
              scales: {
                x: { ticks: { color: getComputedStyle(document.documentElement).getPropertyValue('--color-text-dark').trim() || '#fff' } },
                y: { ticks: { color: getComputedStyle(document.documentElement).getPropertyValue('--color-text-dark').trim() || '#fff' } }
              }
            }
          });

          const ctxPieEl = document.getElementById('userPieChart');
          if (!ctxPieEl) return;
          const ctxPie = ctxPieEl.getContext('2d');
          userPieChart = new Chart(ctxPie, {
            type: 'pie',
            data: {
              labels: [],
              datasets: [{
                data: [],
                backgroundColor: ['#3b82f6','#10b981','#f59e0b','#ef4444','#8b5cf6']
              }]
            },
            options: {
              responsive: true,
              plugins: { legend: { labels: { color: getComputedStyle(document.documentElement).getPropertyValue('--color-text-dark').trim() || '#fff' } } }
            }
          });
        },
        update(filtered) {
          const totalPerUser = {};
          filtered.forEach(sale => {
            totalPerUser[sale.user] = (totalPerUser[sale.user] || 0) + sale.value;
          });

          if (salesChart) {
            salesChart.data.labels = filtered.map((_, i) => `Venda ${i + 1}`);
            salesChart.data.datasets[0].data = filtered.map(s => s.value);
            salesChart.update();
          }

          if (userPieChart) {
            userPieChart.data.labels = Object.keys(totalPerUser);
            userPieChart.data.datasets[0].data = Object.values(totalPerUser);
            userPieChart.update();
          }
        },
        updateTheme(isDark) {
          const legendColor = isDark ? getComputedStyle(document.documentElement).getPropertyValue('--color-text-dark').trim() : getComputedStyle(document.documentElement).getPropertyValue('--color-text-light').trim();
          if (salesChart) {
            salesChart.options.plugins.legend.labels.color = legendColor;
            if (salesChart.options.scales && salesChart.options.scales.x) salesChart.options.scales.x.ticks.color = legendColor;
            if (salesChart.options.scales && salesChart.options.scales.y) salesChart.options.scales.y.ticks.color = legendColor;
            salesChart.update();
          }
          if (userPieChart) {
            userPieChart.options.plugins.legend.labels.color = legendColor;
            userPieChart.update();
          }
        }
      };

      const Theme = {
        toggle() {
          document.body.classList.toggle('light');
          const isDark = !document.body.classList.contains('light');
          localStorage.setItem('theme', isDark ? 'dark' : 'light');
          Charts.updateTheme(isDark);
        },
        load() {
          const saved = localStorage.getItem('theme');
          if (saved === 'light') {
            document.body.classList.add('light');
          }
        }
      };

      const Export = {
        csv() {
          if (sales.length === 0) {
            alert('Nenhuma venda para exportar.');
            return;
          }
          let csv = 'Usuário,Valor,Data\n';
          sales.forEach(s => {
            csv += `"${s.user}",${s.value.toFixed(2)},${s.date}\n`;
          });
          const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
          const url = URL.createObjectURL(blob);
          const link = document.createElement('a');
          link.setAttribute('href', url);
          link.setAttribute('download', `vendas_${new Date().toISOString().split('T')[0]}.csv`);
          link.click();
        }
      };

      function bindEvents() {
        DOM.addUserBtn.addEventListener('click', () => {
          const val = DOM.getUserNameInput().value;
          if (Users.add(val)) {
            DOM.getUserNameInput().value = '';
            DOM.getUserNameInput().focus();
          }
        });

        DOM.getUserNameInput().addEventListener('keypress', (e) => {
          if (e.key === 'Enter') DOM.addUserBtn.click();
        });

        DOM.addSaleBtn.addEventListener('click', () => {
          const userIdx = DOM.userSelect.value;
          const amount = DOM.getSaleAmountInput().value;
          const date = DOM.getSaleDateInput().value;
          if (Sales.add(userIdx, amount, date)) {
            DOM.getSaleAmountInput().value = '';
            DOM.getSaleDateInput().value = '';
            DOM.userSelect.focus();
          }
        });

        DOM.getSaleAmountInput().addEventListener('keypress', (e) => {
          if (e.key === 'Enter') DOM.addSaleBtn.click();
        });

        DOM.filterUser.addEventListener('change', () => Sales.render());
        DOM.getFilterStart().addEventListener('change', () => Sales.render());
        DOM.getFilterEnd().addEventListener('change', () => Sales.render());

        DOM.exportBtn.addEventListener('click', () => Export.csv());
        DOM.toggleThemeBtn.addEventListener('click', () => Theme.toggle());
      }

      return {
        init() {
          Storage.load();
          Theme.load();
          Charts.init();
          bindEvents();
          Users.render();
          Sales.render();
        }
      };
    })();

    document.addEventListener('DOMContentLoaded', () => {
      DashboardApp.init();
    });
  </script>
</body>
</html>
    document.addEventListener('DOMContentLoaded', () => {
      DashboardApp.init();
    });
  </script>
</body>
</html>
