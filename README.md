
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>PHANTOM ULTRA</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#080b14;color:#edf2ff;font-family:Arial,sans-serif;padding:16px}
.wrap{max-width:650px;margin:auto}
header{text-align:center;padding:20px 8px}
h1{margin:0;color:#8b5cf6;letter-spacing:4px;font-size:32px}
.sub{color:#9ba8c7;margin-top:8px}
.card{background:#111827;border:1px solid #29334b;border-radius:14px;padding:16px;margin:14px 0}
h2{font-size:17px;margin:0 0 12px;color:#c4b5fd}
textarea,input,select,button{width:100%;padding:12px;border-radius:9px;border:1px solid #374151;background:#080d19;color:white;font-size:15px;margin:5px 0}
textarea{min-height:155px;resize:vertical}
button{background:#6d28d9;border:0;font-weight:bold;cursor:pointer;padding:14px}
.secondary{background:#273449}
.grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:10px}
.stat{background:#080d19;padding:14px;border-radius:10px;border:1px solid #29334b}
.label{font-size:12px;color:#9ba8c7}
.value{font-size:23px;font-weight:bold;margin-top:7px;overflow-wrap:anywhere}
.small{font-size:12px;color:#9ba8c7;line-height:1.5}
table{width:100%;border-collapse:collapse;font-size:13px}
td,th{text-align:left;padding:9px 5px;border-bottom:1px solid #29334b}
.green{color:#4ade80}.red{color:#fb7185}.violet{color:#c4b5fd}
#status{color:#fbbf24;font-size:13px;line-height:1.5}
</style>
</head>
<body>
<div class="wrap">
<header>
<h1>PHANTOM ULTRA</h1>
<div class="sub">NUMBER & COLOUR HISTORY ANALYZER</div>
</header>

<div class="card">
<h2>01 / Enter Actual Results</h2>
<p class="small">Enter one result per line: Period,Number,Colour. Example: 1005,7,Green</p>
<textarea id="history" placeholder="Period,Number,Colour&#10;1005,7,Green&#10;1004,2,Red&#10;1003,9,Green"></textarea>
<select id="order">
<option value="new">Newest result first</option>
<option value="old">Oldest result first</option>
</select>
<button onclick="analyse()">ANALYSE HISTORY</button>
<button class="secondary" onclick="sample()">LOAD EXAMPLE DATA</button>
<button class="secondary" onclick="clearAll()">CLEAR</button>
<div id="status"></div>
</div>

<div class="card">
<h2>02 / History Summary</h2>
<div class="grid">
<div class="stat"><div class="label">TOTAL RESULTS</div><div class="value" id="total">0</div></div>
<div class="stat"><div class="label">MOST FREQUENT NUMBER</div><div class="value" id="topnum">—</div></div>
<div class="stat"><div class="label">MOST FREQUENT COLOUR</div><div class="value" id="topcolour">—</div></div>
<div class="stat"><div class="label">LATEST NUMBER</div><div class="value" id="latest">—</div></div>
</div>
</div>

<div class="card">
<h2>03 / Number Frequency (0–9)</h2>
<table><thead><tr><th>Number</th><th>Count</th><th>Frequency</th></tr></thead><tbody id="numbers"></tbody></table>
</div>

<div class="card">
<h2>04 / Colour Frequency</h2>
<table><thead><tr><th>Colour</th><th>Count</th><th>Frequency</th></tr></thead><tbody id="colours"></tbody></table>
</div>

<div class="card">
<h2>05 / Recent Recorded Results</h2>
<table><thead><tr><th>Period</th><th>Number</th><th>Colour</th></tr></thead><tbody id="recent"></tbody></table>
</div>

<div class="card">
<h2>06 / Statistical Trend</h2>
<p id="trend">Enter actual results above to calculate historical trends.</p>
<p class="small">Historical frequency describes past data only. It does not predict the next result or guarantee profit. This page is not connected to YaarWin and does not fetch live results.</p>
</div>
</div>

<script>
let records=[];
function colourName(s){
 s=String(s||"").trim().toLowerCase();
 if(s==="g"||s==="green")return "Green";
 if(s==="r"||s==="red")return "Red";
 if(s==="v"||s==="violet"||s==="purple")return "Violet";
 return "";
}
function analyse(){
 const lines=document.getElementById("history").value.trim().split(/\n+/);
 let arr=[],seen=new Set(),bad=0;
 for(let line of lines){
  line=line.trim();
  if(!line||/^period\s*[,;\t]/i.test(line))continue;
  let p=line.split(/[,;\t]+/).map(x=>x.trim());
  if(p.length<3){p=line.split(/\s+/);}
  if(p.length<3){bad++;continue;}
  let period=p[0],num=Number(p[1]),col=colourName(p[2]);
  if(!period||!Number.isInteger(num)||num<0||num>9||!col){bad++;continue;}
  if(seen.has(period))continue;
  seen.add(period);arr.push({period,num,col});
 }
 if(document.getElementById("order").value==="old")arr.reverse();
 records=arr;
 document.getElementById("total").textContent=arr.length;
 const nc=Array(10).fill(0),cc={Green:0,Red:0,Violet:0};
 arr.forEach(r=>{nc[r.num]++;cc[r.col]++;});
 const bestN=Math.max(...nc);
 document.getElementById("topnum").textContent=arr.length?nc.indexOf(bestN)+" ("+bestN+")":"—";
 let bestC=Object.keys(cc).sort((a,b)=>cc[b]-cc[a])[0];
 document.getElementById("topcolour").textContent=arr.length&&cc[bestC]>0?bestC+" ("+cc[bestC]+")":"—";
 document.getElementById("latest").textContent=arr.length?arr[0].num:"—";
 document.getElementById("numbers").innerHTML=nc.map((n,i)=>`<tr><td>${i}</td><td>${n}</td><td>${arr.length?(n*100/arr.length).toFixed(1):"0.0"}%</td></tr>`).join("");
 document.getElementById("colours").innerHTML=Object.entries(cc).map(([c,n])=>`<tr><td class="${c.toLowerCase()}">${c}</td><td>${n}</td><td>${arr.length?(n*100/arr.length).toFixed(1):"0.0"}%</td></tr>`).join("");
 document.getElementById("recent").innerHTML=arr.slice(0,15).map(r=>`<tr><td>${escapeHtml(r.period)}</td><td>${r.num}</td><td class="${r.col.toLowerCase()}">${r.col}</td></tr>`).join("");
 document.getElementById("trend").textContent=arr.length?`Across ${arr.length} recorded results, number ${nc.indexOf(bestN)} appeared most often (${bestN} times). ${cc[bestC]} was the most frequent colour (${cc[bestC]} times). These are historical statistics, not a reliable next-result prediction.`:"Enter valid results to see trends.";
 document.getElementById("status").textContent=`Analysed ${arr.length} valid results.${bad?" Skipped "+bad+" invalid line(s).":""}`;
}
function escapeHtml(s){return String(s).replace(/[&<>"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]));}
function sample(){
 document.getElementById("history").value="Period,Number,Colour\n1005,7,Green\n1004,2,Red\n1003,9,Green\n1002,0,Violet\n1001,5,Red";
 analyse();
 document.getElementById("status").textContent="Example data loaded — replace it with actual results.";
}
function clearAll(){
 document.getElementById("history").value="";
 records=[];
 ["total","topnum","topcolour","latest"].forEach(id=>document.getElementById(id).textContent="0");
 document.getElementById("topnum").textContent="—";
 document.getElementById("topcolour").textContent="—";
 document.getElementById("latest").textContent="—";
 document.getElementById("numbers").innerHTML="";
 document.getElementById("colours").innerHTML="";
 document.getElementById("recent").innerHTML="";
 document.getElementById("trend").textContent="Enter actual results above to calculate historical trends.";
 document.getElementById("status").textContent="Cleared.";
}
</script>
</body>
</html>
