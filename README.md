<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إدارة القضايا | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body {margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8;padding:20px;}
header {background:#0a2a43;color:#fff;padding:20px;text-align:center;border-radius:12px;margin-bottom:20px;box-shadow:0 5px 15px rgba(0,0,0,.1);}
header h1 {margin:0;}
.container {max-width:1200px;margin:auto;}
.card {background:#fff;padding:25px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:25px;}
.card h2 {margin-top:0;color:#0a2a43;}
input, select, button, textarea {width:100%;padding:12px;margin-bottom:12px;font-family:'Cairo';border-radius:8px;border:1px solid #ccc;}
textarea {resize: vertical;}
button {background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer;font-size:16px;transition:.3s;}
button:hover {opacity:.9;}
table {width:100%;border-collapse:collapse;margin-top:15px;}
th, td {padding:12px;border-bottom:1px solid #ddd;text-align:center;}
th {background:#0a2a43;color:#fff;}
.actions button {margin:0 5px;padding:6px 12px;font-size:14px;}
.filter-group {display:flex;gap:15px;flex-wrap:wrap;margin-bottom:15px;}
.filter-group select, .filter-group input {flex:1;min-width:180px;padding:10px;border-radius:6px;border:1px solid #ccc;}
</style>
</head>
<body>

<header>
<h1>إدارة القضايا | Cabinet Juridique</h1>
</header>

<div class="container">

<!-- نموذج إضافة / تعديل قضية -->
<div class="card">
<h2>إضافة / تعديل قضية</h2>
<input type="hidden" id="caseIndex">
<input type="text" id="title" placeholder="عنوان القضية">
<input type="text" id="ref" placeholder="رقم / مرجع القضية">
<select id="clientId"><option value="">اختر الموكل</option></select>
<select id="lawyerId"><option value="">اختر المحامي</option></select>
<select id="status">
<option value="">الحالة</option>
<option value="جارية">جارية</option>
<option value="مغلقة">مغلقة</option>
</select>
<input type="text" id="fees" placeholder="أتعاب المحامي للقضية (د.م)">
<button onclick="saveCase()">💾 حفظ القضية</button>
<button onclick="exportCasesPDF()">📄 تصدير PDF القضايا الاحترافي</button>
</div>

<!-- جدول القضايا مع الفلاتر -->
<div class="card">
<h2>قائمة القضايا</h2>
<div class="filter-group">
<input type="text" id="searchCases" placeholder="ابحث عن قضية">
<select id="filterClient"><option value="">الموكل</option></select>
<select id="filterLawyer"><option value="">المحامي</option></select>
<select id="filterStatusTable"><option value="">الحالة</option><option value="جارية">جارية</option><option value="مغلقة">مغلقة</option></select>
</div>
<table>
<thead>
<tr>
<th>#</th>
<th>رقم / مرجع القضية</th>
<th>الموكل</th>
<th>عنوان القضية</th>
<th>المحامي</th>
<th>الحالة</th>
<th>أتعاب المحامي</th>
<th>إجراءات</th>
</tr>
</thead>
<tbody id="casesTable"></tbody>
</table>
</div>

</div>

<!-- jsPDF -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>

<script>
// ===== LocalStorage =====
let clients = JSON.parse(localStorage.getItem('clients')) || [];
let lawyers = JSON.parse(localStorage.getItem('lawyers')) || [];
let cases = JSON.parse(localStorage.getItem('cases')) || [];

// تعبئة قوائم الموكلين والمحامين
function populateSelects(){
    const clientSelect = document.getElementById('clientId');
    const lawyerSelect = document.getElementById('lawyerId');
    const filterClient = document.getElementById('filterClient');
    const filterLawyer = document.getElementById('filterLawyer');

    clientSelect.innerHTML = '<option value="">اختر الموكل</option>';
    filterClient.innerHTML = '<option value="">الموكل</option>';
    lawyerSelect.innerHTML = '<option value="">اختر المحامي</option>';
    filterLawyer.innerHTML = '<option value="">المحامي</option>';

    clients.forEach((c,i)=>{
        let o = document.createElement('option'); o.value = i; o.innerText = c.name;
        clientSelect.appendChild(o);
        filterClient.appendChild(o.cloneNode(true));
    });

    lawyers.forEach((l,i)=>{
        let o = document.createElement('option'); o.value = i; o.innerText = l.name;
        lawyerSelect.appendChild(o);
        filterLawyer.appendChild(o.cloneNode(true));
    });
}

// حفظ / تعديل قضية
function saveCase(){
    const index = document.getElementById('caseIndex').value;
    const newCase = {
        ref: document.getElementById('ref').value,
        clientId: parseInt(document.getElementById('clientId').value),
        title: document.getElementById('title').value,
        lawyerId: parseInt(document.getElementById('lawyerId').value),
        status: document.getElementById('status').value,
        fees: parseFloat(document.getElementById('fees').value) || 0,
        index: index==='' ? cases.length : parseInt(index)
    };
    if(index===''){ cases.push(newCase); } else { cases[index] = newCase; }
    localStorage.setItem('cases', JSON.stringify(cases));
    resetForm();
    renderCases();
}

// تحرير قضية
function editCase(i){
    const c = cases[i];
    document.getElementById('caseIndex').value = i;
    document.getElementById('ref').value = c.ref;
    document.getElementById('clientId').value = c.clientId;
    document.getElementById('title').value = c.title;
    document.getElementById('lawyerId').value = c.lawyerId;
    document.getElementById('status').value = c.status;
    document.getElementById('fees').value = c.fees;
}

// حذف قضية
function deleteCase(i){
    if(confirm('هل أنت متأكد من حذف هذه القضية؟')){
        cases.splice(i,1);
        localStorage.setItem('cases', JSON.stringify(cases));
        renderCases();
    }
}

// إعادة تعيين النموذج
function resetForm(){
    document.getElementById('caseIndex').value='';
    document.getElementById('ref').value='';
    document.getElementById('clientId').value='';
    document.getElementById('title').value='';
    document.getElementById('lawyerId').value='';
    document.getElementById('status').value='';
    document.getElementById('fees').value='';
}

// عرض القضايا مع الفلاتر
function renderCases(){
    const term = document.getElementById('searchCases').value.toLowerCase();
    const clientFilter = document.getElementById('filterClient').value;
    const lawyerFilter = document.getElementById('filterLawyer').value;
    const statusFilter = document.getElementById('filterStatusTable').value;
    const table = document.getElementById('casesTable');
    table.innerHTML='';

    cases.forEach((c,i)=>{
        const clientName = clients[c.clientId]?.name || "غير معروف";
        const lawyerName = lawyers[c.lawyerId]?.name || "غير معروف";

        if((c.title.toLowerCase().includes(term) || clientName.toLowerCase().includes(term) || lawyerName.toLowerCase().includes(term) || c.ref.toLowerCase().includes(term))
           && (clientFilter===''||c.clientId==clientFilter)
           && (lawyerFilter===''||c.lawyerId==lawyerFilter)
           && (statusFilter===''||c.status===statusFilter)){
            table.innerHTML += `<tr>
                <td>${i+1}</td>
                <td>${c.ref}</td>
                <td>${clientName}</td>
                <td>${c.title}</td>
                <td>${lawyerName}</td>
                <td>${c.status}</td>
                <td>${c.fees} د.م</td>
                <td class="actions">
                    <button onclick="editCase(${i})">✏️ تعديل</button>
                    <button onclick="deleteCase(${i})">🗑️ حذف</button>
                </td>
            </tr>`;
        }
    });
}

// أحداث البحث والفلاتر
document.getElementById('searchCases').addEventListener('input',renderCases);
document.getElementById('filterClient').addEventListener('change',renderCases);
document.getElementById('filterLawyer').addEventListener('change',renderCases);
document.getElementById('filterStatusTable').addEventListener('change',renderCases);

// تفعيل عند تحميل الصفحة
populateSelects();
renderCases();

// ==== PDF احترافي للقضايا + أتعاب المحامي ====
async function exportCasesPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF('p','pt','a4');

    // إضافة خط عربي Cairo
    const fontUrl = 'https://raw.githubusercontent.com/google/fonts/main/ofl/cairo/Cairo-Regular.ttf';
    const fontResponse = await fetch(fontUrl);
    const fontBuffer = await fontResponse.arrayBuffer();
    doc.addFileToVFS("Cairo-Regular.ttf", fontBuffer);
    doc.addFont("Cairo-Regular.ttf","Cairo","normal");
    doc.setFont("Cairo");

    let y = 40;
    doc.setFontSize(22);
    doc.text("🏛️ مكتب الاستشارات القانونية", 297.5, y, {align:"center"});
    y += 25;
    doc.setFontSize(16);
    doc.text("تقرير القضايا + أتعاب المحامي", 297.5, y, {align:"center"});
    y += 30;

    // إعداد بيانات القضايا
    let tableData = [["#", "مرجع القضية", "الموكل", "عنوان القضية", "المحامي", "الحالة", "أتعاب المحامي"]];
    cases.forEach((c,i)=>{
        const clientName = clients[c.clientId]?.name || "غير معروف";
        const lawyerName = lawyers[c.lawyerId]?.name || "غير معروف";
        tableData.push([i+1, c.ref, clientName, c.title, lawyerName, c.status, c.fees + " د.م"]);
    });

    doc.autoTable({
        startY:y,
        head: [tableData[0]],
        body: tableData.slice(1),
        theme: 'grid',
        headStyles: { fillColor:[10,42,67], textColor:255 },
        styles: { font: 'Cairo', fontSize:12, cellPadding:4, halign:'center' },
        margin: { left: 40, right: 40 },
        didDrawCell: function(data) {
            if(data.column.dataKey === 3 && data.row.index >=0){
                const caseRef = tableData[data.row.index+1][1];
                const clientId = cases[data.row.index].clientId;
                const url = `client-profile.html?index=${clientId}&case=${caseRef}`;
                doc.link(data.cell.x, data.cell.y, data.cell.width, data.cell.height, { url });
            }
        }
    });

    y = doc.lastAutoTable.finalY + 20;

    // ملخص إحصائي
    const totalCases = tableData.length - 1;
    const totalFees = cases.reduce((sum,c)=>sum + (c.fees||0),0);
    doc.setFontSize(16);
    doc.text("ملخص إحصائي:", 40, y); y += 20;
    doc.setFontSize(14);
    doc.text(`عدد القضايا: ${totalCases}`, 40, y); y += 18;
    doc.text(`إجمالي أتعاب المحامي: ${totalFees} د.م`, 40, y); y += 18;

    doc.save("تقرير_القضايا_الاحترافي.pdf");
}
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>ملف الموكل | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body {margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8;padding:20px;}
header {background:#0a2a43;color:#fff;padding:20px;text-align:center;border-radius:12px;margin-bottom:20px;box-shadow:0 5px 15px rgba(0,0,0,.1);}
header h1 {margin:0;}
.container {max-width:1200px;margin:auto;}
.card {background:#fff;padding:25px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:25px;}
.card h2 {margin-top:0;color:#0a2a43;}
table {width:100%;border-collapse:collapse;margin-top:15px;}
th, td {padding:12px;border-bottom:1px solid #ddd;text-align:center;}
th {background:#0a2a43;color:#fff;}
.actions button {margin:0 5px;padding:6px 12px;font-size:14px;}
button {background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer;font-size:16px;transition:.3s;margin-top:10px;}
button:hover {opacity:.9;}
</style>
</head>
<body>

<header>
<h1>ملف الموكل | Cabinet Juridique</h1>
</header>

<div class="container">

<div class="card">
<h2>بيانات الموكل</h2>
<p><strong>الاسم:</strong> <span id="clientName"></span></p>
<p><strong>البريد الإلكتروني:</strong> <span id="clientEmail"></span></p>
<p><strong>الهاتف:</strong> <span id="clientPhone"></span></p>
<button onclick="exportClientPDF()">📄 تصدير PDF موكل</button>
</div>

<div class="card">
<h2>قضايا الموكل</h2>
<table>
<thead>
<tr>
<th>#</th>
<th>رقم / مرجع القضية</th>
<th>عنوان القضية</th>
<th>المحامي</th>
<th>الحالة</th>
<th>أتعاب المحامي</th>
</tr>
</thead>
<tbody id="clientCasesTable"></tbody>
</table>
</div>

</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>

<script>
// ===== بيانات LocalStorage =====
let clients = JSON.parse(localStorage.getItem('clients')) || [];
let lawyers = JSON.parse(localStorage.getItem('lawyers')) || [];
let cases = JSON.parse(localStorage.getItem('cases')) || [];

// ===== استدعاء موكل معين من الرابط =====
const urlParams = new URLSearchParams(window.location.search);
const clientIndex = parseInt(urlParams.get('index')) || 0;
const client = clients[clientIndex] || {name:'غير معروف', email:'', phone:''};

document.getElementById('clientName').innerText = client.name;
document.getElementById('clientEmail').innerText = client.email;
document.getElementById('clientPhone').innerText = client.phone;

// عرض القضايا الخاصة بالموكل
function renderClientCases(){
    const table = document.getElementById('clientCasesTable');
    table.innerHTML='';
    let clientCases = cases.filter(c=>c.clientId===clientIndex);
    clientCases.forEach((c,i)=>{
        const lawyerName = lawyers[c.lawyerId]?.name || "غير معروف";
        table.innerHTML += `<tr>
            <td>${i+1}</td>
            <td>${c.ref}</td>
            <td>${c.title}</td>
            <td>${lawyerName}</td>
            <td>${c.status}</td>
            <td>${c.fees} د.م</td>
        </tr>`;
    });
}
renderClientCases();

// تصدير PDF لكل موكل
async function exportClientPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF('p','pt','a4');

    // إضافة خط عربي Cairo
    const fontUrl = 'https://raw.githubusercontent.com/google/fonts/main/ofl/cairo/Cairo-Regular.ttf';
    const fontResponse = await fetch(fontUrl);
    const fontBuffer = await fontResponse.arrayBuffer();
    doc.addFileToVFS("Cairo-Regular.ttf", fontBuffer);
    doc.addFont("Cairo-Regular.ttf","Cairo","normal");
    doc.setFont("Cairo");

    let y = 40;
    doc.setFontSize(22);
    doc.text("🏛️ مكتب الاستشارات القانونية", 297.5, y, {align:"center"});
    y += 25;
    doc.setFontSize(16);
    doc.text(`ملف الموكل: ${client.name}`, 297.5, y, {align:"center"});
    y += 30;

    // بيانات الموكل
    doc.setFontSize(14);
    doc.text(`البريد الإلكتروني: ${client.email}`, 40, y); y+=18;
    doc.text(`الهاتف: ${client.phone}`, 40, y); y+=25;

    // إعداد بيانات القضايا
    let tableData = [["#", "مرجع القضية", "عنوان القضية", "المحامي", "الحالة", "أتعاب المحامي"]];
    let clientCases = cases.filter(c=>c.clientId===clientIndex);
    clientCases.forEach((c,i)=>{
        const lawyerName = lawyers[c.lawyerId]?.name || "غير معروف";
        tableData.push([i+1, c.ref, c.title, lawyerName, c.status, c.fees + " د.م"]);
    });

    doc.autoTable({
        startY:y,
        head: [tableData[0]],
        body: tableData.slice(1),
        theme: 'grid',
        headStyles: { fillColor:[10,42,67], textColor:255 },
        styles: { font: 'Cairo', fontSize:12, cellPadding:4, halign:'center' },
        margin: { left: 40, right: 40 }
    });

    y = doc.lastAutoTable.finalY + 20;

    // ملخص إحصائي
    const totalCases = tableData.length - 1;
    const totalFees = clientCases.reduce((sum,c)=>sum+(c.fees||0),0);
    doc.setFontSize(16);
    doc.text("ملخص إحصائي:", 40, y); y+=20;
    doc.setFontSize(14);
    doc.text(`عدد القضايا: ${totalCases}`, 40, y); y+=18;
    doc.text(`إجمالي أتعاب المحامي: ${totalFees} د.م`, 40, y); y+=18;

    doc.save(`ملف_${client.name}_القضايا.pdf`);
}
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إدارة الموكلين | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body {
    margin:0;
    font-family:'Cairo',sans-serif;
    background:#f4f6f8;
    padding:20px;
}
header {
    background:#0a2a43;
    color:#fff;
    padding:20px;
    text-align:center;
    border-radius:12px;
    box-shadow:0 5px 15px rgba(0,0,0,.1);
    margin-bottom:20px;
}
header h1 {margin:0;}
.container {
    max-width:1000px;
    margin:auto;
}
.card {
    background:#fff;
    padding:25px;
    border-radius:12px;
    box-shadow:0 5px 15px rgba(0,0,0,.08);
    margin-bottom:25px;
}
.card h2 {margin-top:0; color:#0a2a43;}
input, select, button, textarea {
    width:100%;
    padding:12px;
    margin-bottom:12px;
    font-family:'Cairo';
    border-radius:8px;
    border:1px solid #ccc;
}
textarea { resize: vertical; }
button {
    background:#0a2a43;
    color:#fff;
    border:none;
    border-radius:8px;
    cursor:pointer;
    font-size:16px;
    transition:0.3s;
}
button:hover { opacity:.9; }
table {
    width:100%;
    border-collapse:collapse;
    margin-top:15px;
}
th, td {
    padding:12px;
    border-bottom:1px solid #ddd;
    text-align:center;
}
th {background:#0a2a43;color:#fff;}
.actions button {margin:0 5px;padding:6px 12px;font-size:14px;}
</style>
</head>
<body>

<header>
<h1>إدارة الموكلين | Cabinet Juridique</h1>
</header>

<div class="container">

<!-- نموذج إضافة / تعديل موكل -->
<div class="card">
<h2>إضافة / تعديل موكل</h2>
<input type="hidden" id="index">
<input type="text" id="name" placeholder="الاسم الكامل">
<input type="text" id="cin" placeholder="رقم البطاقة الوطنية">
<input type="text" id="phone" placeholder="رقم الهاتف">
<input type="text" id="address" placeholder="العنوان">
<input type="text" id="fees" placeholder="أتعاب المحامي (د.م)">
<textarea id="notes" placeholder="ملاحظات"></textarea>
<button onclick="saveClient()">💾 حفظ الموكل</button>
</div>

<!-- جدول عرض الموكلين -->
<div class="card">
<h2>قائمة الموكلين</h2>
<table>
<thead>
<tr>
<th>#</th>
<th>الاسم</th>
<th>CIN</th>
<th>الهاتف</th>
<th>العنوان</th>
<th>أتعاب المحامي</th>
<th>ملاحظات</th>
<th>إجراءات</th>
</tr>
</thead>
<tbody id="clientsTable"></tbody>
</table>
</div>

</div>

<script>
// ===== LocalStorage =====
let clients = JSON.parse(localStorage.getItem('clients')) || [];

// حفظ / تعديل موكل
function saveClient() {
    const idx = document.getElementById('index').value;
    const client = {
        name: document.getElementById('name').value,
        cin: document.getElementById('cin').value,
        phone: document.getElementById('phone').value,
        address: document.getElementById('address').value,
        fees: document.getElementById('fees').value,
        notes: document.getElementById('notes').value
    };
    if(idx === '') { clients.push(client); }
    else { clients[idx] = client; }
    localStorage.setItem('clients', JSON.stringify(clients));
    resetForm();
    renderClients();
}

// تحرير موكل
function editClient(i) {
    const c = clients[i];
    document.getElementById('index').value = i;
    document.getElementById('name').value = c.name;
    document.getElementById('cin').value = c.cin;
    document.getElementById('phone').value = c.phone;
    document.getElementById('address').value = c.address;
    document.getElementById('fees').value = c.fees;
    document.getElementById('notes').value = c.notes;
}

// حذف موكل
function deleteClient(i) {
    if(confirm('هل أنت متأكد من حذف هذا الموكل؟')){
        clients.splice(i,1);
        localStorage.setItem('clients', JSON.stringify(clients));
        renderClients();
    }
}

// إعادة تعيين النموذج
function resetForm() {
    document.getElementById('index').value='';
    document.getElementById('name').value='';
    document.getElementById('cin').value='';
    document.getElementById('phone').value='';
    document.getElementById('address').value='';
    document.getElementById('fees').value='';
    document.getElementById('notes').value='';
}

// عرض الموكلين
function renderClients() {
    const table = document.getElementById('clientsTable');
    table.innerHTML = '';
    clients.forEach((c,i)=>{
        table.innerHTML += `<tr>
            <td>${i+1}</td>
            <td>${c.name}</td>
            <td>${c.cin}</td>
            <td>${c.phone}</td>
            <td>${c.address}</td>
            <td>${c.fees} د.م</td>
            <td>${c.notes}</td>
            <td class="actions">
                <button onclick="editClient(${i})">✏️ تعديل</button>
                <button onclick="deleteClient(${i})">🗑️ حذف</button>
            </td>
        </tr>`;
    });
}

// عرض عند تحميل الصفحة
renderClients();
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>لوحة التحكم | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body{margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8;display:flex}
.sidebar{width:220px;background:#0a2a43;color:#fff;min-height:100vh;padding:20px}
.sidebar h2{text-align:center;margin-bottom:30px}
.sidebar a{display:block;color:#fff;text-decoration:none;padding:10px 15px;margin-bottom:10px;border-radius:6px}
.sidebar a:hover{background:#1b4b6b}
.content{flex:1;padding:30px}
.topbar{display:flex;justify-content:space-between;align-items:center;margin-bottom:30px}
.cards{display:flex;gap:20px;flex-wrap:wrap}
.card{flex:1;min-width:200px;background:#fff;padding:20px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);text-align:center}
.table-container{background:#fff;padding:15px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:30px}
table{width:100%;border-collapse:collapse;margin-top:10px}
th,td{padding:10px;border-bottom:1px solid #ddd;text-align:center}
th{background:#f0f0f0;cursor:pointer}
.actions button{margin:0 5px;padding:5px 10px}
input,select{padding:8px;margin-bottom:5px;border-radius:6px;border:1px solid #ccc;width:100%}
.sortable:hover{background:#e0e0e0}
</style>
</head>
<body>

<aside class="sidebar">
<h2>Cabinet Juridique</h2>
<nav class="menu">
  <a href="#" data-role="all">🏠 لوحة التحكم</a>
  <a href="lawyers.html" data-role="admin">⚖️ إدارة المحامين</a>
  <a href="cases.html" data-role="admin,lawyer">📁 القضايا</a>
  <a href="clients.html" data-role="admin,lawyer">👤 الموكلون</a>
  <a href="sessions.html" data-role="admin,assistant">📅 الجلسات</a>
  <a href="documents.html" data-role="admin,lawyer">📁 الوثائق</a>
  <a href="statistics.html" data-role="admin">📊 الإحصائيات</a>
  <a href="#" onclick="logout()">🚪 تسجيل الخروج</a>
</nav>
</aside>

<main class="content">
<div class="topbar">
<h1>لوحة التحكم</h1>
<div class="role" id="userRole">الدور: مدير</div>
</div>

<section class="cards">
  <div class="card">
    <h3>عدد المحامين</h3>
    <strong id="totalLawyers">0</strong>
  </div>
  <div class="card">
    <h3>عدد الموكلين</h3>
    <strong id="totalClients">0</strong>
  </div>
  <div class="card">
    <h3>عدد القضايا</h3>
    <strong id="totalCases">0</strong>
  </div>
  <div class="card">
    <h3>عدد الجلسات</h3>
    <strong id="totalSessions">0</strong>
  </div>
</section>

<!-- جدول القضايا -->
<div class="table-container">
<h3>قائمة القضايا</h3>
<div style="display:flex; gap:10px; flex-wrap:wrap; margin-bottom:10px;">
  <input type="text" id="searchCases" placeholder="ابحث...">
  <select id="filterClient"><option value="">الموكل</option></select>
  <select id="filterLawyer"><option value="">المحامي</option></select>
  <select id="filterStatus"><option value="">الحالة</option><option value="جارية">جارية</option><option value="مغلقة">مغلقة</option></select>
</div>
<table id="casesTableMain">
<thead>
<tr>
  <th data-column="title" class="sortable">عنوان القضية ▲▼</th>
  <th data-column="client" class="sortable">الموكل ▲▼</th>
  <th data-column="lawyer" class="sortable">المحامي ▲▼</th>
  <th data-column="status" class="sortable">الحالة ▲▼</th>
  <th>إجراءات</th>
</tr>
</thead>
<tbody id="casesTable"></tbody>
</table>
</div>

<!-- جدول الجلسات -->
<div class="table-container">
<h3>قائمة الجلسات</h3>
<div style="display:flex; gap:10px; flex-wrap:wrap; margin-bottom:10px;">
  <input type="text" id="searchSessions" placeholder="ابحث...">
  <select id="filterCourt"><option value="">المحكمة</option></select>
  <select id="filterJudge"><option value="">القاضي</option></select>
  <select id="filterResult"><option value="">النتيجة</option></select>
</div>
<table id="sessionsTableMain">
<thead>
<tr>
  <th data-column="caseRef" class="sortable">القضية ▲▼</th>
  <th data-column="client" class="sortable">الموكل ▲▼</th>
  <th data-column="lawyer" class="sortable">المحامي ▲▼</th>
  <th data-column="date" class="sortable">التاريخ ▲▼</th>
  <th data-column="court" class="sortable">المحكمة ▲▼</th>
  <th data-column="room" class="sortable">القاعة ▲▼</th>
  <th data-column="judge" class="sortable">القاضي ▲▼</th>
  <th data-column="result" class="sortable">النتيجة ▲▼</th>
  <th>إجراءات</th>
</tr>
</thead>
<tbody id="sessionsTable"></tbody>
</table>
</div>

<script>
// محاكاة دور المستخدم
const currentRole = localStorage.getItem('role') || 'admin';
document.getElementById('userRole').innerText = 'الدور: ' + (currentRole==='admin'?'مدير':currentRole==='lawyer'?'محامٍ':'مساعد');

// إخفاء الروابط غير المصرح بها
document.querySelectorAll('.menu a').forEach(link=>{
  const roles = link.dataset.role;
  if(roles && roles !== 'all' && !roles.split(',').includes(currentRole)){
    link.style.display = 'none';
  }
});

// بيانات التخزين
const lawyers = JSON.parse(localStorage.getItem('lawyers')) || [];
const clients = JSON.parse(localStorage.getItem('clients')) || [];
const cases = JSON.parse(localStorage.getItem('cases')) || [];
const sessions = JSON.parse(localStorage.getItem('sessions')) || [];

document.getElementById('totalLawyers').innerText = lawyers.length;
document.getElementById('totalClients').innerText = clients.length;
document.getElementById('totalCases').innerText = cases.length;
document.getElementById('totalSessions').innerText = sessions.length;

function logout(){ localStorage.removeItem('role'); window.location.href='login.html'; }

// ==== وظائف البحث + الفلترة + الترتيب ====

function populateFilters(){
  const clientSelect = document.getElementById('filterClient');
  clients.forEach(c=>{ let o=document.createElement('option'); o.value=c.name;o.innerText=c.name;clientSelect.appendChild(o); });
  const lawyerSelect = document.getElementById('filterLawyer');
  lawyers.forEach(l=>{ let o=document.createElement('option'); o.value=l.name;o.innerText=l.name;lawyerSelect.appendChild(o); });
  const courtSelect = document.getElementById('filterCourt');
  [...new Set(sessions.map(s=>s.court))].forEach(c=>{ let o=document.createElement('option');o.value=c;o.innerText=c;courtSelect.appendChild(o); });
  const judgeSelect = document.getElementById('filterJudge');
  [...new Set(sessions.map(s=>s.judge))].forEach(j=>{ let o=document.createElement('option');o.value=j;o.innerText=j;judgeSelect.appendChild(o); });
  const resultSelect = document.getElementById('filterResult');
  [...new Set(sessions.map(s=>s.result))].forEach(r=>{ let o=document.createElement('option'); o.value=r; o.innerText=r; resultSelect.appendChild(o); });
}

function filterAndRenderCases(){
  const term = document.getElementById('searchCases').value.toLowerCase();
  const clientFilter = document.getElementById('filterClient').value;
  const lawyerFilter = document.getElementById('filterLawyer').value;
  const statusFilter = document.getElementById('filterStatus').value;

  const table = document.getElementById('casesTable');
  table.innerHTML='';
  cases.forEach((c,i)=>{
    const clientName = clients[c.clientId]?clients[c.clientId].name:'غير معروف';
    const lawyerName = lawyers[c.lawyerId]?lawyers[c.lawyerId].name:'غير معروف';
    const text = `${c.title} ${clientName} ${lawyerName} ${c.status}`.toLowerCase();
    if(text.includes(term) && (clientFilter===''||clientName===clientFilter) && (lawyerFilter===''||lawyerName===lawyerFilter) && (statusFilter===''||c.status===statusFilter)){
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${i+1}</td>
      <td>${c.title}</td>
      <td><a href="client-profile.html?index=${c.clientId}" target="_blank">${clientName}</a></td>
      <td><a href="lawyer-profile.html?index=${c.lawyerId}" target="_blank">${lawyerName}</a></td>
      <td>${c.status}</td>
      <td class="actions"><button onclick="alert('فتح تفاصيل القضية')">✏️ فتح</button></td>`;
      table.appendChild(tr);
    }
  });
}

function filterAndRenderSessions(){
  const term=document.getElementById('searchSessions').value.toLowerCase();
  const courtFilter=document.getElementById('filterCourt').value;
  const judgeFilter=document.getElementById('filterJudge').value;
  const resultFilter=document.getElementById('filterResult').value;
  const table=document.getElementById('sessionsTable');
  table.innerHTML='';
  sessions.forEach((s,i)=>{
    const c = cases.find(c=>c.ref===s.caseRef);
    const clientName = c && clients[c.clientId]?clients[c.clientId].name:'غير معروف';
    const lawyerName = c && lawyers[c.lawyerId]?lawyers[c.lawyerId].name:'غير معروف';
    if((`${s.caseRef} ${clientName} ${lawyerName} ${s.court} ${s.judge} ${s.result}`).toLowerCase().includes(term)
      && (courtFilter===''||s.court===courtFilter)
      && (judgeFilter===''||s.judge===judgeFilter)
      && (resultFilter===''||s.result===resultFilter)){
        const tr=document.createElement('tr');
        tr.innerHTML=`<td>${i+1}</td>
        <td>${s.caseRef}</td>
        <td><a href="client-profile.html?index=${c?c.clientId:0}" target="_blank">${clientName}</a></td>
        <td><a href="lawyer-profile.html?index=${c?c.lawyerId:0}" target="_blank">${lawyerName}</a></td>
        <td>${s.date}</td>
        <td>${s.court}</td>
        <td>${s.room}</td>
        <td>${s.judge}</td>
        <td>${s.result}</td>
        <td class="actions"><button onclick="alert('فتح تفاصيل الجلسة')">✏️ فتح</button></td>`;
        table.appendChild(tr);
      }
  });
}

// ترتيب الأعمدة
function sortTable(tableId, columnKey){
  const table = document.getElementById(tableId);
  let rows=Array.from(table.querySelectorAll('tr'));
  let asc = table.getAttribute('data-sort')!=='asc';
  rows.sort((a,b)=>{
    let i=Array.from(a.parentNode.children).indexOf(a);
    let valA = a.cells[columnKeyIndex(columnKey, tableId)].innerText.toLowerCase();
    let valB = b.cells[columnKeyIndex(columnKey, tableId)].innerText.toLowerCase();
    return asc?valA.localeCompare(valB):valB.localeCompare(valA);
  });
  table.setAttribute('data-sort',asc?'asc':'desc');
  rows.forEach(r=>table.appendChild(r));
}

function columnKeyIndex(key,tableId){
  const headers = document.getElementById(tableId).closest('.table-container').querySelectorAll('th');
  for(let i=0;i<headers.length;i++){if(headers[i].dataset.column===key)return i;} return 0;
}

// أحداث البحث + فلترة + ترتيب
document.getElementById('searchCases').addEventListener('input',filterAndRenderCases);
document.getElementById('filterClient').addEventListener('change',filterAndRenderCases);
document.getElementById('filterLawyer').addEventListener('change',filterAndRenderCases);
document.getElementById('filterStatus').addEventListener('change',filterAndRenderCases);

document.getElementById('searchSessions').addEventListener('input',filterAndRenderSessions);
document.getElementById('filterCourt').addEventListener('change',filterAndRenderSessions);
document.getElementById('filterJudge').addEventListener('change',filterAndRenderSessions);
document.getElementById('filterResult').addEventListener('change',filterAndRenderSessions);

document.querySelectorAll('.sortable').forEach(th=>th.addEventListener('click',()=>{sortTable(th.closest('.table-container').querySelector('tbody').id,th.dataset.column);}))

// تفعيل عند تحميل الصفحة
populateFilters();
filterAndRenderCases();
filterAndRenderSessions();

</script>
</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إدارة الوثائق | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body{margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8}
header{background:#0a2a43;color:#fff;padding:15px;text-align:center}
.container{padding:30px}
.form-box,.table-box{background:#fff;padding:20px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:30px}
h2{margin-top:0}
input,select,button{width:100%;padding:10px;margin-bottom:10px;font-family:'Cairo'}
button{background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer}
button:hover{opacity:.9}
table{width:100%;border-collapse:collapse}
th,td{padding:10px;border-bottom:1px solid #ddd;text-align:center}
th{background:#f0f0f0}
.actions button{width:auto;margin:0 5px;padding:6px 10px}
</style>
</head>
<body>


<header>
<h1>إدارة الوثائق</h1>
</header>


<div class="container">


<!-- Form -->
<div class="form-box">
<h2>إضافة وثيقة</h2>
<input type="hidden" id="index">
<select id="caseRef"></select>
<input type="text" id="title" placeholder="عنوان الوثيقة">
<input type="file" id="file" accept=".pdf,.jpg,.png">
<button onclick="saveDocument()">💾 حفظ</button>
</div>


<!-- Table -->
<div class="table-box">
<h2>قائمة الوثائق</h2>
<table>
<thead>
<tr>
<th>#</th>
<th>القضية</th>
<th>العنوان</th>
<th>الملف</th>
<th>إجراءات</th>
</tr>
</thead>
<tbody id="docsTable"></tbody>
</table>
</div>


</div>


<script>
let documents = JSON.parse(localStorage.getItem('documents')) || [];
let cases = JSON.parse(localStorage.getItem('cases')) || [];


function loadCases(){
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>أتعاب المحامي | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">

<style>
body{margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8}
header{background:#0a2a43;color:#fff;padding:15px;text-align:center}
.container{padding:30px;max-width:1100px;margin:auto}
.box{background:#fff;padding:20px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:30px}
h2{margin-top:0;color:#0a2a43}
input,select,textarea,button{
  width:100%;padding:10px;margin-bottom:10px;font-family:'Cairo'
}
button{
  background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer
}
button:hover{opacity:.9}
table{width:100%;border-collapse:collapse}
th,td{padding:10px;border-bottom:1px solid #ddd;text-align:center}
th{background:#f0f0f0}
.badge{
  padding:4px 10px;border-radius:20px;color:#fff;font-size:13px
}
.ok{background:#2ecc71}
.wait{background:#f39c12}
</style>
</head>

<body>

<header>
  <h1>أتعاب المحامي</h1>
</header>

<div class="container">

<!-- ===== نموذج ===== -->
<div class="box">
<h2>إضافة / تعديل أتعاب</h2>

<input type="hidden" id="index">

<select id="caseId">
  <option value="">اختر القضية</option>
</select>

<input type="number" id="total" placeholder="المبلغ المتفق عليه (درهم)">
<input type="number" id="paid" placeholder="المبلغ المؤدى (درهم)" oninput="calcRest()">
<input type="number" id="rest" placeholder="المبلغ المتبقي" readonly>

<textarea id="notes" placeholder="ملاحظات"></textarea>

<button onclick="saveFee()">💾 حفظ</button>
</div>

<!-- ===== جدول ===== -->
<div class="box">
<h2>قائمة الأتعاب</h2>

<table>
<thead>
<tr>
  <th>#</th>
  <th>القضية</th>
  <th>المجموع</th>
  <th>المؤدى</th>
  <th>المتبقي</th>
  <th>الحالة</th>
  <th>إجراءات</th>
</tr>
</thead>
<tbody id="feesTable"></tbody>
</table>
</div>

</div>

<script>
// ===== البيانات =====
let fees  = JSON.parse(localStorage.getItem("fees")) || [];
let cases = JSON.parse(localStorage.getItem("cases")) || [];

// ===== تحميل القضايا =====
function loadCases(){
  caseId.innerHTML = `<option value="">اختر القضية</option>`;
  cases.forEach((c,i)=>{
    caseId.innerHTML += `<option value="${i}">${c.title}</option>`;
  });
}

// ===== حساب الباقي =====
function calcRest(){
  const total = parseFloat(document.getElementById("total").value) || 0;
  const paid  = parseFloat(document.getElementById("paid").value) || 0;
  document.getElementById("rest").value = total - paid;
}

// ===== حفظ =====
function saveFee(){
  const index = document.getElementById("index").value;

  const fee = {
    caseId: parseInt(caseId.value),
    total: parseFloat(total.value),
    paid: parseFloat(paid.value),
    rest: parseFloat(rest.value),
    notes: notes.value
  };

  if(index === ""){
    fees.push(fee);
  }else{
    fees[index] = fee;
  }

  localStorage.setItem("fees", JSON.stringify(fees));
  resetForm();
  renderFees();
}

// ===== تعديل =====
function editFee(i){
  const f = fees[i];
  index.value = i;
  caseId.value = f.caseId;
  total.value = f.total;
  paid.value  = f.paid;
  rest.value  = f.rest;
  notes.value = f.notes;
}

// ===== حذف =====
function deleteFee(i){
  if(confirm("هل تريد حذف هذه البيانات؟")){
    fees.splice(i,1);
    localStorage.setItem("fees", JSON.stringify(fees));
    renderFees();
  }
}

// ===== إعادة تعيين =====
function resetForm(){
  index.value="";
  caseId.value="";
  total.value="";
  paid.value="";
  rest.value="";
  notes.value="";
}

// ===== عرض =====
function renderFees(){
  feesTable.innerHTML = "";

  fees.forEach((f,i)=>{
    const caseTitle = cases[f.caseId]?.title || "غير محددة";
    const status = f.rest <= 0
      ? `<span class="badge ok">مكتملة</span>`
      : `<span class="badge wait">غير مكتملة</span>`;

    feesTable.innerHTML += `
      <tr>
        <td>${i+1}</td>
        <td>${caseTitle}</td>
        <td>${f.total} DH</td>
        <td>${f.paid} DH</td>
        <td>${f.rest} DH</td>
        <td>${status}</td>
        <td>
          <button onclick="editFee(${i})">✏️</button>
          <button onclick="deleteFee(${i})">🗑️</button>
        </td>
      </tr>
    `;
  });
}

// ===== تشغيل =====
loadCases();
renderFees();
</script>

</body>
</html>
<!DOCTYPE html>
<title>Cabinet Juridique | مكتب للاستشارات القانونية</title>
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&family=Poppins:wght@400;600&display=swap" rel="stylesheet">
<style>
:root{
--primary:#0a2a43;
--secondary:#6c757d;
--light:#f8f9fa;
}
body{margin:0;font-family:'Cairo',sans-serif;background:var(--light);color:#222}
header{background:var(--primary);color:#fff;padding:20px}
nav{display:flex;justify-content:space-between;align-items:center}
nav ul{list-style:none;display:flex;gap:20px;margin:0;padding:0}
nav a{color:#fff;text-decoration:none;font-weight:600}
.lang-switch{font-family:'Poppins',sans-serif}
.hero{padding:80px 20px;text-align:center;background:#fff}
.hero h1{font-size:36px;margin-bottom:10px}
.hero p{font-size:18px;color:#555}
.sections{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;padding:40px}
.card{background:#fff;border-radius:12px;padding:20px;box-shadow:0 5px 15px rgba(0,0,0,.08)}
footer{background:var(--primary);color:#fff;text-align:center;padding:15px;font-size:14px}
.fr{display:none;direction:ltr;font-family:'Poppins',sans-serif}
</style>
</head>
<body>


<header>
<nav>
<strong>Cabinet Juridique</strong>
<ul>
<li><a href="#home">الرئيسية</a></li>
<li><a href="#about">من نحن</a></li>
<li><a href="#services">التخصصات</a></li>
<li><a href="#contact">اتصل بنا</a></li>
</ul>
<div class="lang-switch">
<button onclick="setLang('ar')">AR</button>
<button onclick="setLang('fr')">FR</button>
</div>
</nav>
</header>


<section class="hero" id="home">
<div class="ar">
<h1>مكتب للاستشارات القانونية</h1>
<p>نضع خبرتنا القانونية رهن إشارتكم للدفاع عن حقوقكم.</p>
</div>
<div class="fr">
<h1>Cabinet Juridique</h1>
<p>Nous mettons notre expertise juridique à votre service.</p>
</div>
</section>


<section class="sections" id="services">
<div class="card ar">القانون الجنائي</div>
<div class="card ar">القانون المدني</div>
<div class="card ar">القانون التجاري</div>


<div class="card fr">Droit pénal</div>
<div class="card fr">Droit civil</div>
<div class="card fr">Droit commercial</div>
</section>


<footer>
© 2026 Cabinet Juridique – جميع الحقوق محفوظة
</footer>


<script>
function setLang(lang){
document.querySelectorAll('.ar').forEach(e=>e.style.display = lang==='ar'?'block':'none');
document.querySelectorAll('.fr').forEach(e=>e.style.display = lang==='fr'?'block':'none');
document.documentElement.lang = lang;
document.documentElement.dir = lang==='ar'?'rtl':'ltr';
}
</script>


</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إدارة القضايا | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">

<style>
body{
  margin:0;
  font-family:'Cairo',sans-serif;
  background:#f4f6f8;
}
header{
  background:#0a2a43;
  color:#fff;
  padding:15px;
  text-align:center;
}
.container{
  padding:30px;
  max-width:1100px;
  margin:auto;
}
.box{
  background:#fff;
  padding:20px;
  border-radius:12px;
  box-shadow:0 5px 15px rgba(0,0,0,.08);
  margin-bottom:30px;
}
h2{margin-top:0;color:#0a2a43}
input,select,button{
  width:100%;
  padding:10px;
  margin-bottom:10px;
  font-family:'Cairo';
}
button{
  background:#0a2a43;
  color:#fff;
  border:none;
  border-radius:8px;
  cursor:pointer;
}
button:hover{opacity:.9}
table{
  width:100%;
  border-collapse:collapse;
}
th,td{
  padding:10px;
  border-bottom:1px solid #ddd;
  text-align:center;
}
th{
  background:#f0f0f0;
}
.actions button{
  width:auto;
  margin:0 4px;
  padding:5px 10px;
}
</style>
</head>

<body>

<header>
  <h1>إدارة القضايا</h1>
</header>

<div class="container">

<!-- ===== نموذج إضافة / تعديل قضية ===== -->
<div class="box">
  <h2>إضافة / تعديل قضية</h2>

  <input type="hidden" id="index">

  <input type="text" id="title" placeholder="عنوان القضية">

  <input type="text" id="ref" placeholder="رقم / مرجع القضية">

  <select id="clientId">
    <option value="">اختر الموكل</option>
  </select>

  <select id="lawyerId">
    <option value="">اختر المحامي</option>
  </select>

  <select id="status">
    <option value="">الحالة</option>
    <option value="جارية">جارية</option>
    <option value="مغلقة">مغلقة</option>
  </select>

  <button onclick="saveCase()">💾 حفظ القضية</button>
</div>

<!-- ===== جدول القضايا ===== -->
<div class="box">
  <h2>قائمة القضايا</h2>

  <table>
    <thead>
      <tr>
        <th>#</th>
        <th>عنوان القضية</th>
        <th>الموكل</th>
        <th>المحامي</th>
        <th>الحالة</th>
        <th>إجراءات</th>
      </tr>
    </thead>
    <tbody id="casesTable"></tbody>
  </table>
</div>

</div>

<script>
// ===== البيانات =====
let cases   = JSON.parse(localStorage.getItem("cases"))   || [];
let clients = JSON.parse(localStorage.getItem("clients")) || [];
let lawyers = JSON.parse(localStorage.getItem("lawyers")) || [];

// ===== تحميل الموكلين =====
function loadClients(){
  const select = document.getElementById("clientId");
  select.innerHTML = `<option value="">اختر الموكل</option>`;
  clients.forEach((c,i)=>{
    select.innerHTML += `<option value="${i}">${c.name}</option>`;
  });
}

// ===== تحميل المحامين =====
function loadLawyers(){
  const select = document.getElementById("lawyerId");
  select.innerHTML = `<option value="">اختر المحامي</option>`;
  lawyers.forEach((l,i)=>{
    select.innerHTML += `<option value="${i}">${l.name}</option>`;
  });
}

// ===== حفظ القضية =====
function saveCase(){
  const index = document.getElementById("index").value;

  const caseObj = {
    title: document.getElementById("title").value,
    ref: document.getElementById("ref").value,
    clientId: parseInt(document.getElementById("clientId").value),
    lawyerId: parseInt(document.getElementById("lawyerId").value),
    status: document.getElementById("status").value
  };

  if(index === ""){
    cases.push(caseObj);
  }else{
    cases[index] = caseObj;
  }

  localStorage.setItem("cases", JSON.stringify(cases));
  resetForm();
  renderCases();
}

// ===== تعديل قضية =====
function editCase(i){
  const c = cases[i];
  document.getElementById("index").value = i;
  document.getElementById("title").value = c.title;
  document.getElementById("ref").value = c.ref;
  document.getElementById("clientId").value = c.clientId;
  document.getElementById("lawyerId").value = c.lawyerId;
  document.getElementById("status").value = c.status;
}

// ===== حذف قضية =====
function deleteCase(i){
  if(confirm("هل تريد حذف هذه القضية؟")){
    cases.splice(i,1);
    localStorage.setItem("cases", JSON.stringify(cases));
    renderCases();
  }
}

// ===== إعادة تعيين النموذج =====
function resetForm(){
  document.getElementById("index").value = "";
  document.getElementById("title").value = "";
  document.getElementById("ref").value = "";
  document.getElementById("clientId").value = "";
  document.getElementById("lawyerId").value = "";
  document.getElementById("status").value = "";
}

// ===== عرض القضايا =====
function renderCases(){
  const table = document.getElementById("casesTable");
  table.innerHTML = "";

  cases.forEach((c,i)=>{
    const clientName = clients[c.clientId]?.name || "غير محدد";
    const lawyerName = lawyers[c.lawyerId]?.name || "غير محدد";

    table.innerHTML += `
      <tr>
        <td>${i+1}</td>
        <td>${c.title}</td>
        <td>${clientName}</td>
        <td>${lawyerName}</td>
        <td>${c.status}</td>
        <td class="actions">
          <button onclick="editCase(${i})">✏️</button>
          <button onclick="deleteCase(${i})">🗑️</button>
        </td>
      </tr>
    `;
  });
}

// ===== تشغيل =====
loadClients();
loadLawyers();
renderCases();
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إدارة المحامين | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body{margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8}
header{background:#0a2a43;color:#fff;padding:15px;text-align:center}
.container{padding:30px}
.form-box,.table-box{background:#fff;padding:20px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:30px}
h2{margin-top:0}
input,button{width:100%;padding:10px;margin-bottom:10px;font-family:'Cairo'}
button{background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer}
button:hover{opacity:.9}
table{width:100%;border-collapse:collapse}
th,td{padding:10px;border-bottom:1px solid #ddd;text-align:center}
th{background:#f0f0f0}
.actions button{width:auto;margin:0 5px;padding:6px 10px}
</style>
</head>
<body>

<header>
<h1>إدارة المحامين</h1>
</header>

<div class="container">

  <!-- Form -->
  <div class="form-box">
    <h2>إضافة / تعديل محامي</h2>
    <input type="hidden" id="index">
    <input type="text" id="name" placeholder="الاسم الكامل">
    <input type="text" id="specialty" placeholder="التخصص">
    <input type="text" id="phone" placeholder="رقم الهاتف">
    <input type="email" id="email" placeholder="البريد الإلكتروني">
    <button onclick="saveLawyer()">💾 حفظ</button>
  </div>

  <!-- Table -->
  <div class="table-box">
    <h2>قائمة المحامين</h2>
    <table>
      <thead>
        <tr>
          <th>#</th>
          <th>الاسم</th>
          <th>التخصص</th>
          <th>الهاتف</th>
          <th>الإيميل</th>
          <th>إجراءات</th>
        </tr>
      </thead>
      <tbody id="lawyersTable"></tbody>
    </table>
  </div>

</div>

<script>
let lawyers = JSON.parse(localStorage.getItem('lawyers')) || [];

function saveLawyer(){
  const index = document.getElementById('index').value;
  const lawyer = {
    name: name.value,
    specialty: specialty.value,
    phone: phone.value,
    email: email.value
  };

  if(index === ''){
    lawyers.push(lawyer);
  } else {
    lawyers[index] = lawyer;
  }

  localStorage.setItem('lawyers', JSON.stringify(lawyers));
  resetForm();
  renderTable();
}

function editLawyer(i){
  const l = lawyers[i];
  index.value = i;
  name.value = l.name;
  specialty.value = l.specialty;
  phone.value = l.phone;
  email.value = l.email;
}

function deleteLawyer(i){
  if(confirm('هل أنت متأكد من الحذف؟')){
    lawyers.splice(i,1);
    localStorage.setItem('lawyers', JSON.stringify(lawyers));
    renderTable();
  }
}

function resetForm(){
  index.value='';
  name.value='';
  specialty.value='';
  phone.value='';
  email.value='';
}

function renderTable(){
  lawyersTable.innerHTML='';
  lawyers.forEach((l,i)=>{
    lawyersTable.innerHTML += `
      <tr>
        <td>${i+1}</td>
        <td>${l.name}</td>
        <td>${l.specialty}</td>
        <td>${l.phone}</td>
        <td>${l.email}</td>
        <td class="actions">
          <button onclick="editLawyer(${i})">✏️</button>
          <button onclick="deleteLawyer(${i})">🗑️</button>
        </td>
      </tr>`;
  });
}

renderTable();
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>تسجيل الدخول | Cabinet Juridique</title>
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body{font-family:Cairo;background:#f4f6f8;display:flex;align-items:center;justify-content:center;height:100vh}
.card{background:#fff;padding:30px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.15);width:360px}
h2{text-align:center;color:#0a2a43}
input,button{width:100%;padding:10px;margin-top:10px;font-family:Cairo}
button{background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer}
.error{color:red;text-align:center;margin-top:10px}
</style>
</head>
<body>

<div class="card">
<h2>🔐 تسجيل الدخول</h2>
<input id="username" placeholder="اسم المستخدم">
<input id="password" type="password" placeholder="كلمة المرور">
<button onclick="login()">دخول</button>
<div class="error" id="error"></div>
</div>

<script>
const users = [
  {username:"admin", password:"1234", role:"admin"},
  {username:"lawyer", password:"1234", role:"lawyer"},
  {username:"assistant", password:"1234", role:"assistant"}
];

function login(){
  const u = username.value;
  const p = password.value;
  const user = users.find(x=>x.username===u && x.password===p);
  if(!user){
    error.innerText="❌ بيانات غير صحيحة";
    return;
  }
  localStorage.setItem("currentUser", JSON.stringify(user));
  window.location.href="dashboard.html";
}
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>ملف الموكل | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<style>
body {margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8;padding:20px;}
header {background:#0a2a43;color:#fff;padding:20px;text-align:center;border-radius:12px;margin-bottom:20px;box-shadow:0 5px 15px rgba(0,0,0,.1);}
header h1 {margin:0;}
.container {max-width:1200px;margin:auto;}
.card {background:#fff;padding:25px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:25px;}
.card h2 {margin-top:0;color:#0a2a43;}
table {width:100%;border-collapse:collapse;margin-top:15px;}
th, td {padding:12px;border-bottom:1px solid #ddd;text-align:center;}
th {background:#0a2a43;color:#fff;}
.actions button {margin:0 5px;padding:6px 12px;font-size:14px;}
button {background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer;font-size:16px;transition:.3s;margin-top:10px;}
button:hover {opacity:.9;}
</style>
</head>
<body>

<header>
<h1>ملف الموكل | Cabinet Juridique</h1>
</header>

<div class="container">

<div class="card">
<h2>بيانات الموكل</h2>
<p><strong>الاسم:</strong> <span id="clientName"></span></p>
<p><strong>البريد الإلكتروني:</strong> <span id="clientEmail"></span></p>
<p><strong>الهاتف:</strong> <span id="clientPhone"></span></p>
<button onclick="exportClientPDF()">📄 تصدير PDF موكل</button>
</div>

<div class="card">
<h2>قضايا الموكل</h2>
<table>
<thead>
<tr>
<th>#</th>
<th>رقم / مرجع القضية</th>
<th>عنوان القضية</th>
<th>المحامي</th>
<th>الحالة</th>
<th>أتعاب المحامي</th>
</tr>
</thead>
<tbody id="clientCasesTable"></tbody>
</table>
</div>

</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>

<script>
// ===== بيانات LocalStorage =====
let clients = JSON.parse(localStorage.getItem('clients')) || [];
let lawyers = JSON.parse(localStorage.getItem('lawyers')) || [];
let cases = JSON.parse(localStorage.getItem('cases')) || [];

// ===== استدعاء موكل معين من الرابط =====
const urlParams = new URLSearchParams(window.location.search);
const clientIndex = parseInt(urlParams.get('index')) || 0;
const client = clients[clientIndex] || {name:'غير معروف', email:'', phone:''};

document.getElementById('clientName').innerText = client.name;
document.getElementById('clientEmail').innerText = client.email;
document.getElementById('clientPhone').innerText = client.phone;

// عرض القضايا الخاصة بالموكل
function renderClientCases(){
    const table = document.getElementById('clientCasesTable');
    table.innerHTML='';
    let clientCases = cases.filter(c=>c.clientId===clientIndex);
    clientCases.forEach((c,i)=>{
        const lawyerName = lawyers[c.lawyerId]?.name || "غير معروف";
        table.innerHTML += `<tr>
            <td>${i+1}</td>
            <td>${c.ref}</td>
            <td>${c.title}</td>
            <td>${lawyerName}</td>
            <td>${c.status}</td>
            <td>${c.fees} د.م</td>
        </tr>`;
    });
}
renderClientCases();

// تصدير PDF لكل موكل
async function exportClientPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF('p','pt','a4');

    // إضافة خط عربي Cairo
    const fontUrl = 'https://raw.githubusercontent.com/google/fonts/main/ofl/cairo/Cairo-Regular.ttf';
    const fontResponse = await fetch(fontUrl);
    const fontBuffer = await fontResponse.arrayBuffer();
    doc.addFileToVFS("Cairo-Regular.ttf", fontBuffer);
    doc.addFont("Cairo-Regular.ttf","Cairo","normal");
    doc.setFont("Cairo");

    let y = 40;
    doc.setFontSize(22);
    doc.text("🏛️ مكتب الاستشارات القانونية", 297.5, y, {align:"center"});
    y += 25;
    doc.setFontSize(16);
    doc.text(`ملف الموكل: ${client.name}`, 297.5, y, {align:"center"});
    y += 30;

    // بيانات الموكل
    doc.setFontSize(14);
    doc.text(`البريد الإلكتروني: ${client.email}`, 40, y); y+=18;
    doc.text(`الهاتف: ${client.phone}`, 40, y); y+=25;

    // إعداد بيانات القضايا
    let tableData = [["#", "مرجع القضية", "عنوان القضية", "المحامي", "الحالة", "أتعاب المحامي"]];
    let clientCases = cases.filter(c=>c.clientId===clientIndex);
    clientCases.forEach((c,i)=>{
        const lawyerName = lawyers[c.lawyerId]?.name || "غير معروف";
        tableData.push([i+1, c.ref, c.title, lawyerName, c.status, c.fees + " د.م"]);
    });

    doc.autoTable({
        startY:y,
        head: [tableData[0]],
        body: tableData.slice(1),
        theme: 'grid',
        headStyles: { fillColor:[10,42,67], textColor:255 },
        styles: { font: 'Cairo', fontSize:12, cellPadding:4, halign:'center' },
        margin: { left: 40, right: 40 }
    });

    y = doc.lastAutoTable.finalY + 20;

    // ملخص إحصائي
    const totalCases = tableData.length - 1;
    const totalFees = clientCases.reduce((sum,c)=>sum+(c.fees||0),0);
    doc.setFontSize(16);
    doc.text("ملخص إحصائي:", 40, y); y+=20;
    doc.setFontSize(14);
    doc.text(`عدد القضايا: ${totalCases}`, 40, y); y+=18;
    doc.text(`إجمالي أتعاب المحامي: ${totalFees} د.م`, 40, y); y+=18;

    doc.save(`ملف_${client.name}_القضايا.pdf`);
}
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إدارة الجلسات | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">

<style>
body{margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8}
header{background:#0a2a43;color:#fff;padding:15px;text-align:center}
.container{padding:30px;max-width:1100px;margin:auto}
.box{background:#fff;padding:20px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:30px}
h2{margin-top:0;color:#0a2a43}
input,select,textarea,button{
  width:100%;padding:10px;margin-bottom:10px;font-family:'Cairo'
}
textarea{resize:vertical}
button{
  background:#0a2a43;color:#fff;border:none;border-radius:8px;cursor:pointer
}
button:hover{opacity:.9}
table{width:100%;border-collapse:collapse}
th,td{padding:10px;border-bottom:1px solid #ddd;text-align:center}
th{background:#f0f0f0}
.actions button{width:auto;margin:0 4px;padding:5px 10px}
</style>
</head>

<body>

<header>
  <h1>إدارة الجلسات</h1>
</header>

<div class="container">

<!-- ===== إضافة / تعديل جلسة ===== -->
<div class="box">
<h2>إضافة / تعديل جلسة</h2>

<input type="hidden" id="index">

<select id="caseId">
  <option value="">اختر القضية</option>
</select>

<input type="date" id="date">

<select id="type">
  <option value="">نوع الجلسة</option>
  <option value="جلسة عادية">جلسة عادية</option>
  <option value="جلسة مداولة">جلسة مداولة</option>
  <option value="جلسة حكم">جلسة حكم</option>
</select>

<textarea id="notes" placeholder="ملاحظات الجلسة"></textarea>

<button onclick="saveSession()">💾 حفظ الجلسة</button>
</div>

<!-- ===== جدول الجلسات ===== -->
<div class="box">
<h2>قائمة الجلسات</h2>

<table>
<thead>
<tr>
  <th>#</th>
  <th>القضية</th>
  <th>التاريخ</th>
  <th>النوع</th>
  <th>ملاحظات</th>
  <th>إجراءات</th>
</tr>
</thead>
<tbody id="sessionsTable"></tbody>
</table>
</div>

</div>

<script>
// ===== البيانات =====
let sessions = JSON.parse(localStorage.getItem("sessions")) || [];
let cases    = JSON.parse(localStorage.getItem("cases")) || [];

// ===== تحميل القضايا =====
function loadCases(){
  const select = document.getElementById("caseId");
  select.innerHTML = `<option value="">اختر القضية</option>`;
  cases.forEach((c,i)=>{
    select.innerHTML += `<option value="${i}">${c.title}</option>`;
  });
}

// ===== حفظ الجلسة =====
function saveSession(){
  const index = document.getElementById("index").value;

  const session = {
    caseId: parseInt(document.getElementById("caseId").value),
    date: document.getElementById("date").value,
    type: document.getElementById("type").value,
    notes: document.getElementById("notes").value
  };

  if(index === ""){
    sessions.push(session);
  }else{
    sessions[index] = session;
  }

  localStorage.setItem("sessions", JSON.stringify(sessions));
  resetForm();
  renderSessions();
}

// ===== تعديل =====
function editSession(i){
  const s = sessions[i];
  index.value = i;
  caseId.value = s.caseId;
  date.value = s.date;
  type.value = s.type;
  notes.value = s.notes;
}

// ===== حذف =====
function deleteSession(i){
  if(confirm("هل تريد حذف هذه الجلسة؟")){
    sessions.splice(i,1);
    localStorage.setItem("sessions", JSON.stringify(sessions));
    renderSessions();
  }
}

// ===== إعادة تعيين =====
function resetForm(){
  index.value = "";
  caseId.value = "";
  date.value = "";
  type.value = "";
  notes.value = "";
}

// ===== عرض =====
function renderSessions(){
  const table = document.getElementById("sessionsTable");
  table.innerHTML = "";

  sessions.forEach((s,i)=>{
    const caseTitle = cases[s.caseId]?.title || "غير محددة";

    table.innerHTML += `
      <tr>
        <td>${i+1}</td>
        <td>${caseTitle}</td>
        <td>${s.date}</td>
        <td>${s.type}</td>
        <td>${s.notes}</td>
        <td class="actions">
          <button onclick="editSession(${i})">✏️</button>
          <button onclick="deleteSession(${i})">🗑️</button>
        </td>
      </tr>
    `;
  });
}

// ===== تشغيل =====
loadCases();
renderSessions();
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>إحصائيات | Cabinet Juridique</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body{margin:0;font-family:'Cairo',sans-serif;background:#f4f6f8}
header{background:#0a2a43;color:#fff;padding:15px;text-align:center}
.container{padding:30px}
.cards{display:flex;gap:20px;flex-wrap:wrap;margin-bottom:30px}
.card{flex:1;min-width:200px;background:#fff;padding:20px;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);text-align:center}
canvas{background:#fff;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,.08);margin-bottom:30px}
</style>
</head>
<body>

<header>
<h1>إحصائيات المكتب</h1>
</header>

<div class="container">
<div class="cards">
<div class="card"><h3>عدد المحامين</h3><strong id="totalLawyers">0</strong></div>
<div class="card"><h3>عدد الموكلين</h3><strong id="totalClients">0</strong></div>
<div class="card"><h3>عدد القضايا</h3><strong id="totalCases">0</strong></div>
<div class="card"><h3>عدد الجلسات</h3><strong id="totalSessions">0</strong></div>
</div>

<canvas id="casesChart" height="200"></canvas>
<canvas id="sessionsChart" height="200"></canvas>
</div>

<script>
const lawyers = JSON.parse(localStorage.getItem('lawyers')) || [];
const clients = JSON.parse(localStorage.getItem('clients')) || [];
const cases = JSON.parse(localStorage.getItem('cases')) || [];
const sessions = JSON.parse(localStorage.getItem('sessions')) || [];

// تحديث الأرقام
document.getElementById('totalLawyers').innerText = lawyers.length;
document.getElementById('totalClients').innerText = clients.length;
document.getElementById('totalCases').innerText = cases.length;
document.getElementById('totalSessions').innerText = sessions.length;

// مخطط القضايا حسب الحالة
const ctxCases = document.getElementById('casesChart').getContext('2d');
const casesByStatus = {جارية:0, مؤجلة:0, مغلقة:0};
cases.forEach(c=>{casesByStatus[c.status]++});
new Chart(ctxCases,{type:'pie',data:{labels:Object.keys(casesByStatus),datasets:[{data:Object.values(casesByStatus),backgroundColor:['#0a2a43','#ff9f43','#28c76f']}]},options:{responsive:true,title:{display:true,text:'حالة القضايا'}}});

// مخطط الجلسات حسب النتيجة
const ctxSessions = document.getElementById('sessionsChart').getContext('2d');
const sessionsByResult = {مجدولة:0, مؤجلة:0, حكم:0};
sessions.forEach(s=>{sessionsByResult[s.result]++});
new Chart(ctxSessions,{type:'bar',data:{labels:Object.keys(sessionsByResult),datasets:[{label:'عدد الجلسات',data:Object.values(sessionsByResult),backgroundColor:['#0a2a43','#ff9f43','#28c76f']}]},options:{responsive:true,plugins:{legend:{display:false}},scales:{y:{beginAtZero:true}}}});
</script>

</body>
</html>
