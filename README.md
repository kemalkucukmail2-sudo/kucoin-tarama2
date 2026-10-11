<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kripto Tarama Paneli</title>
<style>
:root{
  --bg:#f1f5f9;
  --panel:#ffffff;
  --text:#111827;
  --muted:#475569;
  --border:#cbd5e1;
  --green:#087443;
  --red:#c62828;
  --yellow:#854d0e;
  --blue:#1d4ed8;
}
*{box-sizing:border-box}
body{
  margin:0;padding:14px;
  background:var(--bg);color:var(--text);
  font-family:Arial,Helvetica,sans-serif;
  font-size:14px;font-weight:500
}
h1{font-size:23px;margin:5px 0 8px;color:#0f172a}
h2{font-size:18px;margin:0 0 12px;color:#0f172a}
p{color:var(--muted);line-height:1.5}
.panel{
  background:var(--panel);border:1px solid var(--border);
  border-radius:12px;padding:14px;margin:12px 0;
  box-shadow:0 2px 5px #0f172a0b
}
button{
  border:1px solid #94a3b8;border-radius:8px;
  padding:10px 12px;background:#fff;color:#111827;
  font-size:13px;font-weight:700;cursor:pointer
}
button.active{background:#172554;color:#fff;border-color:#172554}
button:disabled{opacity:.55;cursor:wait}
.controls{display:flex;flex-wrap:wrap;gap:7px;margin:10px 0}
.primary{background:#1d4ed8;color:white;border-color:#1d4ed8}
.status{font-weight:700;color:#1e293b;margin:8px 0}
.small{font-size:12px;color:#475569}
.table-wrap{overflow-x:auto;width:100%}
table{width:100%;border-collapse:collapse;white-space:nowrap}
th{
  background:#e2e8f0;color:#0f172a;text-align:left;
  padding:11px 9px;border-bottom:2px solid #94a3b8
}
td{
  padding:11px 9px;border-bottom:1px solid #dbe2ea;
  color:#111827;font-weight:600
}
tbody tr:nth-child(even){background:#f8fafc}
.coin{font-weight:800;color:#0f172a}
.price{font-weight:800;color:#111827}
.signal{font-weight:800}
.buy{color:var(--green)}
.sell{color:var(--red)}
.watch{color:var(--yellow)}
.neutral{color:#475569}
.tag{
  display:inline-block;border-radius:5px;padding:4px 6px;
  background:#e2e8f0;color:#1e293b;font-weight:800
}
.note{background:#f8fafc;border-left:4px solid #64748b;padding:10px}
.hidden{display:none}
@media(max-width:600px){
  body{padding:9px}
  h1{font-size:21px}
  .panel{padding:10px}
  button{padding:9px 10px}
  th,td{padding:10px 8px;font-size:13px}
}
</style>
</head>
<body>

<h1>Kripto Tarama Paneli</h1>
<p>Binance Spot · USDT pariteleri · 1 saatlik analiz · Manuel tarama</p>

<div class="panel">
  <button id="scanBtn" class="primary" onclick="startScan()">TARAMAYI BAŞLAT</button>
  <div id="status" class="status">Tarama başlatılmayı bekliyor.</div>
  <div class="small">
    Alım sinyalleri için 4 saatlik EMA200 filtresi uygulanır.
    Sinyaller kapanmış 1 saatlik mumlara göre hesaplanır.
  </div>
</div>

<div class="panel">
  <h2>Güncel Tarama Sonuçları</h2>
  <div class="controls" id="currentFilters">
    <button class="active" data-filter="ALL">Tümü</button>
    <button data-filter="UP">Yükseliş Öncesi Takip</button>
    <button data-filter="BUY">AL</button>
    <button data-filter="STRONG_BUY">Güçlü AL</button>
    <button data-filter="DOWN">Düşüş Öncesi Takip</button>
    <button data-filter="SELL">SAT</button>
    <button data-filter="STRONG_SELL">Güçlü SAT</button>
  </div>
  <div id="currentCount" class="small">Henüz tarama yapılmadı.</div>
  <div class="table-wrap">
    <table>
      <thead><tr>
        <th>Coin</th><th>Son fiyat</th><th>Sinyal</th>
        <th>RSI</th><th>4s EMA200</th><th>Mum kapanışı</th>
      </tr></thead>
      <tbody id="currentBody">
        <tr><td colspan="6">Tarama başlatıldığında sonuçlar burada gösterilecek.</td></tr>
      </tbody>
    </table>
  </div>
</div>

<div class="panel">
  <h2>Kapanmış Mum Sinyal Geçmişi</h2>
  <p class="small">
    Tarama sırasında son 24 kapanmış saatlik mum kontrol edilir.
    Geçmiş kayıtlar bu tarayıcıda saklanır.
  </p>
  <div class="controls" id="historyFilters">
    <button class="active" data-hfilter="ALL">Tüm geçmiş</button>
    <button data-hfilter="BUY">AL</button>
    <button data-hfilter="STRONG_BUY">Güçlü AL</button>
  </div>
  <div id="historyCount" class="small">Henüz geçmiş sinyal bulunmuyor.</div>
  <div class="table-wrap">
    <table>
      <thead><tr>
        <th>Coin</th><th>Sinyal</th><th>Sinyal fiyatı</th>
        <th>RSI</th><th>Mum kapanış zamanı</th>
      </tr></thead>
      <tbody id="historyBody">
        <tr><td colspan="5">Geçmiş sinyaller burada gösterilecek.</td></tr>
      </tbody>
    </table>
  </div>
  <button onclick="clearHistory()">Geçmişi Temizle</button>
</div>

<div class="panel note">
  <b>Önemli:</b> Bu panel otomatik emir vermez. Geçmiş, bu tarayıcıda
  yapılan taramalarla bulunup kaydedilen sinyallerden oluşur; geçmişe
  dönük sonuçlar kârlılık garantisi değildir.
</div>

<script>
const API = "https://data-api.binance.vision";
const HISTORY_KEY = "kucoinTarama_closedBuyHistory_v1";
const MAX_HISTORY = 3000;

let allResults = [];
let currentFilter = "ALL";
let historyFilter = "ALL";
let scanning = false;

function getHistory(){
  try{
    return JSON.parse(localStorage.getItem(HISTORY_KEY) || "[]");
  }catch(e){return []}
}
function saveHistory(items){
  localStorage.setItem(HISTORY_KEY,JSON.stringify(items.slice(0,MAX_HISTORY)));
}
function fmtPrice(n){
  if(!Number.isFinite(n)) return "-";
  return n.toLocaleString("en-US",{maximumFractionDigits:8});
}
function fmtTime(t){
  return new Date(t).toLocaleString("tr-TR",{
    day:"2-digit",month:"2-digit",year:"numeric",
    hour:"2-digit",minute:"2-digit"
  });
}
function signalClass(s){
  if(s==="AL"||s==="GÜÇLÜ AL") return "buy";
  if(s==="SAT"||s==="GÜÇLÜ SAT") return "sell";
  if(s.includes("TAKİP")) return "watch";
  return "neutral";
}
function signalCode(s){
  return ({
    "YÜKSELİŞ ÖNCESİ TAKİP":"UP",
    "AL":"BUY",
    "GÜÇLÜ AL":"STRONG_BUY",
    "DÜŞÜŞ ÖNCESİ TAKİP":"DOWN",
    "SAT":"SELL",
    "GÜÇLÜ SAT":"STRONG_SELL"
  })[s] || "NONE";
}
function ema(values,period){
  if(values.length<period) return null;
  const k=2/(period+1);
  let v=values.slice(0,period).reduce((a,b)=>a+b,0)/period;
  for(let i=period;i<values.length;i++) v=values[i]*k+v*(1-k);
  return v;
}
function rsi(values,period=14){
  if(values.length<period+1) return null;
  let gains=0,losses=0;
  for(let i=values.length-period;i<values.length;i++){
    const d=values[i]-values[i-1];
    if(d>0) gains+=d; else losses-=d;
  }
  if(losses===0) return 100;
  const rs=(gains/period)/(losses/period);
  return 100-(100/(1+rs));
}
function macd(values){
  if(values.length<35) return null;
  const fast=[],slow=[];
  let ef=values[0],es=values[0];
  const kf=2/13,ks=2/27;
  for(let i=0;i<values.length;i++){
    ef=i===0?values[i]:values[i]*kf+ef*(1-kf);
    es=i===0?values[i]:values[i]*ks+es*(1-ks);
    fast.push(ef);slow.push(es);
  }
  const line=fast.map((v,i)=>v-slow[i]);
  let sig=line[0],prevSig=sig;
  const k=2/10;
  for(let i=1;i<line.length;i++){
    prevSig=sig;sig=line[i]*k+sig*(1-k);
  }
  return {line:line[line.length-1],signal:sig,
    prevLine:line[line.length-2],prevSignal:prevSig};
}
function candleSignal(candles,closes,index,trendOK){
  if(index<35) return null;
  const c=candles[index],p=candles[index-1];
  const values=closes.slice(0,index+1);
  const prevValues=closes.slice(0,index);
  const e20=ema(values,20),e50=ema(values,50);
  const pe20=ema(prevValues,20),pe50=ema(prevValues,50);
  const rv=rsi(values,14);
  const m=macd(values);
  const pm=macd(prevValues);
  if(e20===null||e50===null||rv===null||!m||!pm) return null;

  const bullCross=pe20<=pe50 && e20>e50;
  const bearCross=pe20>=pe50 && e20<e50;
  const macdUp=m.line>m.signal && pm.line<=pm.signal;
  const macdDown=m.line<m.signal && pm.line>=pm.signal;
  const bullishCandle=c.close>c.open && c.close>p.high;
  const bearishCandle=c.close<c.open && c.close<p.low;
  const volumeOK=c.volume>=c.avgVolume;
  const trend=e20>e50 && c.close>e20;

  if(trendOK && (bullCross || (macdUp&&trend)) && rv>=40 && volumeOK){
    return {signal:(rv>=50&&trend)?"GÜÇLÜ AL":"AL",rsi:rv};
  }
  if(trendOK && trend && rv>=45 && m.line>m.signal &&
     bullishCandle && volumeOK){
    return {signal:"YÜKSELİŞ ÖNCESİ TAKİP",rsi:rv};
  }
  if(bearCross || (macdDown && rv<=60) ||
     (bearishCandle && rv<45 && volumeOK)){
    return {signal:(rv<=50&&e20<e50)?"GÜÇLÜ SAT":"SAT",rsi:rv};
  }
  if(e20<e50 && m.line<m.signal && rv<50){
    return {signal:"DÜŞÜŞ ÖNCESİ TAKİP",rsi:rv};
  }
  return {signal:"SİNYAL YOK",rsi:rv};
}
async function fetchJSON(url){
  const res=await fetch(url);
  if(!res.ok) throw new Error("API yanıtı: "+res.status);
  return res.json();
}
async function analyzeSymbol(symbol){
  const [hourly,fourHour,priceObj]=await Promise.all([
    fetchJSON(`${API}/api/v3/klines?symbol=${symbol}&interval=1h&limit=100`),
    fetchJSON(`${API}/api/v3/klines?symbol=${symbol}&interval=4h&limit=250`),
    fetchJSON(`${API}/api/v3/ticker/price?symbol=${symbol}`)
  ]);
  const candles=hourly.map(k=>({
    time:k[6],open:+k[1],high:+k[2],low:+k[3],close:+k[4],
    volume:+k[5],avgVolume:0
  }));
  for(let i=0;i<candles.length;i++){
    const start=Math.max(0,i-20);
    const vols=candles.slice(start,i).map(c=>c.volume);
    candles[i].avgVolume=vols.length?
      vols.reduce((a,b)=>a+b,0)/vols.length:0;
  }
  // Binance son kline'ı açık mum olabilir; yalnızca kapanmış mumları kullan.
  const now=Date.now();
  const closed=candles.filter(c=>c.time<now);
  const closedHourly=hourly.filter(k=>k[6]<now);
  const closes=closed.map(c=>c.close);
  if(closed.length<55) return null;

  const fourCloses=fourHour.filter(k=>k[6]<now).map(k=>+k[4]);
  const e200=ema(fourCloses,200);
  const trendOK=e200!==null && closed[closed.length-1].close>e200;
  const latest=closed.length-1;
  const result=candleSignal(closed,closes,latest,trendOK);
  if(!result) return null;

  const current={
    symbol,price:+priceObj.price,signal:result.signal,
    rsi:result.rsi,trendOK,
    closeTime:closed[latest].time
  };

  const historyItems=[];
  const first=Math.max(35,closed.length-24);
  for(let i=first;i<closed.length;i++){
    const vals=closed.slice(0,i+1).map(c=>c.close);
    const tClose=closed[i].time;
    // Mumun kapanışından önce kapanmış son 4s mumun EMA200'ü.
    const available4h=fourHour.filter(k=>k[6]<tClose).map(k=>+k[4]);
    const e4=ema(available4h,200);
    const trendAt=e4!==null && closed[i].close>e4;
    const sig=candleSignal(closed,vals,i,trendAt);
    if(sig && (sig.signal==="AL"||sig.signal==="GÜÇLÜ AL")){
      historyItems.push({
        key:symbol+"_"+tClose,symbol,signal:sig.signal,
        price:closed[i].close,rsi:sig.rsi,time:tClose
      });
    }
  }
  return {current,historyItems};
}
function renderCurrent(){
  const filtered=allResults.filter(r=>
    currentFilter==="ALL" || signalCode(r.signal)===currentFilter
  );
  document.getElementById("currentCount").textContent=
    `Gösterilen: ${filtered.length} / ${allResults.length} coin`;
  const body=document.getElementById("currentBody");
  if(!filtered.length){
    body.innerHTML='<tr><td colspan="6">Bu filtrede sonuç yok.</td></tr>';
    return;
  }
  body.innerHTML=filtered.map(r=>`
    <tr>
      <td class="coin">${r.symbol.replace("USDT","")} <span class="small">/USDT</span></td>
      <td class="price">${fmtPrice(r.price)}</td>
      <td class="signal ${signalClass(r.signal)}">${r.signal}</td>
      <td>${r.rsi.toFixed(1)}</td>
      <td class="${r.trendOK?"buy":"sell"}">${r.trendOK?"ÜZERİNDE":"ALTINDA"}</td>
      <td>${fmtTime(r.closeTime)}</td>
    </tr>`).join("");
}
function renderHistory(){
  let history=getHistory();
  history.sort((a,b)=>b.time-a.time);
  const filtered=history.filter(r=>
    historyFilter==="ALL" || signalCode(r.signal)===historyFilter
  );
  document.getElementById("historyCount").textContent=
    `Gösterilen: ${filtered.length} / ${history.length} kayıt`;
  const body=document.getElementById("historyBody");
  if(!filtered.length){
    body.innerHTML='<tr><td colspan="5">Bu filtrede geçmiş sinyal yok. Yeni tarama yap.</td></tr>';
    return;
  }
  body.innerHTML=filtered.slice(0,500).map(r=>`
    <tr>
      <td class="coin">${r.symbol.replace("USDT","")} <span class="small">/USDT</span></td>
      <td class="signal ${signalClass(r.signal)}">${r.signal}</td>
      <td class="price">${fmtPrice(r.price)}</td>
      <td>${Number(r.rsi).toFixed(1)}</td>
      <td>${fmtTime(r.time)}</td>
    </tr>`).join("");
}
function setupFilters(){
  document.querySelectorAll("#currentFilters button").forEach(btn=>{
    btn.addEventListener("click",()=>{
      currentFilter=btn.dataset.filter;
      document.querySelectorAll("#currentFilters button")
        .forEach(b=>b.classList.toggle("active",b===btn));
      renderCurrent();
    });
  });
  document.querySelectorAll("#historyFilters button").forEach(btn=>{
    btn.addEventListener("click",()=>{
      historyFilter=btn.dataset.hfilter;
      document.querySelectorAll("#historyFilters button")
        .forEach(b=>b.classList.toggle("active",b===btn));
      renderHistory();
    });
  });
}
async function startScan(){
  if(scanning) return;
  scanning=true;
  const btn=document.getElementById("scanBtn");
  btn.disabled=true;
  document.getElementById("status").textContent="USDT pariteleri alınıyor...";
  try{
    const info=await fetchJSON(`${API}/api/v3/exchangeInfo`);
    const symbols=info.symbols
      .filter(s=>s.status==="TRADING"&&s.quoteAsset==="USDT"&&
        s.isSpotTradingAllowed!==false)
      .map(s=>s.symbol);

    const prices=await fetchJSON(`${API}/api/v3/ticker/24hr`);
    const liquid=new Set(prices
      .filter(p=>symbols.includes(p.symbol)&&+p.quoteVolume>=1000000)
      .sort((a,b)=>+b.quoteVolume-+a.quoteVolume)
      .slice(0,506).map(p=>p.symbol));

    const list=symbols.filter(s=>liquid.has(s));
    let done=0;
    const results=[];
    const additions=[];
    let cursor=0;
    async function worker(){
      while(cursor<list.length){
        const sym=list[cursor++];
        try{
          const out=await analyzeSymbol(sym);
          if(out){
            results.push(out.current);
            additions.push(...out.historyItems);
          }
        }catch(e){
          // Tek bir coin API hatası nedeniyle tüm tarama durmasın.
        }
        done++;
        document.getElementById("status").textContent=
          `Tarama sürüyor: ${done}/${list.length} coin`;
      }
    }
    await Promise.all([worker(),worker(),worker(),worker()]);
    allResults=results.sort((a,b)=>{
      const order={"GÜÇLÜ AL":0,"AL":1,"YÜKSELİŞ ÖNCESİ TAKİP":2,
        "DÜŞÜŞ ÖNCESİ TAKİP":3,"SAT":4,"GÜÇLÜ SAT":5,"SİNYAL YOK":6};
      return (order[a.signal]??7)-(order[b.signal]??7);
    });

    const existing=getHistory();
    const seen=new Set(existing.map(r=>r.key));
    for(const item of additions){
      if(!seen.has(item.key)){
        existing.push(item);
        seen.add(item.key);
      }
    }
    existing.sort((a,b)=>b.time-a.time);
    saveHistory(existing);
    renderCurrent();
    renderHistory();
    document.getElementById("status").textContent=
      `Tarama tamamlandı. ${results.length} coin analiz edildi.`;
  }catch(e){
    document.getElementById("status").textContent=
      "Tarama yapılamadı. İnternet bağlantısını kontrol edip tekrar dene.";
  }finally{
    scanning=false;
    btn.disabled=false;
  }
}
function clearHistory(){
  if(confirm("Kayıtlı sinyal geçmişi silinsin mi?")){
    localStorage.removeItem(HISTORY_KEY);
    renderHistory();
  }
}
setupFilters();
renderHistory();
</script>
</body>
</html>
