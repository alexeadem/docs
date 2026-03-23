---
title: Units Calculator
---

## Unit Calculator

{% raw %}

<div>
  <style>
    table { border-collapse: collapse; width: 100%; margin-bottom: 1em; }
    th, td { padding: 0.5em; border: 1px solid #ccc; text-align: left;}
    select, input { width: 80px; border-radius: 5px;padding: 5px; }
    tfoot td { font-weight: bold; white-space: nowrap; }
    .export-btn, .discount-btn, .gpu-btn {
      padding: 10px 16px;
      border-radius: 5px;
      background: #4783ef;
      color: white;
      border: 0;
      transition: background 0.2s ease-in-out;
      cursor: pointer;
    }
    #calc-body > tr:nth-child(even) {
      background: rgba(255, 255, 0, 0.1);
    }
    .export-btn.open-sheet {
      background: white;
      color: #4783ef;
      border: 1px solid #4783ef;
    }
    .export-btn.generating {
      background: #ccc;
      cursor: wait;
    }
    .article-content table td {
      padding: 10px;
    }
    .article-content table th {
      padding: 10px;
      width: auto;
    }
    #customer_email {
      box-sizing: border-box;
      width: 100%;
      padding: 5px;
    }
    .email-wrapper {
      position: relative;
      display: inline-block;
      width: 100%;
    }
    #email_table {
      height: 80px;
    }
    #export-btn:disabled {
      background-color: #ccc;
      color: #666;
      cursor: not-allowed;
      border: 1px solid #aaa;
      opacity: 1;
    }
    #email-error {
      position: absolute;
      top: 100%;
      left: 0;
      color: red;
      font-size: 0.8em;
      white-space: nowrap;
      pointer-events: none;
      line-height: 1.5;
      z-index: 10;
    }
    #env_select {
      width: 200px;
    }
    .discount-controls, .gpu-controls {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .discount-controls span {
      min-width: 40px;
      display: inline-block;
    }
    .gpu_model {
      width:
       200px;
    }
    .unit-select {
      font-size: 0.95em;
      height: 20px;
      padding: 0px;
      width: 140px;
      /* border-radius: 4px; */
      border: 0px;
      background: transparent;
      color: #0047AB;
      font-weight: bold;
      text-align-last: right;
      direction: rtl;
      
    }
    .unit-select:focus {
      outline: 0px
    }
    .summary-table {
      width: 100%;
      border-collapse: collapse;
      table-layout: fixed;
      margin-bottom: 1em;
    }

    .summary-logo-cell {
      border: 1px solid #ccc;
      padding: 0.5em;
      width: 150px;
      text-align: center;
    }

    .summary-logo-cell img {
      max-width: 60px;
      max-height: 60px;
      display: block;
      margin: 0 auto;
    }
    .summary-display-cell {
      position: relative;
      height: 100px;
      border: 1px solid #ccc;
      width: 100%;
    }

    .summary-rate-line {
      position: absolute;
      top: 0.25em;
      left: 0.5em;
      font-size: 0.75em;
      color: gray;
      font-family: 'Courier New', monospace;
    }

    .summary-display {
      position: absolute;
      bottom: 0.5em;
      right: 0.5em;
      font-family: 'Courier New', monospace;
      color: #0047AB;
      text-shadow: 0 0 1px #0047AB;
      text-align: right;
    }

    .summary-total {
      font-size: 4em;
      font-weight: bold;
      line-height: 0.8;
    }

    .calculator-units {

      text-align: right;
    }

    .gpu_units {

      text-align: right;
    }

    .gpu_hourly {

      text-align: right;
    }

    @font-face {
      font-family: 'Digital7Mono';
      src: url('/fonts/digital-7-mono.ttf') format('truetype');
      font-weight: normal;
      font-style: normal;
    }
    #big_total_units {
      font-family: 'Digital7Mono', monospace;
      font-size: 1.1em;
      letter-spacing: 5px;
      color: #0047AB;
      font-weight: normal;
    }

  </style>
  <script src="https://accounts.google.com/gsi/client" async defer></script>
  <script src="https://apis.google.com/js/api.js"></script>
</div>
  <div>

  <table class="summary-table">
    <tr>
      <td class="summary-logo-cell">
        <!-- <img src="/logo.svg" alt="QBO Logo" style="max-width: 60px; max-height: 60px; display: block; margin: 0 auto;"> -->
        <img src="/logo.svg" alt="QBO Logo" class="summary-logo-img">
      </td>
      <td class="summary-display-cell">

        <!-- <div class="summary-rate-line">
          1 QBO Unit = <span id="usd_per_unit_top">1.00 USD</span>
        </div> -->

        <div class="summary-display">
          <div class="summary-total">
            <span id="big_total_units">0.00</span>
          </div>
          <select id="unit_display_select" class="unit-select" onchange="updateUnitDisplay()">
            <option value="year" selected>units/year</option>
            <option value="month">units/month</option>
            <option value="hour">units/hour</option>
          </select>
        </div>
      </td>
    </tr>

  </table>


  <table>
    <thead>
      <tr>
        <th>Resource Type</th>
        <th>Quantity</th>
        <th>Hourly Reference</th>
        <th>Monthly Reference</th>
      </tr>
    </thead>
    <tbody id="calc-body">
      <tr>
        <td>CPU Core</td>
        <td><input type="number" value="1" id="cpu_qty" min="0"></td>
        <td class="calculator-units" id="cpu_hourly"></td>
        <td class="calculator-units" id="cpu_units"></td>
      </tr>
  
      <!-- GPU rows will be inserted here dynamically -->
      <!-- We use a dummy anchor row to insert before RAM -->
      <tr id="gpu-anchor"></tr>
  
      <tr>
        <td>RAM (GB)</td>
        <td><input type="number" value="0" id="ram_qty" min="0" onchange="calculate()"></td>
        <td class="calculator-units" id="ram_hourly"></td>
        <td class="calculator-units" id="ram_units"></td>
      </tr>
      <tr>
        <td>Disk (GB)</td>
        <td><input type="number" value="0" id="disk_qty" min="0" onchange="calculate()"></td>
        <td class="calculator-units" id="disk_hourly"></td>
        <td class="calculator-units" id="disk_units"></td>
      </tr>
      <tr>
        <td>Network (TB)</td>
        <td><input type="number" value="0" id="net_qty" min="0" onchange="calculate()"></td>
        <td class="calculator-units" id="net_hourly"></td>
        <td class="calculator-units" id="net_units"></td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td id="total_label" colspan="2">Total</td>
        <td class="calculator-units" id="total_units_per_hour"></td>
        <td class="calculator-units" id="total_units_per_month"></td>
      </tr>
    </tfoot>
  </table>
  

  <script>

    const cpuRate = 0.020;
    const ramRate = 0.005;
    const diskRate = 0.000005;
    const netRate = 0.05;
    const minSpend = 25000;


    const gpuRates = {
      "GB300": 4.00,
      "GB200": 5.50,
      "B200": 5.45,
      "H200": 3.40,
      "A2000": 0.45,
      "A100 80GB": 1.70,
      "A100 40GB": 1.06,
      "H100": 2.50,
      "L40S": 1.30,
      "L4": 1.25,
      "RTX A6000": 1.75,
      "RTX 6000 Ada": 2.25,
      "RTX A5000": 1.50,
      "RTX 3090": 1.20,
      "RTX 3080": 1.00,
      "RTX 3070": 0.80,
      "T4": 0.60,
      "V100 32GB": 1.10,
      "V100 16GB": 0.85,
      "P100": 1.00,
      "P40": 0.75,
      "A2": 0.40,
      "K80": 0.20,
      "Jetson Nano": 0.08,
      "Jetson TX1": 0.10,
      "Jetson TX2": 0.12,
      "Jetson Xavier NX": 0.25,
      "Jetson AGX Xavier": 0.40,
      "Jetson Orin NX": 0.60,
      "Jetson Orin AGX": 0.80
    };


    const jetsonCpuCores = {
      "Jetson Nano": 4, "Jetson TX1": 4, "Jetson TX2": 6, "Jetson Xavier NX": 6,
      "Jetson AGX Xavier": 8, "Jetson Orin NX": 8, "Jetson Orin AGX": 12
    };


    

    function toggleFields() {
      ["ram_qty", "disk_qty", "net_qty"].forEach(id => {
        document.getElementById(id).disabled = false; // Always enabled
      });
      calculate();
    }

    function addGpuRow(model = "A100 80GB", qty = 0) {
      const anchor = document.getElementById("gpu-anchor");
      const row = document.createElement("tr");

      row.innerHTML = `
        <td>
          <div class="gpu-controls">
            GPU
            <select class="gpu_model">
              ${Object.keys(gpuRates).map(g => `<option value="${g}" ${g === model ? "selected" : ""}>${g}</option>`).join('')}
            </select>
            <button class="gpu-btn gpu-minus">−</button>
            <button class="gpu-btn gpu-plus">+</button>
          </div>
        </td>
        <td><input type="number" class="gpu_qty" value="${qty}" min="0"></td>
        <td class="gpu_hourly"></td>
        <td class="gpu_units"></td>
      `;

      // Insert before the anchor row in the main tbody
      anchor.insertAdjacentElement("beforebegin", row);

      attachGpuHandlers();
      updateMinusButtons();
      calculate();
    }

    function attachGpuHandlers() {
      document.querySelectorAll('.gpu_model, .gpu_qty').forEach(el => el.onchange = calculate);
      document.querySelectorAll('.gpu-plus').forEach(btn => btn.onclick = () => addGpuRow());
      document.querySelectorAll('.gpu-minus').forEach(btn => btn.onclick = (e) => {
        if (document.querySelectorAll('.gpu-minus').length > 1) {
          e.target.closest('tr').remove();
          calculate();
        }
        updateMinusButtons();
      });
    }

    function updateMinusButtons() {
      document.querySelectorAll('.gpu-minus').forEach((btn, i, arr) => {
        btn.disabled = arr.length <= 1;
      });
    }

    function applyZebraStripes() {
      const rows = Array.from(document.querySelectorAll("#calc-body > tr"))
        .filter(row => !row.id || row.id !== "gpu-anchor");

      rows.forEach((row, i) => {
        row.style.backgroundColor = i % 2 === 0 ? "rgba(255, 255, 0, 0.1)" : "transparent";
      });
    }

    function calculate() {
      const envMultiplier = 1;
      const discount = 0;
      const multiplier = envMultiplier;
      const usdDiscountFactor = 1 - discount / 100;
      const hours = 730;

      const cpuField = document.getElementById("cpu_qty");
      let cpuQty = parseFloat(cpuField.value);
      let gpuTotal = 0;
      let gpuTotalHourly = 0;
      let cpuOverridden = false;

      // FIX: Select GPU rows based on class inside #calc-body
      const gpuRows = Array.from(document.querySelectorAll("#calc-body tr")).filter(
        row => row.querySelector(".gpu-controls")
      );

      gpuRows.forEach(row => {
        const model = row.querySelector(".gpu_model").value;
        const qty = parseFloat(row.querySelector(".gpu_qty").value);
        const rate = gpuRates[model];
        const rateAdj = rate * multiplier;
        const hourly = qty * rateAdj;
        const monthly = hourly * hours;

        row.querySelector(".gpu_hourly").textContent = hourly.toFixed(4);
        row.querySelector(".gpu_units").textContent = Math.round(monthly);

        gpuTotal += monthly;
        gpuTotalHourly += hourly;

        if (jetsonCpuCores[model] && qty > 0) {
          cpuQty = jetsonCpuCores[model];
          cpuField.value = cpuQty;
          cpuField.readOnly = true;
          cpuOverridden = true;
        }
      });

      if (!cpuOverridden) {
        cpuField.readOnly = false;
      }

      const ramQty = parseFloat(document.getElementById("ram_qty").value);
      const diskQty = parseFloat(document.getElementById("disk_qty").value);
      const netQty = parseFloat(document.getElementById("net_qty").value);

      const cpuMonthly = cpuQty * cpuRate * multiplier * hours;
      const ramMonthly = ramQty * ramRate * multiplier * hours;
      const diskMonthly = diskQty * diskRate * multiplier * hours;
      const netMonthly = netQty * netRate * multiplier * hours;

      const cpuHourly = cpuQty * cpuRate * multiplier;
      const ramHourly = ramQty * ramRate * multiplier;
      const diskHourly = diskQty * diskRate * multiplier;
      const netHourly = netQty * netRate * multiplier;

      const totalUnits = cpuMonthly + gpuTotal + ramMonthly + diskMonthly + netMonthly;
      const totalUnitsHourly = cpuHourly + gpuTotalHourly + ramHourly + diskHourly + netHourly;

      const totalUSD = totalUnits * usdDiscountFactor;

      document.getElementById("cpu_hourly").textContent = cpuHourly.toFixed(4);
      document.getElementById("cpu_units").textContent = Math.round(cpuMonthly);

      document.getElementById("ram_hourly").textContent = ramHourly.toFixed(4);
      document.getElementById("ram_units").textContent = Math.round(ramMonthly);

      document.getElementById("disk_hourly").textContent = diskHourly.toFixed(4);
      document.getElementById("disk_units").textContent = Math.round(diskMonthly);

      document.getElementById("net_hourly").textContent = netHourly.toFixed(4);
      document.getElementById("net_units").textContent = Math.round(netMonthly);

      document.getElementById("total_units_per_month").textContent = Math.round(totalUnits);
      document.getElementById("total_units_per_hour").textContent = totalUnitsHourly.toFixed(4);

      const bigUnitsEl = document.getElementById("big_total_units");
      bigUnitsEl.dataset.monthlyUnits = totalUnits;
      updateUnitDisplay();
      applyZebraStripes();
    }


  function updateUnitDisplay() {
    const select = document.getElementById("unit_display_select");
    const displayEl = document.getElementById("big_total_units");
    const monthly = parseFloat(displayEl.dataset.monthlyUnits || 0);

    let converted = monthly;
    let label = select.value;

    if (label === "hour") {
      converted = monthly / 730;
      displayEl.textContent = converted.toFixed(2); // show decimals for hourly
    } else if (label === "year") {
      converted = monthly * 12;
      displayEl.textContent = Math.round(converted);
    } else {
      displayEl.textContent = Math.round(monthly);
    }
  }

  addGpuRow();
  toggleFields();
  calculate();
  document.getElementById("cpu_qty").addEventListener("input", calculate);
  </script>
</div>

{% endraw %}

### What Are QBO Units?

**QBO Units** are a standardized way to represent infrastructure class within QBO. They normalize CPU, GPU, RAM, disk, and network specifications into a single, unified licensing unit, making it easier to plan and scale infrastructure across any hardware or environment.

Think of QBO Units as a normalized hardware profile for your infrastructure. They represent the class of system capacity made available to QBO, not metered consumption over time. This calculator helps answer the core question: **“How many QBO Units apply to this hardware profile?”**

For reference, 1 QBO Unit is generally equivalent to **$1.00 USD**. If you're planning a large deployment or want custom pricing, please [contact our sales team](https://qbo.io/contact-us/) to learn more about volume discounts.

---

### Why Use QBO Units?

- **Simplified Planning:** Classify infrastructure consistently across diverse hardware types.
- **Hardware-Agnostic:** From A100s to Jetsons, all resources are priced and compared the same way.
- **Cross-Environment Compatible:** Use the same unit system in cloud, on-prem, airgapped, or hybrid setups.
- **Not Tied to Utilization:** QBO Units represent a fixed platform license based on the hardware profile. There are no overage charges as long as the hardware configuration remains unchanged.
- **Predictable Accounting:** Because each unit maps to $1.00 USD, teams can easily forecast and track costs.
- **Budget & Quota Friendly:** Assign QBO Units by system class and forecast platform cost consistently across teams and environments.

---

### Using the Calculator

1. **Define Your Hardware Profile**

   - Enter the number of **CPU cores** and select your **GPU models**.
   - Specify the system’s **RAM**, **disk**, and **network** specifications.
   - If a Jetson model is selected, CPU values are auto-filled and locked.

2. **Get Your Estimate**

- View the annualized QBO Unit profile for the selected hardware configuration.
- Use this estimate to classify the system and determine the fixed QBO platform license for that hardware profile.

---

### Annual Minimum for Customer-Provided Metal

When deploying on your own hardware (On-Prem), QBO requires a minimum commitment. This ensures sufficient usage to support onboarding, support, and platform operations across your infrastructure.

This minimum only applies to customer-owned infrastructure. There is no minimum when running workloads in QBO’s cloud environments.

---

**Important:** QBO Units are not tied to real-time usage or bandwidth metering. They represent a fixed annual platform license based on the hardware profile. For full details, read our [QBO Units Terms of Service](terms).
