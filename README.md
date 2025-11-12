<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Gerador de Planilha Mensal</title>

  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 900px;
      margin: 30px auto;
      padding: 20px;
      background: #f8fafc;
    }
    h1 {
      text-align: center;
      margin-bottom: 20px;
    }
    label {
      display: block;
      margin: 10px 0 5px;
    }
    input, select, button {
      padding: 8px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      background-color: #007bff;
      color: white;
      cursor: pointer;
      border: none;
      margin-right: 10px;
    }
    button:hover {
      background-color: #0056b3;
    }
    .months {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 5px;
    }
    .preview {
      margin-top: 20px;
      background: white;
      border: 1px solid #ccc;
      padding: 10px;
      border-radius: 8px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 6px;
      font-size: 14px;
      text-align: left;
    }
    th {
      background-color: #f1f5f9;
    }
  </style>

  <!-- Biblioteca XLSX -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.19.3/dist/xlsx.full.min.js"></script>
</head>
<body>

<h1>📅 Gerador de Planilha Mensal</h1>

<label for="ano">Ano:</label>
<input type="number" id="ano" min="2000" max="2100" value="2025">

<label>Selecione os meses:</label>
<div class="months" id="meses"></div>

<label for="qtdAbas">Quantidade de abas:</label>
<input type="number" id="qtdAbas" min="1" max="12" value="1">

<br><br>
<button id="gerarBtn">Gerar Arquivo XLSX</button>
<button id="previewBtn" style="background-color:#28a745;">Ver Exemplo</button>

<div class="preview" id="preview"></div>

<script>
  const nomesMeses = [
    "Janeiro","Fevereiro","Março","Abril","Maio","Junho",
    "Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"
  ];

  // Gerar checkboxes dos meses
  const mesesDiv = document.getElementById("meses");
  nomesMeses.forEach((m, i) => {
    const lbl = document.createElement("label");
    lbl.innerHTML = `<input type="checkbox" value="${i}"> ${m}`;
    mesesDiv.appendChild(lbl);
  });

  // Função para pegar meses selecionados
  function mesesSelecionados() {
    const checks = mesesDiv.querySelectorAll("input:checked");
    return Array.from(checks).map(c => parseInt(c.value));
  }

  // Gerar dias de um mês
  function gerarDias(ano, mes) {
    const dias = [];
    const ultimoDia = new Date(ano, mes + 1, 0).getDate();
    for (let d = 1; d <= ultimoDia; d++) {
      const data = new Date(ano, mes, d);
      const diasSemana = ["Domingo","Segunda","Terça","Quarta","Quinta","Sexta","Sábado"];
      dias.push({
        data: `${d.toString().padStart(2,"0")}/${(mes+1).toString().padStart(2,"0")}/${ano}`,
        diaSemana: diasSemana[data.getDay()]
      });
    }
    return dias;
  }

  // Montar a planilha
  function gerarPlanilha(ano, meses, qtdAbas) {
    const wb = XLSX.utils.book_new();

    for (let i = 0; i < Math.min(qtdAbas, meses.length); i++) {
      const mes = meses[i];
      const dias = gerarDias(ano, mes);
      const dados = [["Data","Dia da Semana","Atividade","Observações"]];
      dias.forEach(d => dados.push([d.data, d.diaSemana,"",""]));
      const ws = XLSX.utils.aoa_to_sheet(dados);
      XLSX.utils.book_append_sheet(wb, ws, nomesMeses[mes]);
    }

    XLSX.writeFile(wb, `planejamento_${ano}.xlsx`);
  }

  // Mostrar exemplo
  function mostrarExemplo(ano, meses, qtdAbas) {
    const preview = document.getElementById("preview");
    preview.innerHTML = "";
    if (meses.length === 0) {
      preview.innerHTML = "<p style='color:red'>Selecione pelo menos um mês!</p>";
      return;
    }
    const mes = meses[0];
    const dias = gerarDias(ano, mes);
    let html = `<h3>${nomesMeses[mes]} ${ano}</h3><table><tr><th>Data</th><th>Dia da Semana</th><th>Atividade</th><th>Observações</th></tr>`;
    dias.slice(0, 10).forEach(d => {
      html += `<tr><td>${d.data}</td><td>${d.diaSemana}</td><td></td><td></td></tr>`;
    });
    html += "</table><p><em>Exibindo os 10 primeiros dias...</em></p>";
    preview.innerHTML = html;
  }

  // Botões
  document.getElementById("gerarBtn").addEventListener("click", () => {
    const ano = parseInt(document.getElementById("ano").value);
    const meses = mesesSelecionados();
    const qtdAbas = parseInt(document.getElementById("qtdAbas").value);
    if (meses.length === 0) {
      alert("Selecione ao menos um mês!");
      return;
    }
    gerarPlanilha(ano, meses, qtdAbas);
  });

  document.getElementById("previewBtn").addEventListener("click", () => {
    const ano = parseInt(document.getElementById("ano").value);
    const meses = mesesSelecionados();
    const qtdAbas = parseInt(document.getElementById("qtdAbas").value);
    mostrarExemplo(ano, meses, qtdAbas);
  });
</script>

</body>
</html>
