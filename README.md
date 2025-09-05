<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>اپ کنکور</title>
<style>
body {
  font-family: "Tahoma", sans-serif;
  direction: rtl;
  background: #f2f2f2;
  margin: 0;
  padding: 0;
}
.container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
}
h1, h2 { text-align: center; }
.task-form, .video-form {
  background: #fff;
  padding: 10px;
  margin-bottom: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}
input, select, button {
  display: block;
  width: 100%;
  margin: 8px 0;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
}
button {
  background: #4A90E2;
  color: white;
  border: none;
  cursor: pointer;
}
button:hover { background: #357ABD; }
ul { list-style: none; padding: 0; }
li {
  background: #fff;
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}
</style>
<link rel="manifest" href="manifest.json">
</head>
<body>
<div class="container">
<h1>اپ کنکور</h1>

<div class="task-form">
<input id="taskTitle" placeholder="عنوان کار">
<input id="taskTime" type="time">
<select id="taskCategory">
<option>درس</option>
<option>ورزش</option>
<option>استراحت</option>
</select>
<input id="taskVideo" placeholder="لینک ویدئو (اختیاری)">
<button onclick="addTask()">➕ اضافه کردن کار</button>
</div>

<h2>کارهای امروز</h2>
<ul id="taskList"></ul>

<div class="video-form">
<input id="videoTitle" placeholder="عنوان ویدئو">
<input id="videoURL" placeholder="لینک ویدئو">
<input id="videoTime" type="time">
<button onclick="addVideo()">🎬 اضافه کردن ویدئو</button>
</div>

<h2>ویدئوهای روزانه</h2>
<ul id="videoList"></ul>
</div>

<script>
let tasks = JSON.parse(localStorage.getItem('tasks') || '[]');
let videos = JSON.parse(localStorage.getItem('videos') || '[]');

function renderTasks() {
  const list = document.getElementById('taskList');
  list.innerHTML = '';
  tasks.forEach(task => {
    const li = document.createElement('li');
    li.innerHTML = `<b>${task.title}</b> (${task.category}) - ${task.time}
      ${task.video ? `<button onclick="playVideo('${task.video}')">▶️ ویدئو</button>` : ''}`;
    li.style.borderRight = `8px solid ${getColor(task.category)}`;
    list.appendChild(li);
  });
}

function renderVideos() {
  const list = document.getElementById('videoList');
  list.innerHTML = '';
  videos.forEach(video => {
    const li = document.createElement('li');
    li.innerHTML = `<b>${video.title}</b> - ${video.time}
      <button onclick="playVideo('${video.url}')">▶️ پخش</button>`;
    li.style.borderRight = "8px solid #FF9800";
    list.appendChild(li);
  });
}

function addTask() {
  const title = document.getElementById('taskTitle').value;
  const time = document.getElementById('taskTime').value;
  const category = document.getElementById('taskCategory').value;
  const video = document.getElementById('taskVideo').value;
  if(title && time){
    tasks.push({title, time, category, video});
    localStorage.setItem('tasks', JSON.stringify(tasks));
    renderTasks();
  }
}

function addVideo() {
  const title = document.getElementById('videoTitle').value;
  const url = document.getElementById('videoURL').value;
  const time = document.getElementById('videoTime').value;
  if(title && url && time){
    videos.push({title, url, time});
    localStorage.setItem('videos', JSON.stringify(videos));
    renderVideos();
  }
}

function playVideo(url){ window.open(url, '_blank'); }

function getColor(category){
  if(category === "درس") return "#2196F3";
  if(category === "ورزش") return "#4CAF50";
  if(category === "استراحت") return "#FF9800";
  return "#9E9E9E";
}

renderTasks();
renderVideos();
</script>
</body>
</html>
