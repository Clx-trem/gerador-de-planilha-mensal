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

  /* Neon global */
  .neon{
    text-shadow: 0 0 5px #00eaff, 0 0 10px #00eaff, 0 0 20px #00eaff;
  }

  @keyframes fadeIn{
    from{opacity:0; transform:translateY(10px);}
    to{opacity:1; transform:translateY(0);}
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

  #siteArea{
    display:none;
    padding:20px;
    animation:fadeIn .5s;
  }

  /* engrenagem */
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

  /* painel admin */
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
    z-index:9999;
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

  /* botão sair */
  #logoutBtn{
    position:fixed;
    left:20px;
    top:20px;
    background:#ff0044;
    color:white;
    padding:10px 15px;
    border-radius:8px;
    cursor:pointer;
    font-weight:bold;
    display:none;
    z-index:9999;
  }

  /* Foto padrão */
  #perfilFoto{
    width:90px;
    height:90px;
    border-radius:50%;
    background:#111;
    border:2px solid #00eaff;
    display:flex;
    align-items:center;
    justify-content:center;
    font-size:40px;
    color:#00eaff;
    margin-bottom:10px;
  }
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginBox">
  <div class="card">
    <h2 class="neon">Acesso</h2>

    <input id="usuario" placeholder="Usuário" />
    <input id="senha" type="password" placeholder="Senha" />

    <label style="display:flex; gap:8px; align-items:center; margin-top:10px;">
      <input type="checkbox" id="lembrarLogin" /> Lembrar login
    </label>

    <button onclick="login()">Entrar</button>
  </div>
</div>

<!-- Foto padrão -->
<div id="perfilFoto" style="display:none;">👤</div>

<!-- Botão sair -->
<div id="logoutBtn" onclick="logout()">Sair</div>

<!-- Engrenagem admin -->
<div id="gear" onclick="abrirPainel()">⚙️</div>

<!-- Painel admin -->
<div id="adminPanel">
  <h3 class="neon">Gerenciar Logins</h3>

  <input id="newUser" placeholder="Novo usuário">
  <input id="newPass" placeholder="Nova senha">
  <button class="ok" onclick="addUser()">Adicionar</button>

  <h4>Usuários:</h4>
  <ul id="listaUsers"></ul>

  <hr style="border:1px solid #333;margin:15px 0;">

  <h4>Trocar minha senha:</h4>
  <input id="newMyPass" placeholder="Nova senha">
  <button class="warning" onclick="trocarMinhaSenha()">Trocar senha</button>

  <button style="background:#ff4444;color:white;margin-top:20px;width:100%;" onclick="fecharPainel()">Fechar</button>
</div>

<!-- Conteúdo -->
<div id="siteArea">
  <h1 class="neon">Bem-vindo ao Site Neon!</h1>
  <p>Conteúdo liberado após o login.</p>
</div>

<script>
  /* banco de usuários */
  let users = [
    {user:"admin", pass:"1234", admin:true, blocked:false}
  ];

  function salvar(){ localStorage.setItem("usersCLX", JSON.stringify(users)); }
  function carregar(){
    const d = localStorage.getItem("usersCLX");
    if(d) users = JSON.parse(d);
  }
  carregar();

  let loggedUser = null;

  /* LOGIN */
  function login(){
    const u = usuario.value.trim();
    const s = senha.value.trim();

    const found = users.find(x => x.user === u && x.pass === s);

    if(!found) return alert("Login incorreto!");
    if(found.blocked) return alert("Usuário bloqueado!");

    loggedUser = found;

    if(lembrarLogin.checked){
      localStorage.setItem("lembrarUser", u);
      localStorage.setItem("lembrarPass", s);
    } else{
      localStorage.removeItem("lembrarUser");
      localStorage.removeItem("lembrarPass");
    }

    loginBox.style.display = "none";
    siteArea.style.display = "block";
    perfilFoto.style.display = "flex";
    logoutBtn.style.display = "block";

    document.body.style.background = "#fff";
    document.body.style.color = "black";

    if(found.admin) gear.style.display = "block";
  }

  /* lembrar login */
  window.onload = () => {
    const u = localStorage.getItem("lembrarUser");
    const p = localStorage.getItem("lembrarPass");
    if(u && p){
      usuario.value = u;
      senha.value = p;
      lembrarLogin.checked = true;
    }
  };

  /* sair */
  function logout(){
    loggedUser = null;
    siteArea.style.display = "none";
    gear.style.display = "none";
    perfilFoto.style.display = "none";
    logoutBtn.style.display = "none";

    document.body.style.background = "#03040a";
    document.body.style.color = "white";

    loginBox.style.display = "flex";
  }

  /* painel admin */
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
    alert("Senha alterada!");
  }

  /* listar usuários */
  function renderUsers(){
    listaUsers.innerHTML = "";
    users.forEach((u,i)=>{
      let cls = u.blocked ? "blocked" : "";
      listaUsers.innerHTML += `
      <li class="${cls}">
        ${u.user}
        <button class='btnSmall danger' onclick='remover(${i})'>❌</button>
        <button class='btnSmall warning' onclick='bloquear(${i})'>${u.blocked ? "Desbloquear" : "Bloquear"}</button>
      </li>`;
    });
  }
</script>

</body>
</html>
