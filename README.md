import React from 'react';
import { 
  Plus, Search, Calendar as CalendarIcon, FileText, Upload, Download, 
  Cloud, ChevronLeft, ChevronRight, MoreHorizontal, Settings, 
  HelpCircle, Clock, PieChart, LayoutGrid
} from 'lucide-react';

export default function CalendarApp() {
  // 画像の日付データ：2026年1月（開始日は2025/12/28）
  const calendarData = [
    { day: 28, isPrevMonth: true, events: [{ title: 'ゴルフ', color: 'bg-[#84cc16]' }] },
    { day: 29, isPrevMonth: true, events: [{ title: 'nainai来るよ', color: 'bg-[#00bfa5]' }] },
    { day: 30, isPrevMonth: true, events: [] },
    { day: 31, isPrevMonth: true, events: [] },
    { day: 1, isPrevMonth: false, events: [] },
    { day: 2, isPrevMonth: false, events: [] },
    { day: 3, isPrevMonth: false, events: [] },
    { day: 4, isPrevMonth: false, events: [{ title: 'ゴルフ', color: 'bg-[#84cc16]' }] },
    { day: 5, isPrevMonth: false, events: [] },
    { day: 6, isPrevMonth: false, events: [] },
    { day: 7, isPrevMonth: false, events: [{ title: '始業式', color: 'bg-[#3b82f6]' }] },
    { day: 8, isPrevMonth: false, events: [] },
    { day: 9, isPrevMonth: false, events: [] },
    { day: 10, isPrevMonth: false, events: [] },
    { day: 11, isPrevMonth: false, events: [{ title: 'ゴルフ', color: 'bg-[#84cc16]' }] },
    { day: 12, isPrevMonth: false, events: [] },
    { day: 13, isPrevMonth: false, events: [] },
    { day: 14, isPrevMonth: false, events: [] },
    { day: 15, isPrevMonth: false, events: [] },
    { day: 16, isPrevMonth: false, events: [] },
    { day: 17, isPrevMonth: false, events: [] },
    { day: 18, isPrevMonth: false, events: [{ title: 'ゴルフ', color: 'bg-[#84cc16]' }] },
    { day: 19, isPrevMonth: false, events: [] },
    { day: 20, isPrevMonth: false, events: [{ title: '半日', color: 'bg-[#0ea5e9]' }] },
    { day: 21, isPrevMonth: false, events: [{ title: '休み', color: 'bg-[#0ea5e9]' }] },
    { day: 22, isPrevMonth: false, events: [{ title: '休み', color: 'bg-[#0ea5e9]' }] },
    { day: 23, isPrevMonth: false, events: [{ title: '英検間近', color: 'bg-[#ec4899]' }] },
    { day: 24, isPrevMonth: false, events: [{ title: 'ゴルフ', color: 'bg-[#84cc16]' }, { title: '武道大会', color: 'bg-[#f97316]' }] },
    { day: 25, isPrevMonth: false, events: [{ title: '英検当日', color: 'bg-[#ef4444]' }, { title: 'ゴルフ', color: 'bg-[#84cc16]' }] },
    { day: 26, isPrevMonth: false, events: [] },
    { day: 27, isPrevMonth: false, events: [{ title: '英検対策', color: 'bg-[#a855f7]' }] },
    { day: 28, isPrevMonth: false, events: [{ title: '百人一首大会', color: 'bg-[#ef4444]' }] },
    { day: 29, isPrevMonth: false, events: [{ title: '英検対策', color: 'bg-[#a855f7]' }] },
    { day: 30, isPrevMonth: false, isToday: true, events: [{ title: '上小田井クリニック', color: 'bg-[#475569]' }] },
    { day: 31, isPrevMonth: false, events: [] },
  ];

  return (
    <div className="flex h-screen bg-[#f3f4f6] text-slate-700 font-sans overflow-hidden">
      {/* ---------------- 左サイドバー ---------------- */}
      <aside className="w-[280px] bg-[#f8f9fc] flex flex-col border-r border-slate-200 shadow-sm z-10">
        
        {/* 左上黒ヘッダー */}
        <div className="bg-black text-white h-12 flex items-center px-4 justify-between shrink-0">
          <div className="flex items-center gap-2">
            <ChevronLeft size={16} className="text-gray-400" />
            <span className="font-bold text-sm">タスク表示</span>
          </div>
        </div>

        {/* メニューエリア */}
        <div className="p-4 flex-1 overflow-y-auto">
          <div className="mb-6">
            <p className="text-xs text-slate-400 font-bold mb-3 px-2">機能</p>
            <nav className="space-y-1">
              <NavItem icon={<FileText size={18} className="text-purple-500" />} text="AI予定作成" activeText />
              <NavItem icon={<FileText size={18} className="text-orange-400" />} text="メモ" />
              <NavItem icon={<Upload size={18} className="text-emerald-500" />} text="インポート" />
              <NavItem icon={<Download size={18} className="text-blue-500" />} text="エクスポート" />
              <NavItem icon={<Cloud size={18} className="text-sky-500" />} text="カレンダー連携" />
            </nav>
          </div>

          <div className="h-px bg-slate-200 my-4" />

          {/* 表示切り替えタブ */}
          <div className="bg-white p-1 rounded-full border border-slate-200 flex mb-6 shadow-sm">
            <TabButton active text="今日" />
            <TabButton text="今週" />
            <TabButton text="今月" />
            <TabButton icon={<PieChart size={14} />} text="統計" />
          </div>

          {/* 今日の予定カード */}
          <div className="space-y-4">
            <ScheduleCard title="今日の予定" count="1件" icon={<CalendarIcon size={16} />}>
              <div className="mt-3">
                <div className="font-bold text-sm text-slate-800">上小田井クリニック</div>
                <div className="flex items-center gap-1 text-xs text-slate-400 mt-1">
                  <Clock size={12} />
                  <span>00:00</span>
                </div>
              </div>
            </ScheduleCard>

            <ScheduleCard title="明日の予定" count="0件" icon={<CalendarIcon size={16} />}>
              <div className="mt-8 text-center text-xs text-slate-400 pb-4">
                予定なし
              </div>
            </ScheduleCard>
          </div>
        </div>

        {/* 左下フッターメニュー */}
        <div className="p-4 border-t border-slate-200 space-y-3 bg-[#f8f9fc]">
          <FooterItem icon={<Clock size={16} />} text="バージョン履歴" />
          <FooterItem icon={<HelpCircle size={16} />} text="ヘルプ" />
          <FooterItem icon={<Settings size={16} />} text="設定" />
        </div>
      </aside>

      {/* ---------------- メインコンテンツ ---------------- */}
      <main className="flex-1 flex flex-col min-w-0 bg-white">
        
        {/* ヘッダーエリア */}
        <header className="px-6 py-4 border-b border-slate-100">
          <div className="flex justify-between items-start mb-4">
            <div>
              <h1 className="text-2xl font-bold text-slate-900">2026年1月</h1>
              <p className="text-xs text-slate-500 mt-1">1月30日(金) 13:39</p>
            </div>
            <button className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg text-sm font-bold flex items-center gap-1 shadow-md transition-colors">
              <Plus size={16} /> 作成
            </button>
          </div>

          <div className="flex justify-between items-center">
            <div className="flex items-center gap-2">
              <button className="px-3 py-1.5 border border-slate-200 rounded-md text-sm font-medium hover:bg-slate-50 bg-white">今日</button>
              <div className="flex border border-slate-200 rounded-md bg-white">
                <button className="p-1.5 hover:bg-slate-50 border-r border-slate-200"><ChevronLeft size={16} /></button>
                <button className="p-1.5 hover:bg-slate-50"><ChevronRight size={16} /></button>
              </div>
            </div>
            
            <div className="flex gap-2 text-sm text-slate-500 bg-white border border-slate-200 rounded-lg p-1">
              <button className="px-3 py-1 rounded-md bg-blue-600 text-white shadow-sm">月</button>
              <button className="px-3 py-1 rounded-md hover:bg-slate-100">週</button>
              <button className="px-3 py-1 rounded-md hover:bg-slate-100">日</button>
            </div>
          </div>
        </header>

        {/* 検索バー */}
        <div className="px-6 py-3 bg-[#fafafa]">
          <div className="relative">
            <Search className="absolute left-3 top-2.5 text-slate-400" size={18} />
            <input 
              type="text" 
              placeholder="イベントを検索..." 
              className="w-full pl-10 pr-4 py-2 bg-white border border-slate-200 rounded-full text-sm focus:outline-none focus:ring-2 focus:ring-blue-100 transition-shadow"
            />
          </div>
        </div>

        {/* カレンダーグリッド */}
        <div className="flex-1 overflow-auto p-4 bg-[#fafafa]">
          <div className="bg-white rounded-xl border border-slate-200 shadow-sm h-full flex flex-col overflow-hidden">
            {/* 曜日ヘッダー */}
            <div className="grid grid-cols-7 border-b border-slate-100">
              {['日', '月', '火', '水', '木', '金', '土'].map((day, i) => (
                <div key={i} className={`py-3 text-center text-xs font-bold ${i === 0 ? 'text-red-500' : i === 6 ? 'text-blue-500' : 'text-slate-500'}`}>
                  {day}
                </div>
              ))}
            </div>

            {/* 日付セル */}
            <div className="grid grid-cols-7 grid-rows-5 flex-1 divide-x divide-y divide-slate-100">
              {calendarData.map((data, index) => (
                <div key={index} className={`p-2 min-h-[100px] flex flex-col relative ${data.isToday ? 'bg-slate-50' : 'bg-white'}`}>
                  
                  {/* 日付数字 */}
                  <span className={`text-xs font-medium mb-1 ${data.isToday ? 'bg-slate-700 text-white w-6 h-6 flex items-center justify-center rounded-full' : 'text-slate-500'}`}>
                    {data.day}
                  </span>

                  {/* イベントリスト */}
                  <div className="flex flex-col gap-1 overflow-y-auto">
                    {data.events.map((event, i) => (
                      <div key={i} className={`${event.color} text-white text-[10px] px-2 py-1 rounded-[4px] font-medium truncate shadow-sm`}>
                        {event.title}
                      </div>
                    ))}
                  </div>
                </div>
              ))}
            </div>
          </div>
        </div>
      </main>
    </div>
  );
}

// ---- サブコンポーネント ----

const NavItem = ({ icon, text, activeText }) => (
  <button className={`w-full flex items-center gap-3 px-2 py-2 rounded-lg text-sm font-medium hover:bg-slate-100 transition-colors ${activeText ? 'text-purple-600' : 'text-slate-600'}`}>
    {icon}
    <span>{text}</span>
  </button>
);

const FooterItem = ({ icon, text }) => (
  <button className="flex items-center gap-3 text-slate-500 text-xs font-medium hover:text-slate-800 transition-colors w-full">
    {icon}
    <span>{text}</span>
  </button>
);

const TabButton = ({ text, active, icon }) => (
  <button className={`flex-1 flex items-center justify-center gap-1 py-1.5 text-xs font-bold rounded-full transition-all ${active ? 'bg-slate-900 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}`}>
    {icon}
    {text}
  </button>
);

const ScheduleCard = ({ title, count, icon, children }) => (
  <div className="bg-white border border-slate-200 rounded-xl p-4 shadow-sm hover:shadow-md transition-shadow">
    <div className="flex justify-between items-center mb-2">
      <div className="flex items-center gap-2">
        <span className="text-slate-400">{icon}</span>
        <h4 className="font-bold text-sm text-slate-800">{title}</h4>
      </div>
      <span className="bg-slate-100 text-slate-600 text-[10px] px-2 py-0.5 rounded-full font-bold">{count}</span>
    </div>
    {children}
  </div>
);
