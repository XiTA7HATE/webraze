import { useState, useEffect, useRef } from "react";

const DEMO_USER = { name: "Raze", calories_goal: 2768, protein_goal: 191, fat_goal: 86, carb_goal: 293, weight: 75, height: 180, age: 22 };
const DEMO_LOG = [
  { id: 1, meal_name: "Овсянка с бананом", calories: 380, protein: 12, fat: 6, carbs: 68, time: "08:15" },
  { id: 2, meal_name: "Куриная грудка с рисом", calories: 520, protein: 48, fat: 8, carbs: 62, time: "13:00" },
];

function Snow() {
  const canvasRef = useRef();
  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
    let W = canvas.width = window.innerWidth;
    let H = canvas.height = window.innerHeight;
    const flakes = Array.from({ length: 80 }, () => ({
      x: Math.random() * W, y: Math.random() * H,
      r: Math.random() * 2.5 + 0.5, speed: Math.random() * 0.8 + 0.2,
      wind: Math.random() * 0.4 - 0.2, opacity: Math.random() * 0.35 + 0.08,
    }));
    let raf;
    function draw() {
      ctx.clearRect(0, 0, W, H);
      flakes.forEach(f => {
        ctx.beginPath(); ctx.arc(f.x, f.y, f.r, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(255,255,255,${f.opacity})`; ctx.fill();
        f.y += f.speed; f.x += f.wind;
        if (f.y > H) { f.y = -5; f.x = Math.random() * W; }
        if (f.x > W) f.x = 0; if (f.x < 0) f.x = W;
      });
      raf = requestAnimationFrame(draw);
    }
    draw();
    const onResize = () => { W = canvas.width = window.innerWidth; H = canvas.height = window.innerHeight; };
    window.addEventListener("resize", onResize);
    return () => { cancelAnimationFrame(raf); window.removeEventListener("resize", onResize); };
  }, []);
  return <canvas ref={canvasRef} style={{ position: "fixed", top: 0, left: 0, width: "100%", height: "100%", pointerEvents: "none", zIndex: 0 }} />;
}

function CalorieRing({ eaten, goal }) {
  const size = 200, stroke = 14, r = (size - stroke) / 2;
  const circ = 2 * Math.PI * r, pct = Math.min(eaten / goal, 1);
  return (
    <div style={{ position: "relative", width: size, height: size, margin: "0 auto" }}>
      <svg width={size} height={size} style={{ transform: "rotate(-90deg)" }}>
        <circle cx={size/2} cy={size/2} r={r} fill="none" stroke="#1a1a1a" strokeWidth={stroke} />
        <circle cx={size/2} cy={size/2} r={r} fill="none" stroke="#22c55e" strokeWidth={stroke}
          strokeDasharray={`${pct * circ} ${circ}`} strokeLinecap="round"
          style={{ transition: "stroke-dasharray 0.8s ease", filter: "drop-shadow(0 0 8px #22c55e99)" }} />
      </svg>
      <div style={{ position: "absolute", inset: 0, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center" }}>
        <div style={{ fontSize: 34, fontWeight: 900, color: "#fff", lineHeight: 1 }}>{eaten}</div>
        <div style={{ fontSize: 12, color: "#555" }}>/ {goal} ккал</div>
        <div style={{ fontSize: 13, color: "#22c55e", marginTop: 4, fontWeight: 700 }}>{Math.round(pct * 100)}%</div>
      </div>
    </div>
  );
}

function MiniRing({ value, max, color, label }) {
  const size = 72, stroke = 6, r = (size - stroke) / 2;
  const circ = 2 * Math.PI * r, pct = Math.min(value / max, 1);
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize: 13, color: "#9ca3af", marginBottom: 6 }}>{label}</div>
      <div style={{ position: "relative", width: size, height: size, margin: "0 auto" }}>
        <svg width={size} height={size} style={{ transform: "rotate(-90deg)" }}>
          <circle cx={size/2} cy={size/2} r={r} fill="none" stroke="#1a1a1a" strokeWidth={stroke} />
          <circle cx={size/2} cy={size/2} r={r} fill="none" stroke={color} strokeWidth={stroke}
            strokeDasharray={`${pct*circ} ${circ}`} strokeLinecap="round" style={{ transition: "stroke-dasharray 0.8s" }} />
        </svg>
        <div style={{ position: "absolute", inset: 0, display: "flex", alignItems: "center", justifyContent: "center" }}>
          <div style={{ fontSize: 14, fontWeight: 700, color: "#fff" }}>{Math.round(pct*100)}%</div>
        </div>
      </div>
      <div style={{ fontSize: 11, color: "#555", marginTop: 4 }}>{value}г<br/>из {max}г</div>
    </div>
  );
}

function WeekBar() {
  const days = ["Пн","Вт","Ср","Чт","Пт","Сб","Вс"];
  const today = new Date().getDay();
  const mon = today === 0 ? 6 : today - 1;
  const dates = Array.from({ length: 7 }, (_, i) => {
    const d = new Date(); d.setDate(d.getDate() - mon + i);
    return { label: days[i], date: d.getDate(), active: i === mon };
  });
  return (
    <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 20 }}>
      {dates.map((d, i) => (
        <div key={i} style={{ textAlign: "center", flex: 1 }}>
          <div style={{ fontSize: 10, color: "#555", marginBottom: 6, textTransform: "uppercase" }}>{d.label}</div>
          <div style={{ width: 32, height: 32, borderRadius: "50%", margin: "0 auto", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 13, fontWeight: d.active ? 700 : 400, color: d.active ? "#22c55e" : "#9ca3af", border: d.active ? "2px solid #22c55e" : "2px solid transparent" }}>{d.date}</div>
        </div>
      ))}
    </div>
  );
}

export default function App() {
  const [tab, setTab] = useState(0);
  const [log, setLog] = useState(DEMO_LOG);
  const [water, setWater] = useState(800);
  const [analyzing, setAnalyzing] = useState(false);
  const [result, setResult] = useState(null);
  const [addMode, setAddMode] = useState("food");
  const [textInput, setTextInput] = useState("");
  const [imagePreview, setImagePreview] = useState(null);
  const [imageBase64, setImageBase64] = useState(null);
  const [avatar, setAvatar] = useState(null);
  const foodRef = useRef();
  const barcodeRef = useRef();
  const avatarRef = useRef();

  const totals = log.reduce((a, m) => ({ cal: a.cal+m.calories, prot: a.prot+m.protein, fat: a.fat+m.fat, carbs: a.carbs+m.carbs }), { cal:0, prot:0, fat:0, carbs:0 });

  function handleFile(e, mode) {
    const file = e.target.files[0]; if (!file) return;
    const reader = new FileReader();
    reader.onload = ev => { setImageBase64(ev.target.result.split(",")[1]); setImagePreview(ev.target.result); setResult(null); };
    reader.readAsDataURL(file);
  }

  async function analyzeFood() {
    if (!imageBase64) return;
    setAnalyzing(true); setResult(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST", headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ model: "claude-sonnet-4-6", max_tokens: 1000, messages: [{ role: "user", content: [
          { type: "image", source: { type: "base64", media_type: "image/jpeg", data: imageBase64 } },
          { type: "text", text: `Ты эксперт по питанию. Посмотри на фото еды и определи КБЖУ. Ответь СТРОГО в JSON (только JSON):
{"name":"название","portion":граммы,"calories":число,"protein":число,"fat":число,"carbs":число,"tip":"совет"}
Если не еда: {"error":"not_food"}` }
        ]}] })
      });
      const data = await res.json();
      const parsed = JSON.parse(data.content[0].text.replace(/```json|```/g,"").trim());
      setResult(parsed.error ? { error: "Не вижу еду на фото 🤔" } : parsed);
    } catch { setResult({ error: "Ошибка анализа. Попробуй ещё раз." }); }
    setAnalyzing(false);
  }

  async function analyzeBarcode() {
    if (!imageBase64) return;
    setAnalyzing(true); setResult(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST", headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ model: "claude-sonnet-4-6", max_tokens: 200, messages: [{ role: "user", content: [
          { type: "image", source: { type: "base64", media_type: "image/jpeg", data: imageBase64 } },
          { type: "text", text: "На фото штрихкод. Прочитай цифры и ответь ТОЛЬКО цифрами без пробелов. Например: 4607026570124" }
        ]}] })
      });
      const data = await res.json();
      const barcode = data.content[0].text.trim().replace(/\D/g,"");
      if (!barcode || barcode.length < 8) { setResult({ error: "Не удалось прочитать штрихкод 😔" }); setAnalyzing(false); return; }
      const off = await fetch(`https://world.openfoodfacts.org/api/v0/product/${barcode}.json`);
      const offData = await off.json();
      if (offData.status !== 1) { setResult({ error: `Продукт не найден (${barcode}) 😔` }); setAnalyzing(false); return; }
      const p = offData.product; const n = p.nutriments || {};
      setResult({ name: p.product_name_ru || p.product_name || "Продукт", portion: 100, calories: Math.round(n["energy-kcal_100g"]||0), protein: Math.round((n.proteins_100g||0)*10)/10, fat: Math.round((n.fat_100g||0)*10)/10, carbs: Math.round((n.carbohydrates_100g||0)*10)/10, tip: `Бренд: ${p.brands||"неизвестен"} · ${barcode}` });
    } catch { setResult({ error: "Ошибка. Попробуй ещё раз." }); }
    setAnalyzing(false);
  }

  async function analyzeText() {
    if (!textInput.trim()) return;
    setAnalyzing(true); setResult(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST", headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ model: "claude-sonnet-4-6", max_tokens: 1000, messages: [{ role: "user", content: `Ты эксперт по питанию. Пользователь написал: "${textInput}". Ответь СТРОГО в JSON (только JSON):
{"name":"название","portion":граммы,"calories":число,"protein":число,"fat":число,"carbs":число,"tip":"совет"}
Если не еда: {"error":"not_food"}` }] })
      });
      const data = await res.json();
      const parsed = JSON.parse(data.content[0].text.replace(/```json|```/g,"").trim());
      setResult(parsed.error ? { error: "Не понял что за еда 🤔" } : parsed);
    } catch { setResult({ error: "Ошибка анализа." }); }
    setAnalyzing(false);
  }

  function addToLog() {
    if (!result || result.error) return;
    const now = new Date();
    const time = `${now.getHours().toString().padStart(2,"0")}:${now.getMinutes().toString().padStart(2,"0")}`;
    setLog(p => [...p, { id: Date.now(), meal_name: result.name, calories: result.calories, protein: result.protein, fat: result.fat, carbs: result.carbs, time }]);
    setResult(null); setImagePreview(null); setImageBase64(null); setTextInput(""); setTab(0);
  }

  const S = {
    page: { padding: "0 16px 100px", position: "relative", zIndex: 1 },
    card: { background: "rgba(14,14,14,0.92)", borderRadius: 16, padding: 16, marginBottom: 12, border: "1px solid #1c1c1c" },
    btn: (v="green") => ({ width:"100%", padding:"14px", borderRadius:12, border:"none", cursor:"pointer", fontSize:15, fontWeight:700, background: v==="green" ? "#22c55e" : "#111", color: v==="green" ? "#000" : "#666" }),
    inp: { width:"100%", padding:"12px 14px", background:"#0d0d0d", border:"1px solid #1c1c1c", borderRadius:10, color:"#fff", fontSize:15, outline:"none", boxSizing:"border-box" },
    tabBar: { position:"fixed", bottom:0, left:"50%", transform:"translateX(-50%)", width:"100%", maxWidth:480, background:"rgba(6,6,6,0.97)", borderTop:"1px solid #161616", display:"flex", padding:"8px 0 18px", zIndex:200 },
    tab: (a) => ({ flex:1, background:"none", border:"none", color: a?"#22c55e":"#333", cursor:"pointer", display:"flex", flexDirection:"column", alignItems:"center", gap:3, padding:"4px 0" }),
  };

  const ImageUpload = ({ refEl, onChange, label, icon }) => (
    <>
      <input ref={refEl} type="file" accept="image/*" onChange={onChange} style={{ display:"none" }} />
      {!imagePreview ? (
        <div onClick={() => refEl.current.click()} style={{ ...S.card, textAlign:"center", padding:"48px 20px", cursor:"pointer", border:"2px dashed #1c1c1c" }}>
          <div style={{ fontSize:52, marginBottom:12 }}>{icon}</div>
          <div style={{ color:"#555", fontSize:14 }}>{label}</div>
        </div>
      ) : (
        <div style={{ ...S.card, padding:0, overflow:"hidden", marginBottom:10 }}>
          <img src={imagePreview} alt="" style={{ width:"100%", maxHeight:220, objectFit:"cover" }} />
        </div>
      )}
    </>
  );

  return (
    <div style={{ minHeight:"100vh", background:"#060606", color:"#fff", fontFamily:"'Inter',-apple-system,sans-serif", maxWidth:480, margin:"0 auto", position:"relative" }}>
      <Snow />

      {tab === 0 && (
        <div style={S.page}>
          <div style={{ paddingTop:16 }}><WeekBar /></div>
          <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", marginBottom:20 }}>
            <div style={{ textAlign:"center" }}>
              <div style={{ fontSize:20, fontWeight:800 }}>{totals.cal}</div>
              <div style={{ fontSize:11, color:"#555" }}>ккал</div>
              <div style={{ fontSize:10, color:"#444", marginTop:2 }}>Потребление</div>
            </div>
            <CalorieRing eaten={totals.cal} goal={DEMO_USER.calories_goal} />
            <div style={{ textAlign:"center" }}>
              <div style={{ fontSize:20, fontWeight:800, color: totals.cal > DEMO_USER.calories_goal ? "#f87171" : "#fff" }}>{Math.max(0, DEMO_USER.calories_goal - totals.cal)}</div>
              <div style={{ fontSize:11, color:"#555" }}>ккал</div>
              <div style={{ fontSize:10, color:"#444", marginTop:2 }}>Осталось</div>
            </div>
          </div>

          <div style={{ ...S.card }}>
            <div style={{ display:"flex", justifyContent:"space-around" }}>
              {[["＋","Добавить",()=>{setTab(1);setAddMode("text");}],["〓","Штрихкод",()=>{setTab(1);setAddMode("barcode");setImagePreview(null);setImageBase64(null);}],["📷","Фото",()=>{setTab(1);setAddMode("food");setImagePreview(null);setImageBase64(null);}],["💧","Вода",null]].map(([ic,lb,ac],i) => (
                <button key={i} onClick={ac} style={{ background:"none", border:"none", cursor: ac?"pointer":"default", display:"flex", flexDirection:"column", alignItems:"center", gap:4, padding:"6px 8px" }}>
                  <div style={{ width:46, height:46, borderRadius:"50%", background:"#111", border:"1px solid #1c1c1c", display:"flex", alignItems:"center", justifyContent:"center", fontSize:22 }}>{ic}</div>
                  <span style={{ fontSize:10, color:"#555" }}>{lb}</span>
                </button>
              ))}
            </div>
          </div>

          <div style={S.card}>
            <div style={{ fontSize:13, color:"#555", fontWeight:700, marginBottom:14, letterSpacing:1 }}>НУТРИЕНТЫ</div>
            <div style={{ display:"flex", justifyContent:"space-around" }}>
              <MiniRing value={totals.prot} max={DEMO_USER.protein_goal} color="#60a5fa" label="Белки" />
              <MiniRing value={totals.carbs} max={DEMO_USER.carb_goal} color="#a78bfa" label="Углеводы" />
              <MiniRing value={totals.fat} max={DEMO_USER.fat_goal} color="#fbbf24" label="Жиры" />
            </div>
          </div>

          <div style={S.card}>
            <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:10 }}>
              <span style={{ fontSize:14, fontWeight:600 }}>💧 Вода</span>
              <span style={{ fontSize:14, color:"#60a5fa", fontWeight:700 }}>{water} / 2000 мл</span>
            </div>
            <div style={{ height:6, background:"#111", borderRadius:3, overflow:"hidden", marginBottom:10 }}>
              <div style={{ height:"100%", width:`${Math.min(water/2000*100,100)}%`, background:"#60a5fa", borderRadius:3, transition:"width 0.4s" }} />
            </div>
            <div style={{ display:"flex", gap:6 }}>
              {[150,200,250,500].map(ml=>(
                <button key={ml} onClick={()=>setWater(w=>w+ml)} style={{ flex:1, padding:"8px 0", background:"#0d0d0d", border:"1px solid #1c1c1c", borderRadius:8, color:"#60a5fa", fontSize:12, cursor:"pointer", fontWeight:600 }}>+{ml}</button>
              ))}
            </div>
          </div>

          {log.length > 0 && (
            <div style={S.card}>
              <div style={{ fontSize:12, color:"#555", fontWeight:700, marginBottom:10, letterSpacing:1 }}>ПРИЁМЫ ПИЩИ</div>
              {log.map(m=>(
                <div key={m.id} style={{ display:"flex", justifyContent:"space-between", alignItems:"center", padding:"10px 0", borderBottom:"1px solid #0d0d0d" }}>
                  <div>
                    <div style={{ fontWeight:600, fontSize:14 }}>{m.meal_name}</div>
                    <div style={{ fontSize:11, color:"#555" }}>Б:{m.protein}г · Ж:{m.fat}г · У:{m.carbs}г · {m.time}</div>
                  </div>
                  <div style={{ fontSize:20, fontWeight:900, color:"#22c55e" }}>{m.calories}</div>
                </div>
              ))}
            </div>
          )}
        </div>
      )}

      {tab === 1 && (
        <div style={S.page}>
          <div style={{ paddingTop:16, fontSize:18, fontWeight:800, marginBottom:14 }}>Добавить приём</div>
          <div style={{ display:"flex", gap:6, marginBottom:16 }}>
            {[["food","📷 Фото"],["barcode","〓 Штрихкод"],["text","✍️ Текст"]].map(([m,l])=>(
              <button key={m} onClick={()=>{setAddMode(m);setResult(null);setImagePreview(null);setImageBase64(null);}} style={{ flex:1, padding:"10px 0", borderRadius:10, border:"none", cursor:"pointer", fontSize:12, fontWeight:700, background: addMode===m?"#22c55e":"#0d0d0d", color: addMode===m?"#000":"#555" }}>{l}</button>
            ))}
          </div>

          {addMode==="food" && (
            <>
              <input ref={foodRef} type="file" accept="image/*" onChange={e=>handleFile(e,"food")} style={{ display:"none" }} />
              {!imagePreview ? (
                <div onClick={()=>foodRef.current.click()} style={{ ...S.card, textAlign:"center", padding:"48px 20px", cursor:"pointer", border:"2px dashed #1c1c1c" }}>
                  <div style={{ fontSize:52, marginBottom:12 }}>📷</div>
                  <div style={{ color:"#555", fontSize:14 }}>Сфотографируй еду</div>
                </div>
              ) : (
                <div style={{ ...S.card, padding:0, overflow:"hidden", marginBottom:10 }}>
                  <img src={imagePreview} alt="" style={{ width:"100%", maxHeight:220, objectFit:"cover" }} />
                </div>
              )}
              {imagePreview && !result && <button onClick={analyzeFood} disabled={analyzing} style={{ ...S.btn(), marginTop:8, opacity:analyzing?0.6:1 }}>{analyzing?"⏳ Анализирую...":"🔍 Определить КБЖУ"}</button>}
              {imagePreview && <button onClick={()=>{setImagePreview(null);setImageBase64(null);setResult(null);}} style={{ ...S.btn("dark"), marginTop:8 }}>Другое фото</button>}
            </>
          )}

          {addMode==="barcode" && (
            <>
              <input ref={barcodeRef} type="file" accept="image/*" onChange={e=>handleFile(e,"barcode")} style={{ display:"none" }} />
              {!imagePreview ? (
                <div onClick={()=>barcodeRef.current.click()} style={{ ...S.card, textAlign:"center", padding:"48px 20px", cursor:"pointer", border:"2px dashed #1c1c1c" }}>
                  <div style={{ fontSize:52, marginBottom:12 }}>〓</div>
                  <div style={{ color:"#555", fontSize:14 }}>Сфотографируй штрихкод</div>
                  <div style={{ color:"#333", fontSize:12, marginTop:8 }}>AI прочитает цифры → найдёт продукт</div>
                </div>
              ) : (
                <div style={{ ...S.card, padding:0, overflow:"hidden", marginBottom:10 }}>
                  <img src={imagePreview} alt="" style={{ width:"100%", maxHeight:220, objectFit:"cover" }} />
                </div>
              )}
              {imagePreview && !result && <button onClick={analyzeBarcode} disabled={analyzing} style={{ ...S.btn(), marginTop:8, opacity:analyzing?0.6:1 }}>{analyzing?"⏳ Читаю штрихкод...":"🔍 Найти продукт"}</button>}
              {imagePreview && <button onClick={()=>{setImagePreview(null);setImageBase64(null);setResult(null);}} style={{ ...S.btn("dark"), marginTop:8 }}>Другое фото</button>}
            </>
          )}

          {addMode==="text" && (
            <>
              <textarea value={textInput} onChange={e=>{setTextInput(e.target.value);setResult(null);}} placeholder="Например: 200г гречки и куриная грудка" style={{ ...S.inp, minHeight:100, resize:"none", marginBottom:10 }} />
              <button onClick={analyzeText} disabled={analyzing||!textInput.trim()} style={{ ...S.btn(), opacity:(analyzing||!textInput.trim())?0.4:1 }}>{analyzing?"⏳ Анализирую...":"🔍 Определить КБЖУ"}</button>
            </>
          )}

          {result && (
            <div style={{ ...S.card, marginTop:12, border: result.error?"1px solid #450a0a":"1px solid #14532d" }}>
              {result.error ? (
                <div style={{ textAlign:"center", color:"#f87171", padding:"12px 0" }}>{result.error}</div>
              ) : (
                <>
                  <div style={{ fontWeight:800, fontSize:17, marginBottom:4 }}>✅ {result.name}</div>
                  <div style={{ color:"#555", fontSize:13, marginBottom:14 }}>~{result.portion}г</div>
                  <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:8, marginBottom:14 }}>
                    {[["🔥 Калории",result.calories,"ккал","#22c55e"],["🥩 Белки",result.protein,"г","#60a5fa"],["🧈 Жиры",result.fat,"г","#fbbf24"],["🍞 Углев.",result.carbs,"г","#a78bfa"]].map(([l,v,u,c])=>(
                      <div key={l} style={{ background:"#0d0d0d", borderRadius:10, padding:"10px 12px" }}>
                        <div style={{ fontSize:11, color:"#555", marginBottom:2 }}>{l}</div>
                        <div style={{ fontSize:20, fontWeight:800, color:c }}>{v}<span style={{ fontSize:12, color:"#444" }}> {u}</span></div>
                      </div>
                    ))}
                  </div>
                  {result.tip && <div style={{ fontSize:12, color:"#555", marginBottom:12 }}>💡 {result.tip}</div>}
                  <button onClick={addToLog} style={S.btn()}>+ Добавить в дневник</button>
                </>
              )}
            </div>
          )}
        </div>
      )}

      {tab === 2 && (
        <div style={S.page}>
          <div style={{ paddingTop:16, fontSize:18, fontWeight:800, marginBottom:14 }}>История</div>
          {[{d:"Сегодня",cal:totals.cal},{d:"Вчера",cal:1980},{d:"11.06",cal:2150},{d:"10.06",cal:1870},{d:"09.06",cal:2300},{d:"08.06",cal:1950},{d:"07.06",cal:2050}].map(({d,cal})=>{
            const pct=Math.min(cal/DEMO_USER.calories_goal*100,100), over=cal>DEMO_USER.calories_goal;
            return (
              <div key={d} style={S.card}>
                <div style={{ display:"flex", justifyContent:"space-between", marginBottom:8 }}>
                  <span style={{ fontWeight:600, fontSize:14 }}>{d}</span>
                  <span style={{ fontSize:14, color:over?"#f87171":"#22c55e", fontWeight:700 }}>{cal} ккал</span>
                </div>
                <div style={{ height:6, background:"#0d0d0d", borderRadius:3, overflow:"hidden" }}>
                  <div style={{ height:"100%", width:`${pct}%`, background:over?"#ef4444":"#22c55e", borderRadius:3 }} />
                </div>
                <div style={{ fontSize:11, color:"#333", marginTop:4 }}>{over?`⚠️ +${cal-DEMO_USER.calories_goal} ккал`:`✅ −${DEMO_USER.calories_goal-cal} ккал`}</div>
              </div>
            );
          })}
        </div>
      )}

      {tab === 3 && (
        <div style={S.page}>
          <div style={{ paddingTop:16, fontSize:18, fontWeight:800, marginBottom:14 }}>Профиль</div>
          <input ref={avatarRef} type="file" accept="image/*" onChange={e=>{const f=e.target.files[0];if(!f)return;const r=new FileReader();r.onload=ev=>setAvatar(ev.target.result);r.readAsDataURL(f);}} style={{ display:"none" }} />
          <div style={{ ...S.card, textAlign:"center", paddingTop:28, paddingBottom:24 }}>
            <div onClick={()=>avatarRef.current.click()} style={{ width:84, height:84, borderRadius:"50%", background:"#0d0d0d", margin:"0 auto 12px", cursor:"pointer", overflow:"hidden", border:"2px solid #22c55e", display:"flex", alignItems:"center", justifyContent:"center", fontSize:36 }}>
              {avatar ? <img src={avatar} style={{ width:"100%", height:"100%", objectFit:"cover" }} alt="" /> : "👤"}
            </div>
            <div style={{ fontSize:11, color:"#444", marginBottom:6 }}>Нажми чтобы изменить фото</div>
            <div style={{ fontSize:22, fontWeight:900 }}>{DEMO_USER.name}</div>
          </div>
          <div style={S.card}>
            <div style={{ fontSize:12, color:"#555", fontWeight:700, marginBottom:12, letterSpacing:1 }}>ДАННЫЕ</div>
            {[["⚖️ Вес",`${DEMO_USER.weight} кг`],["📏 Рост",`${DEMO_USER.height} см`],["🎂 Возраст",`${DEMO_USER.age} лет`]].map(([l,v])=>(
              <div key={l} style={{ display:"flex", justifyContent:"space-between", padding:"10px 0", borderBottom:"1px solid #0d0d0d" }}>
                <span style={{ color:"#777", fontSize:14 }}>{l}</span>
                <span style={{ fontWeight:600 }}>{v}</span>
              </div>
            ))}
          </div>
          <div style={S.card}>
            <div style={{ fontSize:12, color:"#555", fontWeight:700, marginBottom:12, letterSpacing:1 }}>ДНЕВНАЯ НОРМА</div>
            <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:8 }}>
              {[["🔥","Калории",DEMO_USER.calories_goal,"ккал","#22c55e"],["🥩","Белки",DEMO_USER.protein_goal,"г","#60a5fa"],["🧈","Жиры",DEMO_USER.fat_goal,"г","#fbbf24"],["🍞","Углеводы",DEMO_USER.carb_goal,"г","#a78bfa"]].map(([ic,l,v,u,c])=>(
                <div key={l} style={{ background:"#0d0d0d", borderRadius:10, padding:12 }}>
                  <div style={{ fontSize:11, color:"#555" }}>{ic} {l}</div>
                  <div style={{ fontSize:22, fontWeight:900, color:c, marginTop:4 }}>{v}<span style={{ fontSize:11, color:"#333" }}> {u}</span></div>
                </div>
              ))}
            </div>
          </div>
        </div>
      )}

      {tab === 4 && (
        <div style={S.page}>
          <div style={{ paddingTop:16, fontSize:18, fontWeight:800, marginBottom:14 }}>Канал</div>
          <div style={{ ...S.card, textAlign:"center", padding:"44px 20px" }}>
            <div style={{ fontSize:60, marginBottom:16 }}>📢</div>
            <div style={{ fontSize:24, fontWeight:900, color:"#22c55e", marginBottom:8 }}>Raze Company</div>
            <div style={{ fontSize:14, color:"#555", marginBottom:28, lineHeight:1.7 }}>Контент о питании, тренировках<br/>и здоровом образе жизни</div>
            <a href="https://t.me/razecompany" target="_blank" rel="noreferrer" style={{ display:"inline-block", padding:"14px 36px", background:"#22c55e", color:"#000", borderRadius:12, textDecoration:"none", fontWeight:800, fontSize:15 }}>Подписаться ✈️</a>
          </div>
        </div>
      )}

      <div style={S.tabBar}>
        {[["🏠","Дневник"],["➕","Добавить"],["📅","История"],["👤","Профиль"],["📢","Канал"]].map(([ic,lb],i)=>(
          <button key={i} onClick={()=>setTab(i)} style={S.tab(tab===i)}>
            <span style={{ fontSize: i===1?28:20, color: i===1?"#22c55e":tab===i?"#22c55e":"#333" }}>{ic}</span>
            <span style={{ fontSize:9 }}>{lb}</span>
          </button>
        ))}
      </div>
    </div>
  );
}
