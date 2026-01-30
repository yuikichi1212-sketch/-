import React, { useState, useEffect } from 'react'
import { motion, AnimatePresence } from 'framer-motion'
import { Card, CardContent } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Tabs, TabsList, TabsTrigger } from '@/components/ui/tabs'
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog'
import { Switch } from '@/components/ui/switch'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { ChevronLeft, ChevronRight, Plus, Search, Upload, Download, Calendar, BarChart3, Settings, HelpCircle, StickyNote, Menu, Trash2, Edit2, Sun, Paperclip, Users, Link2, Flag } from 'lucide-react'

const days = ['日','月','火','水','木','金','土']
const pad = (n)=>String(n).padStart(2,'0')
const fmt = (d)=>`${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())}`
const parseHM = (hm)=>{ const [h,m]=hm.split(':').map(Number); return {h,m} }

const defaultSettings = {
  layoutWidth: 'standard',
  defaultView: 'month',
  weekStart: 'sun',
  emphasizeWeekend: true,
  showWeekNumber: false,
  monthWeeks: 6,
  density: 'compact',
  timeFormat: '24',
  workStart: '08:00',
  workEnd: '20:00',
  themeColor: 'blue',
  compactMode: false,
  animations: true,
  defaultEventColor: 'bg-blue-500',
  notifications: false,
  defaultReminder: '1日前',
  uiStyle: 'glass' // glass | flat
}

export default function CalendarApp() {
  const [current, setCurrent] = useState(new Date())
  const [view, setView] = useState('month')
  const [query, setQuery] = useState('')
  const [sidebarOpen, setSidebarOpen] = useState(true)
  const [panel, setPanel] = useState(null) // settings | memo | help | versions
  const [events, setEvents] = useState(()=>{ try { return JSON.parse(localStorage.getItem('events')||'{}') } catch { return {} } })
  const [settings, setSettings] = useState(()=>{ try { return { ...defaultSettings, ...(JSON.parse(localStorage.getItem('settings')||'{}')) } } catch { return defaultSettings } })
  const [memos, setMemos] = useState(()=>{ try { return JSON.parse(localStorage.getItem('memos')||'[]') } catch { return [] } })

  const [open, setOpen] = useState(false)
  const [editing, setEditing] = useState(null) // {dateKey, index}
  const [draft, setDraft] = useState({ title:'', allDay:true, start:'', end:'', location:'', reminder:settings.defaultReminder, color:settings.defaultEventColor, repeat:'繰り返さない', category:'仕事', memo:'', attendees:'', priority:'中', url:'', attachments:[] })

  useEffect(()=>{ localStorage.setItem('events', JSON.stringify(events)) }, [events])
  useEffect(()=>{ localStorage.setItem('settings', JSON.stringify(settings)) }, [settings])
  useEffect(()=>{ localStorage.setItem('memos', JSON.stringify(memos)) }, [memos])
  useEffect(()=>{ setView(settings.defaultView) }, [])

  const year = current.getFullYear()
  const month = current.getMonth()
  const start = new Date(year, month, 1)
  const end = new Date(year, month + 1, 0)
  const startDay = start.getDay()
  const total = end.getDate()

  const cells = []
  for (let i = 0; i < startDay; i++) cells.push(null)
  for (let d = 1; d <= total; d++) cells.push(d)

  const prev = () => setCurrent(new Date(year, month - 1, 1))
  const next = () => setCurrent(new Date(year, month + 1, 1))
  const today = () => setCurrent(new Date())

  const openCreate = () => {
    setEditing(null)
    setDraft({ title:'', allDay:true, start:'', end:'', location:'', reminder:settings.defaultReminder, color:settings.defaultEventColor, repeat:'繰り返さない', category:'仕事', memo:'', attendees:'', priority:'中', url:'', attachments:[] })
    setOpen(true)
  }

  const saveEvent = () => {
    if (!draft.title || !draft.start) return
    const dateKey = draft.start.split('T')[0]
    setEvents(prev=>{
      const copy = { ...prev }
      const list = copy[dateKey] ? [...copy[dateKey]] : []
      if (editing && editing.dateKey===dateKey) list[editing.index] = draft
      else list.push(draft)
      copy[dateKey] = list
      return copy
    })
    setOpen(false)
  }

  const deleteEvent = (dateKey, index) => {
    setEvents(prev=>{
      const copy = { ...prev }
      copy[dateKey] = copy[dateKey].filter((_,i)=>i!==index)
      if (copy[dateKey].length===0) delete copy[dateKey]
      return copy
    })
  }

  const editEvent = (dateKey, index) => {
    const e = events[dateKey][index]
    setEditing({ dateKey, index })
    setDraft(e)
    setOpen(true)
  }

  const filteredEvents = Object.fromEntries(Object.entries(events).filter(([k,v])=>{
    if (!query) return true
    return v.some(e=>e.title.toLowerCase().includes(query.toLowerCase()))
  }))

  const exportJSON = () => {
    const blob = new Blob([JSON.stringify(events, null, 2)], { type: 'application/json' })
    const a = document.createElement('a')
    a.href = URL.createObjectURL(blob)
    a.download = 'calendar-events.json'
    a.click()
  }

  const importJSON = (file) => {
    const r = new FileReader()
    r.onload = () => { try { setEvents(JSON.parse(r.result)) } catch {} }
    r.readAsText(file)
  }

  // UI shells
  const glass = 'backdrop-blur bg-white/60 dark:bg-slate-900/50 border border-white/30 shadow-xl rounded-2xl'
  const flat = 'bg-white dark:bg-slate-900 border border-slate-200 rounded-none shadow-none backdrop-blur-0'
  const shell = settings.uiStyle==='glass' ? glass : flat

  // Month grid
  const monthGrid = (
    <Card className={shell}>
      <CardContent className="p-4">
        <div className="grid grid-cols-7 gap-2 mb-2">
          {days.map(d => <div key={d} className={`text-center text-sm font-medium ${settings.emphasizeWeekend && (d==='日'||d==='土')?'text-red-500':''}`}>{d}</div>)}
        </div>
        <div className={`grid grid-cols-7 ${settings.density==='compact'?'gap-2':'gap-3'}`}>
          {cells.map((d,i) => {
            const key = d ? `${year}-${pad(month+1)}-${pad(d)}` : null
            const list = key ? (filteredEvents[key]||[]) : []
            return (
              <div key={i} className={`border/50 min-h-[96px] p-1 ${settings.uiStyle==='glass'?'rounded-xl':'rounded-none'}`}>
                {d && <div className="text-xs mb-1 flex justify-between"><span>{d}</span>{settings.showWeekNumber && <span className="opacity-50">W</span>}</div>}
                {d && list.map((e,idx)=>(
                  <div key={idx} className={`${e.color} text-white text-xs ${settings.uiStyle==='glass'?'rounded':'rounded-none'} px-1 py-0.5 mb-1 flex justify-between items-center`}>
                    <span>{e.title}</span>
                    <span className="flex gap-1">
                      <Edit2 size={12} className="cursor-pointer" onClick={()=>editEvent(key, idx)} />
                      <Trash2 size={12} className="cursor-pointer" onClick={()=>deleteEvent(key, idx)} />
                    </span>
                  </div>
                ))}
              </div>
            )
          })}
        </div>
      </CardContent>
    </Card>
  )

  // Week view
  const getWeekStart = (d) => {
    const date = new Date(d)
    const day = date.getDay()
    const diff = settings.weekStart==='mon' ? (day===0?-6:1-day) : -day
    date.setDate(date.getDate()+diff)
    date.setHours(0,0,0,0)
    return date
  }
  const weekStart = getWeekStart(current)
  const weekDays = Array.from({length:7},(_,i)=>{ const x=new Date(weekStart); x.setDate(x.getDate()+i); return x })
  const {h:wsH,m:wsM}=parseHM(settings.workStart)
  const {h:weH,m:weM}=parseHM(settings.workEnd)
  const hours = []
  for(let h=wsH; h<=weH; h++) hours.push(h)

  const weekView = (
    <Card className={shell}>
      <CardContent className="p-2">
        <div className="grid grid-cols-8 gap-1 text-xs">
          <div></div>
          {weekDays.map((d,i)=>(<div key={i} className={`text-center font-medium ${settings.emphasizeWeekend && (d.getDay()===0||d.getDay()===6)?'text-red-500':''}`}>{days[d.getDay()]} {d.getDate()}</div>))}
          {hours.map(h=> (
            <React.Fragment key={h}>
              <div className="text-right pr-1">{pad(h)}:00</div>
              {weekDays.map((d,di)=>{
                const key = fmt(d)
                const list = filteredEvents[key]||[]
                const slot = list.filter(e=>{
                  if(e.allDay) return false
                  const sh = new Date(e.start).getHours()
                  return sh===h
                })
                return (
                  <div key={di} className={`border/50 min-h-[48px] p-0.5 ${settings.uiStyle==='glass'?'rounded':'rounded-none'}`}>
                    {slot.map((e,ei)=>(
                      <div key={ei} className={`${e.color} text-white text-[10px] ${settings.uiStyle==='glass'?'rounded':'rounded-none'} px-0.5 py-0.5 mb-0.5`}>
                        {e.title}
                      </div>
                    ))}
                  </div>
                )
              })}
            </React.Fragment>
          ))}
        </div>
      </CardContent>
    </Card>
  )

  // Day view
  const dayKey = fmt(current)
  const dayList = filteredEvents[dayKey]||[]
  const dayView = (
    <Card className={shell}>
      <CardContent className="p-3 space-y-2">
        <div className="font-medium">{current.getMonth()+1}月{current.getDate()}日（{days[current.getDay()]}）</div>
        <div className="grid grid-cols-1 gap-1">
          {hours.map(h=>{
            const slot = dayList.filter(e=>!e.allDay && new Date(e.start).getHours()===h)
            return (
              <div key={h} className={`border/50 min-h-[40px] p-1 ${settings.uiStyle==='glass'?'rounded':'rounded-none'}`}>
                <div className="text-xs opacity-60">{pad(h)}:00</div>
                {slot.map((e,ei)=>(
                  <div key={ei} className={`${e.color} text-white text-xs ${settings.uiStyle==='glass'?'rounded':'rounded-none'} px-1 py-0.5 mb-1`}>{e.title}</div>
                ))}
              </div>
            )
          })}
          <div className="pt-2">
            {dayList.filter(e=>e.allDay).map((e,i)=>(
              <div key={i} className={`${e.color} text-white text-xs ${settings.uiStyle==='glass'?'rounded':'rounded-none'} px-1 py-0.5 mb-1`}>終日: {e.title}</div>
            ))}
          </div>
        </div>
      </CardContent>
    </Card>
  )

  const sidebar = sidebarOpen && (
    <div className={`w-72 p-4 space-y-4 ${shell}`}>
      <div className="flex items-center gap-2 text-lg font-semibold"><Calendar/> タスク表示</div>
      <Button className="w-full gap-1" onClick={openCreate}><Plus size={16}/> 作成</Button>
      <div className="flex gap-2">
        <Button variant="outline" size="sm" onClick={today}>今日</Button>
        <Button variant="outline" size="sm" onClick={()=>setView('week')}>今週</Button>
        <Button variant="outline" size="sm" onClick={()=>setView('month')}>今月</Button>
        <Button variant="outline" size="sm" className="ml-auto"><BarChart3 size={16}/> 統計</Button>
      </div>
      <div className="space-y-2">
        <div className="font-medium">今日の予定</div>
        {(events[fmt(new Date())]||[]).length===0 ? <div className="text-sm text-slate-500">予定なし</div> : (events[fmt(new Date())]||[]).map((e,i)=>(
          <div key={i} className="text-sm">• {e.title}</div>
        ))}
      </div>
      <div className="space-y-2">
        <div className="font-medium">明日の予定</div>
        {(events[fmt(new Date(Date.now()+86400000))]||[]).length===0 ? <div className="text-sm text-slate-500">予定なし</div> : (events[fmt(new Date(Date.now()+86400000))]||[]).map((e,i)=>(
          <div key={i} className="text-sm">• {e.title}</div>
        ))}
      </div>
      <div className="pt-4 border-t space-y-2">
        <Button variant="outline" className="w-full gap-2"><Upload size={16}/><label className="cursor-pointer">インポート<input type="file" className="hidden" accept="application/json" onChange={e=>e.target.files && importJSON(e.target.files[0])}/></label></Button>
        <Button variant="outline" className="w-full gap-2" onClick={exportJSON}><Download size={16}/> エクスポート</Button>
        <Button variant="outline" className="w-full gap-2" onClick={()=>setPanel('settings')}><Settings size={16}/> 設定</Button>
        <Button variant="outline" className="w-full gap-2" onClick={()=>setPanel('memo')}><StickyNote size={16}/> メモ</Button>
        <Button variant="outline" className="w-full gap-2" onClick={()=>setPanel('help')}><HelpCircle size={16}/> ヘルプ</Button>
        <Button variant="outline" className="w-full gap-2" onClick={()=>setPanel('versions')}><BarChart3 size={16}/> バージョン履歴</Button>
      </div>
    </div>
  )

  return (
    <div className={`min-h-screen ${settings.uiStyle==='glass' ? 'bg-gradient-to-br from-sky-100 via-white to-purple-100' : 'bg-slate-100 dark:bg-slate-900'} flex ${settings.compactMode?'text-sm':''}`}>
      {sidebar}
      <div className="flex-1 p-4">
        <motion.div initial={settings.animations?{opacity:0,y:20}:{}} animate={settings.animations?{opacity:1,y:0}:{}} transition={{type:'spring',stiffness:140,damping:18}} className="max-w-7xl mx-auto">
          <div className={`flex items-center justify-between mb-4 p-3 ${shell}`}>
            <div className="flex items-center gap-2">
              <Button variant="outline" size="icon" onClick={()=>setSidebarOpen(v=>!v)}><Menu/></Button>
              <h1 className="text-2xl font-semibold">ゆいきちカレンダー</h1>
            </div>
            <div className="flex gap-2 items-center">
              <div className="relative">
                <Search className="absolute left-2 top-2.5 h-4 w-4 text-slate-400" />
                <Input className="pl-8 w-40 sm:w-64" placeholder="イベントを検索..." value={query} onChange={e=>setQuery(e.target.value)} />
              </div>
              <Button variant="outline" size="icon" onClick={prev}><ChevronLeft /></Button>
              <Button variant="outline" size="icon" onClick={next}><ChevronRight /></Button>
              <Button className="gap-1" onClick={openCreate}><Plus size={16}/> 作成</Button>
            </div>
          </div>

          <Tabs value={view} onValueChange={setView} className="mb-2">
            <TabsList className={shell}>
              <TabsTrigger value="month">月</TabsTrigger>
              <TabsTrigger value="week">週</TabsTrigger>
              <TabsTrigger value="day">日</TabsTrigger>
            </TabsList>
          </Tabs>

          <AnimatePresence mode="wait">
            <motion.div key={view} initial={settings.animations?{opacity:0,scale:0.96}:{}} animate={settings.animations?{opacity:1,scale:1}:{}} exit={settings.animations?{opacity:0,scale:1.02}:{}} transition={{type:'spring',stiffness:120,damping:14}}>
              {view==='month' && monthGrid}
              {view==='week' && weekView}
              {view==='day' && dayView}
            </motion.div>
          </AnimatePresence>
        </motion.div>
      </div>

      {/* 予定作成/編集 */}
      <Dialog open={open} onOpenChange={setOpen}>
        <DialogContent className={`max-w-lg ${shell}`}>
          <DialogHeader><DialogTitle>{editing?'予定を編集':'予定を作成'}</DialogTitle></DialogHeader>
          <div className="space-y-2">
            <Input placeholder="タイトル" value={draft.title} onChange={e=>setDraft(d=>({...d, title:e.target.value}))} />
            <div className="flex gap-2 text-sm">
              <label className="flex items-center gap-1"><input type="checkbox" checked={draft.allDay} onChange={e=>setDraft(d=>({...d, allDay:e.target.checked}))}/> 終日</label>
            </div>
            <Input type="datetime-local" value={draft.start} onChange={e=>setDraft(d=>({...d, start:e.target.value}))} />
            <Input type="datetime-local" value={draft.end} onChange={e=>setDraft(d=>({...d, end:e.target.value}))} />
            <Input placeholder="場所" value={draft.location} onChange={e=>setDraft(d=>({...d, location:e.target.value}))} />
            <Select value={draft.reminder} onValueChange={v=>setDraft(d=>({...d, reminder:v}))}>
              <SelectTrigger><SelectValue placeholder="リマインダー" /></SelectTrigger>
              <SelectContent>
                <SelectItem value="なし">なし</SelectItem>
                <SelectItem value="1日前">1日前</SelectItem>
                <SelectItem value="1時間前">1時間前</SelectItem>
              </SelectContent>
            </Select>
            <Select value={draft.repeat} onValueChange={v=>setDraft(d=>({...d, repeat:v}))}>
              <SelectTrigger><SelectValue placeholder="繰り返し" /></SelectTrigger>
              <SelectContent>
                <SelectItem value="繰り返さない">繰り返さない</SelectItem>
                <SelectItem value="毎日">毎日</SelectItem>
                <SelectItem value="毎週">毎週</SelectItem>
                <SelectItem value="毎月">毎月</SelectItem>
              </SelectContent>
            </Select>
            <Select value={draft.category} onValueChange={v=>setDraft(d=>({...d, category:v}))}>
              <SelectTrigger><SelectValue placeholder="カテゴリー" /></SelectTrigger>
              <SelectContent>
                <SelectItem value="仕事">仕事</SelectItem>
                <SelectItem value="プライベート">プライベート</SelectItem>
                <SelectItem value="趣味">趣味</SelectItem>
              </SelectContent>
            </Select>
            <div className="grid grid-cols-2 gap-2">
              <div className="flex items-center gap-2"><Users size={16}/><Input placeholder="参加者（カンマ区切り）" value={draft.attendees} onChange={e=>setDraft(d=>({...d, attendees:e.target.value}))} /></div>
              <div className="flex items-center gap-2"><Flag size={16}/><Select value={draft.priority} onValueChange={v=>setDraft(d=>({...d, priority:v}))}><SelectTrigger><SelectValue/></SelectTrigger><SelectContent><SelectItem value="高">高</SelectItem><SelectItem value="中">中</SelectItem><SelectItem value="低">低</SelectItem></SelectContent></Select></div>
            </div>
            <div className="flex items-center gap-2"><Link2 size={16}/><Input placeholder="関連URL" value={draft.url} onChange={e=>setDraft(d=>({...d, url:e.target.value}))} /></div>
            <div className="flex items-center gap-2"><Paperclip size={16}/><Input type="file" multiple onChange={e=>setDraft(d=>({...d, attachments:[...e.target.files]}))} /></div>
            <textarea className={`w-full border p-2 ${settings.uiStyle==='glass'?'rounded':'rounded-none'} bg-white/60`} placeholder="メモ" value={draft.memo} onChange={e=>setDraft(d=>({...d, memo:e.target.value}))} />
            <div className="flex gap-2">
              {['bg-blue-500','bg-cyan-500','bg-lime-500','bg-pink-500','bg-purple-500','bg-orange-500','bg-red-500','bg-slate-600'].map(c=> (
                <button key={c} onClick={()=>setDraft(d=>({...d, color:c}))} className={`${c} w-6 h-6 ${settings.uiStyle==='glass'?'rounded-full':'rounded-none'} ring-2 ${draft.color===c?'ring-black':'ring-transparent'}`} />
              ))}
            </div>
            <Button className="w-full" onClick={saveEvent}>保存</Button>
          </div>
        </DialogContent>
      </Dialog>

      {/* 設定・メモ・ヘルプ・履歴 */}
      <Dialog open={!!panel} onOpenChange={()=>setPanel(null)}>
        <DialogContent className={`max-w-xl ${shell}`}>
          <DialogHeader><DialogTitle>{panel==='settings'?'設定':panel==='memo'?'メモ':panel==='help'?'ヘルプ':'バージョン履歴'}</DialogTitle></DialogHeader>
          {panel==='settings' && (
            <div className="space-y-4 text-sm">
              <div className="font-medium">表示設定</div>
              <div className="flex items-center justify-between"><span>週の開始日</span><Select value={settings.weekStart} onValueChange={v=>setSettings(s=>({...s, weekStart:v}))}><SelectTrigger className="w-40"><SelectValue/></SelectTrigger><SelectContent><SelectItem value="sun">日曜日</SelectItem><SelectItem value="mon">月曜日</SelectItem></SelectContent></Select></div>
              <div className="flex items-center justify-between"><span>週末を強調表示</span><Switch checked={settings.emphasizeWeekend} onCheckedChange={v=>setSettings(s=>({...s, emphasizeWeekend:v}))} /></div>
              <div className="flex items-center justify-between"><span>週番号を表示</span><Switch checked={settings.showWeekNumber} onCheckedChange={v=>setSettings(s=>({...s, showWeekNumber:v}))} /></div>
              <div className="flex items-center justify-between"><span>イベント密度</span><Select value={settings.density} onValueChange={v=>setSettings(s=>({...s, density:v}))}><SelectTrigger className="w-40"><SelectValue/></SelectTrigger><SelectContent><SelectItem value="compact">コンパクト</SelectItem><SelectItem value="normal">標準</SelectItem></SelectContent></Select></div>
              <div className="font-medium pt-2">時間設定</div>
              <div className="flex items-center justify-between"><span>業務開始</span><Input className="w-24" value={settings.workStart} onChange={e=>setSettings(s=>({...s, workStart:e.target.value}))} /></div>
              <div className="flex items-center justify-between"><span>業務終了</span><Input className="w-24" value={settings.workEnd} onChange={e=>setSettings(s=>({...s, workEnd:e.target.value}))} /></div>
              <div className="font-medium pt-2">テーマ設定</div>
              <div className="flex items-center justify-between"><span>UIスタイル</span><Select value={settings.uiStyle} onValueChange={v=>setSettings(s=>({...s, uiStyle:v}))}><SelectTrigger className="w-40"><SelectValue/></SelectTrigger><SelectContent><SelectItem value="glass">ガラス</SelectItem><SelectItem value="flat">フラット</SelectItem></SelectContent></Select></div>
              <div className="flex items-center justify-between"><span>コンパクトモード</span><Switch checked={settings.compactMode} onCheckedChange={v=>setSettings(s=>({...s, compactMode:v}))} /></div>
              <div className="flex items-center justify-between"><span>アニメーション</span><Switch checked={settings.animations} onCheckedChange={v=>setSettings(s=>({...s, animations:v}))} /></div>
              <div className="font-medium pt-2">イベント設定</div>
              <div className="flex items-center justify-between"><span>デフォルト色</span><Select value={settings.defaultEventColor} onValueChange={v=>setSettings(s=>({...s, defaultEventColor:v}))}><SelectTrigger className="w-40"><SelectValue/></SelectTrigger><SelectContent><SelectItem value="bg-blue-500">青</SelectItem><SelectItem value="bg-cyan-500">水色</SelectItem><SelectItem value="bg-lime-500">黄緑</SelectItem><SelectItem value="bg-pink-500">ピンク</SelectItem></SelectContent></Select></div>
              <div className="flex items-center justify-between"><span>通知</span><Switch checked={settings.notifications} onCheckedChange={v=>setSettings(s=>({...s, notifications:v}))} /></div>
              <div className="flex items-center justify-between"><span>デフォルトリマインダー</span><Select value={settings.defaultReminder} onValueChange={v=>setSettings(s=>({...s, defaultReminder:v}))}><SelectTrigger className="w-40"><SelectValue/></SelectTrigger><SelectContent><SelectItem value="なし">なし</SelectItem><SelectItem value="1日前">1日前</SelectItem><SelectItem value="1時間前">1時間前</SelectItem></SelectContent></Select></div>
            </div>
          )}
          {panel==='memo' && (
            <div className="space-y-2">
              <div className="flex gap-2"><Input placeholder="新しいメモ" onKeyDown={e=>{ if(e.key==='Enter'){ setMemos(m=>[...m, e.target.value]); e.target.value='' } }} /><Button onClick={()=>{}}><Sun/></Button></div>
              <ul className="space-y-1">{memos.map((m,i)=>(<li key={i} className={`p-2 ${settings.uiStyle==='glass'?'rounded bg-white/60':'rounded-none bg-white'} border`}>{m}</li>))}</ul>
            </div>
          )}
          {panel==='help' && <div className="text-sm">・作成ボタンで予定を追加できます。<br/>・設定からUIスタイル（ガラス/フラット）や表示を変更できます。<br/>・JSONでインポート／エクスポート可能です。</div>}
          {panel==='versions' && <div className="text-sm">v0.4 フラット完全対応 / 週・日ビュー本実装</div>}
        </DialogContent>
      </Dialog>
    </div>
  )
}
