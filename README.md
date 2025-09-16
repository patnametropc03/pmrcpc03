<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>L&T Construction – Patna Metro PC-03 Project</title>
<style>
:root{
  --primary:#004c91; /* L&T blue */
  --secondary:#f9b000; /* Metro yellow */
  --bg:#f2f4f7;
  --card:#ffffff;
  --radius:12px;
  font-family:Segoe UI,Roboto,Arial,sans-serif;
}
body{margin:0;background:var(--bg);color:#1e293b;}
header{background:var(--primary);color:white;padding:20px;text-align:center;}
header h1{margin:0;font-size:28px;}
header h2{margin:6px 0 0;font-weight:400;font-size:16px;color:#dbeafe;}
.container{max-width:1100px;margin:20px auto;padding:20px;}
.card{background:var(--card);padding:20px;border-radius:var(--radius);box-shadow:0 4px 12px rgba(0,0,0,0.05);}
label{display:block;margin-top:10px;font-weight:600;}
select,input,textarea{width:100%;padding:10px;margin-top:5px;border-radius:8px;border:1px solid #d1d5db;font-size:14px;box-sizing:border-box;}
button{background:var(--secondary);border:none;color:black;padding:10px 16px;margin-top:15px;border-radius:8px;font-weight:bold;cursor:pointer;}
button:hover{background:#facc15;}
.data-list{margin-top:20px;}
.data-item{background:#f8fafc;padding:12px;border-left:5px solid var(--primary);margin-bottom:10px;border-radius:6px;}
.controls{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:10px;}
.controls select,.controls button{flex:1;}
.chart-container{margin-top:20px;background:white;border-radius:12px;padding:20px;}
footer{text-align:center;margin:30px 0;color:#6b7280;font-size:13px;}
.action-btn{background:#e5e7eb;color:#111827;padding:4px 8px;border-radius:6px;font-size:12px;margin-left:6px;cursor:pointer;}
.action-btn:hover{background:#d1d5db;}
</style>
</head>
<body>
<header>
  <h1>L&T Construction – Patna Metro PC-03 Project</h1>
  <h2>Project Director: Aloke Kumar Dey</h2>
</header>

<div class="container">
  <div class="card">
    <h3>Department Wise Entry</h3>
    <form id="entryForm">
      <input type="hidden" id="editId">
      <label for="department">Select Department</label>
      <select id="department" required>
        <option value="">--Choose Department--</option>
        <option>Civil</option>
        <option>P&amp;M</option>
        <option>Planning</option>
        <option>Contracts</option>
        <option>Design</option>
        <option>IR</option>
        <option>SHE</option>
        <option>EHS</option>
        <option>ISD</option>
        <option>QAQC</option>
        <option>Store</option>
        <option>Account</option>
        <option>Procurements</option>
        <option>Others</option>
      </select>

      <label for="workdone">Work Done</label>
      <textarea id="workdone" rows="3" required></textarea>

      <label for="workpending">Work Pending</label>
      <textarea id="workpending" rows="3"></textarea>

      <label for="notes">Notes</label>
      <textarea id="notes" rows="3"></textarea>

      <button type="submit" id="submitBtn">Submit Entry</button>
    </form>
  </div>

  <div class="card">
    <h3>Saved Entries</h3>
    <div class="controls">
      <select id="filterDept">
        <option value="">All Departments</option>
        <option>Civil</option><option>P&amp;M</option><option>Planning</option>
        <option>Contracts</option><option>Design</option><option>IR</option><option>SHE</option>
        <option>EHS</option><option>ISD</option><option>QAQC</option><option>Store</option>
        <option>Account</option><option>Procurements</option><option>Others</option>
      </select>
      <button onclick="downloadCSV()">Download CSV</button>
      <button onclick="window.print()">Print / Save PDF</button>
    </div>
    <div id="entriesContainer"></div>
  </div>

  <div class="chart-container">
    <h3>Work Done vs Pending (Entries Count per Department)</h3>
    <canvas id="myChart" height="200"></canvas>
  </div>
</div>

<footer>
  © <span id="year"></span> L&T Construction – Patna Metro PC-03 Project
</footer>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
// Dynamic year
document.getElementById('year').innerText=new Date().getFullYear();

// Load existing entries
let entries=JSON.parse(localStorage.getItem('lt_entries')||'[]');
renderEntries();
renderChart();

document.getElementById('entryForm').addEventListener('submit',function(e){
  e.preventDefault();
  const dept=document.getElementById('department').value;
  const done=document.getElementById('workdone').value;
  const pending=document.getElementById('workpending').value;
  const notes=document.getElementById('notes').value;
  const editId=document.getElementById('editId').value;

  if(editId){ // Editing existing
    const idx=entries.findIndex(x=>x.id==editId);
    if(idx>-1){
      entries[idx].department=dept;
      entries[idx].workdone=done;
      entries[idx].workpending=pending;
      entries[idx].notes=notes;
      entries[idx].date=new Date().toLocaleString();
    }
  } else {
    // New entry
    const entry={id:Date.now(),department:dept,workdone:done,workpending:pending,notes:notes,date:new Date().toLocaleString()};
    entries.push(entry);
  }

  localStorage.setItem('lt_entries',JSON.stringify(entries));
  document.getElementById('entryForm').reset();
  document.getElementById('editId').value='';
  document.getElementById('submitBtn').innerText='Submit Entry';

  renderEntries();
  renderChart();
});

document.getElementById('filterDept').addEventListener('change',renderEntries);

function renderEntries(){
  const container=document.getElementById('entriesContainer');
  container.innerHTML='';
  const filter=document.getElementById('filterDept').value;
  let list=entries.slice().reverse();
  if(filter) list=list.filter(e=>e.department===filter);

  if(!list.length){
    container.innerHTML='<p style="color:#6b7280">No entries yet.</p>';
    return;
  }

  list.forEach(ent=>{
    const div=document.createElement('div');
    div.className='data-item';
    div.innerHTML=`
      <strong>${ent.department}</strong> <small style="color:#6b7280">(${ent.date})</small><br>
      <b>Work Done:</b> ${ent.workdone}<br>
      <b>Work Pending:</b> ${ent.workpending||'-'}<br>
      <b>Notes:</b> ${ent.notes||'-'}
      <div style="margin-top:6px;">
        <span class="action-btn" onclick="editEntry(${ent.id})">Edit</span>
        <span class="action-btn" onclick="deleteEntry(${ent.id})">Delete</span>
      </div>
    `;
    container.appendChild(div);
  });
}

function editEntry(id){
  const ent=entries.find(x=>x.id==id);
  if(ent){
    document.getElementById('department').value=ent.department;
    document.getElementById('workdone').value=ent.workdone;
    document.getElementById('workpending').value=ent.workpending;
    document.getElementById('notes').value=ent.notes;
    document.getElementById('editId').value=ent.id;
    document.getElementById('submitBtn').innerText='Update Entry';
    window.scrollTo({top:0,behavior:'smooth'});
  }
}

function deleteEntry(id){
  if(confirm('Delete this entry?')){
    entries=entries.filter(x=>x.id!=id);
    localStorage.setItem('lt_entries',JSON.stringify(entries));
    renderEntries();
    renderChart();
  }
}

// Download CSV
function downloadCSV(){
  if(!entries.length){alert('No data');return;}
  let csv='Department,Work Done,Work Pending,Notes,Date\n';
  entries.forEach(e=>{
    csv+=`"${e.department}","${e.workdone}","${e.workpending}","${e.notes}","${e.date}"\n`;
  });
  const blob=new Blob([csv],{type:'text/csv'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');
  a.href=url;
  a.download='entries.csv';
  a.click();
  URL.revokeObjectURL(url);
}

// Chart
let chart;
function renderChart(){
  const countsDone={},countsPending={};
  entries.forEach(e=>{
    countsDone[e.department]=(countsDone[e.department]||0)+1;
    if(e.workpending && e.workpending.trim()!=''){
      countsPending[e.department]=(countsPending[e.department]||0)+1;
    } else {
      countsPending[e.department]=(countsPending[e.department]||0)+0;
    }
  });
  const depts=[...new Set(entries.map(e=>e.department))];
  const doneData=depts.map(d=>countsDone[d]||0);
  const pendingData=depts.map(d=>countsPending[d]||0);

  if(chart)chart.destroy();
  const ctx=document.getElementById('myChart').getContext('2d');
  chart=new Chart(ctx,{
    type:'bar',
    data:{
      labels:depts,
      datasets:[
        {label:'Entries (Work Done)',data:doneData,backgroundColor:'#004c91'},
        {label:'Entries with Work Pending',data:pendingData,backgroundColor:'#f9b000'}
      ]
    },
    options:{responsive:true,plugins:{legend:{position:'bottom'}}}
  });
}
</script>
</body>
</html>
