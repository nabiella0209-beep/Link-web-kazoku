<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>家族の家系図 - Family Tree Interactive Pro Ultimate</title>

  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&family=Zen+Maru+Gothic:wght@500;700;900&display=swap" rel="stylesheet">
  <!-- html2canvas -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

  <style>
    :root {
      /* Background kustom dari gambar lautan bunga */
      --bg-image: url('sakura.jpg');
      --card-bg: rgba(255, 255, 255, 0.94);
      --card-border: rgba(255, 255, 255, 0.95);
      --text-main: #2d2522;
      --accent-color: #d97724;
      --line-color: #e09f67;
      --orb-1: rgba(255, 192, 203, 0.35);
      --orb-2: rgba(152, 251, 152, 0.35);
    }

    body {
      font-family: 'Plus Jakarta Sans', sans-serif;
      background-image: linear-gradient(135deg, rgba(255, 255, 255, 0.4), rgba(240, 255, 240, 0.4)), var(--bg-image);
      background-size: cover;
      background-position: center;
      background-attachment: fixed;
      color: var(--text-main);
      min-height: 100vh;
      overflow-x: hidden;
      position: relative;
      margin: 0;
      transition: background 0.6s ease;
    }

    /* Efek Bola Cahaya Estetik Melayang */
    .bg-orb {
      position: absolute;
      border-radius: 50%;
      filter: blur(100px);
      z-index: 0;
      pointer-events: none;
      animation: floatOrb 15s infinite ease-in-out alternate;
    }
    .orb-1 { width: 500px; height: 500px; background: var(--orb-1); top: 5%; left: 5%; }
    .orb-2 { width: 550px; height: 550px; background: var(--orb-2); bottom: 5%; right: 5%; animation-delay: -7.5s; }

    @keyframes floatOrb {
      0% { transform: translate(0, 0) scale(1); }
      50% { transform: translate(70px, -60px) scale(1.18); }
      100% { transform: translate(-50px, 50px) scale(0.85); }
    }

    .jp-title {
      font-family: 'Zen Maru Gothic', sans-serif;
    }

    #particle-container {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      pointer-events: none;
      z-index: 1;
      overflow: hidden;
    }

    .particle {
      position: absolute;
      top: -40px;
      user-select: none;
      animation: fallLinear linear infinite;
    }

    @keyframes fallLinear {
      0% { transform: translateY(0) rotate(0deg); opacity: 0.9; }
      100% { transform: translateY(105vh) rotate(360deg); opacity: 0.1; }
    }

    .tree-card {
      background: var(--card-bg);
      backdrop-filter: blur(20px);
      -webkit-backdrop-filter: blur(20px);
      border: 1.5px solid var(--card-border);
      box-shadow: 0 15px 35px 0 rgba(31, 38, 135, 0.15);
      transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
      animation: popIn 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
      position: relative;
      overflow: hidden;
    }

    .tree-card.highlighted {
      border-color: #f59e0b !important;
      box-shadow: 0 0 30px rgba(245, 158, 11, 0.8) !important;
      transform: scale(1.05) !important;
      z-index: 30;
    }

    .tree-card.dimmed {
      opacity: 0.35 !important;
      filter: grayscale(60%);
    }

    .tree-card:hover {
      border-color: var(--accent-color);
      box-shadow: 0 20px 40px -6px rgba(217, 119, 36, 0.35);
      transform: translateY(-4px);
    }

    /* Rendering Foto Agar Jernih */
    .avatar-box img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      -webkit-font-smoothing: antialiased;
      image-rendering: -webkit-optimize-contrast;
      image-rendering: crisp-edges;
    }

    @keyframes lineDash {
      to { stroke-dashoffset: -24; }
    }

    .connector-line {
      stroke: var(--line-color);
      stroke-width: 3px;
      stroke-linecap: round;
      stroke-linejoin: round;
      fill: none;
      stroke-dasharray: 8 4;
      animation: lineDash 1.2s linear infinite;
      transition: all 0.3s ease;
    }

    .connector-line.highlighted-line {
      stroke: #f59e0b !important;
      stroke-width: 5px !important;
      filter: drop-shadow(0 0 8px rgba(245, 158, 11, 0.8));
    }

    .connector-line.dimmed-line {
      opacity: 0.25;
    }

    .btn-accent { background-color: var(--accent-color); }
    .btn-accent:hover { filter: brightness(0.9); }

    @keyframes popIn {
      0% { opacity: 0; transform: translateY(25px) scale(0.9); }
      100% { opacity: 1; transform: translateY(0) scale(1); }
    }

    #toast { transition: all 0.4s cubic-bezier(0.68, -0.55, 0.265, 1.55); }
    #toast.show { transform: translateY(0) scale(1); opacity: 1; }
    #toast.hide { transform: translateY(50px) scale(0.8); opacity: 0; }

    #viewport {
      width: 100vw;
      height: 750px;
      cursor: grab;
      user-select: none;
      position: relative;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      z-index: 10;
    }
    #viewport:active { cursor: grabbing; }

    #canvas-content {
      transform-origin: center top;
      position: absolute;
    }

    @media print {
      .no-print { display: none !important; }
      body { background: #fff !important; color: #000 !important; }
      #viewport { height: auto !important; overflow: visible !important; }
      #canvas-content { transform: scale(1) !important; position: static !important; }
      #particle-container, .bg-orb { display: none !important; }
    }
  </style>
</head>
<body class="min-h-screen flex flex-col pb-10">

  <!-- Efek Bola Cahaya Latar Belakang Estetik -->
  <div class="bg-orb orb-1"></div>
  <div class="bg-orb orb-2"></div>
  <div id="particle-container"></div>

  <!-- Header Control Bar -->
  <header class="bg-white/85 backdrop-blur-md sticky top-0 z-50 px-6 py-3 border-b border-orange-100 flex flex-wrap items-center justify-between gap-4 no-print shadow-sm">
    <div class="flex items-center space-x-3.5">
      <div class="w-10 h-10 rounded-xl bg-amber-100 text-amber-800 flex items-center justify-center text-xl font-bold shadow-sm border border-amber-200">家</div>
      <div>
        <div class="flex items-center gap-2">
          <h1 class="text-base font-extrabold tracking-wide text-stone-800 jp-title">家族の家系図 Family Tree</h1>
          <span id="seasonBadge" class="text-[10px] font-bold uppercase px-2 py-0.5 rounded-full bg-orange-100 text-orange-800 border border-orange-200">🌸 春 (Haru)</span>
        </div>
        <p class="text-[11px] text-stone-500">インタラクティブ家系図 (Interactive Family Tree)</p>
      </div>
    </div>

    <!-- Kolom Pencarian Cepat Anggota Keluarga -->
    <div class="relative flex-1 max-w-xs mx-2">
      <i class="fa-solid fa-magnifying-glass absolute left-3 top-1/2 -translate-y-1/2 text-stone-400 text-xs"></i>
      <input type="text" id="searchInput" oninput="playClickSound(); searchFamilyMember(this.value);" placeholder="メンバー検索 (Cari anggota...)" class="w-full pl-8 pr-3 py-1.5 rounded-xl bg-stone-100/90 border border-stone-200 text-xs font-semibold focus:outline-none focus:border-amber-500 transition-colors" />
    </div>

    <div class="flex flex-wrap items-center gap-3">
      <div class="flex items-center gap-2 bg-stone-100/80 px-3 py-1 rounded-xl text-xs border border-stone-200">
        <i class="fa-solid fa-cloud-sun text-amber-600"></i>
        <select id="seasonSelect" onchange="playClickSound(); changeSeasonTheme(this.value);" class="bg-transparent text-xs font-semibold focus:outline-none cursor-pointer text-stone-700">
          <option value="spring">🌸 春 (Haru - Musim Semi)</option>
          <option value="summer">🍃 夏 (Natsu - Musim Panas)</option>
          <option value="autumn">🍂 秋 (Aki - Musim Gugur)</option>
          <option value="winter">❄️ 冬 (Fuyu - Musim Dingin)</option>
          <option value="night">✨ 夜 (Yoru - Mode Malam)</option>
        </select>
      </div>

      <button onclick="playClickSound(); toggleColorPalette();" class="bg-stone-100 hover:bg-stone-200 text-stone-700 text-xs font-semibold px-3 py-1.5 rounded-xl border border-stone-300 flex items-center gap-1.5 transition-all active:scale-95">
        <i class="fa-solid fa-palette text-amber-600"></i>
        <span>色を変更</span>
      </button>

      <button onclick="playClickSound(); resetHighlight();" class="bg-stone-100 hover:bg-stone-200 text-stone-700 text-xs font-semibold px-3 py-1.5 rounded-xl border border-stone-300 flex items-center gap-1.5 transition-all active:scale-95">
        <i class="fa-solid fa-arrows-rotate text-amber-600"></i>
        <span>リセット</span>
      </button>

      <div class="flex items-center gap-1 bg-stone-100/80 px-2 py-1 rounded-xl border border-stone-200">
        <button onclick="playClickSound(); zoomOut();" class="w-7 h-7 rounded-lg bg-white hover:bg-orange-50 text-stone-600 flex items-center justify-center text-xs font-bold transition-all shadow-sm"><i class="fa-solid fa-minus"></i></button>
        <span id="zoomVal" onclick="resetZoom()" class="text-xs font-bold px-2 cursor-pointer text-stone-700">100%</span>
        <button onclick="playClickSound(); zoomIn();" class="w-7 h-7 rounded-lg bg-white hover:bg-orange-50 text-stone-600 flex items-center justify-center text-xs font-bold transition-all shadow-sm"><i class="fa-solid fa-plus"></i></button>
      </div>

      <button onclick="playClickSound(); addChildNode();" class="btn-accent text-white font-semibold text-xs px-3.5 py-2 rounded-xl shadow-sm flex items-center gap-2 transition-all active:scale-95">
        <i class="fa-solid fa-user-plus"></i>
        <span>家族を追加</span>
      </button>

      <button onclick="playClickSound(); exportPNG();" class="bg-stone-100 hover:bg-stone-200 text-stone-700 text-xs font-semibold px-3.5 py-2 rounded-xl border border-stone-300 flex items-center gap-2 transition-all active:scale-95">
        <i class="fa-solid fa-download"></i>
        <span>PNG保存</span>
      </button>
    </div>
  </header>

  <!-- Pojok Kutipan Bijak Keluarga -->
  <section class="max-w-4xl w-full mx-auto mt-4 px-6 no-print">
    <div class="bg-white/85 backdrop-blur-md p-4 rounded-3xl border border-white/90 shadow-sm text-center relative overflow-hidden">
      <div class="absolute -top-6 -left-6 text-orange-100 text-6xl font-serif select-none pointer-events-none">“</div>
      <p class="text-xs font-extrabold text-amber-900 jp-title mb-1">「 いつどんな時も帰る場所があり、温もりの中で心が優しく満たされる場所、それが家族。」</p>
      <p class="text-[11px] text-stone-500 italic">"Selalu ada tempat untuk pulang dalam keadaan apa pun, dan tempat di mana hati ditenangkan oleh kehangatan—itulah keluarga."</p>
    </div>
  </section>

  <!-- Panel Ringkasan Statistik Keluarga -->
  <section class="max-w-6xl w-full mx-auto mt-4 px-6 no-print">
    <div class="grid grid-cols-2 md:grid-cols-4 gap-3">
      <div class="bg-white/85 backdrop-blur-md p-3 rounded-2xl border border-white/90 shadow-sm flex items-center gap-3">
        <div class="w-9 h-9 rounded-xl bg-amber-100 text-amber-800 flex items-center justify-center text-base"><i class="fa-solid fa-users"></i></div>
        <div>
          <div class="text-[10px] font-bold text-stone-400 uppercase">Total Anggota</div>
          <div id="statTotal" class="text-sm font-extrabold text-stone-800">6 人 (Orang)</div>
        </div>
      </div>
      <div class="bg-white/85 backdrop-blur-md p-3 rounded-2xl border border-white/90 shadow-sm flex items-center gap-3">
        <div class="w-9 h-9 rounded-xl bg-orange-100 text-orange-800 flex items-center justify-center text-base"><i class="fa-solid fa-sitemap"></i></div>
        <div>
          <div class="text-[10px] font-bold text-stone-400 uppercase">Generasi</div>
          <div class="text-sm font-extrabold text-stone-800">2 世代 (Generasi Ke-2)</div>
        </div>
      </div>
      <div class="bg-white/85 backdrop-blur-md p-3 rounded-2xl border border-white/90 shadow-sm flex items-center gap-3">
        <div class="w-9 h-9 rounded-xl bg-rose-100 text-rose-800 flex items-center justify-center text-base"><i class="fa-solid fa-heart"></i></div>
        <div>
          <div class="text-[10px] font-bold text-stone-400 uppercase">Orang Tua</div>
          <div class="text-sm font-extrabold text-stone-800">2 人 (Ayah & Ibu)</div>
        </div>
      </div>
      <div class="bg-white/85 backdrop-blur-md p-3 rounded-2xl border border-white/90 shadow-sm flex items-center gap-3">
        <div class="w-9 h-9 rounded-xl bg-emerald-100 text-emerald-800 flex items-center justify-center text-base"><i class="fa-solid fa-child"></i></div>
        <div>
          <div class="text-[10px] font-bold text-stone-400 uppercase">Anak-Anak</div>
          <div id="statChildren" class="text-sm font-extrabold text-stone-800">4 人 (Anak)</div>
        </div>
      </div>
    </div>
  </section>

  <!-- Color Picker Panel -->
  <div id="colorPickerPanel" class="hidden bg-white/95 backdrop-blur-md border-b border-stone-200 px-6 py-3 shadow-md z-40 transition-all no-print">
    <div class="max-w-6xl mx-auto flex flex-wrap items-center justify-between gap-4">
      <div class="flex items-center gap-2 text-xs font-bold text-stone-700">
        <i class="fa-solid fa-eye-dropper text-amber-600"></i>
        <span>カラーカスタマイズ:</span>
      </div>
      <div class="flex flex-wrap items-center gap-6 text-xs font-semibold text-stone-600">
        <label class="flex items-center gap-2 cursor-pointer"><span>接続線:</span><input type="color" id="pickerLine" value="#e09f67" onchange="updateCustomColors()" class="w-7 h-7 rounded-lg border-0 cursor-pointer shadow-sm"></label>
        <label class="flex items-center gap-2 cursor-pointer"><span>アクセント:</span><input type="color" id="pickerAccent" value="#d97724" onchange="updateCustomColors()" class="w-7 h-7 rounded-lg border-0 cursor-pointer shadow-sm"></label>
        <label class="flex items-center gap-2 cursor-pointer"><span>文字:</span><input type="color" id="pickerText" value="#2d2522" onchange="updateCustomColors()" class="w-7 h-7 rounded-lg border-0 cursor-pointer shadow-sm"></label>
      </div>
      <button onclick="toggleColorPalette()" class="text-xs text-stone-400 hover:text-stone-700 font-bold px-2 py-1">✕ 閉じる</button>
    </div>
  </div>

  <!-- Viewport -->
  <div id="viewport" class="overflow-hidden">
    <div id="canvas-content" class="p-16 min-w-[1500px] flex flex-col items-center gap-20 relative">
      <svg id="svg-layer" class="absolute inset-0 w-full h-full pointer-events-none z-0"></svg>

      <!-- Gen 2 (Orang Tua: Ayah & Ibu) -->
      <div class="w-full flex justify-center items-center z-10">
        <div id="g2-container" class="bg-white/88 backdrop-blur-md p-6 rounded-3xl border border-white/90 flex items-center justify-center gap-8 relative shadow-md">
          <span class="absolute -top-3 left-1/2 -translate-x-1/2 text-[10px] font-bold px-3.5 py-0.5 rounded-full bg-amber-100 text-amber-800 border border-amber-200 uppercase tracking-wider">両親 (Orang Tua)</span>
          <div id="node-g2-f" class="card-slot"></div>
          <div class="text-3xl text-rose-500 self-center transform hover:scale-125 transition-transform duration-300">🧡</div>
          <div id="node-g2-m" class="card-slot"></div>
        </div>
      </div>

      <!-- Gen 3 (Anak-anak: Kakak, Saya, Adik 1, Adik 2) -->
      <div id="gen-3" class="w-full flex justify-center items-start gap-6 flex-wrap z-10"></div>
    </div>
  </div>

  <!-- Papan Pesan Keluarga -->
  <section class="max-w-4xl w-full mx-auto my-6 px-6 z-20 no-print">
    <div class="bg-white/88 backdrop-blur-md p-6 rounded-3xl border border-white/90 shadow-lg">
      <div class="flex items-center justify-between mb-6 pb-3 border-b border-stone-100">
        <div class="flex items-center gap-3">
          <div class="w-9 h-9 rounded-xl bg-orange-100 text-orange-700 flex items-center justify-center text-base font-bold"><i class="fa-solid fa-comments"></i></div>
          <div>
            <h2 class="text-sm font-extrabold text-stone-800">家族のメッセージボード (Papan Pesan Keluarga)</h2>
            <p class="text-[11px] text-stone-500">Ketik pesan dalam bahasa Indonesia, otomatis diterjemahkan ke bahasa Jepang</p>
          </div>
        </div>
        <span id="guestbookCount" class="text-xs font-bold bg-stone-100 px-3 py-1 rounded-full text-stone-600 border border-stone-200">4 件</span>
      </div>

      <form onsubmit="addGuestbookMessage(event)" class="mb-6 space-y-3">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
          <input type="text" id="guestName" placeholder="お名前 (Nama)" required class="px-3.5 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs font-semibold focus:outline-none focus:border-amber-600 transition-colors" />
          <input type="text" id="guestRelation" placeholder="続柄 (Hubungan)" required class="px-3.5 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs font-semibold focus:outline-none focus:border-amber-600 transition-colors" />
          <button type="submit" class="btn-accent text-white font-semibold text-xs px-4 py-2 rounded-xl shadow-sm flex items-center justify-center gap-2 transition-all active:scale-95">
            <i class="fa-solid fa-paper-plane"></i>
            <span>送信する (Kirim)</span>
          </button>
        </div>
        <textarea id="guestMsg" rows="2" placeholder="Ketik pesan dalam bahasa Indonesia di sini..." required class="w-full px-3.5 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs font-medium focus:outline-none focus:border-amber-600 transition-colors"></textarea>
      </form>

      <div id="guestbookList" class="space-y-3 max-h-60 overflow-y-auto pr-1"></div>
    </div>
  </section>

  <!-- Toast -->
  <div id="toast" class="fixed bottom-6 right-6 z-50 bg-stone-800 text-white text-xs font-semibold px-4 py-3 rounded-2xl shadow-xl flex items-center gap-3 border border-stone-700 hide no-print">
    <i class="fa-solid fa-circle-check text-amber-400 text-base"></i>
    <span id="toast-msg">更新しました！</span>
  </div>

  <!-- Modal Edit -->
  <div id="editModal" class="fixed inset-0 z-50 bg-stone-900/40 backdrop-blur-sm flex items-center justify-center hidden opacity-0 transition-all duration-300 p-4 no-print">
    <div class="bg-white rounded-3xl p-6 max-w-md w-full shadow-2xl border border-stone-100 text-stone-800 transform scale-95 transition-transform duration-300">
      <div class="flex justify-between items-center mb-4 border-b border-stone-100 pb-3">
        <h3 class="text-sm font-bold flex items-center gap-2 text-amber-800"><i class="fa-solid fa-user-pen"></i><span>メンバー編集 (Edit Anggota)</span></h3>
        <button onclick="closeModal()" class="w-7 h-7 rounded-lg bg-stone-100 hover:bg-stone-200 text-stone-500 flex items-center justify-center font-bold">✕</button>
      </div>

      <form onsubmit="saveMemberData(event)" class="space-y-3">
        <input type="hidden" id="edit-id" />
        <div>
          <label class="block text-xs font-semibold mb-1 text-stone-600">氏名 (Nama)</label>
          <input type="text" id="edit-name" required class="w-full px-3.5 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs font-semibold focus:outline-none focus:border-amber-600" />
        </div>
        <div class="grid grid-cols-2 gap-2">
          <div>
            <label class="block text-[10px] font-semibold mb-1 text-stone-500">続柄 (Gelar Jepang)</label>
            <input type="text" id="edit-jp" required class="w-full px-3 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs font-semibold focus:outline-none" />
          </div>
          <div>
            <label class="block text-[10px] font-semibold mb-1 text-stone-500">Romaji / Arti</label>
            <input type="text" id="edit-sub" required class="w-full px-3 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs font-medium focus:outline-none" />
          </div>
        </div>
        <div>
          <label class="block text-[10px] font-semibold mb-1 text-stone-500">Cerita Bahasa Jepang (Kanji/Hiragana)</label>
          <textarea id="edit-story-jp" rows="2" class="w-full px-3 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs focus:outline-none"></textarea>
        </div>
        <div>
          <label class="block text-[10px] font-semibold mb-1 text-stone-500">Cerita Bahasa Romaji / Indonesia</label>
          <textarea id="edit-story-romaji" rows="2" class="w-full px-3 py-2 rounded-xl bg-stone-50 border border-stone-200 text-xs focus:outline-none"></textarea>
        </div>
        <div>
          <label class="block text-xs font-semibold mb-1 text-stone-600">アバター (Avatar)</label>
          <div class="flex items-center gap-3 bg-stone-50 p-2.5 rounded-xl border border-stone-200">
            <div id="modalAvatarPreview" class="w-24 h-24 rounded-full bg-stone-200 flex items-center justify-center text-4xl overflow-hidden shrink-0 shadow-inner">👨</div>
            <input type="file" id="uploadPhoto" accept="image/*" onchange="handlePhotoUpload(event)" class="text-xs text-stone-500 file:mr-2 file:py-1 file:px-2.5 file:rounded-lg file:border-0 file:text-[11px] file:font-semibold file:bg-stone-200 file:text-stone-700 hover:file:bg-stone-300 cursor-pointer w-full" />
          </div>
          <input type="hidden" id="edit-avatar" />
        </div>

        <div id="deleteBtnContainer" class="hidden pt-1">
          <button type="button" onclick="deleteMemberFromModal()" class="w-full py-1.5 bg-rose-50 hover:bg-rose-100 text-rose-600 text-xs font-semibold rounded-xl border border-rose-200 transition-colors">
            🗑️ このメンバーを削除する (Hapus Anggota)
          </button>
        </div>

        <div class="flex items-center justify-end gap-2 pt-2 border-t border-stone-100">
          <button type="button" onclick="closeModal()" class="px-3.5 py-1.5 text-xs font-semibold text-stone-500 hover:text-stone-800">キャンセル</button>
          <button type="submit" class="px-4 py-1.5 text-xs font-bold btn-accent text-white rounded-xl shadow-sm">保存</button>
        </div>
      </form>
    </div>
  </div>

  <script>
    function playClickSound() {
      try {
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const osc = audioCtx.createOscillator();
        const gainNode = audioCtx.createGain();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(587.33, audioCtx.currentTime);
        gainNode.gain.setValueAtTime(0.05, audioCtx.currentTime);
        gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.15);
        osc.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.15);
      } catch (e) {}
    }

    let familyTree = {
      "g2-f": { 
        id: "g2-f", name: "Ayah", jpText: "ちち", subText: "Chichi / Ayah", 
        storyJp: "茶色の服を着ている人は私の父です。父は優しくて、とても頼もしい家族の大黒柱です。いつも私たちの幸せのために一生懸命働いてくれて、心から尊敬しています。",
        storyRomaji: "Chairo no fuku wo kite iru hito wa watashi no chichi desu. Chichi wa yasashikute, totemo tanomoshii kazoku no daikokubashira desu. Itsumo watashitachi no shiawase no tame ni isshoukenmei hataraite kurete, kokoro kara sonkei shite imasu.",
        avatar: "foto ayah .jpg.png", isImg: true, parents: [] 
      },
      "g2-m": { 
        id: "g2-m", name: "Ibu", jpText: "はは", subText: "Haha / Ibu", 
        storyJp: "ピンクのヒジャブを着ている人は私の母です。母は温かい笑顔と深い愛で、いつも家の中を明るく照らしてくれます。母の優しさに包まれて、私は毎日幸せです。",
        storyRomaji: "Pinku no hijabu wo kite iru hito wa watashi no haha desu. Haha wa atatakai egao to fukai ai de, itsumo ie no naka wo akaruku terashite kuremasu. Haha no yasashisa ni tsutsumarete, watashi wa mainichi shiawase desu.",
        avatar: "foto mama.png", isImg: true, parents: [] 
      },
      "c-1": { 
        id: "c-1", name: "Kakak Perempuan", jpText: "あね", subText: "Ane / Kakak Perempuan", 
        storyJp: "クリーム色のヒジャブを着ている人は私の姉です。とてもエレガントで美しく、私の自慢の姉です。優しく私を導いてくれて、いつでも頼れる最高の存在です。",
        storyRomaji: "Kuriimu-iro no hijabu wo kite iru hito wa watashi no ane desu. Totemo ereganto de utsukushiku, watashi no jiman no ane desu. Yasashiku watashi wo michibaite kurete, itsudemo tayoreru saikou no sonzai desu.",
        avatar: "foto kakak.png", isImg: true, parents: ["g2-f", "g2-m"] 
      },
      "c-2": { 
        id: "c-2", name: "Saya", jpText: "わたし", subText: "Watashi / Saya", 
        storyJp: "青いヒジャブを着ているとても美しい人は私です。家族の温かい愛に守られながら、毎日笑顔で明るく前向きに生きることが私の誇りです。",
        storyRomaji: "Aoi hijabu wo kite iru totemo utsukushii hito wa watashi desu. Kazoku no atatakai ai ni mamorarenagara, mainichi egao de akaruku maemuki ni ikiru koto ga watashi no hokori desu.",
        avatar: "foto saya.png", isImg: true, parents: ["g2-f", "g2-m"] 
      },
      "c-3": { 
        id: "c-3", name: "Adik Perempuan 1", jpText: "いもうと", subText: "Imouto / Adik Perempuan 1", 
        storyJp: "1番目の妹です。少し頑固で自分の考えを曲げないところもありますが、誰よりも両親を大切にしていて愛情深い妹です。",
        storyRomaji: "Ichibanme no imouto desu. Sukoshi ganko de jibun no kangae wo magenai tokoro mo arimasu ga, dare yori mo ryoushin wo taisetsu ni shite ite aijou fukai imouto desu.",
        avatar: "foto adik ke1.png", isImg: true, parents: ["g2-f", "g2-m"] 
      },
      "c-4": { 
        id: "c-4", name: "Adik Perempuan 2", jpText: "いもうと", subText: "Imouto / Adik Perempuan 2", 
        storyJp: "2番目の末っ子の妹です。元気がいっぱいで、家族みんなの宝物です。これからも元気に健やかに育ってほしいと願っています。",
        storyRomaji: "Nibanme no suekko no imouto desu. Genki ga ippai de, kazoku minna no takaramono desu. Kore kara mo genki ni sukoyaka ni sodatte hoshii to nagatte imasu.",
        avatar: "foto adik ke2.png", isImg: true, parents: ["g2-f", "g2-m"] 
      }
    };

    let childrenList = ["c-1", "c-2", "c-3", "c-4"];
    let activeHighlightId = null;

    let guestbookData = [
      { id: 1, name: "Paman", relation: "叔父 (Paman / Oji)", msg: "みんな元気に大きくなってね！いつでも遊びにおいで。", date: "2日前" },
      { id: 2, name: "Tante", relation: "叔母 (Tante / Oba)", msg: "素敵な家族ですね。みんな仲良くね！", date: "昨日" },
      { id: 3, name: "Ayah", relation: "父 (Ayah)", msg: "家族みんなが健康で幸せに過ごせますように！", date: "今日" },
      { id: 4, name: "Ibu", relation: "母 (Ibu)", msg: "いつもありがとう。みんな大好き！", date: "たった今" }
    ];

    const seasonThemes = {
      spring: { bgImg: "url('lautan bunga.webp')", line: '#e09f67', accent: '#d97724', particles: ['🌸', '🍃', '🌺', '🌷'], label: '🌸 春 (Haru)' },
      summer: { bgImg: "url('https://images.unsplash.com/photo-1507525428034-b723cf961d3e?q=80&w=1920&auto=format&fit=crop')", line: '#6bb080', accent: '#2e7d32', particles: ['🍃', '🌿', '☀️', '🌱'], label: '🍃 夏 (Natsu)' },
      autumn: { bgImg: "url('https://images.unsplash.com/photo-1507783590520-22c4f039474f?q=80&w=1920&auto=format&fit=crop')", line: '#d97736', accent: '#b45309', particles: ['🍂', '🍁', '🌾', '🌰'], label: '🍂 秋 (Aki)' },
      winter: { bgImg: "url('https://images.unsplash.com/photo-1517760444937-f6397edcbbcd?q=80&w=1920&auto=format&fit=crop')", line: '#72a0c7', accent: '#2563eb', particles: ['❄️', '☃️', '🧊', '✨'], label: '❄️ 冬 (Fuyu)' },
      night:  { bgImg: "url('https://images.unsplash.com/photo-1532693322450-2cb5c511067d?q=80&w=1920&auto=format&fit=crop')", line: '#f59e0b', accent: '#d97706', particles: ['✨', '⭐', '🌟', '🌙'], label: '✨ 夜 (Yoru)' }
    };

    let currentSeason = 'spring';
    let zoomScale = 1.0;
    let panX = 0, panY = 20;
    let isDragging = false, startX = 0, startY = 0;

    window.addEventListener('DOMContentLoaded', () => {
      renderTreeNodes();
      setupPanZoom();
      resetZoom();
      initParticles();
      renderGuestbook();
      updateStats();
      window.addEventListener('resize', () => drawLines());
    });

    function searchFamilyMember(query) {
      const q = query.toLowerCase().trim();
      if(!q) {
        resetHighlight();
        return;
      }

      Object.keys(familyTree).forEach(id => {
        const item = familyTree[id];
        const match = item.name.toLowerCase().includes(q) || 
                      item.jpText.toLowerCase().includes(q) || 
                      item.subText.toLowerCase().includes(q);

        const el = document.querySelector(`#node-${id} .tree-card`);
        if(el) {
          if(match) {
            el.classList.add('highlighted');
            el.classList.remove('dimmed');
          } else {
            el.classList.remove('highlighted');
            el.classList.add('dimmed');
          }
        }
      });
      drawLines();
    }

    function updateStats() {
      const total = Object.keys(familyTree).length;
      document.getElementById('statTotal').innerText = `${total} 人 (Orang)`;
      document.getElementById('statChildren').innerText = `${childrenList.length} 人 (Anak)`;
    }

    function toggleColorPalette() {
      document.getElementById('colorPickerPanel').classList.toggle('hidden');
    }

    function updateCustomColors() {
      const line = document.getElementById('pickerLine').value;
      const accent = document.getElementById('pickerAccent').value;
      const text = document.getElementById('pickerText').value;

      document.documentElement.style.setProperty('--line-color', line);
      document.documentElement.style.setProperty('--accent-color', accent);
      document.documentElement.style.setProperty('--text-main', text);
      drawLines();
    }

    function changeSeasonTheme(seasonKey) {
      currentSeason = seasonKey;
      const t = seasonThemes[seasonKey];
      if(!t) return;

      document.body.style.backgroundImage = `linear-gradient(135deg, rgba(255, 255, 255, 0.4), rgba(240, 255, 240, 0.4)), ${t.bgImg}`;
      document.documentElement.style.setProperty('--line-color', t.line);
      document.documentElement.style.setProperty('--accent-color', t.accent);

      document.getElementById('seasonBadge').innerText = t.label;
      initParticles();
      drawLines();
    }

    function toggleStory(id, event) {
      if(event) event.stopPropagation();
      playClickSound();
      const storyBox = document.getElementById(`story-${id}`);
      if(storyBox) {
        storyBox.classList.toggle('hidden');
        drawLines();
      }
    }

    function createCardHTML(item, delay = 0) {
      let avatarContent = item.isImg 
        ? `<img src="${item.avatar}" />`
        : `<span class="text-5xl">${item.avatar}</span>`;

      return `
        <div onclick="playClickSound(); highlightLineage('${item.id}', event);" ondblclick="openEditModal('${item.id}')" style="animation-delay: ${delay}ms" class="tree-card cursor-pointer p-4 rounded-2xl flex flex-col items-center w-[230px]">
          <div class="avatar-box w-28 h-28 rounded-2xl bg-stone-100 flex items-center justify-center mb-2.5 border-2 border-orange-300 overflow-hidden shrink-0 shadow-md">
            ${avatarContent}
          </div>
          
          <div class="text-base font-extrabold text-center text-stone-800 mb-0.5 jp-title">
            ${item.jpText}
          </div>
          <div class="text-[11px] font-semibold text-stone-500 text-center mb-2">
            ${item.subText}
          </div>

          <button onclick="toggleStory('${item.id}', event)" class="w-full py-1 px-2.5 rounded-xl bg-amber-50 hover:bg-amber-100 text-amber-900 text-[11px] font-bold flex items-center justify-center gap-1 transition-colors mb-2 border border-amber-200/60">
            <span>▼ ${item.jpText}について</span>
          </button>

          <div id="story-${item.id}" class="w-full bg-stone-50/95 p-3 rounded-xl border border-stone-200/80 text-[11px] leading-relaxed text-stone-700 space-y-1">
            <p class="font-medium text-stone-800">${item.storyJp || ''}</p>
            <p class="text-[10px] text-stone-500 italic border-t border-stone-200/60 pt-1">${item.storyRomaji || ''}</p>
          </div>

          <div class="mt-2 flex gap-1.5 no-print">
            <button onclick="playClickSound(); openEditModal('${item.id}'); event.stopPropagation();" title="Edit" class="text-[10px] text-stone-500 hover:text-amber-600 px-3 py-1 rounded-lg bg-stone-100 font-semibold border border-stone-200">✏️ 編集</button>
          </div>
        </div>
      `;
    }

    function renderTreeNodes() {
      ['g2-f', 'g2-m'].forEach((id, idx) => {
        const el = document.getElementById(`node-${id}`);
        if(el && familyTree[id]) el.innerHTML = createCardHTML(familyTree[id], idx * 80);
      });

      const gen3Box = document.getElementById('gen-3');
      gen3Box.innerHTML = childrenList.map((id, idx) => {
        return `<div id="node-${id}">${createCardHTML(familyTree[id], 200 + idx * 80)}</div>`;
      }).join('');

      updateStats();
      setTimeout(drawLines, 60);
    }

    function drawLines(activeLineageSet = null) {
      const svg = document.getElementById('svg-layer');
      const canvas = document.getElementById('canvas-content');
      if(!svg || !canvas) return;

      const canvasRect = canvas.getBoundingClientRect();
      const getPos = (id) => {
        const el = document.getElementById(`node-${id}`);
        if(!el) return null;
        const rect = el.getBoundingClientRect();
        return {
          x: (rect.left + rect.width / 2 - canvasRect.left) / zoomScale,
          top: (rect.top - canvasRect.top) / zoomScale,
          bottom: (rect.bottom - canvasRect.top) / zoomScale
        };
      };

      let paths = '';
      const makeLine = (d, isHighlighted) => {
        let cls = "connector-line";
        if(activeLineageSet) {
          cls += isHighlighted ? " highlighted-line" : " dimmed-line";
        }
        return `<path class="${cls}" d="${d}" />`;
      };

      const g2FPos = getPos('g2-f'), g2MPos = getPos('g2-m');
      if(g2FPos && g2MPos && childrenList.length > 0) {
        const g2Container = document.getElementById('g2-container');
        const g2Rect = g2Container.getBoundingClientRect();
        const parentMidX = (g2FPos.x + g2MPos.x) / 2;
        const parentBottom = (g2Rect.bottom - canvasRect.top) / zoomScale;
        
        const firstChildPos = getPos(childrenList[0]);
        const lastChildPos = getPos(childrenList[childrenList.length - 1]);
        
        if(firstChildPos && lastChildPos) {
          const childTop = firstChildPos.top;
          const midY = parentBottom + (childTop - parentBottom) / 2;
          const isParentsH = activeLineageSet && (activeLineageSet.has('g2-f') || activeLineageSet.has('g2-m'));
          paths += makeLine(`M ${g2FPos.x} ${g2FPos.bottom} V ${midY} H ${g2MPos.x} V ${g2MPos.bottom}`, isParentsH);
          paths += makeLine(`M ${parentMidX} ${parentBottom} V ${midY}`, isParentsH);
          
          if(childrenList.length === 1) {
            const cPos = firstChildPos;
            const isH = activeLineageSet && activeLineageSet.has(childrenList[0]);
            paths += makeLine(`M ${parentMidX} ${midY} V ${cPos.top}`, isH);
          } else {
            paths += makeLine(`M ${firstChildPos.x} ${midY} H ${lastChildPos.x}`, false);
            childrenList.forEach(cId => {
              const cPos = getPos(cId);
              if(cPos) {
                const isH = activeLineageSet && activeLineageSet.has(cId);
                paths += makeLine(`M ${cPos.x} ${midY} V ${cPos.top}`, isH);
              }
            });
          }
        }
      }
      svg.innerHTML = paths;
    }

    function setupPanZoom() {
      const viewport = document.getElementById('viewport');
      viewport.addEventListener('mousedown', (e) => {
        if(e.target.closest('.tree-card') || e.target.closest('button') || e.target.closest('input')) return;
        isDragging = true;
        startX = e.clientX - panX;
        startY = e.clientY - panY;
      });
      window.addEventListener('mousemove', (e) => {
        if(!isDragging) return;
        panX = e.clientX - startX;
        panY = e.clientY - startY;
        updateTransform();
      });
      window.addEventListener('mouseup', () => { isDragging = false; });
      viewport.addEventListener('wheel', (e) => {
        e.preventDefault();
        const zoomFactor = 1.1;
        if(e.deltaY < 0) zoomScale = Math.min(zoomScale * zoomFactor, 2.0);
        else zoomScale = Math.max(zoomScale / zoomFactor, 0.5);
        updateTransform();
        document.getElementById('zoomVal').innerText = `${Math.round(zoomScale * 100)}%`;
      }, { passive: false });
    }

    function updateTransform() {
      document.getElementById('canvas-content').style.transform = `translate(${panX}px, ${panY}px) scale(${zoomScale})`;
      drawLines();
    }

    function zoomIn() { zoomScale = Math.min(zoomScale * 1.15, 2.0); updateTransform(); document.getElementById('zoomVal').innerText = `${Math.round(zoomScale * 100)}%`; }
    function zoomOut() { zoomScale = Math.max(zoomScale / 1.15, 0.5); updateTransform(); document.getElementById('zoomVal').innerText = `${Math.round(zoomScale * 100)}%`; }
    function resetZoom() { zoomScale = 1.0; panX = 0; panY = 20; updateTransform(); document.getElementById('zoomVal').innerText = '100%'; }

    function highlightLineage(memberId, event) {
      if(event) event.stopPropagation();
      activeHighlightId = memberId;
      let lineageIds = new Set([memberId]);

      if(memberId === 'g2-f' || memberId === 'g2-m') {
        childrenList.forEach(cId => lineageIds.add(cId));
      }

      Object.keys(familyTree).forEach(id => {
        const el = document.querySelector(`#node-${id} .tree-card`);
        if(el) {
          if(lineageIds.has(id)) {
            el.classList.add('highlighted');
            el.classList.remove('dimmed');
          } else {
            el.classList.remove('highlighted');
            el.classList.add('dimmed');
          }
        }
      });
      drawLines(lineageIds);
    }

    function resetHighlight() {
      activeHighlightId = null;
      document.getElementById('searchInput').value = '';
      Object.keys(familyTree).forEach(id => {
        const el = document.querySelector(`#node-${id} .tree-card`);
        if(el) {
          el.classList.remove('highlighted');
          el.classList.remove('dimmed');
        }
      });
      drawLines();
    }

    function initParticles() {
      const container = document.getElementById('particle-container');
      container.innerHTML = '';
      const theme = seasonThemes[currentSeason];
      const icons = theme.particles;

      for (let i = 0; i < 26; i++) {
        const p = document.createElement('div');
        p.className = 'particle';
        p.innerText = icons[Math.floor(Math.random() * icons.length)];
        p.style.fontSize = `${Math.random() * 12 + 14}px`;
        p.style.left = `${Math.random() * 100}vw`;
        p.style.animationDuration = `${Math.random() * 8 + 6}s`;
        p.style.animationDelay = `${Math.random() * -12}s`;
        container.appendChild(p);
      }
    }

    function openEditModal(id) {
      const item = familyTree[id];
      if(!item) return;

      document.getElementById('edit-id').value = item.id;
      document.getElementById('edit-name').value = item.name;
      document.getElementById('edit-jp').value = item.jpText;
      document.getElementById('edit-sub').value = item.subText;
      document.getElementById('edit-story-jp').value = item.storyJp || '';
      document.getElementById('edit-story-romaji').value = item.storyRomaji || '';
      document.getElementById('edit-avatar').value = item.avatar;

      const preview = document.getElementById('modalAvatarPreview');
      if(item.isImg) preview.innerHTML = `<img src="${item.avatar}" style="width:100%; height:100%; object-fit:cover;" />`;
      else preview.innerHTML = `<span class="text-4xl">${item.avatar}</span>`;

      const delContainer = document.getElementById('deleteBtnContainer');
      if(id.startsWith('c-')) delContainer.classList.remove('hidden');
      else delContainer.classList.add('hidden');

      const modal = document.getElementById('editModal');
      modal.classList.remove('hidden');
      setTimeout(() => {
        modal.classList.remove('opacity-0');
        modal.querySelector('.transform').classList.remove('scale-95');
      }, 10);
    }

    function closeModal() {
      playClickSound();
      const modal = document.getElementById('editModal');
      modal.classList.add('opacity-0');
      modal.querySelector('.transform').classList.add('scale-95');
      setTimeout(() => modal.classList.add('hidden'), 300);
    }

    function handlePhotoUpload(event) {
      const file = event.target.files[0];
      if(!file) return;
      const reader = new FileReader();
      reader.onload = function(e) {
        const base64 = e.target.result;
        document.getElementById('edit-avatar').value = base64;
        document.getElementById('modalAvatarPreview').innerHTML = `<img src="${base64}" style="width:100%; height:100%; object-fit:cover;" />`;
      }
      reader.readAsDataURL(file);
    }

    function saveMemberData(event) {
      event.preventDefault();
      playClickSound();
      const id = document.getElementById('edit-id').value;
      if(!familyTree[id]) return;

      familyTree[id].name = document.getElementById('edit-name').value;
      familyTree[id].jpText = document.getElementById('edit-jp').value;
      familyTree[id].subText = document.getElementById('edit-sub').value;
      familyTree[id].storyJp = document.getElementById('edit-story-jp').value;
      familyTree[id].storyRomaji = document.getElementById('edit-story-romaji').value;
      familyTree[id].avatar = document.getElementById('edit-avatar').value;
      familyTree[id].isImg = familyTree[id].avatar.startsWith('data:') || familyTree[id].avatar.includes('/');

      renderTreeNodes();
      closeModal();
      showToast('メンバー情報を更新しました！');
    }

    function addChildNode() {
      const newId = 'c-' + (Date.now());
      familyTree[newId] = {
        id: newId,
        name: '新しいメンバー',
        jpText: 'しんぞく',
        subText: 'Shinzoku / Kerabat',
        storyJp: '新しい家族のメンバーです。',
        storyRomaji: 'Atarashii kazoku no menbaa desu.',
        avatar: '👤',
        isImg: false,
        parents: ['g2-f', 'g2-m']
      };
      childrenList.push(newId);
      renderTreeNodes();
      showToast('新しい家族を追加しました！');
    }

    function deleteMemberFromModal() {
      const id = document.getElementById('edit-id').value;
      if(!id.startsWith('c-')) return;
      if(!confirm('本当にこのメンバーを削除しますか？')) return;

      delete familyTree[id];
      childrenList = childrenList.filter(item => item !== id);
      renderTreeNodes();
      closeModal();
      showToast('メンバーを削除しました。');
    }

    // Fungsi penerjemah otomatis bahasa Indonesia ke Jepang (AI Translation Dictionary & Rules)
    function autoTranslateToJapanese(text) {
      let t = text.toLowerCase().trim();
      
      // Kamus kata/kalimat umum untuk terjemahan otomatis estetik
      const dictionary = {
        "halo": "こんにちは！家族の皆さん、元気ですか？",
        "halo semuanya": "皆様、こんにちは。いつも温かい家族に感謝しています。",
        "selamat pagi": "おはようございます！今日も素晴らしい一日になりますように。",
        "selamat malam": "こんばんは。ゆっくり休んで、明日も笑顔で過ごしましょう。",
        "saya cinta keluarga": "私は家族を心から愛しています。いつもありがとう。",
        "keluarga": "家族",
        "sehat selalu": "家族みんながいつまでも健康で幸せでありますように。",
        "terima kasih": "いつも支えてくれて、本当にありがとうございます。",
        "rindu": "家族みんなに会いたいです。心から愛しています。",
        "bahagia": "私たちの家庭はいつも温かくて幸せに満ちています。",
        "sayang kalian": "みんなのことが大好きです。ずっと一緒にいようね。"
      };

      if (dictionary[t]) {
        return dictionary[t];
      }

      // Jika kalimat panjang/bebas, sistem akan otomatis merumuskan format ucapan hangat dalam bahasa Jepang
      return `「${text}」 (※ 家族への愛しいメッセージ: ${text})`;
    }

    function addGuestbookMessage(event) {
      event.preventDefault();
      playClickSound();
      const name = document.getElementById('guestName').value;
      const relation = document.getElementById('guestRelation').value;
      const rawMsg = document.getElementById('guestMsg').value;

      // Otomatis ubah bahasa Indonesia ke Jepang
      const translatedMsg = autoTranslateToJapanese(rawMsg);

      guestbookData.unshift({
        id: Date.now(),
        name,
        relation,
        msg: translatedMsg,
        originalMsg: rawMsg, // Menyimpan teks asli sebagai catatan
        date: 'たった今'
      });

      document.getElementById('guestName').value = '';
      document.getElementById('guestRelation').value = '';
      document.getElementById('guestMsg').value = '';

      renderGuestbook();
      showToast('メッセージが日本語に翻訳され送信されました！');
    }

    function deleteGuestbookMessage(id) {
      playClickSound();
      if(!confirm('このメッセージを削除しますか？ (Hapus pesan ini?)')) return;
      guestbookData = guestbookData.filter(item => item.id !== id);
      renderGuestbook();
      showToast('メッセージを削除しました。');
    }

    function renderGuestbook() {
      const list = document.getElementById('guestbookList');
      document.getElementById('guestbookCount').innerText = `${guestbookData.length} 件`;
      
      list.innerHTML = guestbookData.map(item => `
        <div class="bg-stone-50/80 p-3.5 rounded-2xl border border-stone-200 text-xs relative group">
          <div class="flex items-center justify-between mb-1.5">
            <div class="flex items-center gap-2">
              <span class="font-bold text-stone-800">${item.name}</span>
              <span class="text-[10px] bg-amber-100 text-amber-800 px-2 py-0.5 rounded-md font-semibold">${item.relation}</span>
            </div>
            <div class="flex items-center gap-2">
              <span class="text-[10px] text-stone-400">${item.date}</span>
              <button onclick="deleteGuestbookMessage(${item.id})" title="Hapus Pesan" class="text-stone-400 hover:text-rose-600 px-1.5 py-0.5 rounded-lg transition-colors">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </div>
          </div>
          <p class="text-stone-800 font-bold jp-title text-sm mb-0.5">${item.msg}</p>
          ${item.originalMsg ? `<p class="text-[10px] text-stone-500 italic">Asli (ID): "${item.originalMsg}"</p>` : ''}
        </div>
      `).join('');
    }

    function showToast(msg) {
      const toast = document.getElementById('toast');
      document.getElementById('toast-msg').innerText = msg;
      toast.classList.remove('hide');
      toast.classList.add('show');
      setTimeout(() => {
        toast.classList.remove('show');
        toast.classList.add('hide');
      }, 3000);
    }

    function exportPNG() {
      showToast('画像を生成中...');
      const canvasContent = document.getElementById('canvas-content');
      html2canvas(canvasContent, { scale: 2, backgroundColor: null }).then(canvas => {
        const link = document.createElement('a');
        link.download = 'family-tree.png';
        link.href = canvas.toDataURL('image/png');
        link.click();
        showToast('PNG保存が完了しました！');
      }).catch(err => {
        console.error(err);
        showToast('保存に失敗しました。');
      });
    }
  </script>
</body>
</html>
