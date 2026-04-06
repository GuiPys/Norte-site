[painel_administrativo_norte_distribuidora (1).html](https://github.com/user-attachments/files/26508776/painel_administrativo_norte_distribuidora.1.html)
<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Norte Distribuidora - Sistema Profissional</title>

<!-- Firebase (compatível entre si) -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage-compat.js"></script>

<style>
body { margin:0; font-family:Arial; background:#f1f5f9; }
header { background:#0f172a; color:#fff; padding:15px; display:flex; justify-content:space-between; }
input, select { padding:8px; margin:5px 0; width:100%; }
button { padding:10px; border:none; border-radius:5px; cursor:pointer; }

.container { display:flex; }
.sidebar { width:220px; background:#1e293b; color:#fff; padding:15px; }
.sidebar button { width:100%; margin:5px 0; background:#334155; color:#fff; }

.main { flex:1; padding:20px; }
.products { display:grid; grid-template-columns:repeat(auto-fill,minmax(200px,1fr)); gap:15px; }
.card { background:#fff; padding:10px; border-radius:8px; }
.card img { width:100%; height:120px; object-fit:cover; }

.form { background:#fff; padding:15px; margin-bottom:20px; border-radius:8px; display:none; }
#loginBox { max-width:300px; margin:50px auto; background:#fff; padding:20px; border-radius:8px; }
</style>
</head>

<body>

<header>
<h2>Norte Distribuidora</h2>
<input type="text" id="search" placeholder="Buscar..." onkeyup="buscar()">
<button onclick="mostrarLogin()">Admin</button>
</header>

<div id="loginBox" style="display:none;">
<h3>Login Admin</h3>
<input id="email" placeholder="Email">
<input id="senha" type="password" placeholder="Senha">
<button onclick="login()">Entrar</button>
</div>

<div class="container">
<div class="sidebar">
<button onclick="filtrar('all')">Todos</button>
<button onclick="filtrar('ferramentas')">Ferramentas</button>
<button onclick="filtrar('adesivos')">Adesivos</button>
<button onclick="filtrar('mangueiras')">Mangueiras</button>
<button onclick="filtrar('epis')">EPI's</button>
<button onclick="filtrar('eletricos')">Elétricos</button>
</div>

<div class="main">

<div class="form" id="adminPanel">
<h3>Cadastrar Produto</h3>
<input id="nome" placeholder="Nome">
<input type="file" id="file">
<select id="categoria">
<option value="ferramentas">Ferramentas</option>
<option value="adesivos">Adesivos</option>
<option value="mangueiras">Mangueiras</option>
<option value="epis">EPI's</option>
<option value="eletricos">Elétricos</option>
</select>
<button onclick="addProduto()">Cadastrar</button>
</div>

<div class="products" id="products"></div>

</div>
</div>

<script>
// CONFIG FIREBASE
const firebaseConfig = {
  apiKey: "AIzaSyAu5gIF2BBR6boxJ6mJIjMV71CbD-Ey0zk",
  authDomain: "norte-distribuidora.firebaseapp.com",
  projectId: "norte-distribuidora",
  storageBucket: "norte-distribuidora.firebasestorage.app",
  messagingSenderId: "225983951384",
  appId: "1:225983951384:web:291a7b6096f11194e8096d"
};

// INICIALIZAÇÃO SEGURA
let app;
try {
  app = firebase.initializeApp(firebaseConfig);
} catch (e) {
  app = firebase.app();
}

const auth = firebase.auth();
const db = firebase.firestore();
const storage = firebase.storage();

// LOGIN
function login(){
const emailVal = document.getElementById('email').value;
const senhaVal = document.getElementById('senha').value;

auth.signInWithEmailAndPassword(emailVal, senhaVal)
.then(()=>{
 document.getElementById('adminPanel').style.display='block';
 document.getElementById('loginBox').style.display='none';
})
.catch(err=>{
 console.error(err);
 alert('Erro no login');
});
}

function mostrarLogin(){
document.getElementById('loginBox').style.display='block';
}

// UPLOAD
function uploadImagem(file){
let ref = storage.ref('produtos/' + Date.now() + '_' + file.name);
return ref.put(file).then(()=> ref.getDownloadURL());
}

// ADD PRODUTO
function addProduto(){
let nome = document.getElementById('nome').value;
let file = document.getElementById('file').files[0];
let categoria = document.getElementById('categoria').value;

if(!nome || !file) return alert('Preencha tudo');

uploadImagem(file).then(url=>{
 db.collection('produtos').add({ nome, imagem:url, categoria })
 .catch(err=>console.error('Erro ao salvar:', err));
});
}

// LISTAR COM TRATAMENTO DE ERRO
let lista = [];

db.collection('produtos').onSnapshot(snapshot=>{
 lista = [];
 snapshot.forEach(doc=>{
  lista.push({...doc.data(), id:doc.id});
 });
 render(lista);
}, err => {
 console.error('Erro Firestore:', err);
 alert('Erro ao carregar dados. Verifique Firestore.');
});

function render(data){
let html='';
data.forEach(p=>{
html+=`
<div class="card">
<img src="${p.imagem}">
<h4>${p.nome}</h4>
<button onclick="comprar('${p.nome}')">WhatsApp Norte</button>
<button onclick="excluir('${p.id}')" style="background:red;margin-top:5px;">Excluir</button>
</div>`;
});
document.getElementById('products').innerHTML = html;
}

function excluir(id){
db.collection('produtos').doc(id).delete()
.catch(err=>console.error('Erro ao excluir:', err));
}

function comprar(nome){
window.open(`https://wa.me/5594981590065?text=Olá, tenho interesse no produto ${nome}`);
}

function filtrar(cat){
if(cat==='all') return render(lista);
render(lista.filter(p=>p.categoria===cat));
}

function buscar(){
let val = document.getElementById('search').value.toLowerCase();
render(lista.filter(p=>p.nome.toLowerCase().includes(val)));
}
</script>

</body>
</html>
