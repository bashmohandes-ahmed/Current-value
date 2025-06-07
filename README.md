<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8" />
<title>محول العملات</title>
<style>
  body { font-family: Arial, sans-serif; background: #f2f2f2; direction: rtl; text-align: center; margin: 0; padding: 0; }
  .container { background: #fff; border-radius: 10px; padding: 20px; margin: 30px auto; max-width: 600px; box-shadow: 0 0 15px rgba(0,0,0,0.1); }
  input, select, button { width: 100%; margin: 10px 0; padding: 8px; border-radius: 5px; border: 1px solid #ccc; font-size: 16px; box-sizing: border-box; }
  button { background: #4CAF50; color: #fff; border: none; cursor: pointer; font-size: 18px; }
  button:hover { background: #45a049; }
  #result { margin-top: 20px; font-size: 20px; color: #222; text-align: left; direction: ltr; padding-left: 10px; }
  table { width: 100%; border-collapse: collapse; margin-top: 30px; text-align: center; }
  th, td { border: 1px solid #ccc; padding: 8px; }
  th { background: #eee; }
  h3 { margin-top: 30px; margin-bottom: 10px; }
  .currency-name { font-weight: normal; font-size: 14px; color: #555; }
  #oilPrice { margin-top: 20px; font-size: 18px; color: #333; font-weight: bold; }
  .last-update { font-size: 12px; color: #666; margin-top: 5px; }
  .loading { color: #999; font-style: italic; }
  .gold-24 { color: #d4af37; font-weight: bold; }
  .gold-21 { color: #c9b037; }
  .gold-18 { color: #b4a76c; }
  .currency-code { font-weight: bold; }
</style>
</head>
<body>
<div class="container">
  <h1>محول العملات</h1>
  <input type="number" id="amount" placeholder="أدخل المبلغ" />
  <select id="from"></select>
  <select id="to"></select>
  <button onclick="convert()">تحويل</button>
  <div id="result"></div>

  <h3>أسعار العملات مقابل الدولار الأمريكي والجنيه المصري</h3>
  <table id="ratesTable">
    <thead>
      <tr>
        <th>العملة</th>
        <th>السعر مقابل 1 دولار</th>
        <th>السعر مقابل 1 جنيه مصري</th>
      </tr>
    </thead>
    <tbody><tr><td colspan="3" class="loading">جاري تحميل بيانات العملات...</td></tr></tbody>
  </table>

  <h3>أسعار الذهب اليوم</h3>
  <table id="goldTable">
    <thead>
      <tr><th>العيار</th><th>السعر بالجرام (دولار)</th><th>السعر بالجرام (جنيه مصري)</th><th>السعر بالأوقية (دولار)</th></tr>
    </thead>
    <tbody>
      <tr><td>عيار 24</td><td id="gold24USD" class="loading gold-24">جاري التحميل...</td><td id="gold24EGP" class="loading gold-24">جاري التحميل...</td><td id="gold24Ounce" class="loading gold-24">جاري التحميل...</td></tr>
      <tr><td>عيار 21</td><td id="gold21USD" class="loading gold-21">جاري التحميل...</td><td id="gold21EGP" class="loading gold-21">جاري التحميل...</td><td id="gold21Ounce" class="loading gold-21">جاري التحميل...</td></tr>
      <tr><td>عيار 18</td><td id="gold18USD" class="loading gold-18">جاري التحميل...</td><td id="gold18EGP" class="loading gold-18">جاري التحميل...</td><td id="gold18Ounce" class="loading gold-18">جاري التحميل...</td></tr>
    </tbody>
  </table>

  <h3>أسعار المعادن الثمينة</h3>
  <table id="metalsTable">
    <thead>
      <tr><th>المعدن</th><th>السعر بالجرام (بالدولار)</th><th>السعر بالجرام (بالجنيه المصري)</th></tr>
    </thead>
    <tbody>
      <tr><td>الماس</td><td id="diamondUSD">$1000.00</td><td id="diamondEGP" class="loading">جاري التحميل...</td></tr>
      <tr><td>السولتير</td><td id="solitaireUSD">$1500.00</td><td id="solitaireEGP" class="loading">جاري التحميل...</td></tr>
    </tbody>
  </table>

  <div id="oilPrice" class="loading">جاري تحميل سعر برميل البترول...</div>
  <div id="oilUpdateTime" class="last-update"></div>
</div>

<script>
const exchangeApiKey = "c26c0ecc0516e337b7f572af";
const oilApiKey = "O8KQ7FZ5W9X1T3R6";
const goldApiKey = "goldapi-pv9smbmfouca-io";

let rates = {};
const currencyList = {
  USD:"دولار أمريكي", EUR:"يورو", AED:"درهم إماراتي", CAD:"دولار كندي", KWD:"دينار كويتي",
  JPY:"ين ياباني", CHF:"فرنك سويسري", GBP:"جنيه إسترليني", EGP:"جنيه مصري", OMR:"ريال عماني",
  BHD:"دينار بحريني", SAR:"ريال سعودي", QAR:"ريال قطري", ARS:"بيزو أرجنتيني", BRL:"ريال برازيلي",
  CLP:"بيزو تشيلي", CNY:"يوان صيني", THB:"بات تايلاندي", INR:"روبية هندية", AUD:"دولار أسترالي"
};

function populateSelect(id){
  const sel = document.getElementById(id);
  sel.innerHTML = "";
  for(const code in currencyList){
    const opt = document.createElement('option');
    opt.value = code;
    opt.textContent = `${code} - ${currencyList[code]}`;
    sel.appendChild(opt);
  }
}

async function fetchRates(){
  try {
    const res = await fetch(`https://v6.exchangerate-api.com/v6/${exchangeApiKey}/latest/USD`);
    const data = await res.json();
    
    if(data.result === "success"){
      rates = data.conversion_rates;
      fillRatesTable();
      updateMetalsPrices();
      fetchOilPrice();
      fetchGoldPrices();
      setTimeout(fetchRates, 3600000);
    } else {
      throw new Error(data["error-type"]);
    }
  } catch(e){
    console.error("فشل تحميل أسعار العملات:", e);
    document.querySelector("#ratesTable tbody").innerHTML = `
      <tr><td colspan="3" style="color:red;">فشل تحميل بيانات العملات. يرجى المحاولة لاحقاً.</td></tr>
    `;
    setTimeout(fetchRates, 120000);
  }
}

function fillRatesTable(){
  const tbody = document.querySelector("#ratesTable tbody");
  tbody.innerHTML = "";
  
  for(const code in currencyList){

    if(code === "USD") continue;
    
    const usdPrice = rates[code] ? rates[code].toFixed(4) : "-";
    const egpPrice = (rates[code] && rates["EGP"]) ? (rates[code]/rates["EGP"]).toFixed(4) : "-";
    tbody.innerHTML += `
      <tr>
        <td><span class="currency-code">${code}</span> - ${currencyList[code]}</td>
        <td>${usdPrice}</td>
        <td>${egpPrice}</td>
      </tr>`;
  }

  populateSelect("from");
  populateSelect("to");
  document.getElementById("from").value = "USD";
  document.getElementById("to").value = "EGP";
}

async function fetchGoldPrices(){
  try {
    const response = await fetch('https://www.goldapi.io/api/XAU/USD', {
      headers: {
        'x-access-token': goldApiKey
      }
    });
    const data = await response.json();

    if(!data.price){
      throw new Error("بيانات سعر الذهب غير متوفرة");
    }

    const goldPricePerOunce = data.price;
    const gramsPerOunce = 31.1035;
    const gold24PerGramUSD = goldPricePerOunce / gramsPerOunce;
    const gold21PerGramUSD = gold24PerGramUSD * 0.875;
    const gold18PerGramUSD = gold24PerGramUSD * 0.75;
    
    const egpRate = rates["EGP"] || 30;
    
    document.getElementById("gold24USD").textContent = `$${gold24PerGramUSD.toFixed(2)}`;
    document.getElementById("gold24EGP").textContent = `${(gold24PerGramUSD * egpRate).toFixed(2)} ج.م`;
    document.getElementById("gold24Ounce").textContent = `$${goldPricePerOunce.toFixed(2)}`;
    
    document.getElementById("gold21USD").textContent = `$${gold21PerGramUSD.toFixed(2)}`;
    document.getElementById("gold21EGP").textContent = `${(gold21PerGramUSD * egpRate).toFixed(2)} ج.م`;
    document.getElementById("gold21Ounce").textContent = `$${(goldPricePerOunce * 0.875).toFixed(2)}`;
    
    document.getElementById("gold18USD").textContent = `$${gold18PerGramUSD.toFixed(2)}`;
    document.getElementById("gold18EGP").textContent = `${(gold18PerGramUSD * egpRate).toFixed(2)} ج.م`;
    document.getElementById("gold18Ounce").textContent = `$${(goldPricePerOunce * 0.75).toFixed(2)}`;

  } catch(e) {
    console.error("فشل تحميل أسعار الذهب:", e);
    document.querySelector("#goldTable tbody").innerHTML = `
      <tr><td colspan="4" style="color:red;">فشل تحميل أسعار الذهب. يرجى المحاولة لاحقاً.</td></tr>
    `;
  }
}

function convert(){
  const from = document.getElementById("from").value;
  const to = document.getElementById("to").value;
  const amount = parseFloat(document.getElementById("amount").value);
  if(isNaN(amount) || amount <= 0){
    alert("من فضلك أدخل مبلغ صالح للتحويل.");
    return;
  }

  if(!rates[from] || !rates[to]){
    alert("بيانات العملة غير متوفرة حالياً.");
    return;
  }

  const amountInUSD = amount / rates[from];
  const convertedAmount = amountInUSD * rates[to];

  document.getElementById("result").textContent = `${amount} ${from} = ${convertedAmount.toFixed(4)} ${to}`;
}

function updateMetalsPrices(){
  const egpRate = rates["EGP"] || 30;
  document.getElementById("diamondEGP").textContent = (1000 * egpRate).toFixed(2) + " ج.م";
  document.getElementById("solitaireEGP").textContent = (1500 * egpRate).toFixed(2) + " ج.م";
}

async function fetchOilPrice(){
  try {
    const res = await fetch(`https://www.alphavantage.co/query?function=WTI&apikey=${oilApiKey}`);
    const data = await res.json();

    if(!data["Global Quote"] || !data["Global Quote"]["05. price"]){
      throw new Error("خطأ في بيانات النفط");
    }

    const latestPrice = parseFloat(data["Global Quote"]["05. price"]);
    const now = new Date();
    const options = { 
      weekday: 'long', 
      year: 'numeric', 
      month: 'long', 
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
      hour12: true
    };
    const updateTime = now.toLocaleDateString('ar-EG', options);

    document.getElementById("oilPrice").textContent = `سعر برميل البترول: $${latestPrice.toFixed(2)}`;
    document.getElementById("oilUpdateTime").textContent = `آخر تحديث: ${updateTime}`;

  } catch(e) {
    console.error("فشل تحميل سعر النفط:", e);
    document.getElementById("oilPrice").textContent = "فشل تحميل سعر برميل البترول. يرجى المحاولة لاحقاً.";
  }
}

window.onload = () => {
  fetchRates();
};
</script>
</body>
</html>
