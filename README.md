<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gerador de Planilha Personalizada</title>

  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f8fafc;
      max-width: 900px;
      margin: 40px auto;
      padding: 20px;
    }
    h1 {
      text-align: center;
      margin-bottom: 20px;
    }
    label {
      display: block;
      margin-top: 10px;
      font-weight: bold;
    }
    input, select, button {
      padding: 8px;
      border-radius: 5px;
      border: 1px solid #ccc;
      margin-top: 5px;
    }
    button {
      cursor: pointer;
      background: #007bff;
      color: white;
      border: none;
      margin-top: 15px;
      margin-right: 10px;
    }
    button:hover {
      background: #0056b3;
    }
    .months {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
      gap: 5px;
    }
    .preview {
      margin-top: 20px;
      background: white;
      padding: 10px;
      border: 1px solid #ccc;
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
      background: #f1f5f9;
    }
    #colunasContainer input {
      display: block;
      width: 100%;
      margin-top: 5px;
    }
    .modelos {
      display: flex;
      align-items: center;
      gap: 10px;
      flex-wrap: wrap;
      margin-top: 10px;
    }
    footer {
      text-align: center;
      margin-top: 40px;
      font-size: 14px;
      color: #555;
    }
  </style>

  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
</head>

<body>
  <h1>📅 Gerador de Planilha Personalizada</h1>

  <label for="ano">Ano:</label>
  <input type="number" id="ano" min="2000" max="2100" value="2025">

  <label>Selecione os meses:</label>
  <div class="months" id="meses"></div>

  <label for="qtdAbas">Quantidade de abas:</label>
  <input type="number" id="qtdAbas" min="1" max="12" value="1">

  <label>Colunas da planilha:</label>
  <div id="colunasContainer">
    <input type="text" value="Data">
    <input type="text" value="Dia da Semana">
    <input type="text" value="Atividade">
    <input type="text" value="Observações">
  </div>

  <button id="addColunaBtn" style="background:#17a2b8;">+ Adicionar Coluna</button>

  <div class="modelos">
    <select id="modeloSelect">
      <option value="">-- Escolher modelo salvo --</option>
    </select>
    <button id="salvarModeloBtn" style="background:#28a745;">Salvar como Modelo</button>
    <button id="carregarModeloBtn" style="background:#ffc107; color:#000;">Carregar Modelo</button>
    <button id="duplicarModeloBtn" style="background:#6f42c1;">🌀 Duplicar Modelo</button>
    <button id="excluirModeloBtn" style="background:#dc3545;">Excluir Modelo</button>
    <button id="exportarBtn" style="background:#20c997;">⬇️ Exportar Modelos</button>
    <button id="importarBtn" style="background:#fd7e14;">⬆️ Importar Modelos</button>
    <input type="file" id="importFile" accept=".json" style="display:none">
  </div>

  <div>
    <button id="gerarBtn">Gerar Arquivo XLSX</button>
    <button id="previewBtn" style="background:#28a745;">Ver Exemplo</button>
  </div>

  <div class="preview" id="preview"></div>

  <footer>✨ Criado por <strong>CLX</strong></footer>

  <script>
    const nomesMeses = [
      "Janeiro","Fevereiro","Março","Abril","Maio","Junho",
      "Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"
    ];

    // Cria checkboxes dos meses
    const mesesDiv = document.getElementById("meses");
    nomesMeses.forEach((m, i) => {
      const lbl = document.createElement("label");
      lbl.innerHTML = `<input type="checkbox" value="${i}"> ${m}`;
      mesesDiv.appendChild(lbl);
    });

    function mesesSelecionados() {
      const checks = mesesDiv.querySelectorAll("input:checked");
      return Array.from(checks).map(c => parseInt(c.value));
    }

    function gerarDias(ano, mes) {
      const dias = [];
      const ultimoDia = new Date(ano, mes + 1, 0).getDate();
      for (let d = 1; d <= ultimoDia; d++) {
        const data = new Date(ano, mes, d);
        const nomesDias = ["Domingo","Segunda","Terça","Quarta","Quinta","Sexta","Sábado"];
        dias.push({
          data: `${d.toString().padStart(2,"0")}/${(mes+1).toString().padStart(2,"0")}/${ano}`,
          diaSemana: nomesDias[data.getDay()]
        });
      }
      return dias;
    }

    function pegarColunas() {
      const inputs = document.querySelectorAll("#colunasContainer input");
      return Array.from(inputs).map(i => i.value.trim()).filter(v => v !== "");
    }

    document.getElementById("addColunaBtn").addEventListener("click", () => {
      const container = document.getElementById("colunasContainer");
      const input = document.createElement("input");
      input.type = "text";
      input.placeholder = "Nome da nova coluna";
      container.appendChild(input);
    });

    function gerarPlanilha(ano, meses, qtdAbas) {
      const colunas = pegarColunas();
      const wb = XLSX.utils.book_new();

      for (let i = 0; i < Math.min(qtdAbas, meses.length); i++) {
        const mes = meses[i];
        const dias = gerarDias(ano, mes);
        const dados = [colunas];

        dias.forEach(d => {
          const linha = [];
          colunas.forEach(col => {
            if (col.toLowerCase().includes("data")) linha.push(d.data);
            else if (col.toLowerCase().includes("semana")) linha.push(d.diaSemana);
            else linha.push("");
          });
          dados.push(linha);
        });

        const ws = XLSX.utils.aoa_to_sheet(dados);
        XLSX.utils.book_append_sheet(wb, ws, nomesMeses[mes]);
      }

      XLSX.writeFile(wb, `planejamento_${ano}.xlsx`);
    }

    function mostrarExemplo(ano, meses) {
      const preview = document.getElementById("preview");
      preview.innerHTML = "";
      if (meses.length === 0) {
        preview.innerHTML = "<p style='color:red'>Selecione pelo menos um mês!</p>";
        return;
      }

      const colunas = pegarColunas();
      const mes = meses[0];
      const dias = gerarDias(ano, mes);

      let html = `<h3>${nomesMeses[mes]} ${ano}</h3><table><tr>`;
      colunas.forEach(c => html += `<th>${c}</th>`);
      html += `</tr>`;

      dias.slice(0, 10).forEach(d => {
        html += "<tr>";
        colunas.forEach(col => {
          if (col.toLowerCase().includes("data")) html += `<td>${d.data}</td>`;
          else if (col.toLowerCase().includes("semana")) html += `<td>${d.diaSemana}</td>`;
          else html += "<td></td>";
        });
        html += "</tr>";
      });
      html += "</table><p><em>Mostrando os 10 primeiros dias...</em></p>";
      preview.innerHTML = html;
    }

    const modeloSelect = document.getElementById("modeloSelect");

    function atualizarListaModelos() {
      modeloSelect.innerHTML = `<option value="">-- Escolher modelo salvo --</option>`;
      const modelos = JSON.parse(localStorage.getItem("modelosColunas") || "{}");
      for (const nome in modelos) {
        const opt = document.createElement("option");
        opt.value = nome;
        opt.textContent = nome;
        modeloSelect.appendChild(opt);
      }
    }

    atualizarListaModelos();

    function obterModelos() {
      return JSON.parse(localStorage.getItem("modelosColunas") || "{}");
    }

    function salvarModelos(modelos) {
      localStorage.setItem("modelosColunas", JSON.stringify(modelos));
      atualizarListaModelos();
    }

    document.getElementById("salvarModeloBtn").addEventListener("click", () => {
      const nome = prompt("Digite o nome do modelo:");
      if (!nome) return;
      const colunas = pegarColunas();
      const modelos = obterModelos();
      modelos[nome] = colunas;
      salvarModelos(modelos);
      alert(`Modelo "${nome}" salvo com sucesso!`);
    });

    document.getElementById("carregarModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value;
      if (!nome) return alert("Escolha um modelo para carregar!");
      const modelos = obterModelos();
      const colunas = modelos[nome];
      const container = document.getElementById("colunasContainer");
      container.innerHTML = "";
      colunas.forEach(c => {
        const input = document.createElement("input");
        input.type = "text";
        input.value = c;
        container.appendChild(input);
      });
      alert(`Modelo "${nome}" carregado!`);
    });

    document.getElementById("excluirModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value;
      if (!nome) return alert("Escolha um modelo para excluir!");
      if (!confirm(`Excluir modelo "${nome}"?`)) return;
      const modelos = obterModelos();
      delete modelos[nome];
      salvarModelos(modelos);
      alert("Modelo excluído!");
    });

    document.getElementById("duplicarModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value;
      if (!nome) return alert("Escolha um modelo para duplicar!");
      const modelos = obterModelos();
      const colunas = modelos[nome];
      const novoNome = prompt(`Digite o nome do novo modelo (cópia de "${nome}"):`);

      if (!novoNome) return;
      modelos[novoNome] = [...colunas];
      salvarModelos(modelos);
      alert(`Modelo "${novoNome}" criado com sucesso!`);
    });

    // Exportar modelos
    document.getElementById("exportarBtn").addEventListener("click", () => {
      const modelos = obterModelos();
      const blob = new Blob([JSON.stringify(modelos, null, 2)], { type: "application/json" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "modelos_CLX.json";
      a.click();
      URL.revokeObjectURL(url);
    });

    // Importar modelos
    const inputFile = document.getElementById("importFile");
    document.getElementById("importarBtn").addEventListener("click", () => inputFile.click());
    inputFile.addEventListener("change", (e) => {
      const file = e.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (ev) => {
        try {
          const novos = JSON.parse(ev.target.result);
          const modelos = obterModelos();
          Object.assign(modelos, novos);
          salvarModelos(modelos);
          alert("Modelos importados com sucesso!");
        } catch {
          alert("Erro ao importar o arquivo JSON!");
        }
      };
      reader.readAsText(file);
    });

    // Botões principais
    document.getElementById("gerarBtn").addEventListener("click", () => {
      const ano = parseInt(document.getElementById("ano").value);
      const meses = mesesSelecionados();
      const qtdAbas = parseInt(document.getElementById("qtdAbas").value);
      if (meses.length === 0) return alert("Selecione pelo menos um mês!");
      gerarPlanilha(ano, meses, qtdAbas);
    });

    document.getElementById("previewBtn").addEventListener("click", () => {
      const ano = parseInt(document.getElementById("ano").value);
      const meses = mesesSelecionados();
      mostrarExemplo(ano, meses);
    });
  </script>
</body>
</html>
