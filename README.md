
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Love Journal</title>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background-color: #ffe4ec;
    }
    .start-screen {
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      background: #fff0f5;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }
    .start-text {
      font-size: 24px;
      color: #cc4c82;
      font-weight: bold;
      text-align: center;
    }
    .start-button {
      padding: 10px 20px;
      border: none;
      background-color: #ff92ba;
      color: white;
      border-radius: 20px;
      font-size: 18px;
      cursor: pointer;
      margin-top: 20px;
    }

    .toolbar {
      display: flex;
      justify-content: space-between;
      padding: 10px 20px;
      background: rgba(255,255,255,0.6);
      position: sticky;
      top: 0;
      z-index: 10;
    }
    .gallery {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      padding: 20px;
      justify-content: center;
    }
    .photo-container {
      width: 240px;
      border-radius: 10px;
      overflow: hidden;
      background: white;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
      position: relative;
    }
    .photo-container img {
      width: 100%;
    }
    .like-button, .delete-button {
      position: absolute;
      bottom: 10px;
      background: white;
      border: none;
      border-radius: 20px;
      padding: 4px 10px;
      font-size: 14px;
      cursor: pointer;
    }
    .like-button {
      left: 10px;
    }
    .delete-button {
      right: 10px;
    }
    .diary-entry, .comment-entry {
      padding: 10px;
      font-size: 13px;
    }
    .comment-entry input {
      width: 100%;
      padding: 5px;
      font-size: 12px;
    }
    .diary-button, .music-upload {
      position: fixed;
      bottom: 20px;
      right: 20px;
      z-index: 1000;
    }

    .music-upload {
      right: 120px;
    }

    .diary-modal {
      position: fixed;
      top: 15%;
      left: 50%;
      transform: translateX(-50%);
      background: white;
      padding: 20px;
      border-radius: 15px;
      width: 80%;
      max-width: 400px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
      display: none;
      flex-direction: column;
      z-index: 1500;
    }
    .diary-modal textarea {
      width: 100%;
      height: 150px;
      margin-bottom: 10px;
    }

    audio {
      position: fixed;
      bottom: 80px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 999;
    }

  </style>
</head>
<body>

<!-- Start Screen -->
<div class="start-screen" id="start-screen">
  <div class="start-text">for ciaa, with love 🤍</div>
  <button class="start-button" onclick="start()">Masuk ke Galeri</button>
</div>

<!-- Toolbar -->
<div class="toolbar">
  <div>
    Nama: <input type="text" id="username" placeholder="Nama kamu" oninput="saveName()" />
    <input type="color" onchange="changeBackground(this.value)" />
    <input type="file" accept="image/*" id="photo-upload" onchange="uploadPhoto(event)" style="display:none">
    <button onclick="document.getElementById('photo-upload').click()">📸 Upload Foto</button>
  </div>
  <div>
    <input type="file" accept="audio/*" onchange="loadMusic(event)" class="music-upload" />
    <button class="diary-button" onclick="openDiary()">📔 Diary</button>
  </div>
</div>

<!-- Diary Modal -->
<div id="diary-modal" class="diary-modal">
  <textarea id="diary-text"></textarea>
  <button onclick="saveDiary()">Simpan Diary</button>
</div>

<!-- Gallery -->
<div id="gallery" class="gallery"></div>

<!-- Music Player -->
<audio id="music-player" controls loop></audio>

<script>
let photos = [];
let currentDiaryIndex = null;
let pressTimer = null;

function loadPhotos() {
  const stored = localStorage.getItem('photos');
  photos = stored ? JSON.parse(stored) : [];
}

function start() {
  document.getElementById('start-screen').style.display = 'none';
  loadPhotos();
  renderGallery();

  const savedName = localStorage.getItem('username');
  if (savedName) document.getElementById('username').value = savedName;

  const bg = localStorage.getItem('bg-color');
  if (bg) document.body.style.backgroundColor = bg;
}

function renderGallery() {
  const gallery = document.getElementById('gallery');
  gallery.innerHTML = '';
  photos.forEach((photo, index) => {
    const container = document.createElement('div');
    container.className = 'photo-container';

    const img = document.createElement('img');
    img.src = photo.url;

    // Untuk PC: klik kanan untuk hapus
    container.addEventListener('contextmenu', (e) => {
      e.preventDefault(); // Menghindari menu klik kanan default
      if (confirm('Hapus foto ini?')) {
        photos.splice(index, 1);
        savePhotos();
        loadPhotos();
        renderGallery();
      }
    });

    // Untuk HP: long press untuk hapus
    let pressTimer = null;
    container.addEventListener('mousedown', () => {
      pressTimer = setTimeout(() => {
        if (confirm('Hapus foto ini?')) {
          photos.splice(index, 1);
          savePhotos();
          loadPhotos();
          renderGallery();
        }
      }, 1000); // 1 detik untuk long press
    });
    container.addEventListener('mouseup', () => {
      clearTimeout(pressTimer);
    });
    container.addEventListener('mouseleave', () => {
      clearTimeout(pressTimer);
    });

    const likeBtn = document.createElement('button');
    likeBtn.className = 'like-button';
    likeBtn.innerHTML = photo.liked ? '❤️' : '🤍';
    likeBtn.onclick = (e) => {
      e.stopPropagation();
      photo.liked = !photo.liked;
      savePhotos();
      renderGallery();
    };

    const delBtn = document.createElement('button');
    delBtn.className = 'delete-button';
    delBtn.innerHTML = 'Hapus';
    delBtn.style.display = 'none';

    const diary = document.createElement('div');
    diary.className = 'diary-entry';
    diary.innerText = photo.diary || '';

    const comment = document.createElement('div');
    comment.className = 'comment-entry';
    comment.innerHTML = `<input type="text" placeholder="Tulis komentar..." 
      onkeydown="if(event.key==='Enter'){addComment(${index}, this.value); this.value='';}" />`;

    container.appendChild(img);
    container.appendChild(likeBtn);
    container.appendChild(delBtn);
    container.appendChild(diary);
    container.appendChild(comment);
    gallery.appendChild(container);
  });
}

function uploadPhoto(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = function(e) {
    photos.unshift({
      url: e.target.result,
      liked: false,
      diary: '',
      comments: []
    });
    savePhotos();
    loadPhotos();
    renderGallery();
  };
  reader.readAsDataURL(file);
}

function openDiary(index = null) {
  document.getElementById('diary-modal').style.display = 'flex';
  currentDiaryIndex = index;
  document.getElementById('diary-text').value = index !== null ? (photos[index].diary || '') : '';
}

function saveDiary() {
  if (currentDiaryIndex !== null) {
    photos[currentDiaryIndex].diary = document.getElementById('diary-text').value;
    savePhotos();
    loadPhotos();
    renderGallery();
  }
  document.getElementById('diary-modal').style.display = 'none';
}

function addComment(index, comment) {
  if (!photos[index].comments) photos[index].comments = [];
  photos[index].comments.push(comment);
  savePhotos();
  loadPhotos();
  renderGallery();
}

function savePhotos() {
  localStorage.setItem('photos', JSON.stringify(photos));
}

function saveName() {
  const name = document.getElementById('username').value;
  localStorage.setItem('username', name);
}

function changeBackground(color) {
  document.body.style.backgroundColor = color;
  localStorage.setItem('bg-color', color);
}

function loadMusic(event) {
  const file = event.target.files[0];
  if (file) {
    const url = URL.createObjectURL(file);
    const audio = document.getElementById('music-player');
    audio.src = url;
    audio.play();
  }
}
</script>

</body>
</html>
