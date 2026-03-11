<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Estoque Pro - Histórico e Auditoria</title>
    <style>
        :root { --primary: #2c3e50; --accent: #3498db; --success: #27ae60; --danger: #e74c3c; --bg: #f4f7f6; }
        body { font-family: sans-serif; background: var(--bg); padding: 20px; }
        .container { max-width: 1100px; margin: auto; background: white; padding: 25px; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 20px; }
        .card { border: 1px solid #eee; padding: 15px; border-radius: 8px; background: #fff; }
        input, button { width: 100%; padding: 12px; margin: 8px 0; border-radius: 6px; border: 1px solid #ddd; box-sizing: border-box; }
        button { cursor: pointer; font-weight: bold; color: white; border: none; transition: 0.3s; }
        .btn-in { background: var(--success); }
        .btn-out { background: var(--danger); }
        table { width: 100%; border-collapse: collapse; margin-top: 15px; font-size: 0.9em; }
        th, td { padding: 10px; border: 1px solid #eee; text-align: left; }
        th { background: var(--primary); color: white; }
        .historico-container { max-height: 250px; overflow-y: auto; background: #222; color: #0f0; padding: 10px; font-family: monospace; border-radius: 5px; }
        .badge-baixo { background: #ffdada; color: #c0392b; padding: 2px 5px; border-radius: 4px; font-weight: bold; }
    </style>
</head>
<body>

<div class="container">
    <h2>📦 Gestão de Estoque & Play Histórico</h2>

    <div class="grid">
        <div class="card">
            <h3>📥 Entrada de NF</h3>
            <input type="text" id="sku_in" placeholder="Código SKU (Ex: 001)">
            <input type="number" id="qtd_in" placeholder="Quantidade">
            <button class="btn-in" onclick="movimentar('ENTRADA')">REGISTRAR ENTRADA</button>
        </div>

        <div class="card">
            <h3>🛒 Saída PDV</h3>
            <input type="text" id="sku_out" placeholder="Código SKU">
            <input type="number" id="qtd_out" placeholder="Quantidade">
            <button class="btn-out" onclick="movimentar('SAIDA')">REGISTRAR VENDA</button>
        </div>
    </div>

    <h3>📊 Saldo Atualizado</h3>
    <table>
        <thead>
            <tr>
                <th>SKU</th>
                <th>Produto</th>
                <th>Saldo</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody id="corpoTabela"></tbody>
    </table>

    <h3>📜 Play Histórico (Auditoria)</h3>
    <div class="historico-container" id="playHistorico">
        </div>
    <button style="background:#7f8c8d; margin-top:10px;" onclick="limparTudo()">Limpar Banco de Dados</button>
</div>

<script>
    // Banco de Dados persistente no LocalStorage
    let db = JSON.parse(localStorage.getItem('estoqueDB')) || {
        "001": { nome: "Mouse Gamer", saldo: 0, min: 5 },
        "002": { nome: "Teclado Mecânico", saldo: 0, min: 3 }
    };
    
    let logs = JSON.parse(localStorage.getItem('logsDB')) || [];

    function salvar() {
        localStorage.setItem('estoqueDB', JSON.stringify(db));
        localStorage.setItem('logsDB', JSON.stringify(logs));
        renderizar();
    }

    function movimentar(tipo) {
        let sku = tipo === 'ENTRADA' ? document.getElementById('sku_in').value : document.getElementById('sku_out').value;
        let qtd = parseInt(tipo === 'ENTRADA' ? document.getElementById('qtd_in').value : document.getElementById('qtd_out').value);

        if (!db[sku] || isNaN(qtd)) return alert("Dados inválidos ou SKU não cadastrado!");

        if (tipo === 'ENTRADA') {
            db[sku].saldo += qtd;
        } else {
            if (db[sku].saldo < qtd) return alert("Estoque insuficiente!");
            db[sku].saldo -= qtd;
        }

        // Adiciona ao Log de Auditoria
        let data = new Date().toLocaleString();
        logs.unshift(`[${data}] ${tipo}: ${qtd} un do SKU ${sku} (${db[sku].nome})`);
        
        salvar();
    }

    function renderizar() {
        // Renderiza Tabela
        const tbody = document.getElementById('corpoTabela');
        tbody.innerHTML = '';
        for (let sku in db) {
            let item = db[sku];
            let status = item.saldo <= item.min ? '<span class="badge-baixo">REPOR</span>' : '✅ OK';
            tbody.innerHTML += `<tr><td>${sku}</td><td>${item.nome}</td><td>${item.saldo}</td><td>${status}</td></tr>`;
        }

        // Renderiza Play Histórico
        const logBox = document.getElementById('playHistorico');
        logBox.innerHTML = logs.map(l => `<div>> ${l}</div>`).join('');
    }

    function limparTudo() {
        if(confirm("Deseja apagar todo o histórico e saldos?")) {
            localStorage.clear();
            location.reload();
        }
    }

    renderizar();
</script>

</body>
</html>
