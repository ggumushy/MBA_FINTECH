<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>MBA FINTECH GST 2.0 Calculator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: linear-gradient(135deg, #FFD700, #FF8C00, #FF4500, #FF1493);
      background-size: 400% 400%;
      animation: gradientBG 15s ease infinite;
      color: #fff;
    }
    @keyframes gradientBG {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }
    h1 { color: #fff; text-align: center; }
    select, input, button { padding: 5px; margin: 5px; border-radius: 5px; border: none; }
    button { cursor: pointer; background: #FFD700; color: #000; font-weight: bold; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; background: #fff; color: #000; border-radius: 10px; overflow: hidden; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    th { background: #FFA500; color: #fff; }
    .down { color: green; font-weight: bold; }
    .up { color: red; font-weight: bold; }
    .summary { margin-top: 20px; font-size: 20px; font-weight: bold; text-align: center; }
    .grand-total { background: #FFD700; font-weight: bold; }
  </style>
</head>
<body>
  <h1>MBA FINTECH GST 2.0 Calculator</h1>

  <label for="product">Select Product:</label>
  <select id="product"></select>

  <label for="quantity">Quantity:</label>
  <input type="number" id="quantity" value="1" min="1">

  <button onclick="addToCart()">Add to Cart</button>
  <button onclick="calculateBill()">Calculate Composite Bill</button>

  <table id="cartTable" style="display:none;">
    <thead>
      <tr>
        <th>Product</th>
        <th>Quantity</th>
        <th>Base Total</th>
        <th>GST Rates (Old / New)</th>
        <th>Total Price (GST 1)</th>
        <th>Total Price (GST 2)</th>
        <th>SAVING</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody id="cartBody"></tbody>
  </table>

  <div id="results" class="summary" style="display:none;"></div>

  <script>
    const products = {
      "Televisions (over 32 inches)": { price: 35000, gst1: 28, gst2: 18 },
      "Air Conditioners (ACs)": { price: 40000, gst1: 28, gst2: 18 },
      "Small Cars": { price: 700000, gst1: 28, gst2: 18 },
      "Two-wheelers": { price: 85000, gst1: 28, gst2: 18 },
      "Life Insurance Premiums": { price: 20000, gst1: 18, gst2: 0 },
      "Soaps & Shampoos": { price: 150, gst1: 18, gst2: 5 },
      "Packaged Snacks": { price: 50, gst1: 18, gst2: 5 },
      "Chocolates": { price: 80, gst1: 18, gst2: 5 },
      "Cement": { price: 350, gst1: 28, gst2: 18 },
      "Agricultural Machinery": { price: 500000, gst1: 12, gst2: 5 },
      "Marble and Granite": { price: 200, gst1: 12, gst2: 5 },
      "Life-Saving Drugs": { price: 1500, gst1: 12, gst2: 5 },
      "Garments & Apparel": { price: 1000, gst1: 12, gst2: 5 },
      "Carbonated & Sugary Drinks": { price: 80, gst1: 28, gst2: 40 },
      "Yachts & Private Aircraft": { price: 10000000, gst1: 28, gst2: 40 }
    };

    const cart = [];

    const productSelect = document.getElementById("product");
    Object.keys(products).forEach(name => {
      const option = document.createElement("option");
      option.value = name;
      option.textContent = name;
      productSelect.appendChild(option);
    });

    function formatCurrency(num) {
      return "₹" + num.toLocaleString("en-IN", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    }

    function addToCart() {
      const name = document.getElementById("product").value;
      const qty = parseInt(document.getElementById("quantity").value);
      if (!qty || qty < 1) return;

      const existing = cart.find(item => item.name === name);
      if (existing) {
        existing.qty += qty;
      } else {
        cart.push({ name, qty });
      }
      renderCart();
    }

    function removeItem(index) {
      cart.splice(index, 1);
      renderCart();
    }

    function renderCart() {
      const cartTable = document.getElementById("cartTable");
      const tbody = document.getElementById("cartBody");
      tbody.innerHTML = "";
      if (cart.length === 0) {
        cartTable.style.display = "none";
        return;
      }
      cartTable.style.display = "table";

      let totalQty = 0, grandBase = 0, grandGST1 = 0, grandGST2 = 0;

      cart.forEach((item, index) => {
        const { price, gst1, gst2 } = products[item.name];
        const baseTotal = price * item.qty;
        const totalGST1 = baseTotal * (1 + gst1 / 100);
        const totalGST2 = baseTotal * (1 + gst2 / 100);
        const saving = totalGST1 - totalGST2;

        totalQty += item.qty;
        grandBase += baseTotal;
        grandGST1 += totalGST1;
        grandGST2 += totalGST2;

        const row = document.createElement("tr");
        row.innerHTML = `
          <td>${item.name}</td>
          <td>${item.qty}</td>
          <td>${formatCurrency(baseTotal)}</td>
          <td>${gst1}% / ${gst2}%</td>
          <td>${formatCurrency(totalGST1)}</td>
          <td>${formatCurrency(totalGST2)}</td>
          <td class="${saving > 0 ? "down" : saving < 0 ? "up" : ""}">
            ${saving > 0 ? "↓ " + formatCurrency(saving) : saving < 0 ? "↑ " + formatCurrency(Math.abs(saving)) : "No Change"}
          </td>
          <td><button onclick="removeItem(${index})">Remove</button></td>
        `;
        tbody.appendChild(row);
      });

      // Add Grand Total row
      const grandSaving = grandGST1 - grandGST2;
      const grandRow = document.createElement("tr");
      grandRow.className = "grand-total";
      grandRow.innerHTML = `
        <td>GRAND TOTAL</td>
        <td>${totalQty}</td>
        <td>${formatCurrency(grandBase)}</td>
        <td>-</td>
        <td>${formatCurrency(grandGST1)}</td>
        <td>${formatCurrency(grandGST2)}</td>
        <td class="${grandSaving > 0 ? "down" : grandSaving < 0 ? "up" : ""}">
          ${grandSaving > 0 ? "↓ " + formatCurrency(grandSaving) : grandSaving < 0 ? "↑ " + formatCurrency(Math.abs(grandSaving)) : "No Change"}
        </td>
        <td>-</td>
      `;
      tbody.appendChild(grandRow);
    }

    function calculateBill() {
      if (cart.length === 0) {
        alert("Please add at least one product.");
        return;
      }
      renderCart();
    }
  </script>
</body>
</html>
