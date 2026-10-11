<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kripto Al-Sat Tarayıcı</title>
<style>
*{box-sizing:border-box}
body{margin:0;padding:16px;background:#10151d;color:#e8edf5;font-family:Arial,sans-serif}
h2{margin:4px 0 8px}
p{color:#aab6c7;font-size:14px;line-height:1.5}
button{background:#16a085;color:white;border:0;border-radius:8px;padding:13px 18px;font-size:16px;font-weight:bold;cursor:pointer}
button:disabled{opacity:.5}
#status{margin:14px 0;padding:12px;background:#1b2532;border-radius:8px;font-size:14px}
.filters{display:flex;gap:8px;flex-wrap:wrap;margin:12px 0}
.filters button{font-size:13px;padding:9px 12px;background:#283444}
.filters button.active{background:#16a085}
.summary{display:flex;gap:8px;flex-wrap:wrap;margin:12px 0}
.pill{background:#1b2532;padding:10px;border-radius:8px;font-size:13px}
.tablewrap{overflow-x:auto;background:#171f2b;border-radius:8px}
table{border-collapse:collapse;width:100%;min-width:680px;font-size:13px}
th,td{padding:11px 9px;border-bottom:1px solid #2b3544;text-align:left;white-space:nowrap}
th{background:#202b3a;color:#b9c8da;position:sticky;top:0}
.buy{color:#36d399;font-weight:bold}
.sell{color:#ff7474;font-weight:bold}
.watchup{color:#ffd166;font-weight:bold}
.watchdown{color:#ff9f7a;font-weight:bold}
.small{font-size:12px;color:#9cabbf}
</style>
</head>
<body>

<h2>Kripto Al-Sat Tarayıcı</h2>
<p>Binance Spot USDT pariteleri · 1 saatlik sinyal · 4 saatlik EMA200 filtresi</p>

<button id="scanBtn" onclick="startScan()">TEKRAR TARA</button>
<div id="status">Tarama başlatılmaya hazır.</div>

<div class="summary">
  <div class="pill">Taranan: <b id="scanned">0</b></div>
  <div class="pill">Yükseliş takibi: <b id="upCount">0</b></div>
  <div class="pill">Düşüş takibi: <b id="downCount">0</b></div>
  <div class="pill">AL/SAT: <b id="signalCount">0</b></div>
</div>

<div class="filters">
  <button class="active" onclick="setFilter('ALL',this)">TÜM UYARILAR</button>
  <button onclick="setFilter('UP',this)">YÜKSELİŞ ÖNCESİ TAKİP</button>
  <button onclick="setFilter('AL',this)">AL</button>
  <button onclick="setFilter('STRONG_AL',this)">GÜÇLÜ AL</button>
  <button onclick="setFilter('DOWN',this)">DÜŞÜŞ ÖNCESİ TAKİP</button>
  <button onclick="setFilter('SAT',this)">SAT</button>
  <button onclick="setFilter('STRONG_SAT',this)">GÜÇLÜ SAT</button>
</div>

<div class="tablewrap">
<table>
<thead>
<tr>
<th>Parite</th>
<th>Sinyal</th>
<th>Güncel fiyat</th>
<th>24s hacim (USDT)</th>
<th>RSI</th>
<th>4s EMA200</th>
</tr>
</thead>
<tbody id="results">
<tr><td colspan="6">Henüz tarama yapılmadı.</td></tr>
</tbody>
</table>
</div>

<p class="small">
Tarama manuel çalışır ve gerçek alım-satım emri vermez.
Sinyaller kesin kazanç garantisi değildir.
</p>

<script>
const API = "https://data-api.binance.vision";

let allResults = [];
let activeFilter = "ALL";
let isScanning = false;

function setFilter(filter, button) {
  activeFilter = filter;
  document.querySelectorAll(".filters button")
    .forEach(b => b.classList.remove("active"));
  button.classList.add("active");
  renderResults();
}

function ema(values, period) {
  if (!values.length) return [];
  const k = 2 / (period + 1);
  const out = [values[0]];

  for (let i = 1; i < values.length; i++) {
    out.push(values[i] * k + out[i - 1] * (1 - k));
  }
  return out;
}

// TradingView ta.rsi() ile aynı Wilder/RMA yaklaşımı.
function rsiSeries(closes, period = 14) {
  const out = Array(closes.length).fill(null);
  if (closes.length <= period) return out;

  let gainSum = 0;
  let lossSum = 0;

  for (let i = 1; i <= period; i++) {
    const change = closes[i] - closes[i - 1];
    gainSum += Math.max(change, 0);
    lossSum += Math.max(-change, 0);
  }

  let avgGain = gainSum / period;
  let avgLoss = lossSum / period;

  function value(gain, loss) {
    if (loss === 0) return gain === 0 ? 50 : 100;
    if (gain === 0) return 0;
    return 100 - 100 / (1 + gain / loss);
  }

  out[period] = value(avgGain, avgLoss);

  for (let i = period + 1; i < closes.length; i++) {
    const change = closes[i] - closes[i - 1];
    avgGain = (avgGain * (period - 1) + Math.max(change, 0)) / period;
    avgLoss = (avgLoss * (period - 1) + Math.max(-change, 0)) / period;
    out[i] = value(avgGain, avgLoss);
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
  if (!response.ok) throw new Error("API yanıtı: " + response.status);
  return response.json();
}

function analyzePair(symbol, candles1h, candles4h, currentPrice, quoteVolume) {
  // Açık olan son mum sinyal hesabına katılmaz.
  const c1 = candles1h.slice(0, -1);
  const c4 = candles4h.slice(0, -1);

  if (c1.length < 60 || c4.length < 200) return null;

  const close = c1.map(x => Number(x[4]));
  const volume = c1.map(x => Number(x[5]));

  const e20 = ema(close, 20);
  const e50 = ema(close, 50);
  const macd = macdSeries(close);
  const rsi = rsiSeries(close, 14);

  const i = close.length - 1;
  const p = i - 1;

  if (p < 30 || rsi[i] == null || rsi[p] == null) return null;

  const lastClose = close[i];
  const currentRSI = rsi[i];
  const previousRSI = rsi[p];

  const crossUp = e20[p] <= e50[p] && e20[i] > e50[i];
  const crossDown = e20[p] >= e50[p] && e20[i] < e50[i];

  const macdUp = macd.line[i] > macd.signal[i];
  const macdDown = macd.line[i] < macd.signal[i];

  const macdImproving = macd.line[i] > macd.line[p];
  const macdWeakening = macd.line[i] < macd.line[p];

  const gapNow = Math.abs(e20[i] - e50[i]) / lastClose;
  const gapBefore = Math.abs(e20[p] - e50[p]) / close[p];
  const gapNarrowing = gapNow < gapBefore;

  const avgVol =
    volume.slice(-21, -1).reduce((a, b) => a + b, 0) / 20;
  const volumeOK = avgVol > 0 && volume[i] >= avgVol;

  const close4h = c4.map(x => Number(x[4]));
  const ema200_4h = ema(close4h, 200).at(-1);
  const price4h = close4h.at(-1);
  const trendOK = price4h > ema200_4h;

  let signal = "";
  let type = "";
  let cls = "";

  // TradingView:
  // al = 4H trend + EMA20 crossover EMA50 + MACD above signal + RSI >= 40
  // gucluAl = al + RSI >= 50
  const strongBuy =
    trendOK &&
    crossUp &&
    macdUp &&
    currentRSI >= 50;

  // TradingView:
  // sat = EMA20 crossunder EMA50 + MACD below signal + RSI <= 60
  // gucluSat = sat + RSI <= 50
  // Güçlü SAT için 4H filtresi kullanılmaz.
  const strongSell =
    crossDown &&
    macdDown &&
    currentRSI <= 50;

  const normalBuy =
    trendOK &&
    crossUp &&
    macdUp &&
    currentRSI >= 40;

  const normalSell =
    crossDown &&
    macdDown &&
    currentRSI <= 60;

  // Güçlü AL / Güçlü SAT önce kontrol edilir.
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
    // Takip kuralları değiştirilmedi.
    const earlyUp =
      !crossUp &&
      trendOK &&
      e20[i] <= e50[i] &&
      gapNarrowing &&
      gapNow <= 0.008 &&
      macdImproving &&
      currentRSI > previousRSI &&
      currentRSI >= 40 &&
      currentRSI <= 60 &&
      volumeOK;

    const earlyDown =
      !crossDown &&
      e20[i] >= e50[i] &&
      gapNarrowing &&
      gapNow <= 0.008 &&
      macdWeakening &&
      currentRSI < previousRSI &&
      currentRSI >= 40 &&
      currentRSI <= 60 &&
      volumeOK;

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

  if (!signal) return null;

  return {
    symbol,
    signal,
    type,
    cls,
    currentPrice: Number(currentPrice || lastClose),
    quoteVolume: Number(quoteVolume || 0),
    rsi: currentRSI,
    trendOK
  };
}

function formatPrice(n) {
  if (!Number.isFinite(n)) return "-";
  if (n < 1) return n.toFixed(8).replace(/0+$/, "").replace(/\.$/, "");
  return n.toLocaleString("en-US", {
    maximumFractionDigits: 6
  });
}

function formatVolume(n) {
  if (!Number.isFinite(n)) return "-";
  return n.toLocaleString("en-US", {
    maximumFractionDigits: 0
  }) + " USDT";
}

function renderResults() {
  const rows = allResults.filter(r => {
    if (activeFilter === "ALL") return true;
    return r.type === activeFilter;
  });

  const order = {
    STRONG_AL: 0,
    AL: 1,
    UP: 2,
    STRONG_SAT: 3,
    SAT: 4,
    DOWN: 5
  };

  rows.sort((a, b) =>
    order[a.type] - order[b.type] ||
    a.symbol.localeCompare(b.symbol)
  );

  const tbody = document.getElementById("results");

  if (!rows.length) {
    tbody.innerHTML =
      '<tr><td colspan="6">Bu filtrede sinyal bulunamadı.</td></tr>';
    return;
  }

  tbody.innerHTML = rows.map(r => `
    <tr>
      <td><b>${r.symbol}</b></td>
      <td class="${r.cls}">${r.signal}</td>
      <td>${formatPrice(r.currentPrice)}</td>
      <td>${formatVolume(r.quoteVolume)}</td>
      <td>${r.rsi.toFixed(1)}</td>
      <td class="${r.trendOK ? 'buy' : 'sell'}">
        ${r.trendOK ? 'ÜZERİNDE' : 'ALTINDA'}
      </td>
    </tr>
  `).join("");
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
    '<tr><td colspan="6">Pariteler taranıyor, lütfen bekle...</td></tr>';

  try {
    status.textContent = "Binance Spot pariteleri alınıyor...";

    const [exchange, tickers] = await Promise.all([
      getJSON(API + "/api/v3/exchangeInfo"),
      getJSON(API + "/api/v3/ticker/24hr")
    ]);

    const tickerMap = {};
    tickers.forEach(t => {
      tickerMap[t.symbol] = {
        price: Number(t.lastPrice),
        volume: Number(t.quoteVolume)
      };
    });

    const symbols = exchange.symbols
      .filter(s =>
        s.status === "TRADING" &&
        s.quoteAsset === "USDT" &&
        s.isSpotTradingAllowed !== false &&
        tickerMap[s.symbol]
      )
      .map(s => s.symbol);

    status.textContent =
      symbols.length + " Spot USDT paritesi bulundu. Tarama başladı.";

    let completed = 0;
    let cursor = 0;
    const workerCount = 3;

    async function worker() {
      while (true) {
        const index = cursor++;
        if (index >= symbols.length) break;

        const symbol = symbols[index];

        try {
          const [k1, k4] = await Promise.all([
            getJSON(API + "/api/v3/klines?symbol=" +
              symbol + "&interval=1h&limit=500"),
            getJSON(API + "/api/v3/klines?symbol=" +
              symbol + "&interval=4h&limit=500")
          ]);

          const ticker = tickerMap[symbol];

          const result = analyzePair(
            symbol,
            k1,
            k4,
            ticker.price,
            ticker.volume
          );

          if (result) allResults.push(result);

        } catch (e) {
          // Bir paritedeki hata diğer taramaları durdurmaz.
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

    await Promise.all(
      Array.from({length: workerCount}, () => worker())
    );

    document.getElementById("upCount").textContent =
      allResults.filter(r => r.type === "UP").length;

    document.getElementById("downCount").textContent =
      allResults.filter(r => r.type === "DOWN").length;

    document.getElementById("signalCount").textContent =
      allResults.filter(r =>
        ["AL", "SAT", "STRONG_AL", "STRONG_SAT"].includes(r.type)
      ).length;

    renderResults();

    status.textContent =
      "Tarama tamamlandı: " + symbols.length +
      " parite kontrol edildi. " +
      new Date().toLocaleString("tr-TR");

  } catch (e) {
    status.textContent =
      "Tarama başarısız. İnternet bağlantısını kontrol edip tekrar dene.";

    document.getElementById("results").innerHTML =
      '<tr><td colspan="6">Binance verilerine erişilemedi.</td></tr>';

  } finally {
    isScanning = false;
    button.disabled = false;
    button.textContent = "TEKRAR TARA";
  }
}
</script>
</body>
</html>
