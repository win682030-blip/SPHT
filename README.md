<!DOCTYPE html>
<html lang="my">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Our Shared Pet</title>
  <!-- Telegram WebApp SDK -->
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
    }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: #181926;
      color: #cad3f5;
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
      padding: 16px;
    }
    .game-card {
      background: #24273a;
      width: 100%;
      max-width: 380px;
      border-radius: 28px;
      padding: 20px;
      box-shadow: 0 12px 32px rgba(0,0,0,0.45);
      text-align: center;
      border: 1px solid #363a4f;
    }
    .header-info {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 12px;
    }
    .streak-badge {
      background: #f5a97f22;
      color: #f5a97f;
      border: 1px solid #f5a97f55;
      padding: 4px 12px;
      border-radius: 14px;
      font-size: 13px;
      font-weight: bold;
    }
    .level-badge {
      background: #c6a0f622;
      color: #c6a0f6;
      border: 1px solid #c6a0f655;
      padding: 4px 12px;
      border-radius: 14px;
      font-size: 13px;
      font-weight: bold;
    }
    .pet-avatar {
      font-size: 90px;
      margin: 10px 0;
      cursor: pointer;
      transition: transform 0.2s ease;
      display: inline-block;
    }
    .pet-avatar:active {
      transform: scale(1.15) rotate(6deg);
    }
    .pet-name-container {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      margin-bottom: 6px;
    }
    .pet-name {
      font-size: 22px;
      font-weight: bold;
      color: #8aadf4;
    }
    .btn-edit {
      background: transparent;
      border: none;
      color: #8087a2;
      font-size: 14px;
      cursor: pointer;
    }
    .user-tag {
      font-size: 12px;
      color: #a5adcb;
      margin-bottom: 16px;
    }

    /* Stats Progress */
    .stats-group {
      display: flex;
      flex-direction: column;
      gap: 10px;
      margin-bottom: 16px;
    }
    .stat-row {
      display: flex;
      flex-direction: column;
      gap: 3px;
      text-align: left;
      font-size: 12px;
    }
    .stat-header {
      display: flex;
      justify-content: space-between;
      color: #a5adcb;
    }
    .progress-track {
      background: #363a4f;
      border-radius: 8px;
      height: 8px;
      width: 100%;
      overflow: hidden;
    }
    .progress-bar {
      height: 100%;
      border-radius: 8px;
      transition: width 0.3s ease;
    }
    .bar-hunger { background: #ed8796; }
    .bar-growth { background: #a6da95; }

    /* Action Buttons */
    .actions-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-bottom: 16px;
    }
    button.action-btn {
      background: #363a4f;
      color: #cad3f5;
      border: 1px solid #494d64;
      padding: 12px;
      border-radius: 14px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
      transition: transform 0.1s, background 0.15s;
    }
    button.action-btn:active {
      transform: scale(0.96);
      background: #494d64;
    }
    .btn-feed { color: #f5a97f; }
    .btn-play { color: #eed49f; }

    /* Notification & Activity Log */
    .activity-box {
      background: #1e2030;
      border-radius: 12px;
      padding: 10px 12px;
      font-size: 12px;
      color: #b8c0e0;
      text-align: left;
      border-left: 3px solid #8aadf4;
      margin-bottom: 12px;
    }
    .activity-title {
      font-size: 11px;
      color: #8087a2;
      margin-bottom: 4px;
      text-transform: uppercase;
      font-weight: bold;
    }

    .status-panel {
      font-size: 11px;
      color: #8087a2;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
    }
    .status-dot {
      width: 7px;
      height: 7px;
      border-radius: 50%;
      background: #eed49f;
    }
    .status-dot.connected { background: #a6da95; }
  </style>
</head>
<body>

  <div class="game-card">
    <div class="header-info">
      <div class="streak-badge" id="streak-txt">🔥 256 Days</div>
      <div class="level-badge" id="level-txt">Level 12</div>
    </div>

    <div class="pet-avatar" id="avatar" onclick="tapPet()">🐘</div>

    <div class="pet-name-container">
      <div class="pet-name" id="pet-name">Baby animal</div>
      <button class="btn-edit" onclick="renamePet()">✏️</button>
    </div>
    <div class="user-tag" id="current-player">ကစားသူ: စစ်ဆေးနေပါသည်...</div>

    <div class="activity-box">
      <div class="activity-title">နောက်ဆုံး လှုပ်ရှားမှု အသိပေးချက်</div>
      <div id="latest-action">လတ်တလော လှုပ်ရှားမှု မရှိသေးပါ</div>
    </div>

    <div class="stats-group">
      <div class="stat-row">
        <div class="stat-header">
          <span>🍗 ဗိုက်ပြည့်မှု</span>
          <span id="txt-hunger">100%</span>
        </div>
        <div class="progress-track">
          <div class="progress-bar bar-hunger" id="bar-hunger" style="width: 100%;"></div>
        </div>
      </div>

      <div class="stat-row">
        <div class="stat-header">
          <span>🌱 ကြီးထွားမှု အဆင့် (EXP)</span>
          <span id="txt-exp">0 / 5</span>
        </div>
        <div class="progress-track">
          <div class="progress-bar bar-growth" id="bar-growth" style="width: 0%;"></div>
        </div>
      </div>
    </div>

    <div class="actions-grid">
      <button class="action-btn btn-feed" onclick="feedPet()">🍗 အစာကျွေးမည်</button>
      <button class="action-btn btn-play" onclick="playPet()">🎾 ကစားမည်</button>
    </div>

    <div class="status-panel">
      <div class="status-dot" id="dot"></div>
      <span id="status-text">ဆာဗာသို့ ချိတ်ဆက်နေပါသည်...</span>
    </div>
  </div>

  <script>
    // Telegram WebApp Setup
    const tg = window.Telegram?.WebApp;
    if (tg) {
      tg.ready();
      tg.expand();
    }
    const currentTelegramUser = tg?.initDataUnsafe?.user?.first_name || "သူငယ်ချင်း";
    document.getElementById("current-player").innerText = "လက်ရှိကစားသူ: " + currentTelegramUser;

    // Firebase Configuration
    const firebaseConfig = {
      databaseURL: "https://soepaingthu-default-rtdb.asia-southeast1.firebasedatabase.app"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    const petRef = db.ref("shared_pet");

    // Default Starting Values
    let pet = {
      name: "Baby animal",
      level: 12,
      streak: 256,
      hunger: 100,
      growthExp: 0,
      avatar: "🐘",
      lastFedDate: "",
      latestAction: "စတင်လည်ပတ်နေပါပြီ"
    };

    // Realtime Database Listener
    petRef.on("value", (snapshot) => {
      const data = snapshot.val();
      if (data) {
        pet = data;
        render();
        document.getElementById("dot").classList.add("connected");
        document.getElementById("status-text").innerText = "Realtime ချိတ်ဆက်ပြီးပါပြီ";
      } else {
        petRef.set(pet);
      }
    });

    function getTodayString() {
      const d = new Date();
      return `${d.getFullYear()}-${d.getMonth() + 1}-${d.getDate()}`;
    }

    function render() {
      document.getElementById("streak-txt").innerText = `🔥 ${pet.streak || 256} Days`;
      document.getElementById("level-txt").innerText = `Level ${pet.level || 12}`;
      document.getElementById("pet-name").innerText = pet.name || "Baby animal";
      document.getElementById("avatar").innerText = pet.avatar || "🐘";
      document.getElementById("latest-action").innerText = pet.latestAction || "လှုပ်ရှားမှု မရှိသေးပါ";

      const hunger = Math.max(0, Math.min(100, pet.hunger || 0));
      document.getElementById("txt-hunger").innerText = hunger + "%";
      document.getElementById("bar-hunger").style.width = hunger + "%";

      const exp = pet.growthExp || 0;
      document.getElementById("txt-exp").innerText = `${exp} / 5 အစာကျွေးချက်`;
      document.getElementById("bar-growth").style.width = `${(exp / 5) * 100}%`;
    }

    function animateAvatar(emoji) {
      const el = document.getElementById("avatar");
      el.innerText = emoji;
      setTimeout(() => {
        el.innerText = pet.avatar || "🐘";
      }, 700);
    }

    // ၁ ရက် ၁ ကြိမ် အစာကျွေးမှ ကြီးထွားပြီး Streak တက်မည့် စနစ်
    function feedPet() {
      const today = getTodayString();
      let updates = {};
      let actionMsg = "";

      if (pet.lastFedDate !== today) {
        // ဒီနေ့ ပထမဆုံးအကြိမ် ကျွေးခြင်း -> Streak တက်မည်၊ Growth EXP တက်မည်
        const newStreak = (pet.streak || 256) + 1;
        let newExp = (pet.growthExp || 0) + 1;
        let newLevel = pet.level || 12;

        if (newExp >= 5) {
          newLevel += 1;
          newExp = 0;
          actionMsg = `${currentTelegramUser} က ဒီနေ့အတွက် အစာကျွေးပြီး Level ${newLevel} သို့ ကြီးထွားလာပါပြီ! ✨`;
        } else {
          actionMsg = `${currentTelegramUser} က ဒီနေ့အတွက် အစာကျွေးခဲ့ပါသည် (Streak ${newStreak} ရက် ရောက်ရှိ)`;
        }

        updates = {
          streak: newStreak,
          growthExp: newExp,
          level: newLevel,
          lastFedDate: today,
          hunger: 100,
          latestAction: actionMsg
        };
      } else {
        // ဒီနေ့ အစာကျွေးပြီးသားဖြစ်လျှင် ဗိုက်ပြည့်ရုံသာ ပြုလုပ်မည်
        actionMsg = `${currentTelegramUser} က ထပ်မံ အစာကျွေးခဲ့ပါသည် (ဒီနေ့အတွက် Streak ပြည့်ပြီးသားဖြစ်ပါသည်)`;
        updates = {
          hunger: 100,
          latestAction: actionMsg
        };
      }

      petRef.update(updates);
      animateAvatar("🍗");
    }

    function playPet() {
      const actionMsg = `${currentTelegramUser} က အကောင်လေးနှင့် အတူ ဆော့ကစားခဲ့ပါသည် 🎾`;
      petRef.update({
        latestAction: actionMsg
      });
      animateAvatar("🎾");
    }

    function renamePet() {
      const newName = prompt("အကောင်လေး နာမည်အသစ် ရိုက်ထည့်ပါ:", pet.name);
      if (newName && newName.trim() !== "") {
        const actionMsg = `${currentTelegramUser} က နာမည်ကို '${newName.trim()}' ဟု ပြောင်းလဲခဲ့ပါသည် ✏️`;
        petRef.update({
          name: newName.trim(),
          latestAction: actionMsg
        });
      }
    }

    function tapPet() {
      animateAvatar("❤️");
    }
  </script>
</body>
</html>
