<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>My Study Room - پیشرفته نمودار</title>
<style>
body { font-family:'Tahoma',sans-serif; margin:0; padding:0; background:#f0f0f0; color:#333; transition:0.3s; }
header { background:#4CAF50; color:white; padding:1rem; text-align:center; display:flex; justify-content:space-between; align-items:center; }
main { display:flex; height:calc(100vh - 60px); }
nav { width:220px; background:#2E7D32; padding:1rem; display:flex; flex-direction:column; gap:10px; }
nav button { background:#66BB6A; border:none; color:white; padding:0.5rem; cursor:pointer; transition:0.2s; }
nav button:hover { background:#81C784; }
section { flex:1; padding:1rem; overflow-y:auto; }
.task { background:white; padding:10px; margin-bottom:10px; border-radius:5px; box-shadow:0 0 5px rgba(0,0,0,0.2); cursor:pointer; }
.task.done { background:#d0ffd0; text-decoration:line-through; }
.theme-dark { background:#121212; color:white; }
.theme-dark nav { background:#1E1E1E; }
.theme-dark nav button { background:#333; }
.theme-classic { background:#e0f7fa; color:#00695c; }
.theme-classic nav { background:#004d40; }
.theme-classic nav button { background:#00796b; }
.video-container { margin:10px 0; }
.chat-box { border:1px solid #ccc; padding:10px; height:200px; overflow-y:auto; background:white; }
.chat-input { width:100%; padding:5px; margin-top:5px; }
.category-select { margin-bottom:10px; }
.progress-container { background:#ccc; width:100%; border-radius:5px; overflow:hidden; margin:10px 0; }
.progress-bar { background:#4CAF50; height:20px; width:0%; transition:0.3s; }
</style>
</head>
<body>
<header>
  <span>اتاق مطالعه من</span>
  <div>
    <button onclick="toggleTheme()">تغییر تم</button>
    <select id="themeSelect" onchange="changeTheme(this.value)">
      <option value="default">پیش‌فرض</option>
      <option value="dark">تاریک</option>
      <option value="classic">سبز کلاسیک</option>
    </select>
  </div>
</header>
<main>
<nav>
  <button onclick="showSection('tasks')">برنامه روزانه</button>
  <button onclick="showSection('videos')">فیلم‌ها</button>
  <button onclick="showSection('chat')">دستیار هوشمند</button>
  <button onclick="showSection('progress')">پیشرفت</button>
</nav>

<section id="tasks" style="display:block;">
  <h2>برنامه روزانه</h2>
  <input type="text" id="newTask" placeholder="کار جدید">
  <select id="taskCategory" class="category-select">
    <option value="درس">درس</option>
    <option value="ورزش">ورزش</option>
    <option value="تفریح">تفریح</option>
  </select>
  <button onclick="addTask()">اضافه کردن</button>
  <div id="taskList"></div>
</section>

<section id="videos" style="display:none;">
  <h2>فیلم‌های آموزشی</h2>
  <input type="text" id="newVideo" placeholder="لینک ویدئو">
  <select id="videoCategory" class="category-select">
    <option value="درس">درس</option>
    <option value="ورزش">ورزش</option>
    <option value="تفریح">تفریح</option>
  </select>
  <button onclick="addVideo()">اضافه کردن</button>
  <div id="videoList"></div>
</section>

<section id="chat" style="display:none;">
  <h2>چت با دستیار هوشمند</h2>
  <div class="chat-box" id="chatBox"></div>
  <input type="text" id="chatInput" class="chat-input" placeholder="پیام خود را بنویسید">
  <button onclick="sendMessage()">ارسال</button>
</section>

<section id="progress" style="display:none;">
  <h2>پیشرفت مطالعه</h2>
  <canvas id="progressChart" width="400" height="200"></canvas>
  <div class="progress-container"><div id="progressBar" class="progress-bar"></div></div>
  <div id="progressText"></div>
</section>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
let theme='default';
function toggleTheme(){ theme=(theme==='dark'?'default':'dark'); applyTheme(); }
function changeTheme(val){ theme=val; applyTheme(); }
function applyTheme(){ document.body.className=theme==='dark'?'theme-dark':(theme==='classic'?'theme-classic':''); }

function showSection(id){ document.querySelectorAll('main section').forEach(s=>s.style.display='none'); document.getElementById(id).style.display='block'; }

function saveData(key,val){ localStorage.setItem(key,JSON.stringify(val)); }
function loadData(key){ return JSON.parse(localStorage.getItem(key)||'[]'); }

function addTask(){
  let val=document.getElementById('newTask').value.trim();
  let cat=document.getElementById('taskCategory').value;
  if(!val) return;
  let tasks=loadData('tasks');
  tasks.push({text:val, category:cat, done:false, time:new Date().toISOString()});
  saveData('tasks',tasks);
  document.getElementById('newTask').value='';
  renderTasks(); updateProgress(); checkAI(); renderChart();
}
function renderTasks(){
  let tasks=loadData('tasks'); let container=document.getElementById('taskList'); container.innerHTML='';
  tasks.forEach((t,i)=>{ let div=document.createElement('div'); div.className='task'+(t.done?' done':''); div.textContent=`[${t.category}] ${t.text}`; div.onclick=()=>{ t.done=!t.done; saveData('tasks',tasks); renderTasks(); updateProgress(); checkAI(); renderChart(); }; container.appendChild(div); });
}
renderTasks();

function addVideo(){
  let val=document.getElementById('newVideo').value.trim();
  let cat=document.getElementById('videoCategory').value;
  if(!val) return;
  let videos=loadData('videos'); videos.push({url:val, category:cat}); saveData('videos',videos); document.getElementById('newVideo').value=''; renderVideos();
}
function renderVideos(){
  let videos=loadData('videos'); let container=document.getElementById('videoList'); container.innerHTML='';
  videos.forEach(v=>{ let div=document.createElement('div'); div.className='video-container'; div.innerHTML=`<strong>[${v.category}]</strong><video width="320" height="200" controls><source src="${v.url}" type="video/mp4"> مرورگر شما از ویدئو پشتیبانی نمی‌کند.</video>`; container.appendChild(div); });
}
renderVideos();

function sendMessage(){
  let input=document.getElementById('chatInput'); let msg=input.value.trim(); if(!msg) return;
  let box=document.getElementById('chatBox'); let userDiv=document.createElement('div'); userDiv.textContent='شما: '+msg; box.appendChild(userDiv);
  let reply=document.createElement('div'); reply.textContent='دستیار: '+generateReply(msg); box.appendChild(reply); input.value=''; box.scrollTop=box.scrollHeight;
}

function generateReply(msg){
  msg=msg.toLowerCase(); let tasks=loadData('tasks'); let undone=tasks.filter(t=>!t.done);
  if(msg.includes('سلام')) return 'سلام! امروز برنامه ات را مرور کردی؟';
  if(msg.includes('پیشنهاد')) return undone.length? `توصیه: ${undone[0].text} را انجام بده.`:'تمام کارها انجام شده!';
  if(msg.includes('اخطار')) return undone.length? 'یادت باشد زمان مطالعه و استراحت را رعایت کنی!':'همه کارها انجام شده.';
  if(msg.includes('کار بعدی')) return undone.length? 'کار بعدی: '+undone[0].text:'تمام کارها انجام شده!';
  return 'متوجه شدم: '+msg;
}

function checkAI(){ let tasks=loadData('tasks'); let undone=tasks.filter(t=>!t.done); if(undone.length){ let box=document
