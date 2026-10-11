<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kripto Al-Sat Tarayıcı</title>
<style>
:root{color-scheme:dark}
*{box-sizing:border-box}
body{margin:0;padding:14px;background:#10151d;color:#f1f5f9;font-family:Arial,Helvetica,sans-serif}
h2{font-size:22px;margin:5px 0 8px}
h3{font-size:17px;margin:18px 0 10px}
p{color:#b9c5d5;font-size:13px;line-height:1.5}
button{font-family:inherit;cursor:pointer}
.primary{width:100%;padding:13px;background:#138a70;color:white;border:0;border-radius:9px;font-size:16px;font-weight:bold}
.primary:disabled{opacity:.5}
#status{margin:12px 0;padding:12px;background:#1c2735;border-radius:8px;font-size:13px;line-height:1.5}
.stats{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:8px;margin:12px 0}
.stat{padding:12px;background:#1b2532;border-radius:8px;font-size:12px;color:#c3cede}
.stat b{display:block;color:white;font-size:18px;margin-top:5px}
.group{background:#18212d;border:1px solid #2c3949;border-radius:10px;margin:10px 0;padding:10px}
.group-title{font-size:15px;font-weight:bold;margin-bottom:9px}
.subfilters{display:flex;flex-wrap:wrap;gap:7px}
.subfilters button{border:1px solid #3b485a;border-radius:7px;background:#263243;color:#e7edf5;padding:9px 11px;font-size:13px}
.subfilters button.active{background:#147e69;border-color:#25b99b;color:white}
.tablewrap{overflow-x:auto;border:1px solid #293646;border-radius:9px;background:#151d28}
table{width:100%;min-width:690px;border-collapse:collapse;font-size:12px}
th,td{padding:11px 9px;border-bottom:1px solid #2a3544;text-align:left;white-space:nowrap}
th{background:#202b3a;color:#d5dfeb}
td{color:#e7edf5}
.buy{color:#48e0a6;font-weight:bold}
.sell{color:#ff8585;font-weight:bold}
.watchup{color:#ffcf70;font-weight:bold}
.watchdown{color:#ffab83;font-weight:bold}
.muted{color:#a9b6c8}
.section-note{font-size:12px;color:#9eacbf}
.history-tools{display:flex;gap:8px;flex-wrap:wrap;margin:8px 0}
.secondary{padding:9px 11px;background:#293648;border:1px solid #425169;color:white;border-radius:7px;font-size:12px}
.empty{padding:15px;color:#aebbd0}
@media(min-width:700px){body{padding:22px}.stats{grid-template-columns:repeat(4,minmax(0,1fr))}}
</style>
</head>
<body>

<h2>Kripto Al-Sat Tarayıcı</h2>
<p>Binance Spot USDT pariteleri · 1 saatlik sinyaller · 4 saatlik EMA200 filtresi</p>

<button class="primary" id="scanBtn" onclick="startScan()">TARAMAYI BAŞLAT</button>
<div id="status">Tarama başlatılmaya hazır.</div>

<div class="stats">
  <div class="stat">Taranan parite<b id="scanned">0</b></div>
  <div class="stat">Güçlü AL / AL<b id="buyCount">0</b></div>
  <div class="stat">Güçlü SAT / SAT<b id="sellCount">0</b></div>
  <div class="stat">Takip sinyalleri<b id="watchCount">0</b></div>
</div>

<div class="group">
  <div class="group-title">AL Sinyalleri</div>
  <div class="subfilters">
    <button class="active" data-filter="BUY_ALL" onclick="setFilter(this)">Tümü</button>
    <button data-filter="STRONG_AL" onclick="setFilter(this)">GÜÇLÜ AL</button>
    <button data-filter="AL" onclick="setFilter(this)">AL</button>
  </div>
</div>

<div class="group">
  <div class="group-title">SAT Sinyalleri</div>
  <div class="subfilters">
    <button data-filter="SELL_ALL" onclick="setFilter(this)">Tümü</button>
    <button data-filter="STRONG_SAT" onclick="setFilter(this)">GÜÇLÜ SAT</button>
    <button data-filter="SAT" onclick="setFilter(this)">SAT</button>
  </div>
</div>

<div class="group">
  <div class="group-title">Takip Sinyalleri</div>
  <div class="subfilters">
    <button data-filter="WATCH_ALL" onclick="setFilter(this)">Tümü</button>
    <button data-filter="UP" onclick="setFilter(this)">Yükseliş Öncesi Takip</button>
    <button data-filter="DOWN" onclick="setFilter(this)">Düşüş Öncesi Takip</button>
  </div>
</div>

<h3 id="resultsTitle">Anlık Sinyaller</h3>
<div class="tablewrap">
<table>
<thead>
<tr>
<th>Parite</th>
<th>Sinyal</th>
<th>Fiyat (USDT)</th>
<th>24s Hacim (USDT)</th>
<th>RSI</th>
<th>4s EMA200</th>
</tr>
</thead>
<tbody id="results">
<tr><td colspan="6" class="empty">Henüz tarama yapılmadı.</td></tr>
</tbody>
</table>
</div>

<h3>Son 24 Saatte Güçlü AL Veren Coinler</h3>
<p>Son 24 kapanmış 1 saatlik mumda oluşan Güçlü AL sinyalleri. Kayıtlar bu tarayıcıda saklanır.</p>
<div class="history-tools">
  <button class="secondary" onclick="showHistory()">Geçmişi Göster</button>
  <button class="secondary" onclick="clearHistory()">Geçmişi Temizle</button>
</div>
<div class="tablewrap">
<table>
<thead>
<tr>
<th>Parite</th>
<th>Sinyal</th>
<th>Sinyal mumunun kapanışı</th>
<th>RSI</th>
<th>Sinyal zamanı</th>
</tr>
</thead>
<tbody id="history">
<tr><td colspan="5" class="empty">Henüz geçmiş kaydı yok.</td></tr>
</tbody>
</table>
</div>

<p class="section-note">
Sinyaller teknik göstergelere dayanır; alım-satım emri vermez ve kazanç garantisi değildir.
Geçmiş kayıtları bu tarayıcının tarama yaptığı verilerden oluşturulur.
</p>

<script>
"use strict";

const API = "https://data-api.binance.vision";
const HISTORY_KEY = "kripto_tarama_strong_buy_history_v3";
let allResults = [];
let activeFilter = "BUY_ALL";
let isScanning = false;

function setFilter(button) {
  activeFilter = button.dataset.filter;
  document.querySelectorAll(".subfilters button")
    .forEach(b => b.classList.toggle("active", b === button));

  const titles = {
    BUY_ALL:"AL Sinyalleri",
    STRONG_AL:"Güçlü AL Sinyalleri",
    AL:"AL Sinyalleri",
    SELL_ALL:"SAT Sinyalleri",
    STRONG_SAT:"Güçlü SAT Sinyalleri",
    SAT:"SAT Sinyalleri",
    WATCH_ALL:"Takip Sinyalleri",
    UP:"Yükseliş Öncesi Takip",
    DOWN:"Düşüş Öncesi Takip"
  };

  document.getElementById("resultsTitle").textContent = titles[activeFilter];
  renderResults();
}

function ema(values, period) {
  if (!values.length) return [];
  const k = 2 / (period + 1);
  const out = Array(values.length).fill(null);
  out[0] = values[0];

  for (let i = 1; i < values.length; i++) {
    out[i] = values[i] * k + out[i - 1] * (1 - k);
  }
  return out;
}

// Wilder RMA yöntemiyle RSI(14).
function rsiSeries(closes, period = 14) {
  const out = Array(closes.length).fill(null);
  if (closes.length <= period) return out;

  let gain = 0;
  let loss = 0;

  for (let i = 1; i <= period; i++) {
    const d = closes[i] - closes[i - 1];
    gain += Math.max(d, 0);
    loss += Math.max(-d, 0);
  }

  let avgGain = gain / period;
  let avgLoss = loss / period;

  function result(g, l) {
    if (l === 0) return g === 0 ? 50 : 100;
    if (g === 0) return 0;
    return 100 - 100 / (1 + g / l);
  }

  out[period] = result(avgGain, avgLoss);

  for (let i = period + 1; i < closes.length; i++) {
    const d = closes[i] - closes[i - 1];
    avgGain = (avgGain * (period - 1) + Math.max(d, 0)) / period;
    avgLoss = (avgLoss * (period - 1) + Math.max(-d, 0)) / period;
    out[i] = result(avgGain, avgLoss);
  }

  return out;
}

function macdSeries(closes) {
  const fast = ema(closes, 12);
  const slow = ema(closes, 26);
  const line = closes.map((_, i) => fast[i] - slow[i]);
  const signal = ema(line, 9);
  return {line, signal};
}

async function getJSON(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error("HTTP " + response.status);
  return response.json();
}

function getHistory() {
  try {
    const data = JSON.parse(localStorage.getItem(HISTORY_KEY) || "[]");
    return Array.isArray(data) ? data : [];
  } catch {
    return [];
  }
}

function saveHistory(records) {
  const cutoff = Date.now() - 24 * 60 * 60 * 1000;
  const unique = new Map();

  records
    .filter(r => Number(r.time) >= cutoff)
    .forEach(r => unique.set(r.symbol + "_" + r.time, r));

  const clean = [...unique.values()]
    .sort((a, b) => b.time - a.time)
    .slice(0, 3000);

  try {
    localStorage.setItem(HISTORY_KEY, JSON.stringify(clean));
  } catch (e) {
    // Tarama devam eder; tarayıcı depolaması doluysa kayıt yapılamayabilir.
  }
}

function recordStrongBuy(symbol, candleTime, price, rsi) {
  const records = getHistory();
  const key = symbol + "_" + candleTime;

  if (records.some(r => r.symbol + "_" + r.time === key)) return;

  records.push({
    symbol,
    time: candleTime,
    price,
    rsi
  });

  saveHistory(records);
}

function fmtPrice(n) {
  if (!Number.isFinite(n)) return "-";
  return n.toLocaleString("en-US", {maximumFractionDigits:8});
}

function fmtVolume(n) {
  if (!Number.isFinite(n)) return "-";
  return n.toLocaleString("en-US", {maximumFractionDigits:0});
}

function analyzePair(symbol, candles1h, candles4h, ticker) {
  // Son mum açık olabilir. Yalnızca kapanmış mumlar değerlendirilir.
  const closed1h = candles1h.filter(c => Number(c[6]) < Date.now());
  const closed4h = candles4h.filter(c => Number(c[6]) < Date.now());

  if (closed1h.length < 60 || closed4h.length < 200) return null;

  const closes = closed1h.map(c => Number(c[4]));
  const volumes = closed1h.map(c => Number(c[5]));
  const e20 = ema(closes, 20);
  const e50 = ema(closes, 50);
  const macd = macdSeries(closes);
  const rsi = rsiSeries(closes, 14);

  const closes4h = closed4h.map(c => Number(c[4]));
  const e200_4h = ema(closes4h, 200);

  const i = closes.length - 1;
  const p = i - 1;
  if (rsi[i] == null || rsi[p] == null) return null;

  const trendOK = closes4h.at(-1) > e200_4h.at(-1);
  const crossUp = e20[p] <= e50[p] && e20[i] > e50[i];
  const crossDown = e20[p] >= e50[p] && e20[i] < e50[i];
  const macdUp = macd.line[i] > macd.signal[i];
  const macdDown = macd.line[i] < macd.signal[i];

  const gapNow = Math.abs(e20[i] - e50[i]) / closes[i];
  const gapBefore = Math.abs(e20[p] - e50[p]) / closes[p];
  const gapNarrowing = gapNow < gapBefore;

  const avgVol = volumes.slice(-21, -1).reduce((a,b) => a+b, 0) / 20;
  const volumeOK = avgVol > 0 && volumes[i] >= avgVol;

  const currentRSI = rsi[i];
  const previousRSI = rsi[p];

  const strongBuy = trendOK && crossUp && macdUp && currentRSI >= 50;
  const strongSell = crossDown && macdDown && currentRSI <= 50;
  const normalBuy = trendOK && crossUp && macdUp && currentRSI >= 40;
  const normalSell = crossDown && macdDown && currentRSI <= 60;

  let signal = "";
  let type = "";
  let cls = "";

  if (strongBuy) {
    signal = "GÜÇLÜ AL";
    type = "STRONG_AL";
    cls = "buy";
  } else if (strongSell) {
    signal = "GÜÇLÜ SAT";
    type = "STRONG_SAT";
    cls = "sell";
  } else if (normalBuy) {
    signal = "AL";
    type = "AL";
    cls = "buy";
  } else if (normalSell) {
    signal = "SAT";
    type = "SAT";
    cls = "sell";
  } else {
    const earlyUp =
      !crossUp && trendOK && e20[i] <= e50[i] &&
      gapNarrowing && gapNow <= 0.008 &&
      macd.line[i] > macd.line[p] &&
      currentRSI > previousRSI &&
      currentRSI >= 40 && currentRSI <= 60 && volumeOK;

    const earlyDown =
      !crossDown && e20[i] >= e50[i] &&
      gapNarrowing && gapNow <= 0.008 &&
      macd.line[i] < macd.line[p] &&
      currentRSI < previousRSI &&
      currentRSI >= 40 && currentRSI <= 60 && volumeOK;

    if (earlyUp) {
      signal = "YÜKSELİŞ ÖNCESİ TAKİP";
      type = "UP";
      cls = "watchup";
    } else if (earlyDown) {
      signal = "DÜŞÜŞ ÖNCESİ TAKİP";
      type = "DOWN";
      cls = "watchdown";
    }
  }

  // Son 24 kapanmış saatlik mumdaki Güçlü AL sinyallerini bul.
  // Her sinyal kendi mum kapanışında hesaplanır.
  const first = Math.max(35, closed1h.length - 24);

  for (let j = first; j < closed1h.length; j++) {
    if (rsi[j] == null || j < 1) continue;

    const trendAtJ = trendForTime(closed4h, e200_4h, Number(closed1h[j][6]));
    if (trendAtJ === null) continue;

    const crossAtJ = e20[j-1] <= e50[j-1] && e20[j] > e50[j];
    const macdAtJ = macd.line[j] > macd.signal[j];
    const strongAtJ = trendAtJ && crossAtJ && macdAtJ && rsi[j] >= 50;

    if (strongAtJ) {
      recordStrongBuy(
        symbol,
        Number(closed1h[j][6]),
        Number(closed1h[j][4]),
        rsi[j]
      );
    }
  }

  if (!signal) return null;

  return {
    symbol, signal, type, cls,
    price: Number(ticker.lastPrice),
    volume: Number(ticker.quoteVolume),
    rsi: currentRSI,
    trendOK
  };
}

function trendForTime(candles4h, ema200, time) {
  let idx = -1;

  for (let i = 0; i < candles4h.length; i++) {
    if (Number(candles4h[i][6]) <= time) idx = i;
    else break;
  }

  if (idx < 199) return null;
  return Number(candles4h[idx][4]) > ema200[idx];
}

function renderResults() {
  let rows = allResults;

  if (activeFilter === "BUY_ALL") {
    rows = rows.filter(r => ["STRONG_AL","AL"].includes(r.type));
  } else if (activeFilter === "SELL_ALL") {
    rows = rows.filter(r => ["STRONG_SAT","SAT"].includes(r.type));
  } else if (activeFilter === "WATCH_ALL") {
    rows = rows.filter(r => ["UP","DOWN"].includes(r.type));
  } else {
    rows = rows.filter(r => r.type === activeFilter);
  }

  const order = {
    STRONG_AL:0, AL:1, STRONG_SAT:2, SAT:3, UP:4, DOWN:5
  };

  rows.sort((a,b) =>
    order[a.type] - order[b.type] || a.symbol.localeCompare(b.symbol)
  );

  const body = document.getElementById("results");

  if (!rows.length) {
    body.innerHTML = '<tr><td colspan="6" class="empty">Bu grupta sinyal bulunamadı.</td></tr>';
    return;
  }

  body.innerHTML = rows.map(r => `
    <tr>
      <td><b>${r.symbol}</b></td>
      <td class="${r.cls}">${r.signal}</td>
      <td>${fmtPrice(r.price)}</td>
      <td>${fmtVolume(r.volume)}</td>
      <td>${r.rsi.toFixed(1)}</td>
      <td class="${r.trendOK ? "buy" : "sell"}">${r.trendOK ? "ÜZERİNDE" : "ALTINDA"}</td>
    </tr>
  `).join("");
}

function showHistory() {
  const body = document.getElementById("history");
  const cutoff = Date.now() - 24 * 60 * 60 * 1000;
  const rows = getHistory()
    .filter(r => Number(r.time) >= cutoff)
    .sort((a,b) => b.time - a.time);

  if (!rows.length) {
    body.innerHTML = '<tr><td colspan="5" class="empty">Son 24 saat için kayıt yok. Tarama yapıldıkça sinyal geçmişi oluşturulur.</td></tr>';
    return;
  }

  body.innerHTML = rows.map(r => `
    <tr>
      <td><b>${r.symbol}</b></td>
      <td class="buy">GÜÇLÜ AL</td>
      <td>${fmtPrice(Number(r.price))}</td>
      <td>${Number(r.rsi).toFixed(1)}</td>
      <td>${new Date(Number(r.time)).toLocaleString("tr-TR")}</td>
    </tr>
  `).join("");
}

function clearHistory() {
  if (!confirm("Kaydedilmiş son 24 saatlik Güçlü AL geçmişi silinsin mi?")) return;
  localStorage.removeItem(HISTORY_KEY);
  showHistory();
}

async function startScan() {
  if (isScanning) return;

  isScanning = true;
  allResults = [];

  const button = document.getElementById("scanBtn");
  const status = document.getElementById("status");
  button.disabled = true;
  button.textContent = "TARANIYOR...";

  document.getElementById("results").innerHTML =
    '<tr><td colspan="6" class="empty">Tarama devam ediyor...</td></tr>';

  try {
    status.textContent = "Binance Spot pariteleri ve 24 saatlik hacimler alınıyor...";

    const [exchange, tickers] = await Promise.all([
      getJSON(API + "/api/v3/exchangeInfo"),
      getJSON(API + "/api/v3/ticker/24hr")
    ]);

    const tickerMap = {};
    tickers.forEach(t => {
      tickerMap[t.symbol] = t;
    });

    const symbols = exchange.symbols
      .filter(s =>
        s.status === "TRADING" &&
        s.quoteAsset === "USDT" &&
        s.isSpotTradingAllowed !== false &&
        tickerMap[s.symbol]
      )
      .map(s => s.symbol);

    let cursor = 0;
    let completed = 0;
    const workers = 3;

    async function worker() {
      while (true) {
        const idx = cursor++;
        if (idx >= symbols.length) break;

        const symbol = symbols[idx];

        try {
          const [k1, k4] = await Promise.all([
            getJSON(API + "/api/v3/klines?symbol=" + symbol + "&interval=1h&limit=100"),
            getJSON(API + "/api/v3/klines?symbol=" + symbol + "&interval=4h&limit=250")
          ]);

          const result = analyzePair(symbol, k1, k4, tickerMap[symbol]);
          if (result) allResults.push(result);
        } catch (e) {
          // Bir coin başarısız olursa diğer coinler taranmaya devam eder.
        }

        completed++;
        if (completed % 10 === 0 || completed === symbols.length) {
          document.getElementById("scanned").textContent =
            completed + " / " + symbols.length;
          status.textContent =
            "Tarama sürüyor: " + completed + " / " + symbols.length;
        }
      }
    }

    await Promise.all(Array.from({length:workers}, () => worker()));

    document.getElementById("buyCount").textContent =
      allResults.filter(r => ["STRONG_AL","AL"].includes(r.type)).length;
    document.getElementById("sellCount").textContent =
      allResults.filter(r => ["STRONG_SAT","SAT"].includes(r.type)).length;
    document.getElementById("watchCount").textContent =
      allResults.filter(r => ["UP","DOWN"].includes(r.type)).length;

    renderResults();
    showHistory();

    status.textContent =
      "Tarama tamamlandı. Kontrol edilen parite: " + symbols.length +
      ". Son tarama: " + new Date().toLocaleString("tr-TR");

  } catch (e) {
    status.textContent =
      "Veri alınamadı. İnternet bağlantısını kontrol edip tekrar dene.";
  } finally {
    isScanning = false;
    button.disabled = false;
    button.textContent = "TARAMAYI BAŞLAT";
  }
}

showHistory();
</script>
</body>
</html>
