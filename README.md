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
.green{color:#22c55e;}
.red{color:#ef4444;}
table{width:100%;border-collapse:collapse;font-size:12px;}
th,td{border:1px solid #1e293b;padding:6px;}
select,input{padding:6px;margin:5px;}
</style>
</head>

<body>

<h2>Jesus is King Trading AI</h2>

<div class="card">
Balance: $<span id="balance">50</span><br>
Trades: <span id="trades">0</span>/10 |
Wins: <span id="wins">0</span> |
Losses: <span id="losses">0</span> |
Streak: <span id="streak">0</span>
</div>

<div class="card">
<select id="pair">
<option>EUR/USD</option>
<option>GBP/USD</option>
<option>USD/JPY</option>
<option>EUR/USD OTC</option>
<option>GBP/JPY OTC</option>
</select>

<input id="payout" placeholder="Payout %" type="number">

<br>

<button class="buy" onclick="setSignal('BUY')">BUY</button>
<button class="sell" onclick="setSignal('SELL')">SELL</button>
</div>

<div class="card">
Signal: <span id="signal">---</span><br>
Next Stake: $<span id="stake">0</span>
</div>

<div class="card">
<button class="exec" onclick="executeTrade()">EXECUTE</button>
<button class="win" onclick="setResult('win')">WIN</button>
<button class="loss" onclick="setResult('loss')">LOSS</button>
<br>
<button class="reset" onclick="resetSession()">RESET</button>
<button class="print" onclick="window.print()">PRINT</button>
</div>

<div class="card">
<h3>Session Summary</h3>
<table>
<thead>
<tr>
<th>Date</th>
<th>Session</th>
<th>Profit</th>
<th>Best Pairs</th>
</tr>
</thead>
<tbody id="sessionTable"></tbody>
</table>
</div>

<div class="card">
<h3>Trade History</h3>
<table>
<thead>
<tr>
<th>Date</th><th>Pair</th><th>Result</th><th>Profit</th>
</tr>
</thead>
<tbody id="history"></tbody>
</table>
</div>

<script>

// CORE
let balance=50;
let trades=0,wins=0,losses=0,streak=0;

let risk=2;
let recovery=false;
let recoveryAmount=0;

let sessionProfit=0;
let sessionCount=1;
let sessionPairs={};

let currentSignal=null;
let executed=false;

const MIN=1;

// SIGNAL
function setSignal(type){
currentSignal=type;
document.getElementById("signal").innerText=type;
}

// EXECUTE
function executeTrade(){
if(!currentSignal){alert("Select signal");return;}
executed=true;
}

// RESULT
function setResult(res){

if(!executed){alert("Execute first");return;}

let stake=Math.max(balance*(risk/100),MIN);
let payout=parseFloat(document.getElementById("payout").value||80);

let profit = res==="win" ? stake*(payout/100) : -stake;

balance+=profit;
sessionProfit+=profit;
trades++;

if(res==="win"){wins++;streak=0;}
else{losses++;streak++;}

// recovery logic
if(res==="loss"){
recovery=true;
recoveryAmount+=stake;
risk=1;
}

if(recovery && res==="win"){
recoveryAmount-=stake;
if(recoveryAmount<=0){
recovery=false;
risk=2;
recoveryAmount=0;
}
}

// track pair
let pair=document.getElementById("pair").value;
if(!sessionPairs[pair]) sessionPairs[pair]=0;
sessionPairs[pair]+=profit;

// UI update
document.getElementById("balance").innerText=balance.toFixed(2);
document.getElementById("trades").innerText=trades;
document.getElementById("wins").innerText=wins;
document.getElementById("losses").innerText=losses;
document.getElementById("streak").innerText=streak;

addHistory(pair,res,profit);

// SESSION END
if(trades>=10 || streak>=3){
saveSession();
resetSession();
}

executed=false;
}

// SAVE SESSION (GUARANTEED)
function saveSession(){

let table=document.getElementById("sessionTable");

let now=new Date();
let date=now.getDate()+"/"+(now.getMonth()+1)+"/"+now.getFullYear();

let bestPairs=Object.keys(sessionPairs).join(", ") || "-";

let row=document.createElement("tr");

row.innerHTML=
"<td>"+date+"</td>"+
"<td>"+sessionCount+"</td>"+
"<td>"+sessionProfit.toFixed(2)+"</td>"+
"<td>"+bestPairs+"</td>";

table.appendChild(row);

sessionCount++;

// CONFIRM
console.log("Session saved");
}

// RESET
function resetSession(){
trades=0;wins=0;losses=0;streak=0;
sessionProfit=0;
sessionPairs={};
}

// HISTORY
function addHistory(pair,res,profit){

let now=new Date();
let date=now.getDate()+"/"+(now.getMonth()+1)+"/"+now.getFullYear();

let row=
"<tr>"+
"<td>"+date+"</td>"+
"<td>"+pair+"</td>"+
"<td>"+res+"</td>"+
"<td>"+profit.toFixed(2)+"</td>"+
"</tr>";

document.getElementById("history").innerHTML+=row;
}

</script>

</body>
</html>
