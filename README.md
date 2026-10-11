# kucoin-tarama2

<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kripto Al-Sat Tarayıcı</title>
<style>
*{box-sizing:border-box}
body{margin:0;padding:14px;background:#10151d;color:#e8edf5;font-family:Arial,sans-serif}
main{max-width:1100px;margin:auto}
h2,h3{margin:10px 0}
p{color:#aab6c7;font-size:14px;line-height:1.5}
button{background:#16a085;color:white;border:0;border-radius:8px;padding:12px 15px;font-size:14px;font-weight:bold;cursor:pointer;margin:4px}
button:disabled{opacity:.5}
.secondary{background:#344256}.danger{background:#8e3b46}
#status{margin:10px 0;padding:12px;background:#1b2532;border-radius:8px;font-size:13px;line-height:1.6}
.summary{display:flex;gap:7px;flex-wrap:wrap;margin:12px 0}
.pill{background:#1b2532;padding:10px;border-radius:8px;font-size:13px}
.tablewrap{overflow-x:auto;background:#171f2b;border-radius:8px;margin:10px 0 20px}
table{border-collapse:collapse;width:100%;min-width:680px;font-size:12px}
th,td{padding:10px 8px;border-bottom:1px solid #2b3544;text-align:left;white-space:nowrap}
th{background:#202b3a;color:#b9c8da}
.buy{color:#36d399;font-weight:bold}.sell{color:#ff7474;font-weight:bold}
.watchup{color:#ffd166;font-weight:bold}.watchdown{color:#ff9f7a;font-weight:bold}
.muted{color:#aab6c7}.note{font-size:12px}
</style>
</head>
<body>
<main>
<h2>Kripto Al-Sat Tarayıcı</h2>
<p>Binance Spot USDT pariteleri · 1 saatlik sinyal · 4 saatlik EMA200 filtresi.</p>

<button id="scanBtn" onclick="startScan()">TARAMAYI BAŞLAT</button>
<button class="secondary" onclick="exportHistory()">GEÇMİŞİ İNDİR</button>
<button class="danger" onclick="clearHistory()">GEÇMİŞİ TEMİZLE</button>

<div id="status">Tarama başlatılmaya hazır.</div>

<div class="summary">
<div class="pill">Taranan: <b id="scanned">0</b></div>
<div class="pill">Yükseliş takibi: <b id="upCount">0</b></div>
<div class="pill">Düşüş takibi: <b id="downCount">0</b></div>
<div class="pill">Güncel AL/SAT: <b id="signalCount">0</b></div>
<div class="pill">Geçmiş AL: <b id="historyCount">0</b></div>
</div>

<h3>1. Güncel tarama</h3>
<p class="note">Son kapanmış 1 saatlik mumun sinyalleri. Her yeni taramada değişebilir.</p>
<div class="tablewrap">
<table>
<thead><tr><th>Coin</th><th>Sinyal</th><th>Fiyat</th><th>Son kapanış</th><th>RSI</th><th>4s EMA200</th></tr></thead>
<tbody id="results"><tr><td colspan="6">Henüz tarama yapılmadı.</td></tr></tbody>
</table>
</div>

<h3>2. Kapanmış mum sinyal geçmişi</h3>
<p class="note">Her taramada son 24 kapanmış saatlik mum incelenir. Geçmişte bulunan AL ve GÜÇLÜ AL sinyalleri kaydedilir. Aynı coin ve mum için ikinci kayıt oluşturulmaz.</p>
<div class="tablewrap">
<table>
<thead><tr><th>Coin</th><th>Oluşan sinyal</th><th>Mum kapanış zamanı</th><th>Mum kapanışı</th><th>RSI</th><th>Son durum</th></tr></thead>
<tbody id="historyRows"><tr><td colspan="6">Henüz kayıt yok.</td></tr></tbody>
</table>
</div>

<p class="note">Geçmiş kayıtları bu tarayıcının yerel hafızasında saklanır. Tarayıcı kapalıyken oluşan sinyaller otomatik yakalanmaz; yeniden tarama yapılınca son 24 kapanmış mum geriye dönük incelenir. Bu araç emir göndermez ve yatırım tavsiyesi vermez.</p>
</main>

<script>
const API="https://data-api.binance.vision";
const HISTORY_KEY="kucoinTarama_closedBuyHistory_v1";
const $=id=>document.getElementById(id);
let isScanning=false,allResults=[];
let signalHistory=loadHistory();

function loadHistory(){
  try{
    const x=JSON.parse(localStorage.getItem(HISTORY_KEY)||"[]");
    return Array.isArray(x)?x:[];
  }catch(e){return []}
}
function saveHistory(){
  try{
    localStorage.setItem(HISTORY_KEY,JSON.stringify(signalHistory.slice(0,1500)));
  }catch(e){
    $("status").textContent+=" | Yerel kayıt alanı dolu olabilir.";
  }
}
function fmtTime(ms){
  return new Date(Number(ms)).toLocaleString("tr-TR",{hour12:false});
}
function priceFmt(n){
  if(!Number.isFinite(n))return "-";
  return n<1?n.toFixed(8).replace(/0+$/,"").replace(/\.$/,""):n.toFixed(4);
}
function ema(values,period){
  if(!values.length)return [];
  const k=2/(period+1),out=[values[0]];
  for(let i=1;i<values.length;i++)out.push(values[i]*k+out[i-1]*(1-k));
  return out;
}
function rsiSeries(closes,period=14){
  const out=Array(closes.length).fill(null);
  if(closes.length<=period)return out;
  let gains=0,losses=0;
  for(let i=1;i<=period;i++){
    const d=closes[i]-closes[i-1];
    if(d>0)gains+=d;else losses-=d;
  }
  let ag=gains/period,al=losses/period;
  const calc=(g,l)=>l===0?(g===0?50:100):100-100/(1+g/l);
  out[period]=calc(ag,al);
  for(let i=period+1;i<closes.length;i++){
    const d=closes[i]-closes[i-1];
    ag=(ag*(period-1)+Math.max(d,0))/period;
    al=(al*(period-1)+Math.max(-d,0))/period;
    out[i]=calc(ag,al);
  }
  return out;
}
async function getJSON(url){
  const res=await fetch(url);
  if(!res.ok)throw new Error("API yanıtı "+res.status);
  return res.json();
}

/*
  Yalnızca gerçekten kapanmış mumlar kullanılır.
  4 saatlik EMA200, ilgili saatlik mumun kapanış zamanında
  mevcut olan son kapanmış 4 saatlik mumdan hesaplanır.
*/
function analyzeCandles(symbol,k1,k4,currentPrice){
  const now=Date.now();
  const h1=k1.filter(k=>Number(k[6])<now);
  const h4=k4.filter(k=>Number(k[6])<now);

  if(h1.length<60||h4.length<200)return {current:null,buySignals:[]};

  const c=h1.map(k=>Number(k[4]));
  const v=h1.map(k=>Number(k[5]));
  const e20=ema(c,20),e50=ema(c,50);
  const mf=ema(c,12),ms=ema(c,26);
  const ml=c.map((_,i)=>mf[i]-ms[i]);
  const mSignal=ema(ml,9);
  const rsi=rsiSeries(c,14);

  const c4=h4.map(k=>Number(k[4]));
  const e4=ema(c4,200);
  let j4=0;
  const trendAt=[];

  for(let i=0;i<h1.length;i++){
    const closeTime=Number(h1[i][6]);
    while(j4+1<h4.length&&Number(h4[j4+1][6])<=closeTime)j4++;
    if(Number(h4[j4][6])<=closeTime&&j4>=199){
      trendAt[i]=c4[j4]>e4[j4];
    }else{
      trendAt[i]=null;
    }
  }

  function signalAt(i){
    if(i<35||rsi[i]==null||rsi[i-1]==null||trendAt[i]===null)return null;

    const p=i-1;
    const upCross=e20[p]<=e50[p]&&e20[i]>e50[i];
    const downCross=e20[p]>=e50[p]&&e20[i]<e50[i];
    const macdUp=ml[i]>mSignal[i];
    const macdDown=ml[i]<mSignal[i];
    const macdImproving=ml[i]>ml[p];
    const macdWeakening=ml[i]<ml[p];
    const gap=Math.abs(e20[i]-e50[i])/c[i];
    const prevGap=Math.abs(e20[p]-e50[p])/c[p];
    const narrowing=gap<prevGap;
    const avgVol=v.slice(i-20,i).reduce((a,b)=>a+b,0)/20;
    const volOK=avgVol>0&&v[i]>=avgVol;
    const r=rsi[i];

    let signal="",type="",cls="";

    if(upCross&&macdUp&&r>=40&&trendAt[i]){
      signal=r>=50?"GÜÇLÜ AL":"AL";
      type="BUY";cls="buy";
    }else if(downCross&&macdDown&&r<=60){
      signal=r<=50?"GÜÇLÜ SAT":"SAT";
      type="SELL";cls="sell";
    }else{
      const earlyUp=!upCross&&trendAt[i]&&e20[i]<=e50[i]&&
        narrowing&&gap<=0.008&&macdImproving&&
        r>rsi[p]&&r>=40&&r<=60&&volOK;

      const earlyDown=!downCross&&e20[i]>=e50[i]&&
        narrowing&&gap<=0.008&&macdWeakening&&
        r<rsi[p]&&r>=40&&r<=60&&volOK;

      if(earlyUp){
        signal="YÜKSELİŞ ÖNCESİ TAKİP";type="UP";cls="watchup";
      }else if(earlyDown){
        signal="DÜŞÜŞ ÖNCESİ TAKİP";type="DOWN";cls="watchdown";
      }
    }

    if(!signal)return null;

    return {
      symbol,signal,type,cls,
      currentPrice:Number(currentPrice||c[i]),
      lastClose:c[i],rsi:r,
      trendOK:trendAt[i],
      candleOpenTime:Number(h1[i][0]),
      candleCloseTime:Number(h1[i][6])
    };
  }

  const lastIndex=h1.length-1;
  const current=signalAt(lastIndex);
  const buySignals=[];
  const start=Math.max(35,lastIndex-23);

  for(let i=start;i<=lastIndex;i++){
    const r=signalAt(i);
    if(r&&r.type==="BUY")buySignals.push(r);
  }
  return {current,buySignals};
}

function recordHistoricalBuys(rows){
  const existing=new Set(signalHistory.map(h=>h.key));
  const now=Date.now();

  for(const r of rows){
    const key=r.symbol+"_"+r.candleCloseTime;
    if(existing.has(key))continue;
    signalHistory.unshift({
      key,symbol:r.symbol,signal:r.signal,
      candleOpenTime:r.candleOpenTime,
      candleCloseTime:r.candleCloseTime,
      close:r.lastClose,rsi:r.rsi,
      recordedAt:now,
      lastStatus:"Geçmiş mumda sinyal bulundu",
      lastChecked:now
    });
    existing.add(key);
  }
  signalHistory=signalHistory.slice(0,1500);
  saveHistory();
}

function updateHistoryStatus(){
  const latestBySymbol=new Map();
  for(const r of allResults)latestBySymbol.set(r.symbol,r);

  signalHistory=signalHistory.map(h=>{
    const r=latestBySymbol.get(h.symbol);
    let status="Güncel taramada yeni sinyal yok";
    if(r&&r.type==="BUY")status=r.signal+" devam";
    else if(r&&r.type==="UP")status="Yükseliş öncesi takip";
    else if(r&&r.type==="SELL")status="SAT sinyali";
    else if(r&&r.type==="DOWN")status="Düşüş öncesi takip";
    return {...h,lastStatus:status,lastChecked:Date.now()};
  });
  saveHistory();
}

function renderCurrent(){
  const order={BUY:0,SELL:1,UP:2,DOWN:3};
  const rows=[...allResults].sort((a,b)=>
    (order[a.type]??9)-(order[b.type]??9)||a.symbol.localeCompare(b.symbol)
  );
  if(!rows.length){
    $("results").innerHTML='<tr><td colspan="6">Son kapanmış mumda sinyal bulunamadı. Geçmiş sinyaller aşağıda korunur.</td></tr>';
    return;
  }
  $("results").innerHTML=rows.map(r=>
    `<tr><td><b>${r.symbol}</b></td>
    <td class="${r.cls}">${r.signal}</td>
    <td>${priceFmt(r.currentPrice)}</td>
    <td>${priceFmt(r.lastClose)}</td>
    <td>${r.rsi.toFixed(1)}</td>
    <td class="${r.trendOK?"buy":"sell"}">${r.trendOK?"ÜZERİNDE":"ALTINDA"}</td></tr>`
  ).join("");
}

function renderHistory(){
  $("historyCount").textContent=signalHistory.length;
  if(!signalHistory.length){
    $("historyRows").innerHTML='<tr><td colspan="6">Son 24 kapanmış mumda AL sinyali bulunamadı veya tarama henüz tamamlanmadı.</td></tr>';
    return;
  }
  $("historyRows").innerHTML=signalHistory.map(h=>
    `<tr><td><b>${h.symbol}</b></td>
    <td class="buy">${h.signal}</td>
    <td>${fmtTime(h.candleCloseTime)}</td>
    <td>${priceFmt(Number(h.close))}</td>
    <td>${Number(h.rsi).toFixed(1)}</td>
    <td class="muted">${h.lastStatus||"Kaydedildi"}</td></tr>`
  ).join("");
}

async function startScan(){
  if(isScanning)return;
  isScanning=true;
  const btn=$("scanBtn");
  btn.disabled=true;btn.textContent="TARANIYOR...";
  $("scanned").textContent="0";
  $("results").innerHTML='<tr><td colspan="6">Pariteler taranıyor...</td></tr>';
  allResults=[];

  try{
    $("status").textContent="Binance Spot pariteleri alınıyor...";
    const info=await getJSON(API+"/api/v3/exchangeInfo");
    const symbols=info.symbols
      .filter(s=>s.status==="TRADING"&&s.quoteAsset==="USDT"&&s.isSpotTradingAllowed)
      .map(s=>s.symbol);

    let prices={};
    try{
      const p=await getJSON(API+"/api/v3/ticker/price");
      p.forEach(x=>prices[x.symbol]=Number(x.price));
    }catch(e){}

    let cursor=0,completed=0,errors=0;
    const historicalBuys=[];
    const workers=Array.from({length:3},async()=>{
      while(cursor<symbols.length){
        const symbol=symbols[cursor++];
        try{
          const [k1,k4]=await Promise.all([
            getJSON(API+"/api/v3/klines?symbol="+symbol+"&interval=1h&limit=100"),
            getJSON(API+"/api/v3/klines?symbol="+symbol+"&interval=4h&limit=250")
          ]);
          const result=analyzeCandles(symbol,k1,k4,prices[symbol]);
          if(result.current)allResults.push(result.current);
          historicalBuys.push(...result.buySignals);
        }catch(e){errors++}

        completed++;
        if(completed%10===0||completed===symbols.length){
          $("scanned").textContent=completed+" / "+symbols.length;
          $("status").textContent="Tarama sürüyor: "+completed+" / "+symbols.length+
            " parite · Veri alınamayan: "+errors;
        }
      }
    });

    await Promise.all(workers);

    recordHistoricalBuys(historicalBuys);
    updateHistoryStatus();

    $("upCount").textContent=allResults.filter(r=>r.type==="UP").length;
    $("downCount").textContent=allResults.filter(r=>r.type==="DOWN").length;
    $("signalCount").textContent=allResults.filter(r=>r.type==="BUY"||r.type==="SELL").length;

    renderCurrent();
    renderHistory();

    $("status").textContent="Tarama tamamlandı. "+symbols.length+
      " parite kontrol edildi; "+errors+" paritede veri alınamadı. Son 24 kapanmış saatlik mumdaki AL sinyalleri geçmişe işlendi. "+new Date().toLocaleString("tr-TR");
  }catch(e){
    $("status").textContent="Tarama başarısız: Binance verilerine erişilemedi ("+e.message+"). Önceki kayıtlar korundu.";
    renderHistory();
  }finally{
    isScanning=false;
    btn.disabled=false;btn.textContent="TARAMAYI BAŞLAT";
  }
}

function clearHistory(){
  if(!confirm("Tüm kayıtlı AL sinyalleri silinsin mi?"))return;
  signalHistory=[];
  saveHistory();
  renderHistory();
}
function exportHistory(){
  const blob=new Blob([JSON.stringify(signalHistory,null,2)],{type:"application/json"});
  const url=URL.createObjectURL(blob);
  const a=document.createElement("a");
  a.href=url;a.download="kripto-sinyal-gecmisi.json";
  a.click();URL.revokeObjectURL(url);
}
renderHistory();
</script>
</body>
</html>
