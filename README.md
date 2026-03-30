<!DOCTYPE html>
<html>
<head>
<title>Jesus is King Trading AI</title>

<style>
body{background:#020617;color:white;font-family:Arial;text-align:center;}
h2{color:#38bdf8;}
.card{background:#0f172a;border:1px solid #1e293b;padding:12px;margin:10px auto;width:95%;max-width:1000px;border-radius:12px;}
button{padding:10px;margin:5px;border:none;border-radius:6px;font-weight:bold;cursor:pointer;}
.buy{background:#22c55e;}
.sell{background:#ef4444;}
.exec{background:#f59e0b;}
.win{background:#3b82f6;}
.loss{background:#ef4444;}
.reset{background:#64748b;}
.print{background:#06b6d4;}
select,input{padding:6px;margin:5px;}
table{width:100%;border-collapse:collapse;font-size:12px;}
th,td{border:1px solid #1e293b;padding:6px;}
.green{color:#22c55e;}
.red{color:#ef4444;}
</style>
</head>

<body>

<h2>Jesus is King Trading AI</h2>

<div class="card">
<h3>Balance: $<span id="balance">50</span></h3>

Daily Goal: $
<input id="goalInput" value="62" onchange="setGoal()">

<p>Progress: <span id="progress">0</span>%</p>
<p id="goalStatus">Not reached</p>

<p>Session Time (Kampala): <span id="time"></span></p>
</div>

<!-- PERFORMANCE DASHBOARD -->
<div class="card">
<h3>Performance Dashboard</h3>

<p>Total Profit: $<span id="totalProfit">0</span></p>
<p>Win Rate: <span id="winRate">0</span>%</p>
<p>Total Trades: <span id="totalTrades">0</span></p>
<p>Wins: <span id="totalWins">0</span> | Losses: <span id="totalLosses">0</span></p>
<p>Best Pair: <span id="bestPair">---</span></p>
<p>Session Profit: $<span id="sessionProfit">0</span></p>

</div>

<!-- TRADE SETUP -->
<div class="card">
<h3>Quick Trade Setup</h3>

<select id="pairInput">
<option value="">Select Pair</option>

<option>EUR/USD</option>
<option>GBP/USD</option>
<option>USD/JPY</option>
<option>USD/CHF</option>
<option>AUD/USD</option>
<option>USD/CAD</option>
<option>EUR/GBP</option>

<option>EUR/JPY</option>
<option>GBP/JPY</option>
<option>AUD/CAD</option>
<option>AUD/CHF</option>
<option>CAD/JPY</option>
<option>CHF/JPY</option>
<option>EUR/AUD</option>
<option>EUR/CAD</option>
<option>EUR/CHF</option>
<option>GBP/AUD</option>
<option>GBP/CAD</option>
<option>GBP/CHF</option>
<option>CAD/CHF</option>
<option>AUD/JPY</option>

<option>EUR/USD OTC</option>
<option>GBP/USD OTC</option>
<option>USD/JPY OTC</option>
<option>USD/CHF OTC</option>
<option>AUD/USD OTC</option>
<option>USD/CAD OTC</option>
<option>EUR/GBP OTC</option>

<option>AUD/CAD OTC</option>
<option>AUD/CHF OTC</option>
<option>AUD/NZD OTC</option>
<option>CAD/CHF OTC</option>
<option>CAD/JPY OTC</option>
<option>CHF/JPY OTC</option>
<option>EUR/JPY OTC</option>
<option>EUR/NZD OTC</option>
<option>LBP/USD OTC</option>
<option>GBP/JPY OTC</option>

</select>

<br>

Payout %:
<input id="payoutInput" type="number" placeholder="e.g 82">

<br>

<button class="buy" onclick="setSignal('BUY')">BUY</button>
<button class="sell" onclick="setSignal('SELL')">SELL</button>

<p id="quickStatus">No signal selected</p>
</div>

<div class="card">
<p>Pair: <span id="pair">---</span></p>
<p>Signal: <span id="signal">---</span></p>
<p>Confidence: <span id="score">--</span>%</p>
<p>Payout: <span id="payout">--</span>%</p>
<p>Next Stake: $<span id="stake">0</span></p>

<p id="status">READY</p>
<p id="lock"></p>
</div>

<div class="card">
<button class="exec" id="executeBtn" onclick="executeTrade()" disabled>EXECUTE</button>
<button class="win" onclick="setResult('win')">WIN</button>
<button class="loss" onclick="setResult('loss')">LOSS</button>
<br>
<button class="reset" onclick="resetSession()">RESET SESSION</button>
<button class="print" onclick="window.print()">EXPORT / PRINT</button>
</div>

<div class="card">
<p>Trades: <span id="trades">0</span>/10</p>
<p>Wins: <span id="wins">0</span></p>
<p>Losses: <span id="losses">0</span></p>
<p>Loss Streak: <span id="streak">0</span></p>
</div>

<div class="card">
<h3>Trade History</h3>
<table>
<thead>
<tr>
<th>Date</th><th>Pair</th><th>Signal</th>
<th>Payout %</th><th>Result</th><th>Start $</th>
<th>End $</th><th>Profit $</th><th>Risk %</th><th>Next Stake</th>
</tr>
</thead>
<tbody id="history"></tbody>
</table>
</div>

<script>

// EXISTING VARIABLES...
let balance=50, baseBalance=50, goal=62;
let totalProfit=0, sessionProfit=0;
let pairStats={};

function updatePerformance(pair, profit){

totalProfit+=profit;
sessionProfit+=profit;

let totalTrades=trades;
let winRate = totalTrades>0 ? (wins/totalTrades)*100 : 0;

// pair stats
if(!pairStats[pair]){
pairStats[pair]={profit:0,trades:0};
}
pairStats[pair].profit+=profit;
pairStats[pair].trades++;

// best pair
let best="---", maxProfit=-99999;
for(let p in pairStats){
if(pairStats[p].profit>maxProfit){
maxProfit=pairStats[p].profit;
best=p;
}
}

// UI
document.getElementById("totalProfit").innerText=totalProfit.toFixed(2);
document.getElementById("sessionProfit").innerText=sessionProfit.toFixed(2);
document.getElementById("winRate").innerText=winRate.toFixed(1);
document.getElementById("totalTrades").innerText=totalTrades;
document.getElementById("totalWins").innerText=wins;
document.getElementById("totalLosses").innerText=losses;
document.getElementById("bestPair").innerText=best;
}

</script>
