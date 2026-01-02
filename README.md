function finance view ({expenses, setExpenses}) {
  const [showAdd, setShowAdd] = useState(false);
  const RATE_JPY_TO_TWD = 0.22;
  const summary = useMemo(() => {
    let totalTWD = 0; let totalJPY = 0; let debt = 0; 
    expenses.forEach(ex => {
      const amount = parseFloat(ex.amount);
      const isJPY = ex.currency === 'JPY';
      if (isJPY) { totalJPY += amount; totalTWD += amount * RATE_JPY_TO_TWD; } else { totalTWD += amount; totalJPY += amount / RATE_JPY_TO_TWD; }
      const amountInTWD = isJPY ? amount * RATE_JPY_TO_TWD : amount;
      let shareAmount = ex.split === 'split' ? amountInTWD / 2 : (ex.split === 'HongYi' || ex.split === 'YuTing') ? amountInTWD : amountInTWD / 2;
      if (ex.payer === 'YuTing') { if (ex.split === 'split') debt += shareAmount; if (ex.split === 'HongYi') debt += amountInTWD; } else { if (ex.split === 'split') debt -= shareAmount; if (ex.split === 'YuTing') debt -= amountInTWD; }
    });
    return { totalTWD, totalJPY, debt };
  }, [expenses]);
  const handleAdd = (data) => { setExpenses(prev => [...prev, { ...data, id: Date.now() }]); setShowAdd(false); };
  const handleDelete = (id) => { if (window.confirm("確定刪除此筆記帳？")) setExpenses(prev => prev.filter(e => e.id !== id)); };
  const groupedExpenses = useMemo(() => {
    const groups = {};
    expenses.forEach(ex => { if (!groups[ex.date]) groups[ex.date] = []; groups[ex.date].push(ex); });
    return Object.keys(groups).sort((a,b) => new Date(b) - new Date(a)).map(date => ({ date, items: groups[date].sort((a,b) => b.time.localeCompare(a.time)) }));
  }, [expenses]);

  return (
    <div className="p-4 bg-[#F8F8F6] min-h-full pb-24 overflow-y-auto">
      <div className="bg-white p-6 rounded-3xl shadow-sm border border-gray-100 mb-6">
        <h2 className="text-gray-400 text-xs font-bold tracking-widest mb-4">TOTAL EXPENSES</h2>
        <div className="flex justify-between items-end mb-6"><div><div className="flex items-baseline gap-1"><span className="text-sm font-bold text-gray-400">TWD</span><span className="text-3xl font-bold text-[#4A4A4A]">${Math.round(summary.totalTWD).toLocaleString()}</span></div><div className="text-xs text-gray-400 mt-1">(約 JPY ¥{Math.round(summary.totalJPY).toLocaleString()})</div></div></div>
        <div className="bg-[#F8F8F6] rounded-xl p-4 flex items-center justify-between border border-gray-200"><div className="flex items-center gap-2"><Calculator size={18} className="text-gray-400" /><span className="text-sm font-bold text-gray-600">結算</span></div><div className={`text-sm px-3 py-1.5 rounded-lg font-bold ${summary.debt > 0 ? 'bg-[#EBF5FB] text-[#2E86C1]' : summary.debt < 0 ? 'bg-[#FDEDEC] text-[#C0392B]' : 'bg-gray-100 text-gray-500'}`}>{Math.round(summary.debt) === 0 ? '目前結清' : summary.debt > 0 ? <span>宏一 應付 ${Math.abs(Math.round(summary.debt)).toLocaleString()}</span> : <span>于婷 應付 ${Math.abs(Math.round(summary.debt)).toLocaleString()}</span>}</div></div>
      </div>
      <div className="space-y-6">{groupedExpenses.map(group => (<div key={group.date}><h3 className="text-xs font-bold text-gray-400 mb-3 ml-2 flex items-center gap-2"><Calendar size={12} /> {group.date}</h3><div className="space-y-3">{group.items.map(ex => { const catStyle = EXPENSE_CATEGORY_STYLES[ex.category] || EXPENSE_CATEGORY_STYLES['其他']; return (<div key={ex.id} className="bg-white p-4 rounded-2xl shadow-sm border border-gray-50 flex flex-col gap-3 relative group"><button onClick={() => handleDelete(ex.id)} className="absolute top-3 right-3 text-gray-300 hover:text-red-400 p-1"><Trash2 size={14}/></button><div className="flex justify-between items-start pr-6"><div className="flex gap-3"><div className={`w-10 h-10 rounded-full flex items-center justify-center ${catStyle.bg} ${catStyle.text}`}>{catStyle.icon}</div><div><div className="font-bold text-gray-800 text-base">{ex.name}</div><div className="text-xs text-gray-400 mt-0.5 flex items-center gap-2"><span>{ex.time}</span><span className={`px-1.5 rounded text-xs ${catStyle.bg} ${catStyle.text} bg-opacity-50`}>{ex.category}</span></div></div></div><div className="text-right"><div className="font-bold text-gray-800 text-lg font-noto">{ex.currency === 'JPY' ? '¥' : '$'}{parseFloat(ex.amount).toLocaleString()}</div></div></div><div className="h-px w-full bg-gray-100"></div><div className="flex justify-between items-center text-xs text-gray-500"><div className="flex items-center gap-4"><div className="flex items-center gap-1"><span className="text-gray-400">付款:</span><span className={`font-bold px-2 py-0.5 rounded-full ${ex.payer==='YuTing'?'bg-pink-50 text-pink-600':'bg-blue-50 text-blue-600'}`}>{ex.payer==='YuTing'?'于婷':'宏一'}</span></div><div className="flex items-center gap-1"><span className="text-gray-400">分帳:</span><span className="font-medium text-gray-700">{ex.split==='split' ? '平分' : ex.split==='YuTing' ? '于婷全付' : '宏一全付'}</span></div></div>{ex.note && <div className="text-gray-400 italic max-w-[100px] truncate text-right">{ex.note}</div>}</div></div>);})}</div></div>))}</div>
      <button onClick={() => setShowAdd(true)} className="fixed bottom-24 right-6 w-14 h-14 rounded-full bg-[#A89F91] text-white flex items-center justify-center shadow-xl hover:bg-[#968c7e] transition-all z-40 active:scale-95"><Plus size={24} /></button>
      {showAdd && <AddExpenseModal onClose={() => setShowAdd(false)} onAdd={handleAdd} />}
    </div>
  );
}

const AddExpenseModal = ({ onClose, onAdd }) => {
  const [form, setForm] = useState({ date: new Date().toLocaleDateString('zh-TW', {year:'numeric', month:'2-digit', day:'2-digit'}).replace(/\//g, '/'), time: '12:00', amount: '', currency: 'JPY', name: '', category: '食', payer: 'YuTing', split: 'split', note: '' });
  const handleSubmit = () => { if (!form.name || !form.amount) return; onAdd(form); };
  return (
    <div className="fixed inset-0 bg-black/40 backdrop-blur-sm z-50 flex items-end sm:items-center justify-center p-4">
      <div className="bg-white w-full max-w-sm rounded-3xl p-6 animate-slide-up shadow-2xl">
        <div className="flex justify-between items-center mb-4"><h3 className="text-lg font-bold text-[#4A4A4A]">新增消費</h3><button onClick={onClose} className="p-1 rounded-full hover:bg-gray-100"><X size={20} className="text-gray-400"/></button></div>
        <div className="grid grid-cols-2 gap-3 mb-3"><input type="date" className="w-full p-3 bg-gray-50 rounded-xl border border-gray-100 outline-none text-sm" value={form.date.replace(/\//g, '-')} onChange={e => setForm({...form, date: e.target.value.replace(/-/g, '/')})} /><input type="time" className="w-full p-3 bg-gray-50 rounded-xl border border-gray-100 outline-none text-sm" value={form.time} onChange={e => setForm({...form, time: e.target.value})} /></div>
        <div className="flex gap-3 mb-3"><input className="flex-1 w-full p-3 bg-gray-50 rounded-xl border border-gray-100 focus:outline-none focus:ring-2 focus:ring-[#A89F91] text-lg font-bold" type="number" placeholder="金額" value={form.amount} onChange={e=>setForm({...form, amount: e.target.value})} /><select className="w-24 p-3 bg-gray-50 rounded-xl border border-gray-100 outline-none font-bold text-center" value={form.currency} onChange={e=>setForm({...form, currency: e.target.value})}><option value="JPY">JPY</option><option value="TWD">TWD</option></select></div>
        <input className="w-full p-3 bg-gray-50 rounded-xl border border-gray-100 outline-none mb-3" placeholder="名稱" value={form.name} onChange={e=>setForm({...form, name: e.target.value})} />
        <div className="flex gap-2 overflow-x-auto pb-2 no-scrollbar mb-4">{Object.keys(EXPENSE_CATEGORY_STYLES).map(cat => { const style = EXPENSE_CATEGORY_STYLES[cat]; return (<button key={cat} onClick={()=>setForm({...form, category: cat})} className={`flex-shrink-0 px-3 py-2 rounded-lg text-xs font-bold border transition-all flex items-center gap-1 ${form.category===cat ? `${style.bg} ${style.text} border-transparent` : 'bg-white text-gray-500 border-gray-200'}`}>{style.icon} {cat}</button>); })}</div>
        <div className="grid grid-cols-2 gap-3 mb-4 p-3 bg-gray-50 rounded-xl border border-gray-100"><div><label className="text-xs text-gray-400 block mb-1 text-center">付款</label><div className="flex bg-white rounded-lg p-1 border border-gray-100"><button onClick={()=>setForm({...form, payer: 'YuTing'})} className={`flex-1 py-1.5 rounded-md text-xs font-bold transition-all ${form.payer==='YuTing' ? 'bg-pink-100 text-pink-600' : 'text-gray-400'}`}>于婷</button><button onClick={()=>setForm({...form, payer: 'HongYi'})} className={`flex-1 py-1.5 rounded-md text-xs font-bold transition-all ${form.payer==='HongYi' ? 'bg-blue-100 text-blue-600' : 'text-gray-400'}`}>宏一</button></div></div><div><label className="text-xs text-gray-400 block mb-1 text-center">分帳</label><select className="w-full p-1.5 bg-white rounded-lg border border-gray-100 text-xs font-bold text-center outline-none h-[34px]" value={form.split} onChange={e=>setForm({...form, split: e.target.value})}><option value="split">平分</option><option value="YuTing">于婷全付</option><option value="HongYi">宏一全付</option></select></div></div>
        <input className="w-full p-3 bg-gray-50 rounded-xl mb-4 border border-gray-100 outline-none text-xs" placeholder="備註..." value={form.note} onChange={e=>setForm({...form, note: e.target.value})} />
        <button onClick={handleSubmit} className="w-full bg-[#4A4A4A] text-white py-4 rounded-xl font-bold shadow-md hover:bg-[#333]">確認記帳</button>
      </div>
    </div>
  );
};

function InfoView({ infoData, setInfoData, expenses }) {
  const [isEditing, setIsEditing] = useState(false);
  const [tempData, setTempData] = useState(infoData);
  const RATE_JPY_TO_TWD = 0.22;
  const totalSpentTWD = useMemo(() => expenses.reduce((sum, ex) => sum + (ex.currency === 'JPY' ? parseFloat(ex.amount) * RATE_JPY_TO_TWD : parseFloat(ex.amount)), 0), [expenses]);
  const remainingBudget = infoData.budget.total - totalSpentTWD;
  const progress = Math.min(100, (totalSpentTWD / infoData.budget.total) * 100);
  const handleSave = () => { setInfoData(tempData); setIsEditing(false); };

  return (
    <div className="p-4 bg-[#F8F8F6] min-h-full pb-24 overflow-y-auto">
      <div className="flex justify-between items-center mb-6 px-2"><h1 className="text-2xl font-bold text-[#4A4A4A] tracking-widest">TRIP INFO</h1><button onClick={() => { setTempData(infoData); setIsEditing(true); }} className="p-2 bg-white rounded-full shadow-sm text-gray-500 hover:text-[#A89F91]"><Edit2 size={18} /></button></div>
      <div className="bg-white p-5 rounded-3xl shadow-sm border border-gray-100 mb-5">
         <div className="flex items-center gap-2 mb-4 text-[#A89F91]"><PieChart size={20} /><h3 className="font-bold text-sm">預算概況 (TWD)</h3></div>
         <div className="flex justify-between items-end mb-2"><div><p className="text-xs text-gray-400">總支出</p><p className="text-2xl font-bold text-[#4A4A4A]">${Math.round(totalSpentTWD).toLocaleString()}</p></div><div className="text-right"><p className="text-xs text-gray-400">剩餘預算</p><p className={`text-lg font-bold ${remainingBudget < 0 ? 'text-red-500' : 'text-green-600'}`}>${Math.round(remainingBudget).toLocaleString()}</p></div></div>
         <div className="h-2 w-full bg-gray-100 rounded-full overflow-hidden"><div className={`h-full rounded-full ${progress > 100 ? 'bg-red-400' : 'bg-[#A89F91]'}`} style={{ width: `${progress}%` }}></div></div>
      </div>
      <div className="bg-white p-5 rounded-3xl shadow-sm border border-gray-100 mb-5 relative overflow-hidden">
         <div className="absolute -right-4 -top-4 text-blue-50 opacity-50"><Plane size={100} /></div>
         <div className="flex items-center gap-2 mb-4 text-[#8FA3AD] relative z-10"><Plane size={20} /><h3 className="font-bold text-sm">航班資訊</h3></div>
         {['outbound', 'inbound'].map(type => (
             <div key={type} className="mb-4 relative z-10">
                <div className="flex items-center justify-between mb-1"><span className={`text-xs ${type==='outbound'?'bg-[#8FA3AD]':'bg-[#D4A5A5]'} text-white px-2 py-0.5 rounded`}>{type==='outbound'?'去程':'回程'}</span><span className="text-xs text-gray-400">{infoData.flight[type].date}</span></div>
                <div className="bg-gray-50 p-3 rounded-xl border border-gray-100">
                    <div className="flex justify-between items-center mb-2"><div className="text-center"><div className="text-xl font-bold text-gray-700">{infoData.flight[type].from.split(' ')[0]}</div></div><div className="flex flex-col items-center px-2"><span className="text-[10px] text-gray-400 mb-0.5">{infoData.flight[type].airline.split(' ')[0]}</span><div className="h-[1px] w-12 bg-gray-300 relative"><div className="absolute right-0 -top-1 w-2 h-2 rounded-full bg-gray-300"></div></div><span className="text-[10px] font-bold text-gray-600 mt-0.5">{infoData.flight[type].time}</span></div><div className="text-center"><div className="text-xl font-bold text-gray-700">{infoData.flight[type].to.split(' ')[0]}</div></div></div>
                    <div className="flex justify-between border-t border-gray-200 pt-2 mt-2 text-center"><div><div className="text-[10px] text-gray-400">Boarding</div><div className="text-sm font-bold text-[#4A4A4A]">{infoData.flight[type].boardingTime}</div></div><div><div className="text-[10px] text-gray-400">Gate</div><div className="text-sm font-bold text-[#4A4A4A]">{infoData.flight[type].gate}</div></div><div><div className="text-[10px] text-gray-400">Seat</div><div className="text-sm font-bold text-[#4A4A4A]">{infoData.flight[type].seat}</div></div></div>
                </div>
             </div>
         ))}
      </div>
      <div className="bg-white p-5 rounded-3xl shadow-sm border border-gray-100 mb-5">
         <div className="flex items-center gap-2 mb-3 text-[#7E8D75]"><Home size={20} /><h3 className="font-bold text-sm">住宿資訊</h3></div>
         <h4 className="text-lg font-bold text-gray-800 mb-1">{infoData.accommodation.name}</h4>
         <p className="text-xs text-gray-500 mb-3 flex items-center gap-1"><MapPin size={12}/> {infoData.accommodation.address}</p>
         <div className="bg-[#F4F8F0] p-3 rounded-xl text-xs text-gray-600 border border-[#E3E8E1]">{infoData.accommodation.note}</div>
      </div>
      <div className="bg-[#FFF8F8] p-5 rounded-3xl shadow-sm border border-red-50">
         <div className="flex items-center gap-2 mb-4 text-red-400"><Phone size={20} /><h3 className="font-bold text-sm">緊急聯絡</h3></div>
         <div className="grid grid-cols-2 gap-3"><div className="bg-white p-3 rounded-xl text-center border border-red-50"><div className="text-2xl font-bold text-red-500 font-noto">119</div><div className="text-[10px] text-gray-400">救護車/火警</div></div><div className="bg-white p-3 rounded-xl text-center border border-red-50"><div className="text-2xl font-bold text-red-500 font-noto">110</div><div className="text-[10px] text-gray-400">警察局</div></div></div>
      </div>
      {isEditing && (
        <div className="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
            <div className="bg-white w-full max-w-sm rounded-3xl p-6 shadow-2xl max-h-[80vh] overflow-y-auto">
                <h3 className="font-bold text-lg mb-4 text-center">編輯資訊</h3>
                <div className="mb-4"><label className="text-xs text-gray-400 block mb-1">總預算</label><input className="w-full p-2 bg-gray-50 rounded border" type="number" value={tempData.budget.total} onChange={e => setTempData({...tempData, budget: { ...tempData.budget, total: parseInt(e.target.value) }})} /></div>
                {['outbound', 'inbound'].map(type => (
                    <div key={type} className="mb-4 p-3 bg-gray-50 rounded-xl border"><p className="text-xs font-bold mb-2">{type==='outbound'?'去程':'回程'}航班</p><div className="grid grid-cols-2 gap-2"><input className="p-2 rounded border text-xs" placeholder="登機時間" value={tempData.flight[type].boardingTime} onChange={e => setTempData({...tempData, flight: {...tempData.flight, [type]: {...tempData.flight[type], boardingTime: e.target.value}}})} /><input className="p-2 rounded border text-xs" placeholder="登機門" value={tempData.flight[type].gate} onChange={e => setTempData({...tempData, flight: {...tempData.flight, [type]: {...tempData.flight[type], gate: e.target.value}}})} /><input className="p-2 rounded border text-xs col-span-2" placeholder="座位" value={tempData.flight[type].seat} onChange={e => setTempData({...tempData, flight: {...tempData.flight, [type]: {...tempData.flight[type], seat: e.target.value}}})} /></div></div>
                ))}
                <div className="mb-4"><label className="text-xs text-gray-400 block mb-1">住宿名稱</label><input className="w-full p-2 bg-gray-50 rounded border mb-2" value={tempData.accommodation.name} onChange={e => setTempData({...tempData, accommodation: { ...tempData.accommodation, name: e.target.value }})} /><label className="text-xs text-gray-400 block mb-1">備註</label><textarea className="w-full p-2 bg-gray-50 rounded border" value={tempData.accommodation.note} onChange={e => setTempData({...tempData, accommodation: { ...tempData.accommodation, note: e.target.value }})} /></div>
                <div className="flex gap-2"><button onClick={() => setIsEditing(false)} className="flex-1 py-3 text-gray-500 bg-gray-100 rounded-xl">取消</button><button onClick={handleSave} className="flex-1 py-3 text-white bg-[#A89F91] rounded-xl font-bold">儲存</button></div>
            </div>
        </div>
      )}
    </div>
  );
}

function LanguageView({ japaneseData, setJapaneseData }) {
  const [activeTab, setActiveTab] = useState('conversation'); 
  const [activeCategory, setActiveCategory] = useState('全部');
  const [isAdding, setIsAdding] = useState(false);
  const categories = ['全部', '交通', '餐廳', '購物', '緊急'];

  const filteredData = useMemo(() => {
    return japaneseData.filter(item => {
        const typeMatch = activeTab === 'conversation' ? item.type === 'conversation' : item.type === 'vocabulary';
        const catMatch = activeCategory === '全部' || item.category === activeCategory;
        return typeMatch && catMatch;
    });
  }, [activeTab, activeCategory, japaneseData]);

  const speak = (text) => {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'ja-JP';
    window.speechSynthesis.speak(utterance);
  };

  const handleAdd = (newItem) => {
      setJapaneseData(prev => [...prev, { ...newItem, id: Date.now() }]);
      setIsAdding(false);
  };
