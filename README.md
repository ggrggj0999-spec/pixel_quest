# pixel_quest
<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>PIXEL QUEST</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
*{box-sizing:border-box}
body{margin:0;padding:14px;background:linear-gradient(160deg,#17143b,#30205e,#17143b);font-family:system-ui;color:white}
.wrap{max-width:520px;margin:auto}
.panel{background:#24204b;border:1px solid #514878;border-radius:18px;padding:16px;margin-bottom:12px}
.top{display:flex;justify-content:space-between;align-items:center;gap:10px}
h1{font-size:24px;margin:5px 0}
.muted{color:#c9c3ec;font-size:13px}
.level{background:#40316c;border-radius:12px;padding:10px;text-align:center}
.level strong{display:block;font-size:28px;color:#ffe16a}
.avatar{text-align:center;font-size:70px;padding:15px;background:#302957;border-radius:15px;margin:14px 0}
.stats{display:grid;grid-template-columns:repeat(3,1fr);gap:7px;margin-top:12px}
.stat{background:#302957;border-radius:10px;padding:10px 5px;font-size:12px;text-align:center}
.stat b{display:block;font-size:17px;margin-top:5px}
.bar{height:10px;background:#49416e;border-radius:10px;overflow:hidden;margin:8px 0}
.fill{height:100%;width:0;background:#b18aff;transition:.3s}
.tabs{display:grid;grid-template-columns:repeat(3,1fr);gap:6px;margin-bottom:12px}
button{border:0;border-radius:11px;padding:12px;font-weight:800;background:#514078;color:white;cursor:pointer}
.tab.active{background:#8966ee}
.primary{width:100%;background:linear-gradient(90deg,#8966ee,#bd7cff);margin-top:10px}
.item{background:#302957;border:1px solid #514878;border-radius:12px;padding:12px;margin:9px 0}
.item button{width:100%;margin-top:8px}
button:disabled{opacity:.45}
.hidden{display:none}
.notice{padding:12px;background:#17143b;border-radius:10px;margin-top:10px;font-size:13px}
</style>
</head>
<body>
<div class="wrap">
<section class="panel">
<div class="top">
<div><div class="muted">PIXEL QUEST · СЕЗОН 01</div><h1>ПУТЬ ГЕРОЯ</h1><div class="muted">Маленький герой. Большая легенда.</div></div>
<div class="level"><strong id="level">1</strong>УРОВЕНЬ</div>
</div>
<div class="avatar">🧙</div>
<div class="top"><b>⭐ <span id="xp">0</span> XP</b><b><span id="progress">0/100</span></b></div>
<div class="bar"><div class="fill" id="fill"></div></div>
<div class="muted" id="next">100 XP до 2 уровня</div>
<div class="stats">
<div class="stat">🪙 МОНЕТЫ<b id="coins">30</b></div>
<div class="stat">❤️ ЗДОРОВЬЕ<b id="hp">100</b></div>
<div class="stat">🏆 ПОБЕДЫ<b id="wins">0</b></div>
</div>
</section>

<nav class="tabs">
<button class="tab active" data-tab="quests">⚔️ Задания</button>
<button class="tab" data-tab="shop">🛒 Магазин</button>
<button class="tab" data-tab="hero">🛡️ Герой</button>
</nav>

<section class="panel" id="quests">
<h2>⚔️ Доска заданий</h2>
<div class="muted">Выбирай задания, получай опыт и монеты!</div>
<div id="questList"></div>
<div class="notice" id="notice">Выбери задание!</div>
<button class="primary" id="complete">Выполнить задание ⚔️</button>
</section>

<section class="panel hidden" id="shop">
<h2>🛒 Магазин героя</h2>
<div id="shopList"></div>
</section>

<section class="panel hidden" id="hero">
<h2>🛡️ Карточка героя</h2>
<div class="avatar">🧙</div>
<h3 id="heroLevel">Герой · уровень 1</h3>
<p>⭐ Опыт: <b id="heroXp">0</b></p>
<p>🏆 Заданий: <b id="heroWins">0</b>/3</p>
<h3>🎒 Инвентарь</h3>
<div id="inventory"></div>
<button id="reset">Начать заново ↻</button>
</section>
<div class="muted" style="text-align:center;padding:10px">PIXEL QUEST · Твоя легенда начинается!</div>
</div>

<script>
const tg=window.Telegram?.WebApp;
if(tg){tg.ready();tg.expand()}

const initial={xp:0,coins:30,hp:100,completed:[],inventory:[],selected:0};
let state;
try{
state={...initial,...JSON.parse(localStorage.getItem("pixelQuestSave")||"{}")};
}catch(e){state={...initial}}

const quests=[
{name:"Слизень-первопроходец",desc:"Победи слизня в тренировочном бою.",xp:25,coins:15,icon:"🟢",diff:"ЛЕГКО"},
{name:"Охота за сокровищами",desc:"Найди потерянный сундук в лесу.",xp:40,coins:25,icon:"💎",diff:"СРЕДНЕ"},
{name:"Драконье испытание",desc:"Пройди испытание древнего дракона.",xp:60,coins:40,icon:"🐉",diff:"СЛОЖНО"}
];

const items=[
{id:"potion",name:"Зелье здоровья",icon:"🧪",price:20,desc:"Восстанавливает здоровье на 25"},
{id:"crystal",name:"Кристалл опыта",icon:"💠",price:30,desc:"Даёт 20 XP"},
{id:"crown",name:"Корона чемпиона",icon:"👑",price:60,desc:"Украшение героя"}
];

let notice="Выбери задание и отправляйся в приключение!";

function save(){
try{localStorage.setItem("pixelQuestSave",JSON.stringify(state))}catch(e){}
}
function level(){return Math.floor(state.xp/100)+1}

function render(){
const lv=level(),prog=state.xp%100;
document.getElementById("level").textContent=lv;
document.getElementById("xp").textContent=state.xp;
document.getElementById("progress").textContent=prog+"/100";
document.getElementById("fill").style.width=prog+"%";
document.getElementById("next").textContent=(100-prog)+" XP до "+(lv+1)+" уровня";
document.getElementById("coins").textContent=state.coins;
document.getElementById("hp").textContent=state.hp;
document.getElementById("wins").textContent=state.completed.length;
document.getElementById("notice").textContent=notice;

document.getElementById("questList").innerHTML=quests.map((q,i)=>`
<div class="item" style="border:2px solid ${state.selected===i?'#a78bfa':'#514878'}">
<div style="font-size:25px">${q.icon} <b>${q.name}</b></div>
<p class="muted">${q.desc}</p>
<div>⭐ +${q.xp} XP · 🪙 +${q.coins}</div>
<div class="muted">${state.completed.includes(i)?"✅ Выполнено":q.diff}</div>
<button style="margin-top:8px" onclick="selectQuest(${i})">Выбрать</button>
</div>`).join("");

document.getElementById("complete").disabled=state.completed.includes(state.selected);
document.getElementById("complete").textContent=state.completed.includes(state.selected)?"Задание завершено ✓":"Выполнить задание ⚔️";

document.getElementById("shopList").innerHTML=items.map(it=>`
<div class="item">
<div style="font-size:24px">${it.icon} <b>${it.name}</b></div>
<p class="muted">${it.desc}</p>🪙 ${it.price}
<button onclick="buy('${it.id}')" ${state.inventory.includes(it.id)||state.coins<it.price?"disabled":""}>
${state.inventory.includes(it.id)?"Куплено ✓":state.coins<it.price?"Не хватает монет":"Купить"}
</button></div>`).join("");

document.getElementById("heroLevel").textContent="Герой · уровень "+lv;
document.getElementById("heroXp").textContent=state.xp;
document.getElementById("heroWins").textContent=state.completed.length;
document.getElementById("inventory").innerHTML=state.inventory.length?
state.inventory.map(id=>{const it=items.find(x=>x.id===id);return "<p>"+it.icon+" "+it.name+"</p>"}).join("")
:"<p class='muted'>Инвентарь пока пуст.</p>";
save();
}

function selectQuest(i){
state.selected=i;
notice="Выбрано: "+quests[i].name;
render();
}

function buy(id){
const it=items.find(x=>x.id===id);
if(!it||state.inventory.includes(id)||state.coins<it.price)return;
state.coins-=it.price;
state.inventory.push(id);
if(id==="potion")state.hp=Math.min(100,state.hp+25);
if(id==="crystal")state.xp+=20;
notice="Куплено: "+it.name+"!";
render();
}

document.getElementById("complete").addEventListener("click",()=>{
const i=state.selected;
if(state.completed.includes(i))return;
const q=quests[i],old=level();
state.xp+=q.xp;
state.coins+=q.coins;
state.completed.push(i);
notice="Победа! +"+q.xp+" XP и +"+q.coins+" монет!";
if(level()>old)notice+=" 🎉 Новый уровень!";
render();
});

document.querySelectorAll("[data-tab]").forEach(btn=>{
btn.addEventListener("click",()=>{
document.querySelectorAll("[data-tab]").forEach(b=>b.classList.toggle("active",b===btn));
["quests","shop","hero"].forEach(id=>document.getElementById(id).classList.toggle("hidden",id!==btn.dataset.tab));
});
});

document.getElementById("reset").addEventListener("click",()=>{
if(confirm("Начать новую игру?")){
state={...initial,completed:[],inventory:[]};
notice="Новая игра началась! Удачи, герой!";
render();
}
});

render();
</script>
</body>
</html>
