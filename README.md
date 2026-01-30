<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>カレンダーアプリ完全版</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: "Noto Sans JP", system-ui, sans-serif;
    }
    body {
      background: #f5f5f5;
    }
    .app {
      display: flex;
      height: 100vh;
    }
    /* サイドバー */
    .sidebar {
      width: 260px;
      background: #ffffff;
      border-right: 1px solid #ddd;
      padding: 16px;
      display: flex;
      flex-direction: column;
      gap: 16px;
    }
    .sidebar h2 {
      font-size: 16px;
      margin-bottom: 4px;
    }
    .event-list {
      min-height: 60px;
      border: 1px solid #eee;
      border-radius: 6px;
      padding: 8px;
      font-size: 13px;
    }
    .event-item {
      margin-bottom: 4px;
    }
    .sidebar-buttons {
      display: flex;
      flex-direction: column;
      gap: 8px;
      margin-top: auto;
    }
    .sidebar button {
      padding: 8px;
      font-size: 13px;
      border-radius: 6px;
      border: 1px solid #ccc;
      background: #fafafa;
      cursor: pointer;
    }
    .sidebar button:hover {
      background: #eaeaea;
    }
    .memo-area {
      border: 1px solid #eee;
      border-radius: 6px;
      padding: 6px;
      font-size: 12px;
      min-height: 60px;
      white-space: pre-wrap;
    }

    /* メイン */
    .main {
      flex: 1;
      display: flex;
      flex-direction: column;
      padding: 16px;
    }
    .top-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 12px;
    }
    .month-nav {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .month-nav button {
      padding: 4px 8px;
      cursor: pointer;
    }
    #month-label {
      font-size: 18px;
      font-weight: 600;
    }
    .view-switch button {
      padding: 4px 10px;
      margin-left: 4px;
      border-radius: 16px;
      border: 1px solid #ccc;
      background: #fafafa;
      font-size: 13px;
    }
    .view-switch .active {
      background: #1976d2;
      color: #fff;
      border-color: #1976d2;
    }

    /* カレンダー */
    .calendar {
      background: #ffffff;
      border-radius: 8px;
      padding: 12px;
      box-shadow: 0 0 4px rgba(0,0,0,0.05);
      display: flex;
      flex-direction: column;
      height: calc(100% - 40px);
    }
    .calendar-header {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      text-align: center;
      font-size: 13px;
      margin-bottom: 8px;
      color: #555;
    }
    .calendar-grid {
      flex: 1;
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      grid-auto-rows: 1fr;
      gap: 2px;
    }
    .day-cell {
      border: 1px solid #eee;
      padding: 4px;
      font-size: 12px;
      position: relative;
      cursor: pointer;
    }
    .day-cell:hover {
      background: #fafafa;
    }
    .day-number {
      font-size: 12px;
      margin-bottom: 2px;
    }
    .event-chip {
      display: block;
      border-radius: 10px;
      padding: 2px 4px;
      font-size: 10px;
      color: #fff;
      margin-bottom: 2px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    .event-golf { background: #4caf50; }
    .event-exam { background: #2196f3; }
    .event-holiday { background: #f44336; }
    .event-clinic { background: #9c27b0; }
    .event-tournament { background: #ff9800; }
    .event-other { background: #607d8b; }

    /* モーダル */
    .modal-backdrop {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.3);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 10;
    }
    .modal {
      background: #fff;
      border-radius: 8px;
      padding: 16px;
      width: 320px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.2);
      font-size: 14px;
    }
    .modal h3 {
      margin-bottom: 8px;
    }
    .modal label {
      display: block;
      margin-top: 8px;
      font-size: 13px;
    }
    .modal input, .modal select, .modal textarea {
      width: 100%;
      margin-top: 4px;
      padding: 4px;
      font-size: 13px;
    }
    .modal textarea {
      resize: vertical;
      min-height: 60px;
    }
    .modal-buttons {
      margin-top: 12px;
      display: flex;
      justify-content: flex-end;
      gap: 8px;
    }
    .modal-buttons button {
      padding: 4px 10px;
      font-size: 13px;
      cursor: pointer;
    }
    .danger {
      background: #f44336;
      color: #fff;
      border: none;
    }
    .primary {
      background: #1976d2;
      color: #fff;
      border: none;
    }
  </style>
</head>
<body>
  <div class="app">
    <aside class="sidebar">
      <h2>今日の予定</h2>
      <div id="today-events" class="event-list"></div>

      <h2>明日の予定</h2>
      <div id="tomorrow-events" class="event-list"></div>

      <h2>メモ</h2>
      <div id="memo-view" class="memo-area"></div>

      <div class="sidebar-buttons">
        <button id="ai-schedule-btn">AIスケジュール作成</button>
        <button id="memo-edit-btn">メモ編集</button>
        <button id="export-btn">エクスポート</button>
        <button id="import-btn">インポート</button>
        <button id="clear-btn">全データ削除</button>
      </div>
    </aside>

    <main class="main">
      <header class="top-bar">
        <div class="month-nav">
          <button id="prev-month">＜</button>
          <span id="month-label"></span>
          <button id="next-month">＞</button>
        </div>
        <div class="view-switch">
          <button class="active">月</button>
          <button disabled>週</button>
          <button disabled>日</button>
        </div>
      </header>

      <section class="calendar">
        <div class="calendar-header">
          <div>日</div><div>月</div><div>火</div><div>水</div><div>木</div><div>金</div><div>土</div>
        </div>
        <div id="calendar-grid" class="calendar-grid"></div>
      </section>
    </main>
  </div>

  <div id="modal-root"></div>

  <script>
    // ===== データ管理 =====
    const STORAGE_KEY_EVENTS = "calendar_events_v1";
    const STORAGE_KEY_MEMO = "calendar_memo_v1";

    let events = loadEvents();
    let memoText = loadMemo();

    let current = new Date();
    let currentYear = current.getFullYear();
    let currentMonth = current.getMonth();

    const monthLabel = document.getElementById("month-label");
    const calendarGrid = document.getElementById("calendar-grid");
    const todayEventsEl = document.getElementById("today-events");
    const tomorrowEventsEl = document.getElementById("tomorrow-events");
    const memoViewEl = document.getElementById("memo-view");
    const modalRoot = document.getElementById("modal-root");

    document.getElementById("prev-month").addEventListener("click", () => {
      currentMonth--;
      if (currentMonth < 0) {
        currentMonth = 11;
        currentYear--;
      }
      renderCalendar();
    });

    document.getElementById("next-month").addEventListener("click", () => {
      currentMonth++;
      if (currentMonth > 11) {
        currentMonth = 0;
        currentYear++;
      }
      renderCalendar();
    });

    document.getElementById("memo-edit-btn").addEventListener("click", () => {
      const text = prompt("メモを入力してください：", memoText || "");
      if (text !== null) {
        memoText = text;
        saveMemo();
        renderMemo();
      }
    });

    document.getElementById("export-btn").addEventListener("click", () => {
      const data = {
        events,
        memo: memoText,
      };
      const blob = new Blob([JSON.stringify(data, null, 2)], { type: "application/json" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "calendar_export.json";
      a.click();
      URL.revokeObjectURL(url);
    });

    document.getElementById("import-btn").addEventListener("click", () => {
      const input = document.createElement("input");
      input.type = "file";
      input.accept = "application/json";
      input.onchange = (e) => {
        const file = e.target.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = () => {
          try {
            const data = JSON.parse(reader.result);
            if (Array.isArray(data.events)) events = data.events;
            if (typeof data.memo === "string") memoText = data.memo;
            saveEvents();
            saveMemo();
            renderAll();
          } catch (err) {
            alert("インポートに失敗しました。");
          }
        };
        reader.readAsText(file);
      };
      input.click();
    });

    document.getElementById("clear-btn").addEventListener("click", () => {
      if (confirm("全ての予定とメモを削除しますか？")) {
        events = [];
        memoText = "";
        saveEvents();
        saveMemo();
        renderAll();
      }
    });

    document.getElementById("ai-schedule-btn").addEventListener("click", () => {
      runAISchedule();
    });

    function loadEvents() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY_EVENTS);
        if (!raw) return [];
        return JSON.parse(raw);
      } catch {
        return [];
      }
    }

    function saveEvents() {
      localStorage.setItem(STORAGE_KEY_EVENTS, JSON.stringify(events));
    }

    function loadMemo() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY_MEMO);
        if (!raw) return "";
        return raw;
      } catch {
        return "";
      }
    }

    function saveMemo() {
      localStorage.setItem(STORAGE_KEY_MEMO, memoText || "");
    }

    // ===== レンダリング =====
    function renderCalendar() {
      const firstDay = new Date(currentYear, currentMonth, 1);
      const lastDay = new Date(currentYear, currentMonth + 1, 0);
      const startWeekday = firstDay.getDay();
      const daysInMonth = lastDay.getDate();

      monthLabel.textContent = `${currentYear}年 ${currentMonth + 1}月`;

      calendarGrid.innerHTML = "";

      for (let i = 0; i < startWeekday; i++) {
        const cell = document.createElement("div");
        cell.className = "day-cell";
        calendarGrid.appendChild(cell);
      }

      for (let day = 1; day <= daysInMonth; day++) {
        const cell = document.createElement("div");
        cell.className = "day-cell";

        const num = document.createElement("div");
        num.className = "day-number";
        num.textContent = day;
        cell.appendChild(num);

        const dateStr = formatDate(currentYear, currentMonth + 1, day);
        const dayEvents = events.filter(e => e.date === dateStr);

        dayEvents.forEach(ev => {
          const chip = document.createElement("span");
          chip.className = `event-chip event-${ev.type || "other"}`;
          chip.textContent = ev.title;
          cell.appendChild(chip);
        });

        cell.addEventListener("click", () => {
          openEventModal(dateStr);
        });

        calendarGrid.appendChild(cell);
      }

      renderSideBar();
    }

    function renderSideBar() {
      const today = new Date();
      const todayStr = formatDate(today.getFullYear(), today.getMonth() + 1, today.getDate());
      const tomorrow = new Date(today.getTime() + 24 * 60 * 60 * 1000);
      const tomorrowStr = formatDate(tomorrow.getFullYear(), tomorrow.getMonth() + 1, tomorrow.getDate());

      const todayList = events.filter(e => e.date === todayStr);
      const tomorrowList = events.filter(e => e.date === tomorrowStr);

      todayEventsEl.innerHTML = todayList.length
        ? todayList.map(e => `<div class="event-item">${e.title}</div>`).join("")
        : "<div>予定はありません</div>";

      tomorrowEventsEl.innerHTML = tomorrowList.length
        ? tomorrowList.map(e => `<div class="event-item">${e.title}</div>`).join("")
        : "<div>予定はありません</div>";
    }

    function renderMemo() {
      memoViewEl.textContent = memoText || "メモはまだありません。";
    }

    function renderAll() {
      renderCalendar();
      renderMemo();
    }

    // ===== モーダルで予定編集 =====
    function openEventModal(dateStr) {
      const dayEvents = events.filter(e => e.date === dateStr);

      const backdrop = document.createElement("div");
      backdrop.className = "modal-backdrop";

      const modal = document.createElement("div");
      modal.className = "modal";

      const title = document.createElement("h3");
      title.textContent = `${dateStr} の予定`;
      modal.appendChild(title);

      const list = document.createElement("div");
      if (dayEvents.length === 0) {
        list.textContent = "予定はありません。";
      } else {
        dayEvents.forEach((ev, index) => {
          const row = document.createElement("div");
          row.style.marginTop = "6px";
          row.textContent = `${index + 1}. ${ev.title}`;
          row.style.cursor = "pointer";
          row.addEventListener("click", () => {
            openEditForm(dateStr, ev);
            backdrop.remove();
          });
          list.appendChild(row);
        });
      }
      modal.appendChild(list);

      const addBtn = document.createElement("button");
      addBtn.textContent = "予定を追加";
      addBtn.style.marginTop = "10px";
      addBtn.addEventListener("click", () => {
        openEditForm(dateStr, null);
        backdrop.remove();
      });
      modal.appendChild(addBtn);

      const closeArea = document.createElement("div");
      closeArea.className = "modal-buttons";
      const closeBtn = document.createElement("button");
      closeBtn.textContent = "閉じる";
      closeBtn.addEventListener("click", () => backdrop.remove());
      closeArea.appendChild(closeBtn);
      modal.appendChild(closeArea);

      backdrop.appendChild(modal);
      modalRoot.innerHTML = "";
      modalRoot.appendChild(backdrop);
    }

    function openEditForm(dateStr, eventObj) {
      const isEdit = !!eventObj;

      const backdrop = document.createElement("div");
      backdrop.className = "modal-backdrop";

      const modal = document.createElement("div");
      modal.className = "modal";

      const title = document.createElement("h3");
      title.textContent = isEdit ? "予定を編集" : "予定を追加";
      modal.appendChild(title);

      const labelTitle = document.createElement("label");
      labelTitle.textContent = "タイトル";
      const inputTitle = document.createElement("input");
      inputTitle.value = eventObj ? eventObj.title : "";
      labelTitle.appendChild(inputTitle);
      modal.appendChild(labelTitle);

      const labelType = document.createElement("label");
      labelType.textContent = "種類（色分け）";
      const selectType = document.createElement("select");
      const types = [
        { value: "golf", label: "ゴルフ（緑）" },
        { value: "exam", label: "英検・勉強（青）" },
        { value: "holiday", label: "休み（赤）" },
        { value: "clinic", label: "病院（紫）" },
        { value: "tournament", label: "大会（オレンジ）" },
        { value: "other", label: "その他（グレー）" },
      ];
      types.forEach(t => {
        const opt = document.createElement("option");
        opt.value = t.value;
        opt.textContent = t.label;
        if (eventObj && eventObj.type === t.value) opt.selected = true;
        selectType.appendChild(opt);
      });
      labelType.appendChild(selectType);
      modal.appendChild(labelType);

      const labelMemo = document.createElement("label");
      labelMemo.textContent = "メモ（任意）";
      const textareaMemo = document.createElement("textarea");
      textareaMemo.value = eventObj ? (eventObj.note || "") : "";
      labelMemo.appendChild(textareaMemo);
      modal.appendChild(labelMemo);

      const btnArea = document.createElement("div");
      btnArea.className = "modal-buttons";

      if (isEdit) {
        const delBtn = document.createElement("button");
        delBtn.textContent = "削除";
        delBtn.className = "danger";
        delBtn.addEventListener("click", () => {
          if (confirm("この予定を削除しますか？")) {
            events = events.filter(e => e !== eventObj);
            saveEvents();
            renderAll();
            backdrop.remove();
          }
        });
        btnArea.appendChild(delBtn);
      }

      const cancelBtn = document.createElement("button");
      cancelBtn.textContent = "キャンセル";
      cancelBtn.addEventListener("click", () => backdrop.remove());
      btnArea.appendChild(cancelBtn);

      const saveBtn = document.createElement("button");
      saveBtn.textContent = "保存";
      saveBtn.className = "primary";
      saveBtn.addEventListener("click", () => {
        const titleVal = inputTitle.value.trim();
        if (!titleVal) {
          alert("タイトルを入力してください。");
          return;
        }
        const newEvent = {
          id: eventObj ? eventObj.id : generateId(),
          date: dateStr,
          title: titleVal,
          type: selectType.value,
          note: textareaMemo.value.trim(),
        };
        if (isEdit) {
          events = events.map(e => e.id === eventObj.id ? newEvent : e);
        } else {
          events.push(newEvent);
        }
        saveEvents();
        renderAll();
        backdrop.remove();
      });
      btnArea.appendChild(saveBtn);

      modal.appendChild(btnArea);
      backdrop.appendChild(modal);
      modalRoot.innerHTML = "";
      modalRoot.appendChild(backdrop);
    }

    // ===== AIスケジュール（簡易版） =====
    function runAISchedule() {
      const keyword = prompt("どんな予定を自動で入れますか？（例：英検対策）", "英検対策");
      if (!keyword) return;

      const countStr = prompt("何回入れますか？（例：5）", "5");
      const count = parseInt(countStr, 10);
      if (!count || count <= 0) return;

      const today = new Date();
      let added = 0;
      let dayCursor = new Date(today);

      while (added < count && added < 60) {
        const dateStr = formatDate(dayCursor.getFullYear(), dayCursor.getMonth() + 1, dayCursor.getDate());
        const isWeekend = dayCursor.getDay() === 0 || dayCursor.getDay() === 6;
        const hasEvent = events.some(e => e.date === dateStr);

        if (!hasEvent && !isWeekend) {
          events.push({
            id: generateId(),
            date: dateStr,
            title: keyword,
            type: "exam",
            note: "AIスケジュールで自動追加",
          });
          added++;
        }
        dayCursor.setDate(dayCursor.getDate() + 1);
      }

      saveEvents();
      renderAll();
      alert(`${added}件の予定を自動で追加しました。`);
    }

    // ===== ユーティリティ =====
    function formatDate(y, m, d) {
      return [
        y,
        String(m).padStart(2, "0"),
        String(d).padStart(2, "0"),
      ].join("-");
    }

    function generateId() {
      return "ev_" + Math.random().toString(36).slice(2, 10);
    }

    // 初期描画
    renderAll();
  </script>
</body>
</html>
