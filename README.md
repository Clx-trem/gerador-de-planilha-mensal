<!DOCTYPE html>
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
      transition: 0.3s;
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

    /* TEMA ESCURO */
    body.dark {
      background: #1e1e1e;
      color: #fff;
    }
    body.dark input,
    body.dark select,
    body.dark button {
      background: #333;
      color: #fff;
      border-color: #555;
    }
    body.dark .preview {
      background: #2a2a2a;
      border-color: #555;
    }
    body.dark table {
      color: #ffffff;
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
    <button id="salvarModeloBtn" style="background:#28a745;">Salvar Modelo</button>
    <button id="carregarModeloBtn" style="background:#ffc107; color:#000;">Carregar</button>
    <button id="duplicarModeloBtn" style="background:#6f42c1;">Duplicar</button>
    <button id="excluirModeloBtn" style="background:#dc3545;">Excluir</button>
    <button id="exportarBtn" style="background:#20c997;">⬇️ Exportar</button>
    <button id="importarBtn" style="background:#fd7e14;">⬆️ Importar</button>
    <input type="file" id="importFile" accept=".json" style="display:none">
  </div>

  <div class="modelos">
    <button id="modeloSimplesBtn" style="background:#0d6efd;">Modelo Simples</button>
    <button id="modeloProBtn" style="background:#6610f2;">Modelo Profissional</button>
  </div>

  <div>
    <button id="gerarBtn">Gerar XLSX</button>
    <button id="previewBtn" style="background:#28a745;">Ver Exemplo</button>
    <button id="gradearBtn" style="background:#6c757d;">Gradeie</button>
    <button id="bordasBtn" style="background:#343a40;">Bordas Excel</button>
    <button id="congelarBtn" style="background:#198754;">Congelar Cabeçalho</button>
    <button id="pdfBtn" style="background:#dc3545;">📄 Exportar PDF</button>
    <button id="temaBtn" style="background:#000;">🌙 Tema Escuro</button>
  </div>

  <div class="preview" id="preview"></div>

  <footer>✨ Criado por <strong>CLX</strong></footer>

  <script>
    const nomesMeses = [
      "Janeiro","Fevereiro","Março","Abril","Maio","Junho",
      "Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"
    ];

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
      input.placeholder = "Nova coluna";
      container.appendChild(input);
    });

    /* BOTÕES DE MODELO */
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
      const nome = prompt("Nome do modelo:");
      if (!nome) return;

      const colunas = pegarColunas();
      const modelos = obterModelos();
      modelos[nome] = colunas;
      salvarModelos(modelos);

      alert("Modelo salvo!");
    });

    document.getElementById("carregarModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value;
      if (!nome) return alert("Selecione um modelo");

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

      alert("Modelo carregado!");
    });

    document.getElementById("duplicarModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value;
      if (!nome) return alert("Escolha um modelo");

      const novo = prompt("Novo nome da cópia:");
      if (!novo) return;

      const modelos = obterModelos();
      modelos[novo] = [...modelos[nome]];
      salvarModelos(modelos);

      alert("Modelo duplicado!");
    });

    document.getElementById("excluirModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value;
      if (!nome) return alert("Selecione um modelo");

      if (!confirm(`Excluir modelo ${nome}?`)) return;

      const modelos = obterModelos();
      delete modelos[nome];
      salvarModelos(modelos);

      alert("Modelo excluído!");
    });

    document.getElementById("exportarBtn").addEventListener("click", () => {
      const modelos = obterModelos();
      const blob = new Blob([JSON.stringify(modelos, null, 2)], { type: "application/json" });

      const a = document.createElement("a");
      a.href = URL.createObjectURL(blob);
      a.download = "modelos_CLX.json";
      a.click();
    });

    const inputFile = document.getElementById("importFile");
    document.getElementById("importarBtn").addEventListener("click", () => inputFile.click());
    inputFile.addEventListener("change", e => {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (ev) => {
        try {
          const novos = JSON.parse(ev.target.result);
          const modelos = obterModelos();
          Object.assign(modelos, novos);
          salvarModelos(modelos);
          alert("Importado!");
        } catch {
          alert("Erro ao importar JSON");
        }
      };
      reader.readAsText(file);
    });

    /* MODELOS PADRÃO */
    document.getElementById("modeloSimplesBtn").addEventListener("click", () => {
      const container = document.getElementById("colunasContainer");
      container.innerHTML = "";

      ["Data", "Dia da Semana", "Descrição"].forEach(c => {
        const inp = document.createElement("input");
        inp.type = "text";
        inp.value = c;
        container.appendChild(inp);
      });

      alert("Modelo Simples carregado!");
    });

    document.getElementById("modeloProBtn").addEventListener("click", () => {
      const container = document.getElementById("colunasContainer");
      container.innerHTML = "";

      [
        "Data","Dia da Semana","Atividade","Responsável",
        "Setor","Status","Observações"
      ].forEach(c => {
        const inp = document.createElement("input");
        inp.type = "text";
        inp.value = c;
        container.appendChild(inp);
      });

      alert("Modelo Profissional carregado!");
    });

    /* AJUSTE COLUNAS */
    function autoAjustarColunas(ws, colunas) {
      ws['!cols'] = colunas.map(c => ({
        wch: Math.max(12, c.length + 2)
      }));
    }

    /* BORDAS */
    let aplicarBordas = false;

    document.getElementById("bordasBtn").addEventListener("click", () => {
      aplicarBordas = !aplicarBordas;
      alert(aplicarBordas ? "Bordas ativadas!" : "Bordas desativadas!");
    });

    function aplicarBordasNaPlanilha(ws) {
      const range = XLSX.utils.decode_range(ws['!ref']);
      for (let R = range.s.r; R <= range.e.r; R++) {
        for (let C = range.s.c; C <= range.e.c; C++) {
          const cell = XLSX.utils.encode_cell({ r: R, c: C });
          if (!ws[cell]) continue;

          ws[cell].s = {
            border: {
              top: { style: "thin", color: { rgb: "000000" } },
              bottom: { style: "thin", color: { rgb: "000000" } },
              left: { style: "thin", color: { rgb: "000000" } },
              right: { style: "thin", color: { rgb: "000000" } }
            }
          };
        }
      }
    }

    /* CONGELAR CABEÇALHO */
    let congelarCabecalho = false;

    document.getElementById("congelarBtn").addEventListener("click", () => {
      congelarCabecalho = !congelarCabecalho;
      alert(congelarCabecalho ? "Cabeçalho congelado!" : "Desativado");
    });

    function congelarLinha(ws) {
      ws['!freeze'] = { xSplit: 0, ySplit: 1 };
    }

    /* GERAR XLSX */
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

        autoAjustarColunas(ws, colunas);
        if (aplicarBordas) aplicarBordasNaPlanilha(ws);
        if (congelarCabecalho) congelarLinha(ws);

        XLSX.utils.book_append_sheet(wb, ws, nomesMeses[mes]);
      }

      XLSX.writeFile(wb, `planejamento_${ano}.xlsx`);
    }

    /* PDF */
    document.getElementById("pdfBtn").addEventListener("click", () => {
      const preview = document.getElementById("preview").innerHTML;

      if (!preview.trim()) return alert("Clique em Ver Exemplo primeiro!");

      const w = window.open("", "_blank");
      w.document.write(`
        <html>
          <head>
            <title>PDF - CLX</title>
            <style>
              table { width: 100%; border-collapse: collapse; }
              th, td { border: 1px solid #000; padding: 8px; }
              th { background: #eee; }
            </style>
          </head>
          <body>${preview}</body>
        </html>
      `);
      w.document.close();
      w.print();
    });
    /* PREVIEW */
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
      html += "</table><p><em>Mostrando primeiros 10 dias…</em></p>";
      preview.innerHTML = html;
    }
    document.getElementById("gradearBtn").addEventListener("click", () => {
      const tabela = document.querySelector("#preview table");
      if (!tabela) return alert("Clique em Ver Exemplo primeiro!");
      tabela.style.border = "2px solid black";
      tabela.querySelectorAll("td, th").forEach(c => {
        c.style.border = "1px solid black";
      });
      alert("Grade aplicada!");
    });
    /* TEMA ESCURO */
    let darkMode = false;
    document.getElementById("temaBtn").addEventListener("click", () => {
      darkMode = !darkMode;
      document.body.classList.toggle("dark", darkMode);
      document.getElementById("temaBtn").textContent =
        darkMode ? "☀️ Tema Claro" : "🌙 Tema Escuro";
    });
    /* BOTÕES PRINCIPAIS */
    document.getElementById("gerarBtn").addEventListener("click", () => {
      const ano = parseInt(document.getElementById("ano").value);
      const meses = mesesSelecionados();
      const qtdAbas = parseInt(document.getElementById("qtdAbas").value);
      if (meses.length === 0) return alert("Selecione um mês!");
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
