<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Workshop 101 - Stocktaking</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    .tab { margin-bottom: 10px; }
    .tab button { padding: 10px; margin-right: 5px; cursor: pointer; }
    .tabcontent { display: none; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
    input, select { margin: 5px 0; padding: 5px; }
  </style>
</head>
<body>

  <h1>Workshop 101 - Stocktaking</h1>

  <div class="tab">
    <button onclick="openTab(event, 'Issue')">Issue</button>
    <button onclick="openTab(event, 'Return')">Return</button>
    <button onclick="openTab(event, 'Records')">Records</button>
    <button onclick="openTab(event, 'Stock')">Stock</button>
  </div>

  <!-- ISSUE TAB -->
  <div id="Issue" class="tabcontent">
    <h2>Issue Items</h2>
    <form id="issueForm">
      <label>Category:
        <select id="issueCategory" required>
          <option value="">Select</option>
          <option>Parts</option>
          <option>Tools</option>
          <option>Drill Bits</option>
          <option>Oils</option>
        </select>
      </label><br>
      <label>Description: <input type="text" id="issueDesc" required></label><br>
      <label>Part Number: <input type="text" id="issuePartNo"></label><br>
      <label>Quantity: <input type="number" id="issueQty" required></label><br>
      <label>DR: <input type="text" id="issueDR" required></label><br>
      <label>Name: <input type="text" id="issueName" required></label><br>
      <label>Date: <input type="date" id="issueDate" required></label><br>
      <button type="submit">Issue</button>
    </form>
  </div>

  <!-- RETURN TAB -->
  <div id="Return" class="tabcontent">
    <h2>Return Items</h2>
    <form id="returnForm">
      <label>Category:
        <select id="returnCategory" required>
          <option value="">Select</option>
          <option>Parts</option>
          <option>Tools</option>
          <option>Drill Bits</option>
          <option>Oils</option>
        </select>
      </label><br>
      <label>Part Number: <input type="text" id="returnPartNo" required></label><br>
      <label>Quantity: <input type="number" id="returnQty" required></label><br>
      <button type="submit">Return</button>
    </form>
  </div>

  <!-- RECORDS TAB -->
  <div id="Records" class="tabcontent">
    <h2>Transaction Records</h2>
    <input type="text" id="searchRecords" placeholder="Search...">
    <table id="recordsTable">
      <thead>
        <tr>
          <th>Category</th>
          <th>Description</th>
          <th>Part Number</th>
          <th>Quantity</th>
          <th>DR</th>
          <th>Name</th>
          <th>Date</th>
          <th>Type</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <!-- STOCK TAB -->
  <div id="Stock" class="tabcontent">
    <h2>Current Stock</h2>
    <table id="stockTable">
      <thead>
        <tr>
          <th>Category</th>
          <th>Description</th>
          <th>Part Number</th>
          <th>Quantity</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <script>
    const stock = {};
    const records = [];

    function openTab(evt, tabName) {
      const tabcontent = document.getElementsByClassName("tabcontent");
      for (let i = 0; i < tabcontent.length; i++) tabcontent[i].style.display = "none";
      document.getElementById(tabName).style.display = "block";
      if (tabName === 'Stock') renderStock();
      if (tabName === 'Records') renderRecords();
    }

    function getStockKey(category, desc, partNo) {
      return `${category}|${desc}|${partNo || ''}`;
    }

    function updateStock(category, desc, partNo, qty) {
      const key = getStockKey(category, desc, partNo);
      if (!stock[key]) {
        stock[key] = { category, desc, partNo, qty: 0 };
      }
      stock[key].qty += qty;
      if (stock[key].qty < 0) stock[key].qty = 0;
    }

    function renderStock() {
      const tbody = document.querySelector('#stockTable tbody');
      tbody.innerHTML = '';
      for (const key in stock) {
        const { category, desc, partNo, qty } = stock[key];
        if (qty === 0) continue;
        const row = `<tr>
          <td>${category}</td>
          <td>${desc}</td>
          <td>${partNo}</td>
          <td>${qty}</td>
        </tr>`;
        tbody.insertAdjacentHTML('beforeend', row);
      }
    }

    function renderRecords() {
      const tbody = document.querySelector('#recordsTable tbody');
      tbody.innerHTML = '';
      const search = document.getElementById('searchRecords').value.toLowerCase();
      records.forEach(r => {
        if (JSON.stringify(r).toLowerCase().includes(search)) {
          const row = `<tr>
            <td>${r.category}</td>
            <td>${r.desc}</td>
            <td>${r.partNo}</td>
            <td>${r.qty}</td>
            <td>${r.dr || ''}</td>
            <td>${r.name || ''}</td>
            <td>${r.date || ''}</td>
            <td>${r.type}</td>
          </tr>`;
          tbody.insertAdjacentHTML('beforeend', row);
        }
      });
    }

    document.getElementById('issueForm').addEventListener('submit', e => {
      e.preventDefault();
      const category = document.getElementById('issueCategory').value;
      const desc = document.getElementById('issueDesc').value.trim();
      const partNo = document.getElementById('issuePartNo').value.trim();
      const qty = parseInt(document.getElementById('issueQty').value);
      const dr = document.getElementById('issueDR').value.trim();
      const name = document.getElementById('issueName').value.trim();
      const date = document.getElementById('issueDate').value;

      updateStock(category, desc, partNo, -qty);
      records.push({ category, desc, partNo, qty: -qty, dr, name, date, type: 'Issue' });
      e.target.reset();
      alert('Item issued.');
    });

    document.getElementById('returnForm').addEventListener('submit', e => {
      e.preventDefault();
      const category = document.getElementById('returnCategory').value;
      const partNo = document.getElementById('returnPartNo').value.trim();
      const qty = parseInt(document.getElementById('returnQty').value);

      const key = Object.keys(stock).find(k => k.startsWith(`${category}|`) && k.endsWith(`|${partNo}`));
      if (!key) {
        alert('Part not found in stock.');
        return;
      }
      const { desc } = stock[key];
      updateStock(category, desc, partNo, qty);
      records.push({ category, desc, partNo, qty, dr: '', name: '', date: new Date().toISOString().split('T')[0], type: 'Return' });
      e.target.reset();
      alert('Item returned.');
    });

    document.getElementById('searchRecords').addEventListener('input', renderRecords);

    // Open default tab
    document.querySelector('.tab button').click();
  </script>

</body>
</html>
