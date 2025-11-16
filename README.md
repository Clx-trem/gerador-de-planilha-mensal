<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Site Neon CLX</title>
  <style>
    body{
      font-family: Arial, sans-serif;
      background: #03040a;
      color: #fff;
      margin:0;
      padding:0;
      overflow-x:hidden;
    }
    /* efeito neon global */
    .neon{
      text-shadow: 0 0 5px #00eaff, 0 0 10px #00eaff, 0 0 20px #00eaff;
    }
    /* animação fade */
    @keyframes fadeIn{
      from{opacity:0; transform:translateY(10px);} to{opacity:1; transform:translateY(0);}
    }
    #loginBox{
      height:100vh;
      display:flex;
      justify-content:center;
      align-items:center;
      animation:fadeIn .4s;
    }
    .card{
      background:#0b0f19;
      padding:30px;
      width:350px;
      border-radius:12px;
      box-shadow:0 0 20px #00aaff88;
      animation:fadeIn .6s;
    }
    input,button{
      width:100%;
      padding:12px;
      border:none;
      border-radius:6px;
      margin-top:10px;
      font-size:16px;
      transition:.2s;
    }
    input{
      background:#05060c;
      color:#fff;
      border:1px solid #333;
    }
    input:focus{
      box-shadow:0 0 10px #00eaff;
      outline:none;
    }
    button{
      background:#00eaff;
      color:#000;
      font-weight:bold;
      cursor:pointer;
    }
    button:hover{
      box-shadow:0 0 15px #00eaff;
      transform:scale(1.03);
    }
    #siteArea{display:none;padding:20px;animation:fadeIn .5s;}
    #gear{
      position:fixed;
      top:20px;
      right:20px;
      font-size:32px;
      cursor:pointer;
      display:none;
      filter:drop-shadow(0 0 10px #00eaff);
      transition:.2s;
    }
    #gear:hover{transform:rotate(20deg) scale(1.1);} 
    #adminPanel{
      display:none;
      background:#0b0f19;
      padding:20px;
      border-radius:10px;
      width:350px;
      position:fixed;
      right:20px;
      top:70px;
      box-shadow:0 0 20px #00aaffaa;
      animation:fadeIn .3s;
    }
    .blocked{
      opacity:0.4;
      text-decoration: line-through;
      color:#ff5555;
    }
    .btnSmall{
      padding:4px 10px;
      border-radius:4px;
      font-size:12px;
      border:none;
      cursor:pointer;
    }
    .danger{background:#ff4444;color:white;}
    .warning{background:#ffaa00;color:black;}
    .ok{background:#00eaff;color:black;}
  </style>
</head>
<body>

<!-- LOGIN -->
<div id="loginBox">
  <div class="card">
    <h2 class="neon">Acesso</h2>
    <input id="usuario" placeholder="Usuário" />
    <input id="senha" type="password" placeholder="Senha" />
    <button onclick="login()">Entrar</button>
  </div>
</div>

<!-- ENGRANAGEM ADMIN -->
<div id="gear" onclick="abrirPainel()">⚙️</div>

<!-- PAINEL ADMIN -->
<div id="adminPanel">
  <h3 class="neon">Gerenciar Logins</h3>

  <input id="newUser" placeholder="Novo usuário">
  <input id="newPass" placeholder="Nova senha">
  <button class="ok" onclick="addUser()">Adicionar</button>

  <h4 class="neon">Usuários:</h4>
  <ul id="listaUsers"></ul>

  <hr style="border:1px solid #333;margin:15px 0;">

  <h4 class="neon">Trocar minha senha:</h4>
  <input id="newMyPass" placeholder="Nova senha">
  <button class="warning" onclick="trocarMinhaSenha()">Trocar senha</button>

  <button style="background:#ff4444;color:white;margin-top:20px;width:100%;" onclick="fecharPainel()">Fechar</button>
</div>

<!-- CONTEÚDO DO SITE -->
<div id="siteArea">
  <h1 class="neon">Bem-vindo ao site Neon!</h1>
  <p>Seu conteúdo real fica aqui.</p>
</div>

<script>
  let users = [
    {user:"admin", pass:"1234", admin:true, blocked:false}
  ];

  function salvar(){ localStorage.setItem("usersCLX", JSON.stringify(users)); }
  function carregar(){ const d = localStorage.getItem("usersCLX"); if(d) users = JSON.parse(d); }
  carregar();

  let loggedUser = null;

  function login(){
    const u = usuario.value.trim();
    const s = senha.value.trim();

    const found = users.find(x => x.user === u && x.pass === s);

    if(!found) return alert("Login incorreto!");
    if(found.blocked) return alert("Este usuário está bloqueado pelo administrador.");

    loggedUser = found;

    loginBox.style.display = "none";
    siteArea.style.display = "block";

    if(found.admin) gear.style.display = "block";
  }

  function abrirPainel(){ adminPanel.style.display = "block"; renderUsers(); }
  function fecharPainel(){ adminPanel.style.display = "none"; }

  function addUser(){
    const u = newUser.value.trim();
    const p = newPass.value.trim();
    if(!u || !p) return alert("Preencha tudo!");

    users.push({user:u, pass:p, admin:false, blocked:false});
    salvar();
    renderUsers();
    newUser.value="";
    newPass.value="";
  }

  function remover(i){ users.splice(i,1); salvar(); renderUsers(); }

  function bloquear(i){ users[i].blocked = !users[i].blocked; salvar(); renderUsers(); }

  function trocarMinhaSenha(){
    const nova = newMyPass.value.trim();
    if(!nova) return alert("Digite uma nova senha!");

    loggedUser.pass = nova;
    salvar();
    newMyPass.value="";
    alert("Senha alterada com sucesso!");
  }

  function renderUsers(){
    listaUsers.innerHTML = "";

    users.forEach((u,i)=>{
      let cls = u.blocked ? "blocked" : "";

      listaUsers.innerHTML += `
        <li class="${cls}">
          ${u.user}
          <button class='btnSmall danger' onclick='remover(${i})'>Remover</button>
          <button class='btnSmall warning' onclick='bloquear(${i})'>${u.blocked ? "Desbloquear" : "Bloquear"}</button>
        </li>`;
    });
  }
</script>

  <!-- BOTÃO ENGRENAGEM (ADMIN) -->
  <div id="adminGear" style="position: fixed; bottom: 20px; right: 20px; cursor: pointer; display:none; z-index:9999;">
    ⚙️
  </div>

  <script>
    // Mostrar engrenagem só para admin CLX
    const savedUser = localStorage.getItem('logado');
    if (savedUser === 'CLX') {
      document.getElementById('adminGear').style.display = 'block';
    }

    // Abrir painel admin ao clicar
    document.getElementById('adminGear').onclick = () => {
      alert('Painel do administrador — aqui você vai gerenciar usuários.');
    };
  </script>
  <!-- PAINEL ADMIN (JANELA) -->
  <div id="adminPanel" style="position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.7); backdrop-filter: blur(6px); display:none; justify-content:center; align-items:center; z-index:99999;">
    <div style="width:420px; background:#0d0d0f; padding:20px; border-radius:20px; box-shadow:0 0 20px #00eaff; color:white; font-family:Arial; animation: pop 0.3s ease;">
      <h2 style="text-align:center; color:#00eaff;">⚙️ Painel Administrativo</h2>
      <h3>Usuários cadastrados</h3>
      <div id="userList" style="max-height:200px; overflow-y:auto; padding:10px; border:1px solid #00eaff; border-radius:10px; margin-bottom:20px;"></div>
      <h3>Adicionar Usuário</h3>
      <input id="newUser" placeholder="Usuário" style="width:100%; padding:8px; border-radius:8px; margin-bottom:8px;">
      <input id="newPass" placeholder="Senha" style="width:100%; padding:8px; border-radius:8px; margin-bottom:8px;">
      <button onclick="addUser()" style="width:100%; padding:10px; background:#00eaff; color:black; border:none; border-radius:10px; font-weight:bold; cursor:pointer;">Adicionar</button>
      <button onclick="closePanel()" style="width:100%; padding:10px; background:#ff0044; color:white; border:none; border-radius:10px; font-weight:bold; cursor:pointer; margin-top:15px;">Fechar</button>
    </div>
  </div>

  <style>
    @keyframes pop{
      from{transform:scale(0.6); opacity:0;} to{transform:scale(1); opacity:1;}
    }
    body { background: #ffffff !important; color: black !important; }
  </style>

  <script>
    let usuarios = JSON.parse(localStorage.getItem('usuariosCLX') || '{}');

    function salvarUsuarios(){ localStorage.setItem('usuariosCLX', JSON.stringify(usuarios)); }

    function renderUsers(){
      const box = document.getElementById('userList');
      box.innerHTML = '';
      Object.keys(usuarios).forEach(u => {
        const item = document.createElement('div');
        item.style.marginBottom = '10px';
        item.innerHTML = `
          <div style="padding:10px; border-radius:10px; background:#001f29; border:1px solid #00eaff; display:flex; justify-content:space-between; align-items:center;">
            <strong>${u}</strong>
            <div style="display:flex; gap:10px;">
              <button onclick="editPass('${u}')" style="cursor:pointer;">✏️</button>
              <button onclick="toggleBlock('${u}')" style="cursor:pointer;">${usuarios[u].bloqueado ? '🔓' : '🔒'}</button>
              <button onclick="deleteUser('${u}')" style="cursor:pointer;">❌</button>
            </div>
          </div>`;
        box.appendChild(item);
      });
    }

    function addUser(){
      const u = document.getElementById('newUser').value.trim();
      const p = document.getElementById('newPass').value.trim();
      if(!u || !p) return alert('Usuário e senha necessários');
      usuarios[u] = { senha:p, bloqueado:false };
      salvarUsuarios(); renderUsers();
      document.getElementById('newUser').value = '';
      document.getElementById('newPass').value = '';
    }

    function editPass(user){
      const nova = prompt('Nova senha para ' + user);
      if(!nova) return;
      usuarios[user].senha = nova;
      salvarUsuarios(); renderUsers();
    }

    function toggleBlock(user){
      usuarios[user].bloqueado = !usuarios[user].bloqueado;
      salvarUsuarios(); renderUsers();
    }

    function deleteUser(user){
      if(!confirm('Excluir usuário ' + user + '?')) return;
      delete usuarios[user]; salvarUsuarios(); renderUsers();
    }

    document.getElementById('adminGear').onclick = () => {
      document.getElementById('adminPanel').style.display = 'flex';
      renderUsers();
    };

    function closePanel(){ document.getElementById('adminPanel').style.display = 'none'; }

  </script>
</body>
</html>
