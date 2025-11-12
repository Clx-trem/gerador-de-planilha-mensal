<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Gerador de Planilha Mensal</title>
  <style>
    body{font-family:Inter, system-ui, Arial; max-width:980px; margin:28px auto; padding:12px;}
    h1{font-size:20px; margin-bottom:6px;}
    label{display:block; margin:8px 0 4px;}
    .row{display:flex; gap:12px; align-items:center; flex-wrap:wrap;}
    .months{display:grid; grid-template-columns:repeat(4, 1fr); gap:6px; margin-top:6px;}
    .box{padding:10px; border-radius:8px; background:#f7f7f7; border:1px solid #e1e1e1}
    button{padding:10px 14px; border-radius:8px; border:0; background:#0b74de; color:white; cursor:pointer;}
    input[type="number"]{width:110px; padding:8px; border-radius:6px; border:1px solid #ccc;}
    .small{font-size:13px; color:#444}
    footer{margin-top:18px; font-size:13px; color:#666}
    .opt-inline{display:flex; gap:8px; align-items:center;}
    .preview{margin-top:14px;}
    .example{font-size:13px; color:#333}
    .hint{font-size:13px; color:#666; margin-top:6px;}
  </style>
  <!-- SheetJS (xlsx) -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
</head>
<body>
  <h1>Gerador de Planilha Mensal</h1>
  <p class="small">Escolha os meses e o ano — o site vai gerar uma planilha `.xlsx` com uma aba por mês. Os dias terão o nome do dia da semana correto.</p>

  <div class="box">
    <label for="year">Ano</label>
    <input id="year" type="number" min="1900" max="2100" value="2025">
    <label style="margin-top:10px">Selecione os meses</label>
    <div class="months" id="monthsContainer"></div>
    <label style="margin-top:8px">Formato dos dias da semana</label>
    <div class="row">
      <div class="opt-inline">
        <input type="radio" name="locale" id="pt" value="pt" checked> <label for="pt" style="margin:0">Seg — Dom (pt-BR)</label>
      </div>
      <div class="opt-inline">
        <input type="radio" name="locale" id="en" value="en"> <label for="en" style="margin:0">Mon — Sun (en)</label>
      </div>
    </div>
    <label style="margin-top:10px">Opções</label>
    <div class="row">
      <div><input type="checkbox" id="oneSheetPerMonth" checked> <label for="oneSheetPerMonth" style="margin:0">Uma aba por mês</label></div>
      <div><input type="checkbox" id="includeWeekNumbers"> <label for="includeWeekNumbers" style="margin:0">Incluir coluna Nº semana</label></div>
    </div>
    <label style="margin-top:10px">Nome do arquivo</label>
    <input id="filename" type="text" value="planejamento_{{year}}" style="width:260px; padding:8px; border-radius:6px; border:1px solid #ccc;">
    <div style="margin-top:14px;">
      <button id="generateBtn">Gerar planilha (.xlsx)</button>
      <button id="previewBtn" style="background:#2dbe60;">Visualizar Exemplo</button>
    </div>
    <div class="hint">
      <strong>Observação:</strong> o layout é gerado dinamicamente e exportado. Se quiser personalizar colunas (ex.: tarefas, meta, notas), eu adapto.
    </div>
  </div>

  <div class="preview" id="previewArea"></div>

  <footer>
    Desenvolvido — copia e cole o arquivo `index.html` no seu projeto. Usa apenas o navegador (client-side).
  </footer>

<script>
(function(){
  const monthNames = [
    'Janeiro','Fevereiro','Março','Abril','Maio','Junho',
    'Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'
  ];

  const monthsContainer = document.getElementById('monthsContainer');
  monthNames.forEach((m,i) => {
    const id = 'm' + i;
    const div = document.createElement('div');
    div.innerHTML = \`<label style="display:flex;gap:8px;align-items:center"><input type="checkbox" id="\${id}" data-index="\${i}"> \${m}</label>\`;
    monthsContainer.appendChild(div);
  });

  function getSelectedMonths(){
    const checks = monthsContainer.querySelectorAll('input[type=checkbox]');
    const arr = [];
    checks.forEach(c => { if (c.checked) arr.push(parseInt(c.dataset.index)); });
    return arr;
  }

  // Return array of {date, dowName, dowIndex}
  function daysInMonth(year, monthIndex, localeMode){
    const days = [];
    const first = new Date(year, monthIndex, 1);
    const lastDay = new Date(year, monthIndex + 1, 0).getDate();
    for(let d=1; d<=lastDay; d++){
      const dt = new Date(year, monthIndex, d);
      // in JS: 0=Sunday,1=Monday,...6=Saturday
      const jsDow = dt.getDay();
      let name;
      if(localeMode === 'pt'){
        // Map JS day to Portuguese names starting Mon..Sun
        const pt = ['Domingo','Segunda','Terça','Quarta','Quinta','Sexta','Sábado'];
        name = pt[jsDow];
      } else {
        const en = ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'];
        name = en[jsDow];
      }
      days.push({date: d, dowName: name, dowIndex: jsDow, iso: dt.toISOString().slice(0,10)});
    }
    return days;
  }

  function buildSheetForMonth(year, monthIndex, options){
    const localeMode = options.locale;
    const includeWeekNumbers = options.includeWeekNumbers;

    // Build a 2D array representing the sheet
    // Example layout:
    // Row0: [ MÊS NOME - centered across columns ]
    // Row1: [Colunas headers: (Semana?), Dia, Dia da semana, Campo1, Campo2 ...]
    // RowN: one row per date
    const rows = [];

    // Header row
    rows.push([ monthNames[monthIndex] + ' ' + year ]);
    // empty spacer
    rows.push([]);

    // Column headers
    const headers = [];
    if(includeWeekNumbers) headers.push('Semana #');
    headers.push('Data');
    headers.push('Dia da semana');
    headers.push('Tarefa / Atividade');
    headers.push('Observações');
    headers.push('Status');
    rows.push(headers);

    // day rows
    const days = daysInMonth(year, monthIndex, localeMode);
    // Optionally compute week number ISO-like (simple)
    days.forEach(d => {
      const row = [];
      if(includeWeekNumbers){
        // simple week number: number of week-of-year (Mon-Sun) - approximate using ISO week number
        const wk = isoWeekNumber(new Date(year, monthIndex, d.date));
        row.push(wk);
      }
      // date in dd/mm/yyyy
      const dd = String(d.date).padStart(2,'0');
      const mm = String(monthIndex+1).padStart(2,'0');
      row.push(dd + '/' + mm + '/' + year);
      row.push(d.dowName);
      row.push(''); // task
      row.push(''); // obs
      row.push(''); // status
      rows.push(row);
    });

    return rows;
  }

  // ISO week number helper
  function isoWeekNumber(dt){
    // Copy date so don't modify original
    const d = new Date(Date.UTC(dt.getFullYear(), dt.getMonth(), dt.getDate()));
    // Set to nearest Thursday: current date + 4 - current day number
    const dayNum = d.getUTCDay() || 7;
    d.setUTCDate(d.getUTCDate() + 4 - dayNum);
    // Year start
    const yearStart = new Date(Date.UTC(d.getUTCFullYear(),0,1));
    // Calculate full weeks to nearest Thursday
    const weekNo = Math.ceil( ( ( (d - yearStart) / 86400000) + 1 ) / 7 );
    return weekNo;
  }

  function generateWorkbook(selectedMonths, year, options){
    const wb = XLSX.utils.book_new();

    if(selectedMonths.length === 0){
      alert('Selecione pelo menos 1 mês.');
      return null;
    }

    selectedMonths.forEach(mi => {
      const data = buildSheetForMonth(year, mi, options);
      const ws = XLSX.utils.aoa_to_sheet(data);

      // Basic column widths
      const maxCols = data.reduce((m,r)=>Math.max(m, r.length), 0);
      const wscols = [];
      for(let c=0;c<maxCols;c++){
        // define widths: date narrow, dow narrow, task wide, obs wide, status narrow
        let w = 12;
        if(c>=3 && c<=4) w = 30;
        if(c===2) w = 18;
        wscols.push({wch: w});
      }
      ws['!cols'] = wscols;

      // Merge first row across all columns for month title
      ws['!merges'] = ws['!merges'] || [];
      // merge A1 to last column of row 1 (index 0)
      const lastCol = maxCols - 1;
      if(lastCol >= 0){
        ws['!merges'].push({s:{r:0,c:0}, e:{r:0,c:lastCol}});
        // center style for A1
        if(!ws.A1) ws.A1 = {t:'s', v: data[0][0] || ''};
        // set bold via cell z (note: SheetJS basic)
        ws.A1.s = {font:{bold:true, sz:14}, alignment:{horizontal:"center"}};
      }

      // add the sheet
      let sheetName = monthNames[mi].substring(0,31);
      XLSX.utils.book_append_sheet(wb, ws, sheetName);
    });

    return wb;
  }

  // UI actions
  document.getElementById('generateBtn').addEventListener('click', () => {
    const year = parseInt(document.getElementById('year').value) || (new Date()).getFullYear();
    const selected = getSelectedMonths();
    const locale = document.querySelector('input[name="locale"]:checked').value;
    const includeWeekNumbers = document.getElementById('includeWeekNumbers').checked;
    const filenameInput = document.getElementById('filename').value.trim();
    const filename = filenameInput ? filenameInput.replace('{{year}}', year) : 'planejamento_' + year;

    const wb = generateWorkbook(selected, year, {locale, includeWeekNumbers});
    if(!wb) return;
    XLSX.writeFile(wb, filename + '.xlsx');
  });

  // Preview: show a small HTML preview of the first selected month
  document.getElementById('previewBtn').addEventListener('click', () => {
    const selected = getSelectedMonths();
    if(selected.length === 0){
      alert('Selecione pelo menos 1 mês para a pré-visualização.');
      return;
    }
    const year = parseInt(document.getElementById('year').value) || (new Date()).getFullYear();
    const locale = document.querySelector('input[name="locale"]:checked').value;
    const includeWeekNumbers = document.getElementById('includeWeekNumbers').checked;
    const rows = buildSheetForMonth(year, selected[0], {locale, includeWeekNumbers});
    renderPreview(rows);
  });

  function renderPreview(rows){
    const preview = document.getElementById('previewArea');
    preview.innerHTML = '';
    const table = document.createElement('table');
    table.style.borderCollapse = 'collapse';
    table.style.width = '100%';
    table.style.marginTop = '10px';
    table.style.fontSize = '13px';
    rows.forEach((r,ri) => {
      const tr = document.createElement('tr');
      r.forEach((cell,ci) => {
        const td = document.createElement(ri < 2 ? 'th' : 'td');
        td.style.border = '1px solid #ddd';
        td.style.padding = '6px';
        td.textContent = cell === undefined ? '' : cell;
        if(ri === 0){
          td.colSpan = r.length;
          td.style.fontWeight = '700';
          td.style.textAlign = 'center';
          td.style.background = '#f0f6ff';
          tr.appendChild(td);
          // break out of the forEach (we already appended merged cell)
          return;
        }
        tr.appendChild(td);
      });
      table.appendChild(tr);
    });
    preview.appendChild(table);
    const ex = document.createElement('div');
    ex.className = 'example';
    ex.textContent = 'Exemplo da primeira aba (visualização). Use "Gerar planilha" para baixar o arquivo completo.';
    preview.appendChild(ex);
  }

  // initial default selection: current month
  const now = new Date();
  const def = now.getMonth();
  document.querySelector('#m' + def + ' input, #m' + def).checked;
  const defChk = document.querySelector('#m' + def);
  if(defChk) defChk.checked = true;

})();
</script>

</body>
</html>
