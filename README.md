<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Dashboard avançado para gerenciamento e registro de vendas com filtros e gráficos analíticos.">
  <meta name="theme-color" content="#1e293b">
  <title>Dashboard de Vendas - Gerenciamento de Vendas</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js" defer></script>
  <style>
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
    }
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    html, body {
      height: 100%;
      font-family: 'Inter', sans-serif;
    }
    body {
      background: linear-gradient(135deg, var(--color-bg-dark), var(--color-bg-dark-2));
      color: var(--color-text-dark);
      padding: var(--spacing-xl);
      transition: background 0.4s, color 0.4s;
    }
    body.light {
      background: linear-gradient(135deg, var(--color-bg-light), var(--color-bg-light-2));
      color: var(--color-text-light);
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: var(--spacing-lg) var(--spacing-xl);
      border-radius: var(--radius-lg);
      background: rgba(30, 41, 59, 0.8);
      backdrop-filter: blur(10px);
      box-shadow: var(--shadow-sm);
      margin-bottom: var(--spacing-2xl);
      transition: background 0.4s, box-shadow 0.4s;
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
      transition: background 0.3s, box-shadow 0.2s;
    }
    button:hover {
      background: linear-gradient(135deg, var(--color-primary-dark), var(--color-primary-darker));
      box-shadow: 0 2px 8px rgba(59, 130, 246, 0.4);
    }
    button:focus-visible {
      outline: 2px solid var(--color-primary);
      outline-offset: 2px;
    }
    button.action {
      background: linear-gradient(135deg, var(--color-danger), var(--color-danger-dark));
      font-size: 0.8rem;
      padding: var(--spacing-xs) 14px;
    }
    button.action:hover {
      background: linear-gradient(135deg, var(--color-danger-dark), var(--color-danger-darker));
      box-shadow: 0 2px 8px rgba(239, 68, 68, 0.4);
    }
    main {
      display: grid;
      gap: var(--spacing-xl);
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
      transition: background 0.4s;
    }
    body.light .summary {
      background: rgba(255, 255, 255, 0.85);
      box-shadow: var(--shadow-light);
    }
    .summary > div {
      flex: 1;
      font-size: 1.1rem;
      font-weight: 600;
    }
    section {
      background: rgba(30, 41, 59, 0.85);
      backdrop-filter: blur(10px);
      padding: var(--spacing-xl);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-sm);
      transition: background 0.4s;
    }
    body.light section {
      background: rgba(255, 255, 255, 0.85);
    }
    h2 {
      font-size: 1.4rem;
      margin-bottom: var(--spacing-lg);
      color: #60a5fa;
    }
    body.light h2 {
      color: var(--color-text-light-sec);
    }
    label {
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
      <div>
        <strong>Total de Vendas:</strong> R$ <span id="totalSales" aria-label="Total de vendas">0.00</span>
      </div>
      <div>
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
          if (!sanitized || sanitized.length < 2) {
            alert('Digite um nome válido com pelo menos 2 caracteres.');
            return false;
          }
          if (users.includes(sanitized)) {
            alert('Este usuário já existe.');
            return false;
          }
          users.push(sanitized);
          Storage.save();
          this.render();
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
          if (!confirm('Deseja excluir esta venda?')) return;
          sales.splice(index, 1);
          Storage.save();
          this.render();
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
            userCell.setAttribute('role', 'gridcell');

            const valueCell = document.createElement('td');
            valueCell.textContent = `R$ ${sale.value.toFixed(2)}`;
            valueCell.setAttribute('role', 'gridcell');

            const dateCell = document.createElement('td');
            dateCell.textContent = sale.date;
            dateCell.setAttribute('role', 'gridcell');

            const actionCell = document.createElement('td');
            actionCell.setAttribute('role', 'gridcell');
            const btn = document.createElement('button');
            btn.className = 'action';
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
        init() {
          const ctxLine = document.getElementById('salesChart').getContext('2d');
          salesChart = new Chart(ctxLine, {
            type: 'line',
            data: {
              labels: [],
              datasets: [{
                label: 'Vendas (R$)',
                data: [],
                borderColor: '#3b82f6',
                fill: true,
                backgroundColor: 'rgba(59,130,246,0.2)',
                tension: 0.4
              }]
            },
            options: {
              responsive: true,
              plugins: { legend: { labels: { color: '#fff' } } },
              scales: {
                x: { ticks: { color: '#fff' } },
                y: { ticks: { color: '#fff' } }
              }
            }
          });

          const ctxPie = document.getElementById('userPieChart').getContext('2d');
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
              plugins: { legend: { labels: { color: '#fff' } } }
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
          const color = isDark ? '#fff' : '#222';
          if (salesChart) {
            salesChart.options.plugins.legend.labels.color = color;
            salesChart.options.scales.x.ticks.color = color;
            salesChart.options.scales.y.ticks.color = color;
            salesChart.update();
          }
          if (userPieChart) {
            userPieChart.options.plugins.legend.labels.color = color;
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
          const val = DOM.userNameInput().value;
          if (Users.add(val)) {
            DOM.userNameInput().value = '';
            DOM.userNameInput().focus();
          }
        });

        DOM.userNameInput().addEventListener('keypress', (e) => {
          if (e.key === 'Enter') DOM.addUserBtn.click();
        });

        DOM.addSaleBtn.addEventListener('click', () => {
          const userIdx = DOM.userSelect.value;
          const amount = DOM.saleAmountInput().value;
          const date = DOM.saleDateInput().value;
          if (Sales.add(userIdx, amount, date)) {
            DOM.saleAmountInput().value = '';
            DOM.saleDateInput().value = '';
            DOM.userSelect.focus();
          }
        });

        DOM.saleAmountInput().addEventListener('keypress', (e) => {
          if (e.key === 'Enter') DOM.addSaleBtn.click();
        });

        DOM.filterUser.addEventListener('change', () => Sales.render());
        DOM.filterStart().addEventListener('change', () => Sales.render());
        DOM.filterEnd().addEventListener('change', () => Sales.render());

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
