<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Logística & Almoxarifado - Reservas</title>
  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#2563eb">
  <style>
    * { box-sizing: border-box; font-family: 'Segoe UI', system-ui, sans-serif; }
    body { background-color: #f1f5f9; margin: 0; padding: 15px; color: #1e293b; }
    .container { max-width: 1000px; margin: auto; background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); }
    
    /* Login / Escolha de Perfil */
    #loginScreen { text-align: center; padding: 40px 20px; }
    #loginScreen h1 { color: #0f172a; margin-bottom: 10px; }
    #loginScreen p { color: #64748b; margin-bottom: 30px; }
    .login-buttons { display: flex; justify-content: center; gap: 15px; flex-wrap: wrap; }
    .btn-profile { background: #2563eb; color: white; border: none; padding: 15px 30px; font-size: 16px; font-weight: bold; border-radius: 8px; cursor: pointer; transition: 0.2s; }
    .btn-profile.admin { background: #0f172a; }
    .btn-profile:hover { opacity: 0.9; transform: translateY(-2px); }

    .app-screen { display: none; }
    .header-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; border-bottom: 2px solid #e2e8f0; padding-bottom: 12px; }
    .header-bar h2 { margin: 0; font-size: 20px; color: #0f172a; }
    .btn-logout { background: #ef4444; color: white; border: none; padding: 6px 12px; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: bold; }

    /* Abas e Painéis */
    .tabs { display: flex; gap: 10px; margin-bottom: 20px; border-bottom: 1px solid #cbd5e1; padding-bottom: 10px; }
    .tab-btn { background: #e2e8f0; border: none; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-weight: bold; color: #475569; }
    .tab-btn.active { background: #2563eb; color: white; }

    .form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 12px; margin-bottom: 15px; }
    .form-group { display: flex; flex-direction: column; }
    .form-group label { font-weight: bold; margin-bottom: 4px; font-size: 12px; color: #475569; }
    .form-group input, .form-group select { padding: 9px; border: 1px solid #cbd5e1; border-radius: 6px; font-size: 14px; }
    
    button.btn-action { background-color: #2563eb; color: white; border: none; padding: 10px 15px; font-weight: bold; border-radius: 6px; cursor: pointer; }
    button.btn-action:hover { background-color: #1d4ed8; }
    
    .excel-box { background: #f8fafc; border: 2px dashed #cbd5e1; padding: 15px; border-radius: 8px; margin-bottom: 20px; text-align: center; }
    
    .table-container { overflow-x: auto; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { padding: 10px 12px; text-align: left; border-bottom: 1px solid #e2e8f0; font-size: 13px; }
    th { background-color: #f8fafc; color: #475569; font-weight: 600; }
    
    .badge-pos { background-color: #e0f2fe; color: #0369a1; padding: 3px 8px; border-radius: 4px; font-weight: bold; font-size: 11px; }
    .badge-reserva { background-color: #fef3c7; color: #d97706; padding: 4px 8px; border-radius: 6px; font-weight: bold; display: inline-block; margin-bottom: 8px; }
    
    .card-reserva { background: #fffbeb; border: 1px solid #fde68a; padding: 15px; border-radius: 8px; margin-bottom: 15px; }
    .search-input { width: 100%; padding: 10px; border: 1px solid #cbd5e1; border-radius: 6px; margin-bottom: 15px; font-size: 14px; }
  </style>
</head>
<body>

<div class="container">

  <!-- TELA DE LOGIN / ESCOLHA DE PERFIL -->
  <div id="loginScreen">
    <h1>📦 Sistema de Logística & EPIs</h1>
    <p>Selecione como deseja acessar o sistema:</p>
    <div class="login-buttons">
      <button class="btn-profile" onclick="openApp('user')">👤 Entrar como Usuário (Fazer Reserva)</button>
      <button class="btn-profile admin" onclick="openApp('admin')">🔑 Entrar como Administrador (Almoxarifado)</button>
    </div>
  </div>

  <!-- TELA DO USUÁRIO -->
  <div id="userScreen" class="app-screen">
    <div class="header-bar">
      <h2>👤 Painel do Usuário - Catálogo e Reservas</h2>
      <button class="btn-logout" onclick="logout()">Sair</button>
    </div>

    <input type="text" id="userSearch" class="search-input" onkeyup="renderUserCatalog()" placeholder="Pesquisar EPI, fardamento ou peça por código ou descrição...">

    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>Código</th>
            <th>Descrição</th>
            <th>UC</th>
            <th>Posição</th>
            <th>Disponível</th>
            <th>Ação</th>
          </tr>
        </thead>
        <tbody id="userCatalogBody"></tbody>
      </table>
    </div>
  </div>

  <!-- TELA DO ADMINISTRADOR -->
  <div id="adminScreen" class="app-screen">
    <div class="header-bar">
      <h2>🔑 Painel do Administrador - Almoxarifado</h2>
      <button class="btn-logout" onclick="logout()">Sair</button>
    </div>

    <div class="tabs">
      <button class="tab-btn active" onclick="switchTab('estoque')">📦 Estoque & Cadastro</button>
      <button class="tab-btn" onclick="switchTab('pedidos')" id="tabPedidosBtn">🔔 Solicitações de Reserva (0)</button>
    </div>

    <!-- Aba Estoque -->
    <div id="tabEstoque" class="admin-tab">
      <div class="excel-box">
        <p style="margin:0 0 8px 0; font-weight:bold; font-size:13px; color:#334155;">⚡ Cadastro Rápido via Planilha Excel (CSV)</p>
        <p style="margin:0 0 10px 0; font-size:12px; color:#64748b;">O arquivo deve conter as colunas exatas: <b>Codigo, Descricao, UC, Posicao, Quantidade</b></p>
        <input type="file" id="csvFile" accept=".csv" style="font-size:12px;">
        <button class="btn-action" onclick="importExcel()" style="margin-top:8px; padding:6px 12px; font-size:12px;">Importar Produtos</button>
      </div>

      <input type="text" class="search-input" id="adminSearch" onkeyup="renderAdminStock()" placeholder="Pesquisar no estoque...">

      <div class="table-container">
        <table>
          <thead>
            <tr>
              <th>Código</th>
              <th>Descrição</th>
              <th>UC</th>
              <th>Posição</th>
              <th>Qtd.</th>
              <th>Ações</th>
            </tr>
          </thead>
          <tbody id="adminStockBody"></tbody>
        </table>
      </div>
    </div>

    <!-- Aba Pedidos / Reservas -->
    <div id="tabPedidos" class="admin-tab" style="display:none;">
      <h3 style="margin-top:0; font-size:16px; color:#1e293b;">Reservas Feitas pelos Usuários para Separação</h3>
      <div id="adminReservasContainer"></div>
    </div>

  </div>

</div>

<script>
  // Banco de Dados Local
  let products = JSON.parse(localStorage.getItem('almox_products')) || [];
  let reservations = JSON.parse(localStorage.getItem('almox_reservas')) || [];

  // Service Worker PWA
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js').catch(() => {});
  }

  function openApp(profile) {
    document.getElementById('loginScreen').style.display = 'none';
    if (profile === 'user') {
      document.getElementById('userScreen').style.display = 'block';
      renderUserCatalog();
    } else {
      document.getElementById('adminScreen').style.display = 'block';
      renderAdminStock();
      updateReservationsBadge();
    }
  }

  function logout() {
    document.getElementById('loginScreen').style.display = 'block';
    document.getElementById('userScreen').style.display = 'none';
    document.getElementById('adminScreen').style.display = 'none';
  }

  // --- MÓDULO DO USUÁRIO ---
  function renderUserCatalog() {
    const query = document.getElementById('userSearch').value.toLowerCase();
    const tbody = document.getElementById('userCatalogBody');
    tbody.innerHTML = '';
    
    const filtered = products.filter(p => 
      p.code.toLowerCase().includes(query) || 
      p.description.toLowerCase().includes(query) ||
      p.position.toLowerCase().includes(query)
    );

    if (filtered.length === 0) {
      tbody.innerHTML = `<tr><td colspan="6" style="text-align:center; color:#64748b;">Nenhum produto encontrado.</td></tr>`;
      return;
    }

    filtered.forEach((item, index) => {
      const realIndex = products.indexOf(item);
      tbody.innerHTML += `
        <tr>
          <td><strong>${item.code}</strong></td>
          <td>${item.description}</td>
          <td>${item.uc}</td>
          <td><span class="badge-pos">${item.position}</span></td>
          <td>${item.quantity}</td>
          <td>
            <input type="number" id="qty_${realIndex}" min="1" max="${item.quantity}" value="1" style="width:50px; padding:4px;">
            <button class="btn-action" style="padding:4px 8px; font-size:12px;" onclick="fazerReserva(${realIndex})">Reservar</button>
          </td>
        </tr>
      `;
    });
  }

  function fazerReserva(index) {
    const qtdReq = parseInt(document.getElementById(`qty_${index}`).value);
    if (!qtdReq || qtdReq <= 0) return alert('Informe uma quantidade válida.');
    if (qtdReq > products[index].quantity) return alert('Quantidade solicitada maior que o estoque disponível.');

    const numReserva = 'RES-' + Math.floor(100000 + Math.random() * 900000);
    
    const novaReserva = {
      id: numReserva,
      data: new Date().toLocaleDateString('pt-BR') + ' ' + new Date().toLocaleTimeString('pt-BR', {hour: '2-digit', minute:'2-digit'}),
      code: products[index].code,
      description: products[index].description,
      position: products[index].position,
      quantity: qtdReq,
      uc: products[index].uc
    };

    reservations.unshift(novaReserva);
    localStorage.setItem('almox_reservas', JSON.stringify(reservations));

    // Baixa automática provisória ou guarda para o adm separar
    alert(`Reserva realizada com sucesso!\nNúmero da Reserva: ${numReserva}\nA mensagem foi enviada para o painel do Administrador.`);
    document.getElementById(`qty_${index}`).value = 1;
  }

  // --- MÓDULO DO ADMINISTRADOR ---
  function switchTab(tab) {
    document.querySelectorAll('.admin-tab').forEach(el => el.style.display = 'none');
    document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
    
    if (tab === 'estoque') {
      document.getElementById('tabEstoque').style.display = 'block';
      event.target.classList.add('active');
      renderAdminStock();
    } else {
      document.getElementById('tabPedidos').style.display = 'block';
      event.target.classList.add('active');
      renderAdminReservas();
    }
  }

  function renderAdminStock() {
    const query = document.getElementById('adminSearch').value.toLowerCase();
    const tbody = document.getElementById('adminStockBody');
    tbody.innerHTML = '';

    const filtered = products.filter(p => 
      p.code.toLowerCase().includes(query) || p.description.toLowerCase().includes(query)
    );

    filtered.forEach((item, index) => {
      const realIndex = products.indexOf(item);
      tbody.innerHTML += `
        <tr>
          <td><strong>${item.code}</strong></td>
          <td>${item.description}</td>
          <td>${item.uc}</td>
          <td><span class="badge-pos">${item.position}</span></td>
          <td>${item.quantity}</td>
          <td><button style="background:#ef4444; color:white; border:none; padding:4px 8px; border-radius:4px; cursor:pointer; font-size:11px;" onclick="deleteProduct(${realIndex})">Excluir</button></td>
        </tr>
      `;
    });
  }

  function deleteProduct(index) {
    if (confirm('Deseja excluir este item do estoque?')) {
      products.splice(index, 1);
      localStorage.setItem('almox_products', JSON.stringify(products));
      renderAdminStock();
    }
  }

  // Importar Excel via arquivo CSV
  function importExcel() {
    const fileInput = document.getElementById('csvFile');
    const file = fileInput.files[0];
    if (!file) return alert('Selecione um arquivo CSV gerado pelo Excel.');

    const reader = new FileReader();
    reader.onload = function(e) {
      const text = e.target.result;
      const lines = text.split('\n');
      let count = 0;

      for (let i = 1; i < lines.length; i++) {
        const line = lines[i].trim();
        if (!line) continue;
        const cols = line.split(',').map(c => c.replace(/^"|"$/g, '').trim());
        if (cols.length >= 5) {
          products.push({
            code: cols[0],
            description: cols[1],
            uc: cols[2],
            position: cols[3],
            quantity: parseInt(cols[4]) || 0
          });
          count++;
        }
      }

      localStorage.setItem('almox_products', JSON.stringify(products));
      alert(`Sucesso! ${count} produtos foram cadastrados via planilha.`);
      fileInput.value = '';
      renderAdminStock();
    };
    reader.readAsText(file);
  }

  function renderAdminReservas() {
    const container = document.getElementById('adminReservasContainer');
    container.innerHTML = '';

    if (reservations.length === 0) {
      container.innerHTML = `<p style="color:#64748b; text-align:center;">Nenhuma solicitação de reserva pendente no momento.</p>`;
      return;
    }

    reservations.forEach((res, index) => {
      container.innerHTML += `
        <div class="card-reserva">
          <span class="badge-reserva">Reserva #${res.id}</span>
          <span style="float:right; font-size:12px; color:#64748b;">${res.data}</span>
          <p style="margin:4px 0; font-size:14px;"><b>Item:</b> ${res.description} (Cód: <b>${res.code}</b>)</p>
          <p style="margin:4px 0; font-size:14px;"><b>Quantidade Solicitada:</b> ${res.quantity} ${res.uc}</p>
          <p style="margin:4px 0; font-size:14px;"><b>Posição no Almoxarifado para Separação:</b> <span class="badge-pos">${res.position}</span></p>
          <button class="btn-action" style="margin-top:10px; background:#10b981; font-size:12px; padding:6px 12px;" onclick="concluirSeparacao(${index})">✅ Separado / Concluir Pedido</button>
        </div>
      `;
    });
  }

  function concluirSeparacao(index) {
    const res = reservations[index];
    // Baixar o estoque do produto correspondente
    const prod = products.find(p => p.code === res.code);
    if (prod) {
      prod.quantity = Math.max(0, prod.quantity - res.quantity);
      localStorage.setItem('almox_products', JSON.stringify(products));
    }

    reservations.splice(index, 1);
    localStorage.setItem('almox_reservas', JSON.stringify(reservations));
    renderAdminReservas();
    updateReservationsBadge();
    alert('Pedido dado como separado e estoque baixado com sucesso!');
  }

  function updateReservationsBadge() {
    const btn = document.getElementById('tabPedidosBtn');
    if (btn) btn.innerText = `🔔 Solicitações de Reserva (${reservations.length})`;
  }

  // Inicializa contagem no botão
  updateReservationsBadge();
</script>

</body>
</html>
