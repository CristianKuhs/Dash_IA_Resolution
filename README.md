<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Dashboard de Vendas</title>

<style>
:root {
  --bg-dark: #0f172a;
  --bg-dark-2: #1e293b;
  --bg-light: #f8fafc;
  --text-dark: #e5e7eb;
  --text-light: #1e293b;
  --primary: #3b82f6;
  --danger: #ef4444;
  --success: #10b981;
  --radius: 12px;
  --transition: 0.3s ease;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: system-ui, sans-serif;
}

body {
  background: linear-gradient(135deg, var(--bg-dark), var(--bg-dark-2));
  color: var(--text-dark);
  padding: 20px;
  transition: var(--transition);
}

body.light {
  background: var(--bg-light);
  color: var(--text-light);
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

button {
  padding: 10px 16px;
  border-radius: var(--radius);
  border: none;
  cursor: pointer;
  font-weight: 600;
  background: var(--primary);
  color: white;
}

button.danger {
  background: var(--danger);
}

section {
  background: rgba(255,255,255,0.05);
  padding: 20px;
  border-radius: var(--radius);
  margin-bottom: 24px;
}

body.light section {
  background: white;
}

input, select {
  width: 100%;
  padding: 10px;
  margin: 6px 0 12px;
  border-radius: var(--radius);
  border: 1px solid #cbd5e1;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 12px;
  border-bottom: 1px solid #334155;
}

body.light th, body.light td {
  border-color: #e5e7eb;
}
</style>
</head>

<body>
<header>
  <h1>📊 Dashboard de Vendas</h1>
  <button id="themeBtn">Alternar Tema</button>
</header>

<section>
  <h2>Usuários</h2>
  <input id="userName" placeholder="Nome do usuário" />
  <button id="addUser">Adicionar</button>
  <ul id="users"></ul>
</section>

<section>
  <h2>Registrar Venda</h2>
  <select id="saleUser"></select>
  <input type="number" id="saleValue" placeholder="Valor" />
  <input type="date" id="saleDate" />
  <button id="addSale">Registrar</button>
</section>

<section>
  <h2>Vendas</h2>
  <table>
    <thead>
      <tr>
        <th>Usuário</th>
        <th>Valor</th>
        <th>Data</th>
        <th></th>
      </tr>
    </thead>
    <tbody id="sales"></tbody>
  </table>
</section>

<script>
const State = {
  users: JSON.parse(localStorage.getItem('users') || '[]'),
  sales: JSON.parse(localStorage.getItem('sales') || '[]'),
  save() {
    localStorage.setItem('users', JSON.stringify(this.users));
    localStorage.setItem('sales', JSON.stringify(this.sales));
  }
};

const UI = {
  users: document.getElementById('users'),
  sales: document.getElementById('sales'),
  saleUser: document.getElementById('saleUser'),

  renderUsers() {
    this.users.innerHTML = '';
    this.saleUser.innerHTML = '<option value="">Selecione</option>';
    State.users.forEach((u, i) => {
      const li = document.createElement('li');
      li.textContent = u;
      this.users.appendChild(li);

      const opt = document.createElement('option');
      opt.value = i;
      opt.textContent = u;
      this.saleUser.appendChild(opt);
    });
  },

  renderSales() {
    this.sales.innerHTML = '';
    State.sales.forEach((s, i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${s.user}</td>
        <td>R$ ${s.value.toFixed(2)}</td>
        <td>${s.date}</td>
        <td><button class="danger">X</button></td>
      `;
      tr.querySelector('button').onclick = () => {
        State.sales.splice(i, 1);
        State.save();
        this.renderSales();
      };
      this.sales.appendChild(tr);
    });
  }
};

document.getElementById('addUser').onclick = () => {
  const name = document.getElementById('userName').value.trim();
  if (!name || State.users.includes(name)) return;
  State.users.push(name);
  State.save();
  UI.renderUsers();
};

document.getElementById('addSale').onclick = () => {
  const idx = UI.saleUser.value;
  const value = parseFloat(document.getElementById('saleValue').value);
  const date = document.getElementById('saleDate').value || new Date().toISOString().slice(0,10);
  if (idx === '' || value <= 0) return;

  State.sales.push({ user: State.users[idx], value, date });
  State.save();
  UI.renderSales();
};

document.getElementById('themeBtn').onclick = () => {
  document.body.classList.toggle('light');
};

UI.renderUsers();
UI.renderSales();
</script>
</body>
</html>
