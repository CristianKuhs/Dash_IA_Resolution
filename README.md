<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard de Vendas Avançado</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    /* ===== RESET ===== */
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
      background: linear-gradient(135deg, #0f172a, #1e293b);
      color: #e2e8f0;
      padding: 20px;
      transition: background 0.4s, color 0.4s;
    }
    body.light {
      background: linear-gradient(135deg, #f1f5f9, #ffffff);
      color: #1e293b;
    }
    /* ===== HEADER ===== */
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 20px;
      border-radius: 16px;
      background: rgba(30, 41, 59, 0.8);
      backdrop-filter: blur(10px);
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.4);
      margin-bottom: 25px;
      transition: background 0.4s, box-shadow 0.4s;
    }
    body.light header {
      background: rgba(255, 255, 255, 0.8);
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    header h1 {
      font-size: 2rem;
      font-weight: 700;
    }
    /* ===== BOTÕES ===== */
    button {
      background: linear-gradient(135deg, #3b82f6, #2563eb);
      border: none;
      color: #fff;
      padding: 10px 18px;
      border-radius: 10px;
      cursor: pointer;
      font-size: 0.95rem;
      font-weight: 600;
      transition: transform 0.2s, background 0.3s;
    }
    button:hover {
      transform: scale(1.07);
      background: linear-gradient(135deg, #2563eb, #1d4ed8);
    }
    button.action {
      background: linear-gradient(135deg, #ef4444, #dc2626);
      font-size: 0.8rem;
      padding: 6px 14px;
    }
    button.action:hover {
      background: linear-gradient(135deg, #dc2626, #b91c1c);
    }
    /* ===== LAYOUT ===== */
    .container {
      display: grid;
      gap: 20px;
    }
    .summary {
      display: flex;
      justify-content: space-between;
      background: rgba(30, 41, 59, 0.85);
      backdrop-filter: blur(10px);
      padding: 20px;
      border-radius: 16px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
      text-align: center;
      transition: background 0.4s;
    }
    body.light .summary {
      background: rgba(255, 255, 255, 0.85);
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    .summary div {
      flex: 1;
      font-size: 1.1rem;
      font-weight: 600;
    }
    /* ===== FORM CONTAINER ===== */
    .form-container {
      background: rgba(30, 41, 59, 0.85);
      backdrop-filter: blur(10px);
      padding: 20px;
      border-radius: 16px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.4);
      transition: background 0.4s;
    }
    body.light .form-container {
      background: rgba(255, 255, 255, 0.85);
    }
    h2 {
      font-size: 1.4rem;
      margin-bottom: 15px;
      color: #60a5fa;
    }
    body.light h2 {
      color: #2563eb;
    }
    /* ===== INPUTS ===== */
    input, select {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border: 1px solid #334155;
      border-radius: 10px;
      font-size: 1rem;
      background: rgba(15, 23, 42, 0.7);
      color: #f1f5f9;
      transition: border 0.3s, background 0.3s, transform 0.2s;
    }
    input:focus, select:focus {
      border-color: #3b82f6;
      outline: none;
      transform: scale(1.02);
    }
    body.light input, body.light select {
      background: rgba(241, 245, 249, 0.9);
      color: #1e293b;
      border: 1px solid #cbd5e1;
    }
    /* ===== LISTA DE USUÁRIOS ===== */
    #userList {
      list-style: none;
      padding: 0;
      margin: 10px 0 0;
    }
    #userList li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 8px 12px;
      margin-bottom: 6px;
      border-radius: 8px;
      background: rgba(15, 23, 42, 0.8);
      font-size: 0.95rem;
      transition: transform 0.2s;
    }
    #userList li:hover {
      transform: scale(1.02);
    }
    body.light #userList li {
      background: rgba(241, 245, 249, 0.9);
    }
    /* ===== TABELA ===== */
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
      font-size: 0.95rem;
      overflow: hidden;
      border-radius: 10px;
    }
    th, td {
      padding: 12px;
      text-align: left;
      border-bottom: 1px solid #334155;
    }
    th {
      background: rgba(30, 41, 59, 0.9);
      color: #93c5fd;
    }
    body.light th {
      background: rgba(226, 232, 240, 0.9);
      color: #1e293b;
    }
    /* ===== GRÁFICOS ===== */
    .charts {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 20px;
    }
    canvas {
      background: rgba(15, 23, 42, 0.85);
      padding: 10px;
      border-radius: 16px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.4);
    }
    body.light canvas {
      background: rgba(255, 255, 255, 0.85);
    }
    /* ===== RESPONSIVO ===== */
    @media (max-width: 768px) {
      header {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
      }
      .summary {
        flex-direction: column;
        gap: 12px;
      }
    }
  </style>
</head>
<body>
  <header>
    <h1>📊 Dashboard de Vendas</h1>
    <div>
      <button id="toggleTheme">🌙 Tema</button>
      <button onclick="exportCSV()">⬇ Exportar CSV</button>
    </div>
  </header>
  <div class="container">
    <div class="summary">
      <div><strong>Total de Vendas:</strong> R$ <span id="totalSales">0.00</span></div>
      <div><strong>Usuários Cadastrados:</strong> <span id="totalUsers">0</span></div>
    </div>
    <!-- Gerenciar Usuários -->
    <div class="form-container">
      <h2>Gerenciar Usuários</h2>
      <input type="text" id="userName" placeholder="Nome do usuário">
      <button onclick="addUser()">Adicionar Usuário</button>
      <ul id="userList"></ul>
    </div>
    <!-- Registro de Vendas -->
    <div class="form-container">
      <h2>Registrar Venda</h2>
      <select id="userSelect"></select>
      <input type="number" id="saleAmount" placeholder="Valor da venda (R$)">
      <input type="date" id="saleDate">
      <button onclick="addSale()">Adicionar Venda</button>
    </div>
    <!-- Filtros -->
    <div class="form-container">
      <h2>Filtrar Vendas</h2>
      <select id="filterUser" onchange="renderSales()">
        <option value="">Todos os Usuários</option>
      </select>
      <input type="date" id="filterStart" onchange="renderSales()">
      <input type="date" id="filterEnd" onchange="renderSales()">
    </div>
    <!-- Vendas Registradas -->
    <div class="form-container">
      <h2>Vendas Registradas</h2>
      <table id="salesTable">
        <thead>
          <tr>
            <th>Usuário</th>
            <th>Valor (R$)</th>
            <th>Data</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>
    <!-- Gráficos -->
    <div class="charts">
      <canvas id="salesChart"></canvas>
      <canvas id="userPieChart"></canvas>
    </div>
  </div>
  <script>
    /* ===== JS FUNCIONAL ===== */
    let users = JSON.parse(localStorage.getItem('users')) || [];
    let sales = JSON.parse(localStorage.getItem('sales')) || [];
    const userSelect = document.getElementById('userSelect');
    const filterUser = document.getElementById('filterUser');
    const salesTable = document.getElementById('salesTable').querySelector('tbody');
    const totalSalesEl = document.getElementById('totalSales');
    const totalUsersEl = document.getElementById('totalUsers');
    const userList = document.getElementById('userList');
    function saveData() {
      localStorage.setItem('users', JSON.stringify(users));
      localStorage.setItem('sales', JSON.stringify(sales));
    }
    function addUser() {
      const name = document.getElementById('userName').value.trim();
      if (!name) return alert('Digite um nome.');
      users.push(name);
      document.getElementById('userName').value = '';
      saveData();
      updateUserSelect();
    }
    function deleteUser(index) {
      if (!confirm('Deseja excluir este usuário? As vendas dele permanecerão.')) return;
      users.splice(index, 1);
      saveData();
      updateUserSelect();
    }
    function updateUserSelect() {
      userSelect.innerHTML = '';
      filterUser.innerHTML = '<option value="">Todos os Usuários</option>';
      userList.innerHTML = '';
      users.forEach((user, index) => {
        const opt1 = document.createElement('option');
        opt1.value = index;
        opt1.textContent = user;
        userSelect.appendChild(opt1);
        const opt2 = document.createElement('option');
        opt2.value = user;
        opt2.textContent = user;
        filterUser.appendChild(opt2);
        const li = document.createElement('li');
        li.innerHTML = `${user} <button class="action" onclick="deleteUser(${index})">Excluir</button>`;
        userList.appendChild(li);
      });
      totalUsersEl.innerText = users.length;
    }
    function addSale() {
      const userIndex = userSelect.value;
      const amount = parseFloat(document.getElementById('saleAmount').value);
      const date = document.getElementById('saleDate').value || new Date().toISOString().slice(0,10);
      if (userIndex === '' || isNaN(amount) || amount <= 0) return alert('Preencha os campos corretamente.');
      sales.push({ user: users[userIndex], value: amount, date });
      document.getElementById('saleAmount').value = '';
      document.getElementById('saleDate').value = '';
      saveData();
      renderSales();
    }
    function deleteSale(index) {
      if (!confirm('Excluir esta venda?')) return;
      sales.splice(index, 1);
      saveData();
      renderSales();
    }
    function renderSales() {
      salesTable.innerHTML = '';
      let total = 0;
      const filterU = filterUser.value;
      const filterStart = document.getElementById('filterStart').value;
      const filterEnd = document.getElementById('filterEnd').value;
      const filteredSales = sales.filter(s => {
        if (filterU && s.user !== filterU) return false;
        if (filterStart && s.date < filterStart) return false;
        if (filterEnd && s.date > filterEnd) return false;
        return true;
      });
      filteredSales.forEach((sale, index) => {
        total += sale.value;
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${sale.user}</td>
          <td>R$ ${sale.value.toFixed(2)}</td>
          <td>${sale.date}</td>
          <td><button class="action" onclick="deleteSale(${index})">Excluir</button></td>
        `;
        salesTable.appendChild(row);
      });
      totalSalesEl.innerText = total.toFixed(2);
      updateCharts(filteredSales);
    }
    function exportCSV() {
      let csv = 'Usuário,Valor,Data\n';
      sales.forEach(s => {
        csv += `${s.user},${s.value},${s.date}\n`;
      });
      const blob = new Blob([csv], { type: 'text/csv' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'vendas.csv';
      a.click();
    }
    /* ===== GRÁFICOS ===== */
    const ctxSales = document.getElementById('salesChart').getContext('2d');
    const salesChart = new Chart(ctxSales, {
      type: 'line',
      data: { labels: [], datasets: [{ label: 'Vendas (R$)', data: [], borderColor: '#3b82f6', fill: true, backgroundColor: 'rgba(59,130,246,0.2)', tension: 0.4 }] },
      options: { responsive: true, plugins: { legend: { labels: { color: '#fff' } } }, scales: { x: { ticks: { color: '#fff' } }, y: { ticks: { color: '#fff' } } } }
    });
    const ctxPie = document.getElementById('userPieChart').getContext('2d');
    const userPieChart = new Chart(ctxPie, {
      type: 'pie',
      data: { labels: [], datasets: [{ data: [], backgroundColor: ['#3b82f6','#10b981','#f59e0b','#ef4444','#8b5cf6'] }] },
      options: { plugins: { legend: { labels: { color: '#fff' } } } }
    });
    function updateCharts(filteredSales) {
      const totalPerUser = {};
      filteredSales.forEach(sale => {
        totalPerUser[sale.user] = (totalPerUser[sale.user] || 0) + sale.value;
      });
      salesChart.data.labels = filteredSales.map((s, i) => `Venda ${i + 1}`);
      salesChart.data.datasets[0].data = filteredSales.map(s => s.value);
      salesChart.update();
      userPieChart.data.labels = Object.keys(totalPerUser);
      userPieChart.data.datasets[0].data = Object.values(totalPerUser);
      userPieChart.update();
    }
    /* ===== TEMA ===== */
    document.getElementById('toggleTheme').addEventListener('click', () => {
      document.body.classList.toggle('light');
      const color = document.body.classList.contains('light') ? '#222' : '#fff';
      salesChart.options.plugins.legend.labels.color = color;
      salesChart.options.scales.x.ticks.color = color;
      salesChart.options.scales.y.ticks.color = color;
      userPieChart.options.plugins.legend.labels.color = color;
      salesChart.update();
      userPieChart.update();
    });
    updateUserSelect();
    renderSales();
  </script>
</body>
</html>
