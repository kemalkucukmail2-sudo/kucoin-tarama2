<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kripto Al-Sat Tarayıcı</title>
<style>
*{box-sizing:border-box}
:root{color-scheme:light}
body{margin:0;padding:14px;background:#eef2f7;color:#172033;font-family:Arial,sans-serif}
h1{font-size:23px;margin:4px 0 8px;color:#111827}
h2{font-size:19px;margin:0 0 12px;color:#111827}
p{color:#334155;line-height:1.5;font-size:14px}
button{border:1px solid #cbd5e1;border-radius:8px;padding:10px 12px;background:#fff;color:#172033;font-weight:700;font-size:13px;cursor:pointer}
button:disabled{opacity:.55}
button.active{background:#172033;color:#fff;border-color:#172033}
.primary{background:#087f5b;color:#fff;border:0;padding:13px 18px;font-size:15px}
#status{padding:12px;background:#fff;border:1px solid #cbd5e1;border-radius:9px;color:#172033;font-size:14px;margin:12px 0}
.summary{display:flex;flex-wrap:wrap;gap:8px;margin:12px 0}
.pill{background:#fff;border:1px solid #d5dce6;border-radius:8px;padding:10px;color:#172033;font-size:13px}
.section{background:#fff;border:1px solid #d5dce6;border-radius:12px;padding:13px;margin:16px 0;box-shadow:0 2px 5px #1720330a}
.section-title{display:flex;align-items:center;justify-content:space-between;gap:8px;flex-wrap:wrap}
.section-title small{color:#475569;font-size:12px}
.filters{display:flex;flex-wrap:wrap;gap:7px;margin:12px 0}
.filters button{font-size:12px;padding:9px 10px}
.filters button[data-filter="STRONG_BUY"].active,.filters button[data-filter="BUY"].active{background:#087f5b;border-color:#087f5b;color:#fff}
.filters button[data-filter="STRONG_SELL"].active,.filters button[data-filter="SELL"].active{background:#c92a2a;border-color:#c92a2a;color:#fff}
.filters button[data-filter="WATCH_UP"].active{background:#b77900;border-color:#b77900;color:#fff}
.filters button[data-filter="WATCH_DOWN"].active{background:#b54708;border-color:#b54708;color:#fff}
.tablewrap{overflow-x:auto;border:1px solid #d5dce6;border-radius:8px}
table{border-collapse:collapse;width:100%;min-width:790px;font-size:13px;background:#fff}
th,td{padding:11px 9px;border-bottom:1px solid #e2e8f0;text-align:left;white-space:nowrap;color:#172033}
th{background:#e2e8f0;color:#172033;font-weight:800;position:sticky;top:0}
tbody tr:nth-child(even){background:#f8fafc}
.buy{color:#087f5b;font-weight:800}
.sell{color:#c92a2a;font-weight:800}
.watchup{color:#946200;font-weight:800}
.watchdown{color:#b54708;font-weight:800}
.neutral{color:#334155}
.empty{padding:18px;text-align:center;color:#334155}
.small{font-size:12px;color:#475569}
</style>
</head>
<body>
<h1>Kripto Al-Sat Tarayıcı</h1>
<p>Binance Spot USDT pariteleri · 1 saatlik sinyaller · 4 saatlik EMA200 filtresi</p>

<button id="scanBtn" class="primary" onclick="startScan()">TARAMAYI BAŞLAT</button>
<div id="status">Tarama başlatılmaya hazır. Tarama yalnızca düğmeye bastığında çalışır.</div>

<div class="summary">
  <div class="pill">Taranan parite: <b id="scanned">0</b></div>
  <div class="pill">Anlık sinyal: <b id="liveCount">0</b></div>
  <div class="pill">Son saat kapanışı: <b id="closedCount">0</b></div>
  <div class="pill">Son 24 saat sinyali: <b id="historyCount">0</b></div>
  <div class="pill">Son tarama: <b id="lastScan">Henüz yok</b></div>
</div>

<div id="liveSection" class="section">
  <div class="section-title">
    <h2>1. ANLIK</h2>
    <small>Mevcut 1 saatlik mumun güncel verisi</small>
  </div>
  <div class="filters" data-panel="live">
    <button class="active" data-filter="ALL">Tümü</button>
    <button data-filter="STRONG_BUY">Güçlü AL</button>
    <button data-filter="BUY">AL</button>
    <button data-filter="STRONG_SELL">Güçlü SAT</button>
    <button data-filter="SELL">SAT</button>
    <button data-filter="WATCH_UP">Yükseliş Öncesi Takip</button>
    <button data-filter="WATCH_DOWN">Düşüş Öncesi Takip</button>
  </div>
  <div class="tablewrap"><table>
    <thead><tr><th>Parite</th><th>Sinyal</th><th>Güncel fiyat</th><th>24s hacim (USDT)</th><th>RSI</th><th>4s EMA200</th></tr></thead>
    <tbody id="liveRows"><tr><td colspan="6" class="empty">Henüz tarama yapılmadı.</td></tr></tbody>
  </table></div>
</div>

<div id="closedSection" class="section">
  <div class="section-title">
    <h2>2. SON SAAT KAPANIŞI</h2>
    <small>Son tamamlanmış 1 saatlik mumun sinyali</small>
  </div>
  <div class="filters" data-panel="closed">
    <button class="active" data-filter="ALL">Tümü</button>
    <button data-filter="STRONG_BUY">Güçlü AL</button>
    <button data-filter="BUY">AL</button>
    <button data-filter="STRONG_SELL">Güçlü SAT</button>
    <button data-filter="SELL">SAT</button>
    <button data-filter="WATCH_UP">Yükseliş Öncesi Takip</button>
    <button data-filter="WATCH_DOWN">Düşüş Öncesi Takip</button>
  </div>
  <div class="tablewrap"><table>
    <thead><tr><th>Parite</th><th>Sinyal</th><th>Kapanış fiyatı</th><th>24s hacim (USDT)</th><th>RSI</th><th>4s EMA200</th></tr></thead>
    <tbody id="closedRows"><tr><td colspan="6" class="empty">Henüz tarama yapılmadı.</td></tr></tbody>
  </table></div>
</div>

<div id="historySection" class="section">
  <div class="section-title">
    <h2>3. SON 24 SAAT</h2>
    <small>Son 24 saatte kapanmış mumlarda oluşan sinyaller</small>
  </div>
  <div class="filters" data-panel="history">
    <button class="active" data-filter="ALL">Tümü</button>
    <button data-filter="STRONG_BUY">Güçlü AL</button>
    <button data-filter="BUY">AL</button>
    <button data-filter="STRONG_SELL">Güçlü SAT</button>
    <button data-filter="SELL">SAT</button>
    <button data-filter="WATCH_UP">Yükseliş Öncesi Takip</button>
    <button data-filter="WATCH_DOWN">Düşüş Öncesi Takip</button>
  </div>
  <div class="tablewrap"><table>
    <thead><tr><th>Parite</th><th>Sinyal</th><th>Sinyal zamanı</th><th>Fiyat</th><th>24s hacim (USDT)</th><th>RSI</th><th>4s EMA200</th></tr></thead>
    <tbody id="historyRows"><tr><td colspan="7" class="empty">Henüz tarama yapılmadı.</td></tr></tbody>
  </table></div>
</div>

<p class="small">
Sinyaller teknik göstergelere dayanır; kâr garantisi vermez. Anlık sinyaller mum kapanmadan değişebilir.
Bu sayfa yalnızca analiz yapar, otomatik işlem açmaz. Binance API sınırları nedeniyle bazı paritelerin verileri alınamayabilir.
</p>

<script>
"use strict";

const API = "https://data-api.binance.vision";
const HISTORY_KEY = "kripto_tarama_signal_history_v4";
const ONE_DAY = 24 * 60 * 60 * 1000;

let isScanning = false;
let liveResults = [];
let closedResults = [];
let historyResults = [];
let filters = {live:"ALL",closed:"ALL",history:"ALL"};

function setStatus(message){
  document.getElementById("status").textContent = message;
}

function ema(values, period){
  if(!values.length) return [];
  const k = 2 / (period + 1);
  const out = [values[0]];
  for(let i=1;i<values.length;i++){
    out.push(values[i]*k + out[i-1]*(1-k));
  }
  return out;
}

function rsiSeries(closes, period=14){
  const out = Array(closes.length).fill(null);
  if(closes.length <= period) return out;
  let gains=0, losses=0;
  for(let i=1;i<=period;i++){
    const d=closes[i]-closes[i-1];
    gains+=Math.max(d,0);
    losses+=Math.max(-d,0);
  }
  let avgGain=gains/period, avgLoss=losses/period;
  const calc=(g,l)=>l===0?(g===0?50:100):100-100/(1+g/l);
  out[period]=calc(avgGain,avgLoss);
  for(let i=period+1;i<closes.length;i++){
    const d=closes[i]-closes[i-1];
    avgGain=(avgGain*(period-1)+Math.max(d,0))/period;
    avgLoss=(avgLoss*(period-1)+Math.max(-d,0))/period;
    out[i]=calc(avgGain,avgLoss);
  }
  return out;
}

async function getJSON(url){
  const response=await fetch(url);
  if(!response.ok) throw new Error("API yanıtı: "+response.status);
  return response.json();
}

function getSignalAt(i, closes, e20, e50, macd, macdSignal, rsi, volumes, trendOK){
  if(i<30 || rsi[i]==null || rsi[i-1]==null) return null;

  const prev=i-1;
  const crossUp=e20[prev]<=e50[prev] && e20[i]>e50[i];
  const crossDown=e20[prev]>=e50[prev] && e20[i]<e50[i];
  const macdUp=macd[i]>macdSignal[i];
  const macdDown=macd[i]<macdSignal[i];
  const gapNow=Math.abs(e20[i]-e50[i])/closes[i];
  const gapBefore=Math.abs(e20[prev]-e50[prev])/closes[prev];
  const avgVol=volumes.slice(Math.max(0,i-20),i)
    .reduce((a,b)=>a+b,0)/Math.min(20,i);
  const volumeOK=avgVol>0 && volumes[i]>=avgVol;
  const r=rsi[i];

  if(crossUp && macdUp && r>=40 && trendOK){
    return {key:r>=50?"STRONG_BUY":"BUY",rsi:r};
  }
  if(crossDown && macdDown && r<=60){
    return {key:r<=50?"STRONG_SELL":"SELL",rsi:r};
  }

  const gapNarrowing=gapNow<gapBefore;
  const macdImproving=macd[i]>macd[prev];
  const macdWeakening=macd[i]<macd[prev];

  const earlyUp=!crossUp && trendOK && e20[i]<=e50[i] &&
    gapNarrowing && gapNow<=0.008 && macdImproving &&
    r>rsi[prev] && r>=40 && r<=60 && volumeOK;

  const earlyDown=!crossDown && e20[i]>=e50[i] &&
    gapNarrowing && gapNow<=0.008 && macdWeakening &&
    r<rsi[prev] && r>=40 && r<=60 && volumeOK;

  if(earlyUp) return {key:"WATCH_UP",rsi:r};
  if(earlyDown) return {key:"WATCH_DOWN",rsi:r};
  return null;
}

function prepareIndicators(candles){
  const closes=candles.map(c=>Number(c[4]));
  const volumes=candles.map(c=>Number(c[5]));
  const e20=ema(closes,20);
  const e50=ema(closes,50);
  const fast=ema(closes,12);
  const slow=ema(closes,26);
  const macd=closes.map((_,i)=>fast[i]-slow[i]);
  const macdSignal=ema(macd,9);
  const rsi=rsiSeries(closes,14);
  return {closes,volumes,e20,e50,macd,macdSignal,rsi};
}

function trendAtTime(candles4h, ema4h, time){
  let index=-1;
  for(let i=0;i<candles4h.length;i++){
    if(Number(candles4h[i][6])<=time) index=i;
    else break;
  }
  if(index<199) return false;
  return Number(candles4h[index][4])>ema4h[index];
}

function signalLabel(key){
  return ({
    STRONG_BUY:"GÜÇLÜ AL",
    BUY:"AL",
    STRONG_SELL:"GÜÇLÜ SAT",
    SELL:"SAT",
    WATCH_UP:"YÜKSELİŞ ÖNCESİ TAKİP",
    WATCH_DOWN:"DÜŞÜŞ ÖNCESİ TAKİP"
  })[key] || key;
}

function signalClass(key){
  if(key==="STRONG_BUY" || key==="BUY") return "buy";
  if(key==="STRONG_SELL" || key==="SELL") return "sell";
  if(key==="WATCH_UP") return "watchup";
  return "watchdown";
}

function formatPrice(n){
  if(!Number.isFinite(n)) return "-";
  if(n===0) return "0";
  if(n<1) return n.toFixed(8).replace(/0+$/,"").replace(/\.$/,"");
  return n.toLocaleString("en-US",{maximumFractionDigits:4});
}

function formatVolume(n){
  if(!Number.isFinite(n)) return "-";
  return n.toLocaleString("en-US",{maximumFractionDigits:0});
}

function formatTime(t){
  return new Date(t).toLocaleString("tr-TR");
}

function renderPanel(panel, data){
  const tbody=document.getElementById(
    panel==="live"?"liveRows":panel==="closed"?"closedRows":"historyRows"
  );
  const filter=filters[panel];
  const rows=data.filter(r=>filter==="ALL" || r.key===filter)
    .sort((a,b)=>{
      const order={STRONG_BUY:0,BUY:1,STRONG_SELL:2,SELL:3,WATCH_UP:4,WATCH_DOWN:5};
      return order[a.key]-order[b.key] || a.symbol.localeCompare(b.symbol);
    });

  if(!rows.length){
    tbody.innerHTML=`<tr><td colspan="${panel==="history"?7:6}" class="empty">Bu filtrede sinyal bulunamadı.</td></tr>`;
    return;
  }

  tbody.innerHTML=rows.map(r=>{
    const timeCell=panel==="history"?`<td>${formatTime(r.time)}</td>`:"";
    const price=panel==="closed"?r.close:r.price;
    return `<tr>
      <td><b>${r.symbol}</b></td>
      <td class="${signalClass(r.key)}">${signalLabel(r.key)}</td>
      ${timeCell}
      <td>${formatPrice(price)}</td>
      <td>${formatVolume(r.quoteVolume)}</td>
      <td>${Number(r.rsi).toFixed(1)}</td>
      <td class="${r.trendOK?"buy":"sell"}">${r.trendOK?"ÜZERİNDE":"ALTINDA"}</td>
    </tr>`;
  }).join("");
}

function renderAll(){
  renderPanel("live",liveResults);
  renderPanel("closed",closedResults);
  renderPanel("history",historyResults);
  document.getElementById("liveCount").textContent=liveResults.length;
  document.getElementById("closedCount").textContent=closedResults.length;
  document.getElementById("historyCount").textContent=historyResults.length;
}

document.querySelectorAll(".filters").forEach(group=>{
  group.querySelectorAll("button").forEach(button=>{
    button.addEventListener("click",()=>{
      const panel=group.dataset.panel;
      filters[panel]=button.dataset.filter;
      group.querySelectorAll("button").forEach(b=>b.classList.remove("active"));
      button.classList.add("active");
      renderPanel(panel,panel==="live"?liveResults:panel==="closed"?closedResults:historyResults);
    });
  });
});

function loadStoredHistory(){
  try{
    const saved=JSON.parse(localStorage.getItem(HISTORY_KEY)||"[]");
    return Array.isArray(saved)?saved:[];
  }catch(e){return [];}
}

function saveHistory(records){
  const cutoff=Date.now()-ONE_DAY;
  const combined=[...loadStoredHistory(),...records]
    .filter(r=>Number(r.time)>=cutoff);
  const unique=new Map();
  combined.forEach(r=>unique.set(`${r.symbol}|${r.time}|${r.key}`,r));
  const clean=Array.from(unique.values()).sort((a,b)=>b.time-a.time);
  try{localStorage.setItem(HISTORY_KEY,JSON.stringify(clean));}catch(e){}
  return clean;
}

async function startScan(){
  if(isScanning) return;
  isScanning=true;
  const btn=document.getElementById("scanBtn");
  btn.disabled=true;
  btn.textContent="TARANIYOR...";
  liveResults=[];
  closedResults=[];
  historyResults=[];
  renderAll();
  document.getElementById("scanned").textContent="0";
  setStatus("Binance Spot pariteleri alınıyor...");

  try{
    const exchange=await getJSON(API+"/api/v3/exchangeInfo");
    const symbols=exchange.symbols
      .filter(s=>s.status==="TRADING" && s.quoteAsset==="USDT" && s.isSpotTradingAllowed)
      .map(s=>s.symbol);

    const ticker=await getJSON(API+"/api/v3/ticker/24hr");
    const tickerMap={};
    ticker.forEach(t=>tickerMap[t.symbol]={
      price:Number(t.lastPrice),
      quoteVolume:Number(t.quoteVolume)
    });

    let completed=0, errors=0, cursor=0;
    const live=[], closed=[], historical=[];
    const cutoff=Date.now()-ONE_DAY;
    const workerCount=3;

    setStatus(`${symbols.length} Spot USDT paritesi bulundu. Tarama başladı...`);

    async function worker(){
      while(cursor<symbols.length){
        const symbol=symbols[cursor++];
        try{
          const [k1,k4]=await Promise.all([
            getJSON(`${API}/api/v3/klines?symbol=${symbol}&interval=1h&limit=100`),
            getJSON(`${API}/api/v3/klines?symbol=${symbol}&interval=4h&limit=220`)
          ]);

          const closed4=k4.filter(c=>Number(c[6])<Date.now());
          if(closed4.length<200) throw new Error("Yetersiz 4s veri");

          const closes4=closed4.map(c=>Number(c[4]));
          const e4=ema(closes4,200);
          const trendOK=closes4[closes4.length-1]>e4[e4.length-1];

          const closed1=k1.filter(c=>Number(c[6])<Date.now());
          if(closed1.length<60) throw new Error("Yetersiz 1s veri");

          const closedInd=prepareIndicators(closed1);
          const lastClosedIndex=closed1.length-1;
          const lastCandle=closed1[lastClosedIndex];
          const lastTime=Number(lastCandle[6]);
          const currentTicker=tickerMap[symbol]||{};
          const closedSignal=getSignalAt(
            lastClosedIndex,closedInd.closes,closedInd.e20,closedInd.e50,
            closedInd.macd,closedInd.macdSignal,closedInd.rsi,
            closedInd.volumes,trendOK
          );

          if(closedSignal){
            closed.push({
              symbol,key:closedSignal.key,rsi:closedSignal.rsi,
              close:Number(lastCandle[4]),price:Number(lastCandle[4]),
              quoteVolume:currentTicker.quoteVolume||0,trendOK,time:lastTime
            });
          }

          const firstHistoryIndex=Math.max(30,closed1.length-24);
          for(let i=firstHistoryIndex;i<closed1.length;i++){
            const time=Number(closed1[i][6]);
            if(time<cutoff) continue;
            const historicalTrend=trendAtTime(closed4,e4,time);
            const s=getSignalAt(
              i,closedInd.closes,closedInd.e20,closedInd.e50,
              closedInd.macd,closedInd.macdSignal,closedInd.rsi,
              closedInd.volumes,historicalTrend
            );
            if(s){
              historical.push({
                symbol,key:s.key,rsi:s.rsi,close:Number(closed1[i][4]),
                price:Number(closed1[i][4]),quoteVolume:currentTicker.quoteVolume||0,
                trendOK:historicalTrend,time
              });
            }
          }

          const liveCandles=k1;
          const liveInd=prepareIndicators(liveCandles);
          const liveIndex=liveCandles.length-1;
          const liveTime=Number(liveCandles[liveIndex][6]);
          const liveTrend=trendOK;
          const liveSignal=getSignalAt(
            liveIndex,liveInd.closes,liveInd.e20,liveInd.e50,
            liveInd.macd,liveInd.macdSignal,liveInd.rsi,
            liveInd.volumes,liveTrend
          );
          if(liveSignal){
            live.push({
              symbol,key:liveSignal.key,rsi:liveSignal.rsi,
              price:currentTicker.price||Number(liveCandles[liveIndex][4]),
              close:Number(liveCandles[liveIndex][4]),
              quoteVolume:currentTicker.quoteVolume||0,trendOK:liveTrend,time:liveTime
            });
          }
        }catch(e){
          errors++;
        }

        completed++;
        if(completed%5===0 || completed===symbols.length){
          document.getElementById("scanned").textContent=`${completed} / ${symbols.length}`;
          setStatus(`Tarama sürüyor: ${completed} / ${symbols.length} parite · Veri alınamayan: ${errors}`);
        }
      }
    }

    await Promise.all(Array.from({length:workerCount},()=>worker()));

    liveResults=live;
    closedResults=closed;
    historyResults=saveHistory(historical);
    renderAll();
    document.getElementById("lastScan").textContent=new Date().toLocaleTimeString("tr-TR");
    setStatus(`Tarama tamamlandı. ${symbols.length} parite kontrol edildi; ${errors} paritede veri alınamadı. Son 24 saat bölümünde ${historyResults.length} sinyal kaydı var.`);
  }catch(e){
    setStatus("Tarama başarısız: Binance verilerine erişilemedi. İnternet bağlantısını kontrol edip tekrar dene.");
  }finally{
    isScanning=false;
    btn.disabled=false;
    btn.textContent="TEKRAR TARA";
  }
}

historyResults=loadStoredHistory().filter(r=>Number(r.time)>=Date.now()-ONE_DAY);
renderAll();
</script>
</body>
</html>
