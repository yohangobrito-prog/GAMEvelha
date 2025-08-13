<!doctype html>
<html lang="pt-br">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>GAME VELHA – MULTIPLAYer (HTML only)</title>
  <style>
    :root{
      --bg:#0f1220; --card:#171b2e; --muted:#aeb3cf; --text:#f1f5ff; --acc:#6ee7ff; --good:#22c55e; --bad:#ef4444; --warn:#f59e0b;
      --grid:#242a46; --x:#7dd3fc; --o:#fca5a5;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0; background:radial-gradient(1000px 600px at 10% 0%,#111735,#0f1220 60%); color:var(--text); font:16px/1.4 system-ui,Segoe UI,Roboto,Helvetica,Arial}
    .wrap{max-width:980px;margin:0 auto;padding:24px}
    h1{font-size:28px;margin:0 0 8px}
    h2{font-size:20px;margin:16px 0 8px;color:var(--muted)}
    .card{background:linear-gradient(180deg,rgba(255,255,255,0.03),rgba(255,255,255,0.01)); border:1px solid #232844; border-radius:16px; padding:16px; box-shadow:0 10px 30px rgba(0,0,0,.25)}
    .row{display:flex; gap:12px; flex-wrap:wrap}
    .col{flex:1 1 280px; min-width:280px}
    input,button,select{background:#0e1328; color:var(--text); border:1px solid #243056; border-radius:12px; padding:10px 12px; font:inherit}
    button{cursor:pointer; transition:.15s transform,.15s opacity}
    button:hover{transform:translateY(-1px)}
    .btn{background:linear-gradient(180deg,#1b2040,#12172e); border:1px solid #2a345e}
    .btn-acc{background:linear-gradient(180deg,#0d3a4a,#0a2a39); border-color:#0e556b}
    .btn-good{background:linear-gradient(180deg,#14351f,#0d2416); border-color:#1f6d43}
    .btn-bad{background:linear-gradient(180deg,#3a1414,#2a0d0d); border-color:#6d1f1f}
    .muted{color:var(--muted)}
    .tag{display:inline-flex;align-items:center;gap:6px;padding:4px 8px;border-radius:999px;background:#0d1330;border:1px solid #26316a;color:#cfe5ff;font-size:12px}

    /* Lobby / lista de servidores */
    .server{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:10px;border:1px solid #25305a;border-radius:12px;background:#0f1530}
    .server strong{font-weight:600}
    .server .status{font-size:12px}
    .status.on{color:var(--good)}
    .status.off{color:var(--bad)}

    /* Tabuleiro */
    .boardWrap{display:grid;grid-template-columns:repeat(3,1fr);gap:10px; width:min(440px,90vw); margin:16px auto}
    .cell{height:min(140px,28vw); display:grid; place-items:center; font-weight:800; font-size:42px; border-radius:16px; background:linear-gradient(180deg,#12172e,#0e1326); border:1px solid #242a46; cursor:pointer; user-select:none}
    .cell.disabled{opacity:.45;pointer-events:none}
    .cell.x{color:var(--x)}
    .cell.o{color:var(--o)}
    .turn{display:flex;align-items:center;justify-content:center;gap:10px}
    .score{display:flex;gap:10px;justify-content:center}
    .score .box{padding:8px 12px;border:1px solid #2a3560;border-radius:12px;background:#0e142e}
    .marquee{border:1px dashed #2a2f4f;padding:8px;border-radius:12px}

    .hidden{display:none}
    .center{text-align:center}
    .spacer{height:8px}
    .small{font-size:13px}
  </style>
</head>
<body>
<div class="wrap">
  <header class="card">
    <h1>GAME VELHA – MULTIPLAYer</h1>
    <div class="muted small">Versão só HTML/JS (sem backend). Funciona abrindo duas abas do mesmo arquivo: os jogadores se enxergam via <code>BroadcastChannel</code>/<code>localStorage</code>.</div>
  </header>

  <!-- MENU -->
  <section id="menu" class="card" aria-label="Menu">
    <div class="row">
      <div class="col">
        <label for="playerName">Seu nome</label>
        <input id="playerName" placeholder="Digite seu nome" maxlength="16">
      </div>
      <div class="col center" style="align-self:end">
        <button class="btn-acc" id="btnSaveName">Salvar Nome</button>
      </div>
    </div>
    <div class="spacer"></div>
    <div class="row">
      <div class="col">
        <button class="btn" id="btnRobo">Play Robo</button>
      </div>
      <div class="col">
        <button class="btn" id="btnServer">Play Server</button>
      </div>
      <div class="col">
        <button class="btn-good" id="btnCreate">Play Create Server</button>
      </div>
    </div>
  </section>

  <!-- LOBBY: lista de servidores -->
  <section id="lobby" class="card hidden" aria-label="Lobby">
    <div class="row" style="align-items:center; justify-content:space-between">
      <h2>Servidores</h2>
      <button class="btn" id="btnBackLobby">Voltar</button>
    </div>
    <div id="serverList" class="col" style="display:flex; flex-direction:column; gap:10px"></div>
    <div class="spacer"></div>
    <div class="small muted">Dica: Abra esta página em outra aba/Janela para simular o segundo jogador.</div>
  </section>

  <!-- ESPERA APÓS CRIAR SERVER -->
  <section id="waiting" class="card hidden" aria-label="Aguardando jogador">
    <div class="row" style="align-items:center; justify-content:space-between">
      <h2>Servidor criado</h2>
      <button class="btn" id="btnCancelServer">Encerrar</button>
    </div>
    <div id="serverInfo" class="marquee"></div>
    <div class="spacer"></div>
    <div class="muted small">Compartilhe o <b>Nome do Server</b> e a <b>Key</b> com o outro jogador (nesta demo, ambas abas veem a lista automaticamente).</div>
  </section>

  <!-- JOGO -->
  <section id="game" class="card hidden" aria-label="Jogo da Velha">
    <div class="row" style="align-items:center; justify-content:space-between">
      <h2>Partida</h2>
      <button class="btn" id="btnSair">Sair</button>
    </div>
    <div id="names" class="center muted"></div>
    <div class="spacer"></div>
    <div class="turn" id="turnInfo"></div>
    <div class="boardWrap" id="board"></div>
    <div class="score" id="score"></div>
    <div id="roundMsg" class="center" style="min-height:24px"></div>
  </section>
</div>

<script>
/********************** Util **********************/
const $ = sel => document.querySelector(sel);
const uid = () => Math.random().toString(36).slice(2,10)+Date.now().toString(36).slice(-4);
const STORAGE_SERVERS = 'ttt.servers.v1';
const STORAGE_GAMES = 'ttt.games.v1';
const STORAGE_NAME = 'ttt.playerName';
const bc = ('BroadcastChannel' in window) ? new BroadcastChannel('ttt-bc') : null;
function emitBC(type,payload){
  if(bc) bc.postMessage({type,payload,ts:Date.now()});
}
function playTone(type='click'){
  try{
    const ctx = playTone.ctx || (playTone.ctx=new (window.AudioContext||window.webkitAudioContext)());
    const o = ctx.createOscillator();
    const g = ctx.createGain();
    o.connect(g); g.connect(ctx.destination);
    let dur=0.09, f=420, vol=0.05;
    if(type==='win'){f=440;dur=0.5;vol=0.07}
    if(type==='lose'){f=220;dur=0.5;vol=0.07}
    if(type==='click'){f=520;dur=0.06;vol=0.05}
    o.type='sine'; o.frequency.value=f; g.gain.value=vol;
    o.start(); o.stop(ctx.currentTime+dur);
    // Arpejo simples para win/lose
    if(type!=='click'){
      let steps = type==='win' ? [1,1.25,1.5,2] : [1,0.84,0.66,0.5];
      steps.forEach((m,i)=> setTimeout(()=>{ if(o) o.frequency.setValueAtTime(f*m, ctx.currentTime); }, i*100));
    }
  }catch(e){}
}

function loadServers(){
  try { return JSON.parse(localStorage.getItem(STORAGE_SERVERS)||'[]'); } catch(e){ return []; }
}
function saveServers(list){ localStorage.setItem(STORAGE_SERVERS, JSON.stringify(list)); emitBC('servers:update',null); }
function loadGames(){
  try { return JSON.parse(localStorage.getItem(STORAGE_GAMES)||'{}'); } catch(e){ return {}; }
}
function saveGames(map){ localStorage.setItem(STORAGE_GAMES, JSON.stringify(map)); emitBC('games:update',null); }

/********************** Estado **********************/
let state = {
  me: {name: localStorage.getItem(STORAGE_NAME)||''},
  view: 'menu',
  mode: null, // 'ai' | 'server'
  serverId: null,
  symbol: null, // 'X' or 'O'
  game: null,   // referência ao objeto de jogo da sala
  myWins:0, enemyWins:0, target:4,
};

/********************** UI básicas **********************/
function show(id){ ['menu','lobby','waiting','game'].forEach(v=> $('#'+v).classList.toggle('hidden', v!==id)); state.view=id; }
function setNameUI(){ $('#playerName').value = state.me.name; }

/********************** Lobby / Servidores **********************/
function createServer(){
  const servers = loadServers();
  const id = uid();
  const server = {
    id,
    name: `Server-${id.slice(0,4)}`,
    key: uid().toUpperCase(),
    on: true,
    host: {name: state.me.name||'Jogador'},
    guest: null,
    createdAt: Date.now()
  };
  servers.push(server); saveServers(servers);
  state.serverId = id; state.mode='server'; state.symbol='X';
  show('waiting');
  renderServerInfo(server);
}

function renderServerInfo(s){
  $('#serverInfo').innerHTML = `
    <div class="row" style="flex-wrap:wrap;gap:8px">
      <span class="tag"><strong>Nome server:</strong> ${s.name}</span>
      <span class="tag"><strong>Key:</strong> ${s.key}</span>
      <span class="tag">Status: <span class="status ${s.on?'on':'off'}">${s.on?'ON':'OFF'}</span></span>
      <span class="tag">Host: <strong>${s.host?.name||'-'}</strong></span>
      <span class="tag">Guest: <strong>${s.guest?.name||'-'}</strong></span>
    </div>`;
}

function listServers(){
  const cont = $('#serverList');
  const servers = loadServers().sort((a,b)=> b.createdAt - a.createdAt);
  cont.innerHTML='';
  if(!servers.length){
    cont.innerHTML = '<div class="muted">Nenhum servidor. Use "Play Create Server" em outra aba.</div>';
    return;
  }
  servers.forEach(s=>{
    const div = document.createElement('div');
    div.className='server';
    div.innerHTML = `
      <div>
        <div><strong>${s.name}</strong> <span class="status ${s.on?'on':'off'}">${s.on?'ON':'OFF'}</span></div>
        <div class="small muted">Key: ${s.key}</div>
        <div class="small">Host: <strong>${s.host?.name||'-'}</strong> · Guest: <strong>${s.guest?.name||'-'}</strong></div>
      </div>
      <div style="display:flex; gap:8px">
        <button class="btn" data-id="${s.id}" data-action="join">Entrar</button>
        <button class="btn-bad" data-id="${s.id}" data-action="remove">Remover</button>
      </div>`;
    cont.appendChild(div);
  });
}

function removeServer(id){
  const servers = loadServers();
  const idx = servers.findIndex(s=>s.id===id);
  if(idx>=0){ servers.splice(idx,1); saveServers(servers); }
}

function joinServer(id){
  const servers = loadServers();
  const s = servers.find(x=>x.id===id);
  if(!s){ alert('Servidor não existe mais'); return; }
  if(s.guest && s.guest.name){ alert('Servidor já tem dois jogadores'); return; }
  s.guest = {name: state.me.name||'Jogador'}; s.on=true;
  saveServers(servers);
  state.serverId = id; state.mode='server'; state.symbol='O';
  startMatchFromServer(s);
}

/********************** Jogo **********************/
const wins = [
  [0,1,2],[3,4,5],[6,7,8], // linhas
  [0,3,6],[1,4,7],[2,5,8], // colunas
  [0,4,8],[2,4,6]          // diagonais
];

function emptyBoard(){ return Array(9).fill(''); }
function checkWinner(b){
  for(const [a,b2,c] of wins){
    if(b[a] && b[a]===b[b2] && b[a]===b[c]) return b[a];
  }
  if(b.every(v=>v)) return 'draw';
  return null;
}

function renderBoard(b, myTurn){
  const wrap = $('#board');
  wrap.innerHTML='';
  b.forEach((v,i)=>{
    const cell = document.createElement('div');
    cell.className = 'cell'+(v?(' '+v.toLowerCase()):'')+(myTurn?'':' disabled');
    cell.textContent = v||'';
    cell.addEventListener('click',()=>{ playTone('click'); makeMove(i); });
    wrap.appendChild(cell);
  });
}

function renderScore(){
  $('#score').innerHTML = `
    <div class="box">${state.game?.host?.name||'Host'} (X): <strong>${state.symbol==='X'?state.myWins:state.enemyWins}</strong></div>
    <div class="box">${state.game?.guest?.name||'Guest'} (O): <strong>${state.symbol==='O'?state.myWins:state.enemyWins}</strong></div>
    <div class="box">Alvo: 4 vitórias</div>`;
}

function setNames(){
  const host = state.game?.host?.name||'-';
  const guest = state.game?.guest?.name||'-';
  $('#names').innerHTML = `Jogadores: <strong>${host}</strong> vs <strong>${guest}</strong>`;
}

function turnText(){
  const t = state.game.turn;
  const mine = (t===state.symbol);
  $('#turnInfo').innerHTML = `<span class="tag">Vez: <strong>${t}</strong></span> ${mine?'<span class="tag">Sua vez</span>':'<span class="tag">Aguarde o inimigo</span>'}`;
}

function makeMove(i){
  const g = state.game; if(!g) return;
  if(g.turn!==state.symbol) return; // não é minha vez
  if(g.board[i]) return; // ocupado
  g.board[i] = g.turn;
  g.turn = g.turn==='X'?'O':'X';
  commitGame(g);
}

function nextRound(who){
  if(who==='draw'){
    $('#roundMsg').innerHTML = '<span class="muted">Empate! Nova rodada…</span>';
  }else{
    const iWon = (who===state.symbol);
    iWon ? (state.myWins++) : (state.enemyWins++);
    $('#roundMsg').innerHTML = iWon ? '<div style="color:var(--good);font-weight:700">VC GANHOU a rodada!</div>' : '<div style="color:var(--bad);font-weight:700">INIMIGO GANHOU a rodada!</div>';
    playTone(iWon?'win':'lose');
  }
  renderScore();
  if(state.myWins>=state.target || state.enemyWins>=state.target){
    const finalIWon = state.myWins>state.enemyWins;
    setTimeout(()=>{
      alert(finalIWon? 'VC GANHOU a PARTIDA!': 'INIMIGO GANHOU a PARTIDA!');
    }, 50);
  }
  // resetar tabuleiro e manter placar
  const g = state.game; g.board = emptyBoard();
  // alterna quem começa a próxima? Vamos manter mesma ordem: X começa
  g.turn = 'X';
  commitGame(g,true); // true = nova rodada
}

/********************** Persistência de Jogo por Sala **********************/
function commitGame(g, freshRound=false){
  const games = loadGames();
  games[g.id] = g; saveGames(games);
  updateGameUI();
  const res = checkWinner(g.board);
  if(res){ setTimeout(()=> nextRound(res), 200); }
}

function updateGameUI(){
  const g = state.game = loadGames()[state.serverId];
  if(!g){ return; }
  setNames();
  renderBoard(g.board, g.turn===state.symbol);
  turnText();
  renderScore();
}

function startMatchFromServer(server){
  // Criar estado de jogo compartilhado
  const games = loadGames();
  const existing = games[server.id];
  const g = existing || {
    id: server.id,
    host: server.host,
    guest: server.guest,
    board: emptyBoard(),
    turn: 'X',
    wins: {X:0,O:0}
  };
  games[server.id]=g; saveGames(games);
  state.myWins=0; state.enemyWins=0; $('#roundMsg').textContent='';
  show('game');
  updateGameUI();
}

/********************** IA (Play Robo) **********************/
function startAI(){
  state.mode='ai'; state.symbol='X'; state.serverId = 'ai-'+uid();
  const g = {id:state.serverId, host:{name:state.me.name||'Você'}, guest:{name:'Robo'}, board:emptyBoard(), turn:'X'};
  const games = loadGames(); games[g.id]=g; saveGames(games); state.myWins=0; state.enemyWins=0; $('#roundMsg').textContent='';
  show('game'); updateGameUI();
}
function aiMove(){
  if(state.mode!=='ai') return;
  const g = state.game; if(!g) return;
  if(g.turn!=='O') return;
  // IA simples: ganhar > bloquear > centro > canto > aleatório
  const b = g.board.slice();
  const lines = wins;
  const canPut = (idx)=>!b[idx];
  function findWin(sym){
    for(const [a,c,d] of lines){
      const line=[b[a],b[c],b[d]]; const idxs=[a,c,d];
      if(line.filter(v=>v===sym).length===2 && line.includes('')){ return idxs[line.indexOf('')]; }
    } return -1;
  }
  let move = findWin('O');
  if(move<0) move = findWin('X');
  if(move<0 && canPut(4)) move=4;
  if(move<0){ const corners=[0,2,6,8].filter(canPut); if(corners.length) move=corners[Math.floor(Math.random()*corners.length)]; }
  if(move<0){ const empties=b.map((v,i)=>v?null:i).filter(v=>v!==null); if(empties.length) move=empties[Math.floor(Math.random()*empties.length)]; }
  if(move>=0){ g.board[move]='O'; g.turn='X'; commitGame(g); }
}

/********************** Eventos UI **********************/
$('#btnSaveName').addEventListener('click',()=>{
  const n = ($('#playerName').value||'').trim();
  if(!n){ alert('Digite um nome'); return; }
  state.me.name=n; localStorage.setItem(STORAGE_NAME,n); emitBC('name:update',n);
  playTone('click');
});

$('#btnRobo').addEventListener('click',()=>{ playTone('click'); startAI(); });
$('#btnServer').addEventListener('click',()=>{ playTone('click'); listServers(); show('lobby'); });
$('#btnCreate').addEventListener('click',()=>{ 
  if(!(state.me.name||'').trim()){ alert('Digite e salve seu nome primeiro'); return; }
  playTone('click'); createServer();
});
$('#btnBackLobby').addEventListener('click',()=>{ playTone('click'); show('menu'); });
$('#btnCancelServer').addEventListener('click',()=>{
  playTone('click');
  const servers=loadServers(); const idx=servers.findIndex(s=>s.id===state.serverId);
  if(idx>=0){ servers.splice(idx,1); saveServers(servers); }
  state.serverId=null; show('menu');
});
$('#btnSair').addEventListener('click',()=>{ playTone('click'); show('menu'); });

$('#serverList').addEventListener('click', (e)=>{
  const btn = e.target.closest('button'); if(!btn) return; const id = btn.dataset.id; const action=btn.dataset.action;
  if(action==='remove'){ removeServer(id); listServers(); return; }
  if(action==='join'){
    if(!(state.me.name||'').trim()){ alert('Digite e salve seu nome primeiro'); return; }
    joinServer(id);
  }
});

/********************** Broadcast / Sync **********************/
if(bc){
  bc.onmessage = (ev)=>{
    const {type} = ev.data||{};
    if(type==='servers:update'){
      if(state.view==='lobby') listServers();
      if(state.view==='waiting'){ const s = loadServers().find(x=>x.id===state.serverId); if(s){ renderServerInfo(s); if(s.guest && s.guest.name){ startMatchFromServer(s); } } }
    }
    if(type==='games:update'){
      if(state.view==='game'){ updateGameUI(); if(state.mode==='ai') aiMove(); }
    }
    if(type==='name:update'){
      // Só para forçar refresh do lobby/espera
      if(state.view==='lobby') listServers();
      if(state.view==='waiting'){ const s = loadServers().find(x=>x.id===state.serverId); if(s) renderServerInfo(s); }
    }
  };
}

window.addEventListener('storage', (e)=>{
  if(e.key===STORAGE_SERVERS){ if(state.view==='lobby') listServers(); if(state.view==='waiting'){ const s = loadServers().find(x=>x.id===state.serverId); if(s){ renderServerInfo(s); if(s.guest && s.guest.name){ startMatchFromServer(s); } } } }
  if(e.key===STORAGE_GAMES){ if(state.view==='game'){ updateGameUI(); if(state.mode==='ai') aiMove(); } }
});

/********************** Init **********************/
setNameUI();
show('menu');
</script>
</body>
</html>
