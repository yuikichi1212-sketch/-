<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>カレンダーアプリ 2026年1月</title>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        /* --- 基本設定 --- */
        :root {
            --bg-main: #f3f4f6;
            --bg-sidebar: #f8f9fc;
            --border-color: #e2e8f0;
            --text-primary: #1e293b;
            --text-secondary: #64748b;
            --color-blue: #2563eb;
            --color-green-golf: #84cc16;
            --color-green-nainai: #00bfa5;
            --color-blue-school: #3b82f6;
            --color-sky-work: #0ea5e9;
            --color-pink-exam: #ec4899;
            --color-orange-budo: #f97316;
            --color-red-exam: #ef4444;
            --color-purple-study: #a855f7;
            --color-dark-clinic: #475569;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-main);
            color: var(--text-primary);
            height: 100vh;
            overflow: hidden;
        }

        /* --- レイアウト全体 --- */
        .app-container {
            display: flex;
            height: 100vh;
        }

        /* --- サイドバー --- */
        .sidebar {
            width: 280px;
            background-color: var(--bg-sidebar);
            border-right: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
            flex-shrink: 0;
        }

        .sidebar-header {
            background-color: #000;
            color: #fff;
            height: 48px;
            display: flex;
            align-items: center;
            padding: 0 16px;
            font-weight: bold;
            font-size: 14px;
        }

        .sidebar-header i { margin-right: 8px; color: #9ca3af; }

        .sidebar-content {
            padding: 16px;
            flex: 1;
            overflow-y: auto;
        }

        .menu-label {
            font-size: 12px;
            font-weight: bold;
            color: var(--text-secondary);
            margin-bottom: 12px;
            padding-left: 8px;
        }

        .menu-item {
            display: flex;
            align-items: center;
            padding: 10px 12px;
            border-radius: 8px;
            font-size: 14px;
            font-weight: 500;
            color: var(--text-secondary);
            text-decoration: none;
            margin-bottom: 4px;
            transition: background-color 0.2s;
            cursor: pointer;
        }
        .menu-item:hover { background-color: #f1f5f9; }
        .menu-item.active { color: var(--color-purple-study); }
        .menu-item i { margin-right: 12px; }
        .text-purple { color: var(--color-purple-study); }
        .text-orange { color: var(--color-orange-budo); }
        .text-green { color: var(--color-green-nainai); }
        .text-blue { color: var(--color-blue); }
        .text-sky { color: var(--color-sky-work); }

        .divider { height: 1px; background-color: var(--border-color); margin: 16px 0; }

        .tab-group {
            display: flex;
            background-color: #fff;
            padding: 4px;
            border-radius: 9999px;
            border: 1px solid var(--border-color);
            margin-bottom: 24px;
        }
        .tab-button {
            flex: 1;
            text-align: center;
            padding: 6px 0;
            font-size: 12px;
            font-weight: bold;
            color: var(--text-secondary);
            border-radius: 9999px;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 4px;
        }
        .tab-button.active { background-color: #1e293b; color: #fff; }

        .schedule-card {
            background-color: #fff;
            border: 1px solid var(--border-color);
            border-radius: 12px;
            padding: 16px;
            margin-bottom: 16px;
            box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
        }
        .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
        }
        .card-title {
            display: flex;
            align-items: center;
            font-weight: bold;
            font-size: 14px;
            gap: 8px;
        }
        .card-title i { color: var(--text-secondary); }
        .card-count {
            background-color: #f1f5f9;
            color: var(--text-secondary);
            font-size: 10px;
            padding: 2px 8px;
            border-radius: 999px;
            font-weight: bold;
        }
        .task-item {
            font-weight: bold;
            font-size: 14px;
        }
        .task-time {
            display: flex;
            align-items: center;
            gap: 4px;
            font-size: 12px;
            color: var(--text-secondary);
            margin-top: 4px;
        }
        .no-task {
            text-align: center;
            font-size: 12px;
            color: var(--text-secondary);
            padding: 20px 0;
        }

        .sidebar-footer {
            padding: 16px;
            border-top: 1px solid var(--border-color);
        }
        .footer-item {
            display: flex;
            align-items: center;
            gap: 12px;
            font-size: 12px;
            color: var(--text-secondary);
            font-weight: 500;
            padding: 8px 0;
            cursor: pointer;
        }

        /* --- メインコンテンツ --- */
        .main-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            background-color: #fff;
            min-width: 0; /* Gridのはみ出し防止 */
        }

        /* ヘッダー */
        .main-header {
            padding: 16px 24px;
            border-bottom: 1px solid var(--border-color);
        }
        .header-top {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            margin-bottom: 16px;
        }
        .page-title h1 { font-size: 24px; font-weight: bold; }
        .current-date { font-size: 12px; color: var(--text-secondary); margin-top: 4px; }
        .create-btn {
            background-color: var(--color-blue);
            color: #fff;
            border: none;
            padding: 8px 16px;
            border-radius: 8px;
            font-size: 14px;
            font-weight: bold;
            display: flex;
            align-items: center;
            gap: 4px;
            cursor: pointer;
            box-shadow: 0 4px 6px -1px rgba(37, 99, 235, 0.2);
        }

        .header-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .nav-group { display: flex; align-items: center; gap: 8px; }
        .btn-today {
            padding: 6px 12px;
            border: 1px solid var(--border-color);
            border-radius: 6px;
            font-size: 14px;
            font-weight: 500;
            background-color: #fff;
            cursor: pointer;
        }
        .nav-arrows {
            display: flex;
            border: 1px solid var(--border-color);
            border-radius: 6px;
            background-color: #fff;
        }
        .nav-arrow-btn {
            padding: 6px;
            background: none;
            border: none;
            cursor: pointer;
            display: flex;
            align-items: center;
        }
        .nav-arrow-btn:first-child { border-right: 1px solid var(--border-color); }

        .view-toggle-group {
            display: flex;
            gap: 8px;
            padding: 4px;
            border: 1px solid var(--border-color);
            border-radius: 8px;
        }
        .view-btn {
            padding: 4px 12px;
            border: none;
            background: none;
            font-size: 14px;
            border-radius: 6px;
            color: var(--text-secondary);
            cursor: pointer;
        }
        .view-btn.active { background-color: var(--color-blue); color: #fff; }

        /* 検索バー */
        .search-container {
            padding: 12px 24px;
            background-color: #fafafa;
        }
        .search-input-wrapper {
            position: relative;
        }
        .search-icon {
            position: absolute;
            left: 12px;
            top: 50%;
            transform: translateY(-50%);
            color: var(--text-secondary);
        }
        .search-input {
            width: 100%;
            padding: 10px 16px 10px 40px;
            border: 1px solid var(--border-color);
            border-radius: 9999px;
            font-size: 14px;
            outline: none;
        }
        .search-input:focus { border-color: var(--color-blue); }

        /* カレンダーグリッド本体 */
        .calendar-wrapper {
            flex: 1;
            overflow-y: auto;
            padding: 16px;
            background-color: #fafafa;
        }
        .calendar-grid-container {
            background-color: #fff;
            border: 1px solid var(--border-color);
            border-radius: 12px;
            height: 100%;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        .weekdays-header {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            border-bottom: 1px solid #f1f5f9;
        }
        .weekday {
            padding: 12px 0;
            text-align: center;
            font-size: 12px;
            font-weight: bold;
            color: var(--text-secondary);
        }
        .text-red { color: var(--color-red-exam); }

        .calendar-days-grid {
            flex: 1;
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            grid-template-rows: repeat(5, 1fr); /* 5週分 */
        }
        .day-cell {
            border-right: 1px solid #f1f5f9;
            border-bottom: 1px solid #f1f5f9;
            padding: 8px;
            min-height: 100px;
            display: flex;
            flex-direction: column;
        }
        .day-cell:nth-child(7n) { border-right: none; } /* 右端の線消す */
        .day-cell.bg-today-cell { background-color: #f8fafc; }

        .day-number {
            font-size: 12px;
            font-weight: 500;
            color: var(--text-secondary);
            margin-bottom: 4px;
            width: 24px;
            height: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
        }
        .day-number.today {
            background-color: #334155;
            color: #fff;
        }

        .events-list {
            display: flex;
            flex-direction: column;
            gap: 4px;
            overflow-y: auto;
        }
        .event-chip {
            font-size: 10px;
            color: #fff;
            padding: 4px 8px;
            border-radius: 4px;
            font-weight: 500;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            box-shadow: 0 1px 2px 0 rgba(0,0,0,0.05);
        }

        /* イベント背景色クラス */
        .bg-green { background-color: var(--color-green-golf); }
        .bg-green-dark { background-color: var(--color-green-nainai); }
        .bg-blue { background-color: var(--color-blue-school); }
        .bg-sky { background-color: var(--color-sky-work); }
        .bg-pink { background-color: var(--color-pink-exam); }
        .bg-orange { background-color: var(--color-orange-budo); }
        .bg-red { background-color: var(--color-red-exam); }
        .bg-purple { background-color: var(--color-purple-study); }
        .bg-dark { background-color: var(--color-dark-clinic); }

    </style>
</head>
<body>

<div class="app-container">
    <aside class="sidebar">
        <div class="sidebar-header">
            <i data-lucide="chevron-left" width="16"></i>
            <span>タスク表示</span>
        </div>
        <div class="sidebar-content">
            <div class="menu-section">
                <p class="menu-label">機能</p>
                <div class="menu-item active"><i data-lucide="sparkles" class="text-purple" width="18"></i>AI予定作成</div>
                <div class="menu-item"><i data-lucide="file-text" class="text-orange" width="18"></i>メモ</div>
                <div class="menu-item"><i data-lucide="upload" class="text-green" width="18"></i>インポート</div>
                <div class="menu-item"><i data-lucide="download" class="text-blue" width="18"></i>エクスポート</div>
                <div class="menu-item"><i data-lucide="cloud" class="text-sky" width="18"></i>カレンダー連携</div>
            </div>
            <div class="divider"></div>
            <div class="tab-group">
                <div class="tab-button active">今日</div>
                <div class="tab-button">今週</div>
                <div class="tab-button">今月</div>
                <div class="tab-button"><i data-lucide="pie-chart" width="14"></i>統計</div>
            </div>
            <div class="schedule-cards">
                <div class="schedule-card">
                    <div class="card-header">
                        <div class="card-title"><i data-lucide="calendar" width="16"></i>今日の予定</div>
                        <div class="card-count">1件</div>
                    </div>
                    <div class="task-item">上小田井クリニック</div>
                    <div class="task-time"><i data-lucide="clock" width="12"></i>00:00</div>
                </div>
                <div class="schedule-card">
                    <div class="card-header">
                        <div class="card-title"><i data-lucide="calendar" width="16"></i>明日の予定</div>
                        <div class="card-count">0件</div>
                    </div>
                    <div class="no-task">予定なし</div>
                </div>
            </div>
        </div>
        <div class="sidebar-footer">
            <div class="footer-item"><i data-lucide="history" width="16"></i>バージョン履歴</div>
            <div class="footer-item"><i data-lucide="help-circle" width="16"></i>ヘルプ</div>
            <div class="footer-item"><i data-lucide="settings" width="16"></i>設定</div>
        </div>
    </aside>

    <main class="main-content">
        <header class="main-header">
            <div class="header-top">
                <div class="page-title">
                    <h1>2026年1月</h1>
                    <p class="current-date">1月30日(金) 13:39</p>
                </div>
                <button class="create-btn"><i data-lucide="plus" width="16"></i>作成</button>
            </div>
            <div class="header-controls">
                <div class="nav-group">
                    <button class="btn-today">今日</button>
                    <div class="nav-arrows">
                        <button class="nav-arrow-btn"><i data-lucide="chevron-left" width="16"></i></button>
                        <button class="nav-arrow-btn"><i data-lucide="chevron-right" width="16"></i></button>
                    </div>
                </div>
                <div class="view-toggle-group">
                    <button class="view-btn active">月</button>
                    <button class="view-btn">週</button>
                    <button class="view-btn">日</button>
                </div>
            </div>
        </header>

        <div class="search-container">
            <div class="search-input-wrapper">
                <i data-lucide="search" class="search-icon" width="18"></i>
                <input type="text" class="search-input" placeholder="イベントを検索...">
            </div>
        </div>

        <div class="calendar-wrapper">
            <div class="calendar-grid-container">
                <div class="weekdays-header">
                    <div class="weekday text-red">日</div>
                    <div class="weekday">月</div>
                    <div class="weekday">火</div>
                    <div class="weekday">水</div>
                    <div class="weekday">木</div>
                    <div class="weekday">金</div>
                    <div class="weekday text-blue">土</div>
                </div>
                <div class="calendar-days-grid" id="calendar-grid">
                    </div>
            </div>
        </div>
    </main>
</div>

<script>
    // Lucide Icons の初期化
    lucide.createIcons();

    // カレンダーデータ定義 (2026年1月表示用: 2025/12/28 - 2026/1/31)
    const calendarData = [
        // 前月(12月)
        { day: 28, events: [{ title: 'ゴルフ', colorClass: 'bg-green' }] },
        { day: 29, events: [{ title: 'nainai来るよ', colorClass: 'bg-green-dark' }] },
        { day: 30, events: [] },
        { day: 31, events: [] },
        // 当月(1月)
        { day: 1, events: [] },
        { day: 2, events: [] },
        { day: 3, events: [] },
        { day: 4, events: [{ title: 'ゴルフ', colorClass: 'bg-green' }] },
        { day: 5, events: [] },
        { day: 6, events: [] },
        { day: 7, events: [{ title: '始業式', colorClass: 'bg-blue' }] },
        { day: 8, events: [] },
        { day: 9, events: [] },
        { day: 10, events: [] },
        { day: 11, events: [{ title: 'ゴルフ', colorClass: 'bg-green' }] },
        { day: 12, events: [] },
        { day: 13, events: [] },
        { day: 14, events: [] },
        { day: 15, events: [] },
        { day: 16, events: [] },
        { day: 17, events: [] },
        { day: 18, events: [{ title: 'ゴルフ', colorClass: 'bg-green' }] },
        { day: 19, events: [] },
        { day: 20, events: [{ title: '半日', colorClass: 'bg-sky' }] },
        { day: 21, events: [{ title: '休み', colorClass: 'bg-sky' }] },
        { day: 22, events: [{ title: '休み', colorClass: 'bg-sky' }] },
        { day: 23, events: [{ title: '英検間近', colorClass: 'bg-pink' }] },
        { day: 24, events: [{ title: 'ゴルフ', colorClass: 'bg-green' }, { title: '武道大会', colorClass: 'bg-orange' }] },
        { day: 25, events: [{ title: '英検当日', colorClass: 'bg-red' }, { title: 'ゴルフ', colorClass: 'bg-green' }] },
        { day: 26, events: [] },
        { day: 27, events: [{ title: '英検対策', colorClass: 'bg-purple' }] },
        { day: 28, events: [{ title: '百人一首大会', colorClass: 'bg-red' }] },
        { day: 29, events: [{ title: '英検対策', colorClass: 'bg-purple' }] },
        { day: 30, isToday: true, events: [{ title: '上小田井クリニック', colorClass: 'bg-dark' }] },
        { day: 31, events: [] },
    ];

    // カレンダーグリッドを描画する関数
    function renderCalendar() {
        const gridContainer = document.getElementById('calendar-grid');
        gridContainer.innerHTML = ''; // 一旦クリア

        calendarData.forEach(data => {
            // 日付セルを作成
            const cell = document.createElement('div');
            cell.className = `day-cell ${data.isToday ? 'bg-today-cell' : ''}`;

            // 日付の数字
            const dayNumber = document.createElement('div');
            dayNumber.className = `day-number ${data.isToday ? 'today' : ''}`;
            dayNumber.textContent = data.day;
            cell.appendChild(dayNumber);

            // イベントリスト
            const eventsList = document.createElement('div');
            eventsList.className = 'events-list';
            
            data.events.forEach(event => {
                const chip = document.createElement('div');
                chip.className = `event-chip ${event.colorClass}`;
                chip.textContent = event.title;
                eventsList.appendChild(chip);
            });
            cell.appendChild(eventsList);

            gridContainer.appendChild(cell);
        });
    }

    // 初期化実行
    renderCalendar();

</script>
</body>
</html>
