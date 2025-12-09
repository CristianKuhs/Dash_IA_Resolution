<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Dashboard avançado para gerenciamento e registro de vendas com filtros e gráficos analíticos.">
  <meta name="theme-color" content="#1e293b">
  <title>Dashboard de Vendas</title>

  <!-- Fonte -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">

  <style>
    /* ==============================
        🎨 VARIÁVEIS & RESET GLOBAL
       ============================== */
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

      --color-danger: #ef4444;
      --color-success: #10b981;
      --color-warning: #f59e0b;

      --radius-md: 10px;
      --spacing-md: 12px;

      --shadow-md: 0 4px 12px rgba(0,0,0,.4);
    }

    *{margin:0;padding:0;box-sizing:border-box;}

    body{
      font-family:"Inter",sans-serif;
      background:linear-gradient(135deg,var(--color-bg-dark),var(--color-bg-dark-2));
      color:var(--color-text-dark);
      padding:20px;
      transition:.4s;
    }

    body.light{
      background:linear-gradient(135deg,var(--color-bg-light),var(--color-bg-light-2));
      color:var(--color-text-light);
    }

    /* ==============================
          🔹 HEADER
       ============================== */
    header{
      display:flex;
      justify-content:space-between;
      align-items:center;
      padding:20px;
      margin-bottom:20px;
      background:rgba(30,41,59,.8);
      backdrop-filter:blur(10px);
      border-radius:var(--radius-md);
      box-shadow:var(--shadow-md);
    }

    body.light header{
      background:rgba(255,255,255,.8);
    }

    button{
      background:var(--color-primary);
      padding:10px 18px;
      color:white;
      border:none;
      border-radius:var(--radius-md);
      cursor:pointer;
      font-weight:600;
      transition:.3s;
    }
    button:hover{
      background:var(--color-primary-dark);
    }

    /* ==============================
          📌 CONTAINERS
       ============================== */
    section,.summary{
      background:rgba(30,41,59,.8);
      backdrop-filter:blur(10px);
      padding:20px;
      margin-top:20px;
      border-radius:var(--radius-md);
    }

    body.light section, body.light .summary{
      background:rgba(255,255,255,.85);
    }

    h2{margin-bottom:10px;}

    input,select{
      width:100%;
      padding:10px;
      border-radius:var(--radius-md);
      margin-top:6px;
      border:1px solid rgba(255,255,255,0.2);
      background:rgba(255,255,255,.08);
      color:inherit;
    }

    body.light input, body.light select{
      background:rgba(0,0,0,.08);
      border:1px solid #ccc;
    }

    /* ==============================
          📋 LISTAS & TABELAS
       ============================== */
    ul{margin-top:15px;list-style:none;}
    li{
      display:flex;
      justify-content:space-between;
      padding:10px;
      margin-bottom:6px;
      background:rgba(255,255,255,.05);
      border-radius:6px;
    }

    table{width:100%;margin-top:15px;border-collapse:collapse;font-size:.95rem;}
    th,td{
      padding:12px;
      border-bottom:1px solid rgba(255,255,255,.1);
    }

    th{color:var(--color-text-dark-sec);}

    body.light th{color:var(--color-text-light);}

    .delete-btn{
      background:var(--color-danger);
      font-size:.8rem;
      padding:6px 12px;
    }

    canvas{
      width:100%;
      margin-top:20px;
      border-radius:var(--radius-md);
      padding:10px;
      background:rgba(255,255,255,.05);
    }

  </style>
</head>

<body>

<header>
  <h1>📊 Dashboard de Vendas</h1>
  <div>
    <button id="toggleTheme">🌙 Tema</button>
    <button id="exportButton">⬇ Exportar CSV</button>
  </div>
</header>

<main>

  <!-- RESUMO -->
  <div class="summary">
    <p><strong>Total de Vendas:</strong> R$ <span id="totalSales">0.00</span></p>
    <p><strong>Usuários:</strong> <span id="totalUsers">0</span></p>
  </div>

  <!-- USUÁRIOS -->
  <section>
    <h2>Gerenciar Usuários</h2>
    <input id="userName" placeholder="Nome do usuário...">
    <button id="addUserBtn" style="margin-top:10px">Adicionar Usuário</button>
    <ul id="userList"></ul>
  </section>

  <!-- REGISTRAR VENDA -->
  <section>
    <h2>Registrar Venda</h2>
    <select id="userSelect"><option value="">Selecione um usuário</option></select>
    <input id="saleAmount" type="number" placeholder="Valor da venda (R$)">
    <input id="saleDate" type="date">
    <button id="addSaleBtn" style="margin-top:10px">Registrar Venda</button>
  </section>

  <!-- FILTRO -->
  <section>
    <h2>Filtrar Vendas</h2>
    <select id="filterUser"><option value="">Todos os usuários</option></select>
    <input id="filterStart" type="date">
    <input id="filterEnd" type="date">
  </section>

  <!-- TABELA -->
  <section>
    <h2>Vendas Registradas</h2>
    <table id="salesTable">
      <thead>
        <tr>
          <th>Usuário</th>
          <th>Valor</th>
          <th>Data</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

  <!-- GRÁFICOS -->
  <section>
    <canvas id="salesChart"></canvas>
    <canvas id="userPieChart"></canvas>
  </section>

</main>

<!-- CHART.JS -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
/* ===========================
   📌 STATE / STORAGE
=========================== */
let users = JSON.parse(localStorage.getItem("users")) || [];
let sales = JSON.parse(localStorage.getItem("sales")) || [];

/* ===========================
   ⚙️ ELEMENTOS DOM
=========================== */
const userName = document.getElementById("userName");
const addUserBtn = document.getElementById("addUserBtn");
const userList = document.getElementById("userList");
const userSelect = document.getElementById("userSelect");
const filterUser = document.getElementById("filterUser");

const addSaleBtn = document.getElementById("addSaleBtn");
const saleAmount = document.getElementById("saleAmount");
const saleDate = document.getElementById("saleDate");

const totalUsers = document.getElementById("totalUsers");
const totalSales = document.getElementById("totalSales");
const salesTable = document.querySelector("#salesTable tbody");

/* ===========================
   🚀 FUNÇÕES PRINCIPAIS
=========================== */

function syncStorage() {
  localStorage.setItem("users", JSON.stringify(users));
  localStorage.setItem("sales", JSON.stringify(sales));
}

function addUser() {
  if (!userName.value.trim()) return;
  users.push(userName.value.trim());
  userName.value = "";
  updateUI();
}

function addSale() {
  if (!userSelect.value || !saleAmount.value) return;

  sales.push({
    user: userSelect.value,
    value: Number(saleAmount.value),
    date: saleDate.value || new Date().toISOString().split("T")[0]
  });

  saleAmount.value = "";
  saleDate.value = "";
  updateUI();
}

function deleteSale(index) {
  sales.splice(index, 1);
  updateUI();
}

function updateUI() {
  syncStorage();
  renderUsers();
  renderSales();
  updateTotals();
  updateCharts();
}

/* USERS */
function renderUsers() {
  userList.innerHTML = "";
  userSelect.innerHTML = `<option value="">Selecione</option>`;
  filterUser.innerHTML = `<option value="">Todos</option>`;

  users.forEach((u,i)=>{
    const li=document.createElement("li");
    li.textContent=u;
    userList.appendChild(li);

    userSelect.innerHTML += `<option>${u}</option>`;
    filterUser.innerHTML += `<option>${u}</option>`;
  });

  totalUsers.textContent = users.length;
}

/* SALES */
function renderSales() {
  salesTable.innerHTML = "";

  sales.forEach((s,i)=>{
    const row = `
      <tr>
        <td>${s.user}</td>
        <td>R$ ${s.value.toFixed(2)}</td>
        <td>${s.date}</td>
        <td><button class="delete-btn" onclick="deleteSale(${i})">Excluir</button></td>
      </tr>
    `;
    salesTable.innerHTML += row;
  });
}

function updateTotals() {
  const total = sales.reduce((sum,s)=>sum+s.value,0);
  totalSales.textContent = total.toFixed(2);
}

/* ===========================
   📊 GRÁFICOS
=========================== */
let lineChart, pieChart;

function updateCharts() {
  const ctx1 = document.getElementById("salesChart").getContext("2d");
  const ctx2 = document.getElementById("userPieChart").getContext("2d");

  if(lineChart) lineChart.destroy();
  if(pieChart) pieChart.destroy();

  lineChart = new Chart(ctx1,{
    type:"line",
    data:{
      labels:sales.map(s=>s.date),
      datasets:[{
        label:"Vendas (R$)",
        data:sales.map(s=>s.value),
        borderWidth:2
      }]
    }
  });

  const grouped = users.map(u=> sales.filter(s=>s.user===u).reduce((a,b)=>a+b.value,0));

  pieChart = new Chart(ctx2,{
    type:"pie",
    data:{
      labels:users,
      datasets:[{data:grouped}]
    }
  });
}

/* ===========================
   🌗 TEMA
=========================== */
document.getElementById("toggleTheme").onclick=()=>{
  document.body.classList.toggle("light");
};

/* ===========================
   EXPORTAR CSV
=========================== */
document.getElementById("exportButton").onclick=()=>{
  const csv = "Usuário,Valor,Data\n" + sales.map(s=>`${s.user},${s.value},${s.date}`).join("\n");
  const blob = new Blob([csv],{type:"text/csv"});
  const url = URL.createObjectURL(blob);
  const a=document.createElement("a");

  a.href=url;
  a.download="vendas.csv";
  a.click();
};

/* ===========================
   EVENT LISTENERS
=========================== */
addUserBtn.onclick=addUser;
addSaleBtn.onclick=addSale;

updateUI();
</script>

</body>
</html>
