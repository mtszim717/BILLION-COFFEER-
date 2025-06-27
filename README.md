<!DOCTYPE html><html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Coffeer Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { margin: 0; font-family: Arial, sans-serif; background-color: #000; color: #fff; }
        #login-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: radial-gradient(circle at top, #1c1c1c, #000); display: flex; justify-content: center; align-items: center; }
        #login-box { background: #111; padding: 30px; border-radius: 12px; box-shadow: 0 0 20px gold; text-align: center; color: gold; }
        #login-box input { width: 220px; padding: 10px; margin: 10px 0; border: 1px solid gold; border-radius: 6px; background: #222; color: gold; text-align: center; }
        #login-box button { padding: 10px 25px; border: none; border-radius: 6px; background: gold; color: black; cursor: pointer; font-weight: bold; }
        #login-box button:hover { background: #FFD700; }
        .sidebar { position: fixed; top: 0; left: 0; width: 140px; height: 100%; background-color: #111; padding-top: 10px; }
        .sidebar h2 { color: gold; text-align: center; font-size: 16px; }
        .sidebar a { display: block; padding: 8px; color: white; text-decoration: none; font-size: 13px; }
        .sidebar a:hover, .sidebar a.active { background-color: #333; }
        .content { margin-left: 150px; padding: 15px; }
        table, th, td { border: 1px solid gold; border-collapse: collapse; padding: 6px; text-align: center; }
        input, textarea { background: #222; color: #00FFC8; border: none; text-align: center; }
        canvas { background: #222; margin-bottom: 10px; }
        .card { background: #222; padding: 10px; margin-bottom: 10px; border-radius: 8px; }
        #excel-table input { width: 80px; background: #111; color: yellow; text-align: center; border: 1px solid gold; }
        .metrics { display: flex; flex-wrap: wrap; gap: 10px; }
        .metric-card { flex: 1; min-width: 120px; background: #111; color: white; padding: 10px; border-radius: 8px; box-shadow: 0 0 10px #FFD700; text-align: center; }
    </style>
</head>
<body>
<div id="login-screen">
    <div id="login-box">
        <h2>Login Coffeer</h2>
        <input type="text" id="username" placeholder="Usuário"><br>
        <input type="password" id="password" placeholder="Senha"><br>
        <button onclick="login()">ENTRAR</button>
        <p id="login-msg" style="color:red; margin-top:10px;"></p>
    </div>
</div>
<div class="sidebar" style="display:none;">
    <h2>Coffeer</h2>
    <a href="#" class="active" onclick="showTab('planilha', this)">Planilha</a>
    <a href="#" onclick="showTab('graficos', this)">Gráficos</a>
    <a href="#" onclick="showTab('ia', this)">IA Agro</a>
    <a href="#" onclick="showTab('relatorio', this)">Relatórios</a>
    <a href="#" onclick="showTab('imagens', this)">Imagens</a>
    <a href="#" onclick="showTab('excel', this)">Excel.CF</a>
</div>
<div class="content" id="content-area"></div>
<script>
const campos = ['caixa', 'estoque', 'cotacao', 'producao', 'custo'];
function login() {
    const user = document.getElementById('username').value;
    const pass = document.getElementById('password').value;
    if(user === 'coffeer' && pass === '123456') {
        document.getElementById('login-screen').style.display = 'none';
        document.querySelector('.sidebar').style.display = 'block';
        showTab('planilha', document.querySelector('.sidebar a.active'));
    } else {
        document.getElementById('login-msg').innerText = 'Usuário ou senha incorretos!';
    }
}
function saveData() {
    campos.forEach(c => localStorage.setItem(c, document.getElementById(c).value));
}
function loadData() {
    campos.forEach(c => {
        let val = localStorage.getItem(c);
        if (val !== null) document.getElementById(c).value = val;
    });
}
function showTab(tab, element) {
    document.querySelectorAll('.sidebar a').forEach(a => a.classList.remove('active'));
    element.classList.add('active');
    let content = document.getElementById('content-area');
    if (tab === 'planilha') {
        content.innerHTML = `
            <h1>Planilha</h1>
            <table>
                <tr><th>Caixa</th><th>Estoque</th><th>Cotação</th><th>Produção</th><th>Custo</th></tr>
                <tr>
                    <td><input id="caixa" onchange="saveData()"></td>
                    <td><input id="estoque" onchange="saveData()"></td>
                    <td><input id="cotacao" onchange="saveData()"></td>
                    <td><input id="producao" onchange="saveData()"></td>
                    <td><input id="custo" onchange="saveData()"></td>
                </tr>
            </table>
        `;
        loadData();
    }
    else if (tab === 'graficos') {
        const caixa = parseFloat(localStorage.getItem('caixa')) || 0;
        const estoque = parseFloat(localStorage.getItem('estoque')) || 0;
        const cotacao = parseFloat(localStorage.getItem('cotacao')) || 0;
        const producao = parseFloat(localStorage.getItem('producao')) || 0;
        const custo = parseFloat(localStorage.getItem('custo')) || 0;
        const vendas = producao * cotacao;
        const lucro = vendas - custo;
        content.innerHTML = `
            <h1>Métricas de Faturamento</h1>
            <div class="metrics">
                <div class="metric-card">Faturamento<br><b>R$ ${vendas.toFixed(2)}</b></div>
                <div class="metric-card">Produção<br><b>${producao}</b></div>
                <div class="metric-card">Cotação<br><b>R$ ${cotacao.toFixed(2)}</b></div>
                <div class="metric-card">Custo<br><b>R$ ${custo.toFixed(2)}</b></div>
                <div class="metric-card">Lucro<br><b>R$ ${lucro.toFixed(2)}</b></div>
            </div>
            <canvas id="chart" width="600" height="300"></canvas>
        `;
        const ctx = document.getElementById('chart').getContext('2d');
        new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['Caixa', 'Estoque', 'Produção', 'Cotação', 'Custo', 'Lucro'],
                datasets: [{
                    label: 'Indicadores',
                    data: [caixa, estoque, producao, cotacao, custo, lucro],
                    backgroundColor: ['#FFD700','#00FFC8','#FF6384','#36A2EB','#FFCE56','#00FF00']
                }]
            }
        });
    }
    else if (tab === 'ia') {
        content.innerHTML = `
            <h1>IA Agro</h1>
            <div id="chat-box" style="height:200px;overflow-y:auto;border:1px solid gold;margin-bottom:10px;padding:5px;"></div>
            <input type="text" id="user-question" placeholder="Pergunte algo..." style="width:70%;">
            <button onclick="responderIA()">Perguntar</button>
        `;
    }
    else if (tab === 'relatorio') {
        content.innerHTML = `
            <h1>Gerador de Relatórios</h1>
            <textarea id="relatorioPergunta" placeholder="Digite o que deseja no relatório" style="width:300px;height:100px;"></textarea>
            <br><button onclick="gerarRelatorio()">Gerar Relatório</button>
            <div class="card" id="relatorioResultado"></div>
        `;
    }
    else if (tab === 'imagens') {
        content.innerHTML = `
            <h1>Gerador de Imagens</h1>
            <input type="text" id="imagemTexto" placeholder="Descreva a imagem desejada" style="width:70%;">
            <button onclick="gerarImagem()">Gerar</button>
            <div id="imagemResultado"></div>
        `;
    }
    else if (tab === 'excel') {
        content.innerHTML = `
            <h1>Excel.CF</h1>
            <table id="excel-table">
                <tr style="color:yellow;"><th>Nome do Trabalhador</th><th>Salário Base</th><th>Horas Mês</th><th>% Hora Extra</th><th>Horas Extras</th></tr>
                <tr><td><input></td><td><input></td><td><input></td><td><input></td><td><input></td></tr>
                <tr><td><input></td><td><input></td><td><input></td><td><input></td><td><input></td></tr>
                <tr><td><input></td><td><input></td><td><input></td><td><input></td><td><input></td></tr>
                <tr><td><input></td><td><input></td><td><input></td><td><input></td><td><input></td></tr>
                <tr><td><input></td><td><input></td><td><input></td><td><input></td><td><input></td></tr>
            </table>
        `;
    }
}
function responderIA() {
    let box = document.getElementById('chat-box');
    let pergunta = document.getElementById('user-question').value.toLowerCase();
    let resposta = '';
    if (pergunta.includes('oi') || pergunta.includes('olá')) resposta = 'Olá! Como posso ajudar no Agro hoje?';
    else if (pergunta.includes('café')) resposta = 'O Brasil é líder mundial na produção de café, sendo o Arábica o mais cultivado.';
    else resposta = 'Pergunta muito boa! Sobre qual cultura ou tema agropecuário você quer saber mais?';
    box.innerHTML += `<p><b>Você:</b> ${pergunta}</p><p><b>IA:</b> ${resposta}</p>`;
    document.getElementById('user-question').value = '';
    box.scrollTop = box.scrollHeight;
}
function gerarRelatorio() {
    let txt = document.getElementById('relatorioPergunta').value;
    document.getElementById('relatorioResultado').innerText = 'Relatório Gerado:\n' + txt + '\n(Dados reais serão integrados futuramente)';
}
function gerarImagem() {
    let desc = document.getElementById('imagemTexto').value;
    document.getElementById('imagemResultado').innerHTML = '<p>Imagem buscada para: '+desc+'</p><img src="https://source.unsplash.com/300x200/?'+encodeURIComponent(desc)+'" alt="Imagem gerada">';
}
</script>
</body>
</html>
