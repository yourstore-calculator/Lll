# Lll
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #1e1e1e;
      color: white;
      direction: rtl;
      padding: 30px;
    }
    .logo {
      text-align: center;
      margin-bottom: 20px;
    }
    .logo img {
      max-height: 100px;
    }
    select, input {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      border: none;
      border-radius: 4px;
    }
    .section {
      margin-bottom: 25px;
    }
    button {
      background-color: #444;
      color: white;
      padding: 10px 20px;
      border: none;
      margin-top: 10px;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #666;
    }
    .results {
      background: #2b2b2b;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
    }
    .result-item {
      border-bottom: 1px solid #555;
      padding: 10px 0;
    }
    .summary {
      font-weight: bold;
      color: #00e676;
      padding-top: 10px;
    }
  </style>
</head>
<body>

<div class="logo">
  <img src="https://i.imgur.com/Qdc2SU3.png" alt="شعار الشركة">
</div>

<div class="section">
  <label>الفئة الرئيسية:</label>
  <select id="mainCategory" onchange="loadSubTypes()">
    <option value="">-- اختر الفئة --</option>
    <option value="نوافذ">نوافذ</option>
    <option value="أبواب">أبواب</option>
    <option value="أبواب سحب">أبواب سحب</option>
  </select>
</div>

<div class="section">
  <label>النوع الفرعي:</label>
  <select id="subType"></select>
</div>

<div class="section">
  <label>الطول (متر):</label>
  <input type="number" id="height" step="0.01" value="1" />
</div>

<div class="section">
  <label>العرض (متر):</label>
  <input type="number" id="width" step="0.01" value="1" />
</div>

<div class="section">
  <label>الكمية:</label>
  <input type="number" id="quantity" value="1" />
</div>

<div class="section" id="curtainSection" style="display:none">
  <label>إضافة ستارة داخلية؟</label>
  <input type="checkbox" id="addCurtain" />
</div>

<div class="section" id="netSection" style="display:none">
  <label>اختر نوع الشبك:</label>
  <select id="netType">
    <option value="">-- بدون --</option>
    <option value="ثابت">ثابت</option>
    <option value="سلايد">سلايد</option>
    <option value="فولدينج">فولدينج</option>
    <option value="باب">باب</option>
  </select>
</div>

<button onclick="calculate()">➕ أضف للنتائج</button>
<button onclick="clearResults()">🗑️ مسح النتائج</button>
<button onclick="saveAsWord()">💾 حفظ كـ Word</button>

<div class="results" id="results"></div>

<script>
const prices = {
  "دبل جلاس دبل فريم": 73,
  "دبل جلاس سنجل فريم": 46,
  "سنجل جلاس سنجل فريم": 43,
  "نوافذ السلايدنج": 0,
  "النوافذ الكهربائية": 102,
  "سكاي لايت بدون مكينة": 56,
  "سكاي لايت مع مكينة": 145,
  "كارتن وول ثقيل": 56,
  "كارتن وول خفيف": 45,
  "باب المدخل زينك": 66,
  "باب المدخل ستينلس ستيل": 120,
  "باب المدخل كاست المنيوم": 168,
  "WPC فارغ": 45,
  "WPC مع خشب": 50,
  "WPC مع حشوة ضد الصوت": 60,
  "WPC مع فريم ألمنيوم": 67,
  "سلايدنج WPC": 65,
  "ألمنيوم فارغ": 65,
  "ألمنيوم مع خشب": 75,
  "ألمنيوم فل": 85,
  "ألمنيوم مخفي": 110,
  "ألمنيوم خارجي": 61,
  "دورات المياه الجديد": 55,
  "دورات المياه القديم": 45,
  "دورات المياه مخفي زجاجي": 65,
  "داخلي زجاج": 38,
  "داخلي متين": 41,
  "خارجي جزء مفتوح": 55,
  "خارجي جزئين مفتوحات": 58,
  "WPC سلايد": 61,
};

const factors = {
  "default": 0.13,
  "WPC": 0.11,
  "ألمنيوم": 0.11,
  "دورات المياه": 0.11,
  "باب المدخل": 0.2,
};

let resultsList = [];

function loadSubTypes() {
  const category = document.getElementById("mainCategory").value;
  const subType = document.getElementById("subType");
  subType.innerHTML = "";
  document.getElementById("curtainSection").style.display = "none";
  document.getElementById("netSection").style.display = "none";

  let options = [];

  if (category === "نوافذ") {
    options = [
      "دبل جلاس دبل فريم", "دبل جلاس سنجل فريم", "سنجل جلاس سنجل فريم",
      "نوافذ السلايدنج", "النوافذ الكهربائية",
      "سكاي لايت بدون مكينة", "سكاي لايت مع مكينة",
      "كارتن وول ثقيل", "كارتن وول خفيف"
    ];
  } else if (category === "أبواب") {
    options = [
      "باب المدخل زينك", "باب المدخل ستينلس ستيل", "باب المدخل كاست المنيوم",
      "WPC فارغ", "WPC مع خشب", "WPC مع حشوة ضد الصوت", "WPC مع فريم ألمنيوم",
      "ألمنيوم فارغ", "ألمنيوم مع خشب", "ألمنيوم فل", "ألمنيوم مخفي", "ألمنيوم خارجي",
      "دورات المياه الجديد", "دورات المياه القديم", "دورات المياه مخفي زجاجي"
    ];
  } else if (category === "أبواب سحب") {
    options = [
      "داخلي زجاج", "داخلي متين", "خارجي جزء مفتوح", "خارجي جزئين مفتوحات", "WPC سلايد"
    ];
  }

  options.forEach(opt => {
    const o = document.createElement("option");
    o.value = opt;
    o.textContent = opt;
    subType.appendChild(o);
  });

  updateAddons();
}

function updateAddons() {
  const sub = document.getElementById("subType").value;
  if (["دبل جلاس دبل فريم", "دبل جلاس سنجل فريم", "نوافذ السلايدنج",
       "داخلي زجاج", "داخلي متين", "خارجي جزء مفتوح", "خارجي جزئين مفتوحات", "WPC سلايد"]
      .includes(sub)) {
    document.getElementById("curtainSection").style.display = "block";
  } else {
    document.getElementById("curtainSection").style.display = "none";
  }

  if (["دبل جلاس دبل فريم", "دبل جلاس سنجل فريم", "سنجل جلاس سنجل فريم"].includes(sub)) {
    document.getElementById("netSection").style.display = "block";
  } else {
    document.getElementById("netSection").style.display = "none";
  }
}

document.getElementById("subType").addEventListener("change", updateAddons);

function calculate() {
  const sub = document.getElementById("subType").value;
  const quantity = parseInt(document.getElementById("quantity").value);
  const h = parseFloat(document.getElementById("height").value);
  const w = parseFloat(document.getElementById("width").value);
  const area = (h * w);
  const basePrice = prices[sub] || 0;

  let itemPrice = (sub.includes("نوافذ السلايدنج")) ? ((area * 65) + 10) : basePrice * area;
  let factor = factors["default"];

  if (sub.includes("WPC")) factor = factors["WPC"];
  else if (sub.includes("ألمنيوم")) factor = factors["ألمنيوم"];
  else if (sub.includes("دورات المياه")) factor = factors["دورات المياه"];
  else if (sub.includes("باب المدخل")) factor = factors["باب المدخل"];

  const shipping = (area * factor * 48);
  let total = (itemPrice + shipping);

  if (document.getElementById("addCurtain").checked)
    total += (area * 26);

  const netType = document.getElementById("netType").value;
  if (netType) total += 30;

  const final = total * quantity;
  resultsList.push({ sub, h, w, quantity, shipping, final });

  showResults();
}

function showResults() {
  const container = document.getElementById("results");
  container.innerHTML = "";

  let grandTotal = 0;
  resultsList.forEach(item => {
    grandTotal += item.final;
    container.innerHTML += `
      <div class="result-item">
        النوع: ${item.sub}<br>
        المقاس: ${item.h} × ${item.w}<br>
        الكمية: ${item.quantity}<br>
        🚚 الشحن: ${item.shipping.toFixed(2)}<br>
        💵 السعر الكلي: ${item.final.toFixed(2)}
      </div>
    `;
  });

  const commission = grandTotal * 0.04;
  const finalTotal = grandTotal + commission;

  container.innerHTML += `
    <div class="summary">
      ✅ المجموع الكلي: ${grandTotal.toFixed(2)}<br>
      💼 عمولة المكتب (4%): ${commission.toFixed(2)}<br>
      💰 الناتج النهائي: ${finalTotal.toFixed(2)}<br><br>
      <span style="color:#ccc">* الأسعار شاملة الشحن<br>** الأسعار لا تشمل التركيب</span>
    </div>
  `;
}

function clearResults() {
  resultsList = [];
  document.getElementById("results").innerHTML = "";
}

function saveAsWord() {
  const content = document.getElementById("results").innerHTML;
  const blob = new Blob(['\ufeff', content], {
    type: 'application/msword'
  });
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");
  link.href = url;
  link.download = "النتائج.doc";
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>

</body>
</html>
