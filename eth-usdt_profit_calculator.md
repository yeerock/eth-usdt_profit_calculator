# ETH Profit Calculator

**ETH Profit Calculator** is a responsive web app that calculates potential profit from Ethereum (ETH) trading against USDT, using either real-time Binance prices or manual price inputs.  
Users can toggle between live and manual pricing for both buying and selling ETH, input available USDT, and specify buy/sell fee percentages.  
The calculator dynamically fetches live BTC/USDT rates every 100ms when enabled, auto-updates calculations instantly, and displays detailed breakdowns including:

- Buy fees  
- Net ETH acquired  
- Sell fees  
- Final profit (in USDT and %)  

The app features a clean, iOS-native inspired UI optimized for both mobile and desktop usage. It also includes a real-time line chart plotting "Net Profit (USDT) vs. Time" for the last 5 minutes, with a live ETH price display that has a bouncing animation on change.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ETH Profit Calculator</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-title" content="ETH Profit Calculator">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-adapter-date-fns/dist/chartjs-adapter-date-fns.bundle.min.js"></script>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", sans-serif;
      margin: 0; padding: 20px;
      background: #f9f9f9;
      color: #333;
    }
    h1 {
      font-size: 24px;
      margin-bottom: 20px;
      text-align: center;
      font-weight: 600;
    }
    .block {
      background: #fff;
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.05);
      margin-bottom: 20px;
    }
    .input-group {
      position: relative;
      margin-bottom: 15px;
    }
    label {
      font-size: 14px;
      font-weight: 500;
      margin-bottom: 5px;
      display: block;
    }
    input[type="number"] {
      width: 100%;
      font-size: 18px;
      padding: 10px;
      box-sizing: border-box;
      border: 1px solid #ccc;
      border-radius: 8px;
    }
    .switch-group {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 15px;
    }
    .switch {
      position: relative;
      width: 50px;
      height: 24px;
    }
    .switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }
    .slider {
      position: absolute;
      cursor: pointer;
      top: 0; left: 0; right: 0; bottom: 0;
      background-color: #ccc;
      border-radius: 24px;
      transition: .4s;
    }
    .slider:before {
      position: absolute;
      content: "";
      height: 18px;
      width: 18px;
      left: 3px;
      bottom: 3px;
      background-color: white;
      border-radius: 50%;
      transition: .4s;
    }
    input:checked + .slider {
      background-color: #4CAF50;
    }
    input:checked + .slider:before {
      transform: translateX(26px);
    }
    .output .row {
      display: flex;
      justify-content: space-between;
      margin-bottom: 10px;
    }
    .label {
      width: 50%;
      text-align: right;
      padding-right: 10px;
      color: #777;
    }
    .value {
      width: 50%;
      font-weight: 600;
      text-align: left;
    }
    .note {
      text-align: center;
      color: #888;
      font-size: 14px;
      margin-top: 20px;
    }
    #chartContainer {
        height: 300px;
        margin-top: 20px;
        position: relative; /* For positioning the live price display */
    }
    #livePriceDisplayContainer {
        position: absolute;
        top: 10px; /* Adjust as needed */
        right: 10px; /* Adjust as needed */
        font-size: 13px;
        font-weight: bold;
        color: #555; /* Label color */
        z-index: 10; /* To be above the chart canvas */
        background-color: rgba(255, 255, 255, 0.8); /* Optional: slight background for readability */
        padding: 3px 6px;
        border-radius: 4px;
    }
    #livePriceValue {
        color: #627eea; /* ETH Blue for the value */
    }
    .price-bounce {
        animation: priceBounceAnimation 0.3s ease-out;
        display: inline-block; /* Important for transform to work correctly */
    }
    @keyframes priceBounceAnimation {
        0% { transform: scale(1); }
        50% { transform: scale(1.25); }
        100% { transform: scale(1); }
    }
  </style>
</head>
<body>

<h1>ETH Profit Calculator</h1>

<div class="block">
  <div class="switch-group">
    <label>Use Real-Time Price For BUYING</label>
    <label class="switch">
      <input type="checkbox" id="useRealTimeBuy"> <!-- Default unchecked -->
      <span class="slider"></span>
    </label>
  </div>
  <div class="input-group">
    <label>ETH/USDT Price @ BUYING</label>
    <input type="number" id="manualETHPriceBuy" value="3500" step="0.01">
  </div>

  <div class="switch-group">
    <label>Use Real-Time Price For SELLING</label>
    <label class="switch">
      <input type="checkbox" id="useRealTimeSell" checked> <!-- Default checked -->
      <span class="slider"></span>
    </label>
  </div>
  <div class="input-group">
    <label>ETH/USDT Price @ SELLING</label>
    <input type="number" id="manualETHPriceSell" value="3500" step="0.01" disabled>
  </div>

  <div class="input-group">
    <label>USDT On Hand</label>
    <input type="number" id="availableUSDT" value="15440.95427" step="0.00001">
  </div>

  <div class="input-group">
    <label>Buy Fee (%)</label>
    <input type="number" id="buyFeePercent" value="0.075" step="0.001">
  </div>

  <div class="input-group">
    <label>Sell Fee (%)</label>
    <input type="number" id="sellFeePercent" value="0.075" step="0.001">
  </div>
</div>

<div class="block output" id="results">
  Loading...
</div>

<div class="note">
  Price data updated every 100ms from Binance. Chart updates every 100ms. (High Frequency)
</div>

<div class="block" id="chartContainer">
  <div id="livePriceDisplayContainer">
      <span id="livePriceLabel">Live ETH: </span>
      <span id="livePriceValue">--.--</span>
  </div>
  <canvas id="profitChart"></canvas>
</div>

<script>
let ethPrice = null; // Changed from btcPrice
let profitChart;
let chartDataPoints = [];
let latestProfitUSDTForChart = 0;
let initialPriceFetchedAndSet = false; 

const MAX_CHART_DATAPOINTS = 3000; 
const CHART_UPDATE_INTERVAL = 100;  
const PRICE_FETCH_INTERVAL = 100;   

let previousEthPriceForAnimation = null; // Changed for ETH

async function fetchETHPrice() { // Renamed function
  try {
    const res = await fetch('https://api.binance.com/api/v3/ticker/price?symbol=ETHUSDT'); // API for ETHUSDT
    if (!res.ok) throw new Error('API Error: ' + res.status);
    const data = await res.json();
    const newEthPrice = parseFloat(data.price); // Changed to newEthPrice

    const livePriceValueEl = document.getElementById('livePriceValue');

    if (newEthPrice !== null && newEthPrice !== previousEthPriceForAnimation) {
        if (livePriceValueEl) { 
            livePriceValueEl.classList.remove('price-bounce');
            void livePriceValueEl.offsetWidth; 
            livePriceValueEl.classList.add('price-bounce');
        }
    }
    if (newEthPrice !== null) {
        previousEthPriceForAnimation = newEthPrice; // Changed for ETH
    }

    ethPrice = newEthPrice; // Update global ethPrice

    if (livePriceValueEl) {
        if (ethPrice !== null) {
            livePriceValueEl.textContent = `$${ethPrice.toFixed(2)}`;
        } else {
            livePriceValueEl.textContent = '--.--'; 
        }
    }

    if (!initialPriceFetchedAndSet && ethPrice !== null) {
        document.getElementById('manualETHPriceBuy').value = ethPrice.toFixed(2); // ID for ETH
        document.getElementById('manualETHPriceSell').value = ethPrice.toFixed(2); // ID for ETH
        initialPriceFetchedAndSet = true;
    }
    
    calculate(); 
  } catch (error) {
    console.error("Error fetching ETH price:", error); // Error message for ETH
    document.getElementById('results').innerHTML = "⚠️ Cannot fetch ETH price."; // Error message for ETH
    const livePriceValueEl = document.getElementById('livePriceValue');
    if (livePriceValueEl) livePriceValueEl.textContent = 'Error';
  }
}

function row(label, value) {
  return `<div class="row"><div class="label">${label}</div><div class="value">${value}</div></div>`;
}

function getCurrentETHPrice(isBuy) { // Renamed function
  const manualPriceEl = isBuy ? document.getElementById('manualETHPriceBuy') : document.getElementById('manualETHPriceSell'); // IDs for ETH
  const useRealTimeEl = isBuy ? document.getElementById('useRealTimeBuy') : document.getElementById('useRealTimeSell');

  if (useRealTimeEl.checked && ethPrice !== null) { // Use global ethPrice
    return ethPrice;
  }
  return parseFloat(manualPriceEl.value) || 0;
}

function calculate() {
  const availableUSDT = parseFloat(document.getElementById('availableUSDT').value) || 0;
  const buyFeePercent = parseFloat(document.getElementById('buyFeePercent').value) || 0;
  const sellFeePercent = parseFloat(document.getElementById('sellFeePercent').value) || 0;
  
  if (document.getElementById('useRealTimeBuy').checked && ethPrice !== null) { // Use global ethPrice
      document.getElementById('manualETHPriceBuy').value = ethPrice.toFixed(2); // ID for ETH
  }
  if (document.getElementById('useRealTimeSell').checked && ethPrice !== null) { // Use global ethPrice
      document.getElementById('manualETHPriceSell').value = ethPrice.toFixed(2); // ID for ETH
  }

  const currentBuyETH = getCurrentETHPrice(true); // Changed to ETH
  const currentSellETH = getCurrentETHPrice(false); // Changed to ETH

  let waitingMessage = "";
  if ((document.getElementById('useRealTimeBuy').checked || document.getElementById('useRealTimeSell').checked) && ethPrice === null && !initialPriceFetchedAndSet) { // Use global ethPrice
      waitingMessage = "Waiting ETH Price..."; // Message for ETH
  }
  
  if (waitingMessage) {
      document.getElementById('results').innerHTML = waitingMessage;
      if ((document.getElementById('useRealTimeBuy').checked && currentBuyETH === 0) || // currentBuyETH
          (document.getElementById('useRealTimeSell').checked && currentSellETH === 0)) { // currentSellETH
          latestProfitUSDTForChart = 0; 
          return; 
      }
  }

  const buyFeeUSDT = availableUSDT * (buyFeePercent / 100);
  const buyFeeETH = currentBuyETH > 0 ? buyFeeUSDT / currentBuyETH : 0; // buyFeeETH, currentBuyETH
  const usdtAfterBuyFee = availableUSDT - buyFeeUSDT;
  const maxBoughtETH = currentBuyETH > 0 ? usdtAfterBuyFee / currentBuyETH : 0; // maxBoughtETH, currentBuyETH
  
  const ethOnHandSell = maxBoughtETH * currentSellETH; // ethOnHandSell, maxBoughtETH, currentSellETH
  const sellFeeUSDT = ethOnHandSell * (sellFeePercent / 100);
  const sellFeeETH = currentSellETH > 0 ? sellFeeUSDT / currentSellETH : 0; // sellFeeETH, currentSellETH
  const finalUSDT = ethOnHandSell - sellFeeUSDT;
  
  let profitUSDT = 0;
  let profitPercent = 0;

  if (availableUSDT > 0) {
      profitUSDT = finalUSDT - availableUSDT;
      profitPercent = (profitUSDT / availableUSDT) * 100;
  } else if (finalUSDT > 0) { 
      profitUSDT = finalUSDT;
      profitPercent = Infinity; 
  }

  latestProfitUSDTForChart = profitUSDT;

  document.getElementById('results').innerHTML = `
    ${row('ETH/USDT Price @ BUYING', currentBuyETH.toFixed(2) + ' USDT')} 
    ${row('USDT On Hand', availableUSDT.toFixed(8) + ' USDT')}
    ${row('Buy Fee', `${buyFeePercent}% / ${buyFeeUSDT.toFixed(8)} USDT / Ξ${buyFeeETH.toFixed(8)}`)} 
    ${row('Net ETH Acquired (After Deducting Buy Fee)', `Ξ${maxBoughtETH.toFixed(10)}`)} 
    <br>
    ${row('ETH/USDT Price @ SELLING', currentSellETH.toFixed(2) + ' USDT')} 
    ${row('Sell Fee', `${sellFeePercent}% / ${sellFeeUSDT.toFixed(8)} USDT / Ξ${sellFeeETH.toFixed(8)}`)} 
    ${row('Net Profit', `${profitPercent.toFixed(8)}% / ${profitUSDT.toFixed(8)} USDT`)}
  `;
}

function initChart() {
  const ctx = document.getElementById('profitChart').getContext('2d');
  profitChart = new Chart(ctx, {
    type: 'line',
    data: {
      datasets: [{
        label: 'Net Profit (USDT)',
        data: chartDataPoints,
        borderColor: 'rgb(98, 126, 234)', // ETH Blue color
        backgroundColor: 'transparent', 
        fill: false,                    
        tension: 0.2,
        pointRadius: 0.5, 
        pointHoverRadius: 3
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {
          type: 'time',
          time: {
            unit: 'second', 
            tooltipFormat: 'HH:mm:ss',
            displayFormats: {
              minute: 'HH:mm',
              second: 'HH:mm:ss'
            }
          },
          title: {
            display: true,
            text: 'Time'
          },
          grid: {
            display: false
          }
        },
        y: {
          title: {
            display: true,
            text: 'Net Profit (USDT)'
          },
          ticks: {
            callback: function(value) {
              return value.toFixed(2);
            }
          }
        }
      },
      plugins: {
        legend: {
          display: true,
          position: 'top',
        },
        title: {
            display: true,
            text: 'Net Profit (USDT) vs. Time (Last 5 Min)',
            font: {
                size: 16
            },
            padding: {
                bottom: 10,
                top:5
            }
        }
      },
      animation: {
        duration: 50, 
      },
      interaction: {
        mode: 'nearest',
        axis: 'x',
        intersect: false
      },
    }
  });
}

function updateChart() {
  if (!profitChart) return;

  const now = Date.now();
  chartDataPoints.push({ x: now, y: latestProfitUSDTForChart });

  while (chartDataPoints.length > MAX_CHART_DATAPOINTS) {
    chartDataPoints.shift(); 
  }

  profitChart.update('none'); 
}

function attachEvents() {
  document.querySelectorAll('input[type="number"]').forEach(input => {
    input.addEventListener('input', calculate);
  });

  const useRealTimeBuyCheckbox = document.getElementById('useRealTimeBuy');
  const useRealTimeSellCheckbox = document.getElementById('useRealTimeSell');
  const manualBuyInput = document.getElementById('manualETHPriceBuy'); // ID for ETH
  const manualSellInput = document.getElementById('manualETHPriceSell'); // ID for ETH

  manualBuyInput.disabled = useRealTimeBuyCheckbox.checked; 
  manualSellInput.disabled = useRealTimeSellCheckbox.checked; 

  useRealTimeBuyCheckbox.addEventListener('change', function() {
    manualBuyInput.disabled = this.checked;
    if (this.checked && ethPrice !== null) { // Use global ethPrice
      manualBuyInput.value = ethPrice.toFixed(2);
    } else if (this.checked && ethPrice === null && !initialPriceFetchedAndSet) { 
        fetchETHPrice(); // Call fetchETHPrice
    }
    calculate();
  });

  useRealTimeSellCheckbox.addEventListener('change', function() {
    manualSellInput.disabled = this.checked;
    if (this.checked && ethPrice !== null) { // Use global ethPrice
      manualSellInput.value = ethPrice.toFixed(2);
    } else if (this.checked && ethPrice === null && !initialPriceFetchedAndSet) {
        fetchETHPrice(); // Call fetchETHPrice
    }
    calculate();
  });

  initChart(); 
  setInterval(updateChart, CHART_UPDATE_INTERVAL); 
}

fetchETHPrice(); // Initial call for ETH
attachEvents();  

setInterval(() => {
    fetchETHPrice(); // Fetch ETH price
}, PRICE_FETCH_INTERVAL);

document.getElementById('manualETHPriceBuy').addEventListener('input', function() { // ID for ETH
    if (!document.getElementById('useRealTimeBuy').checked) calculate();
});
document.getElementById('manualETHPriceSell').addEventListener('input', function() { // ID for ETH
    if (!document.getElementById('useRealTimeSell').checked) calculate();
});

document.getElementById('livePriceValue').addEventListener('animationend', function() {
    this.classList.remove('price-bounce');
});

</script>

</body>
</html>
