<script type="text/javascript">
        var gk_isXlsx = false;
        var gk_xlsxFileLookup = {};
        var gk_fileData = {};
        function filledCell(cell) {
          return cell !== '' && cell != null;
        }
        function loadFileData(filename) {
        if (gk_isXlsx && gk_xlsxFileLookup[filename]) {
            try {
                var workbook = XLSX.read(gk_fileData[filename], { type: 'base64' });
                var firstSheetName = workbook.SheetNames[0];
                var worksheet = workbook.Sheets[firstSheetName];

                // Convert sheet to JSON to filter blank rows
                var jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1, blankrows: false, defval: '' });
                // Filter out blank rows (rows where all cells are empty, null, or undefined)
                var filteredData = jsonData.filter(row => row.some(filledCell));

                // Heuristic to find the header row by ignoring rows with fewer filled cells than the next row
                var headerRowIndex = filteredData.findIndex((row, index) =>
                  row.filter(filledCell).length >= filteredData[index + 1]?.filter(filledCell).length
                );
                // Fallback
                if (headerRowIndex === -1 || headerRowIndex > 25) {
                  headerRowIndex = 0;
                }

                // Convert filtered JSON back to CSV
                var csv = XLSX.utils.aoa_to_sheet(filteredData.slice(headerRowIndex)); // Create a new sheet from filtered array of arrays
                csv = XLSX.utils.sheet_to_csv(csv, { header: 1 });
                return csv;
            } catch (e) {
                console.error(e);
                return "";
            }
        }
        return gk_fileData[filename] || "";
        }
        </script><!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="theme-color" content="#1e3a8a">
  <title>Aplikasi Kasir Wrg Alifa</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
  <link rel="manifest" href="/manifest.json">
  <link rel="icon" href="https://via.placeholder.com/192x192.png?text=Wrg+Alifa" type="image/png">
  <style>
    body {
      background: linear-gradient(to bottom, #1e3a8a, #ffffff);
      min-height: 100vh;
      font-size: 16px;
    }
    .card {
      background: white;
      border-radius: 10px;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
      transition: transform 0.2s;
    }
    .card:hover {
      transform: translateY(-5px);
    }
    .btn {
      transition: background-color 0.3s, transform 0.2s;
      padding: 12px 16px;
      font-size: 1rem;
    }
    .btn:hover {
      transform: scale(1.05);
    }
    .receipt {
      border: 2px solid #1e3a8a;
      border-radius: 8px;
      background: #f8fafc;
    }
    @media (max-width: 640px) {
      .container {
        padding: 8px;
      }
      .grid-cols-1 {
        grid-template-columns: 1fr;
      }
      .btn {
        width: 100%;
        padding: 12px;
      }
      .text-4xl {
        font-size: 2rem;
      }
      .text-2xl {
        font-size: 1.5rem;
      }
      table {
        font-size: 0.875rem;
      }
      th, td {
        padding: 8px;
      }
    }
  </style>
</head>
<body class="font-sans">
  <div class="container mx-auto p-4 max-w-4xl">
    <h1 class="text-4xl font-bold text-center mb-8 text-white">
      <i class="fas fa-shopping-cart mr-2"></i>Aplikasi Kasir - Wrg Alifa
    </h1>
    
    <!-- Form Tambah Barang ke Daftar Bawaan -->
    <div class="card p-6 mb-6">
      <h2 class="text-2xl font-semibold mb-4 text-blue-800">
        <i class="fas fa-box-open mr-2"></i>Tambah Barang ke Daftar Bawaan
      </h2>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <input id="newItemName" type="text" placeholder="Nama Barang" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
        <select id="newItemCategory" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
          <option value="Makanan">Makanan</option>
          <option value="Minuman">Minuman</option>
          <option value="Rokok">Rokok</option>
          <option value="Kebutuhan Sehari-hari">Kebutuhan Sehari-hari</option>
        </select>
        <input id="newItemPrice" type="number" placeholder="Harga Barang" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
      </div>
      <button onclick="addNewDefaultItem()" class="mt-4 bg-indigo-500 text-white px-4 py-2 rounded btn hover:bg-indigo-600">
        <i class="fas fa-plus mr-2"></i>Tambah ke Daftar Bawaan
      </button>
    </div>

    <!-- Form Tambah Barang ke Keranjang -->
    <div class="card p-6 mb-6">
      <h2 class="text-2xl font-semibold mb-4 text-blue-800">
        <i class="fas fa-plus-circle mr-2"></i>Tambah Barang ke Keranjang
      </h2>
      <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
        <select id="itemName" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
          <option value="">Pilih Barang atau Masukkan Manual</option>
          <!-- Daftar barang bawaan akan diisi oleh JavaScript -->
        </select>
        <input id="itemNameManual" type="text" placeholder="Nama Barang Manual" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
        <select id="itemCategory" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
          <option value="Makanan">Makanan</option>
          <option value="Minuman">Minuman</option>
          <option value="Rokok">Rokok</option>
          <option value="Kebutuhan Sehari-hari">Kebutuhan Sehari-hari</option>
        </select>
        <input id="itemPrice" type="number" placeholder="Harga Barang" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500">
        <input id="itemQuantity" type="number" placeholder="Jumlah" class="border p-2 rounded bg-blue-50 focus:ring-2 focus:ring-blue-500" min="1">
      </div>
      <button onclick="addItem()" class="mt-4 bg-blue-600 text-white px-4 py-2 rounded btn hover:bg-blue-700">
        <i class="fas fa-cart-plus mr-2"></i>Tambah ke Keranjang
      </button>
    </div>

    <!-- Daftar Barang -->
    <div class="card p-6 mb-6">
      <h2 class="text-2xl font-semibold mb-4 text-blue-800">
        <i class="fas fa-list mr-2"></i>Daftar Barang
      </h2>
      <table id="itemList" class="w-full border-collapse">
        <thead>
          <tr class="bg-blue-100 text-blue-800">
            <th class="border p-2">Nama Barang</th>
            <th class="border p-2">Kategori</th>
            <th class="border p-2">Harga</th>
            <th class="border p-2">Jumlah</th>
            <th class="border p-2">Total</th>
            <th class="border p-2">Aksi</th>
          </tr>
        </thead>
        <tbody id="itemTableBody"></tbody>
      </table>
      <div class="mt-4 text-right">
        <p class="text-lg font-semibold text-blue-800">Total Harga: Rp <span id="totalPrice">0</span></p>
      </div>
    </div>

    <!-- Tombol Selesai Transaksi dan Cetak Struk -->
    <div class="flex flex-col sm:flex-row space-y-4 sm:space-y-0 sm:space-x-4">
      <button onclick="finishTransaction()" class="bg-green-500 text-white px-4 py-2 rounded btn hover:bg-green-600">
        <i class="fas fa-check-circle mr-2"></i>Selesai Transaksi
      </button>
      <button onclick="printReceipt()" class="bg-purple-500 text-white px-4 py-2 rounded btn hover:bg-purple-600">
        <i class="fas fa-print mr-2"></i>Cetak Struk
      </button>
    </div>

    <!-- Area Struk -->
    <div id="receiptArea" class="card p-6 mt-6 hidden receipt">
      <h2 class="text-2xl font-semibold mb-4 text-blue-800">
        <i class="fas fa-receipt mr-2"></i>Struk Belanja
      </h2>
      <pre id="receiptContent" class="font-mono text-sm text-gray-800"></pre>
      <button onclick="downloadReceipt()" class="mt-4 bg-blue-600 text-white px-4 py-2 rounded btn hover:bg-blue-700">
        <i class="fas fa-download mr-2"></i>Unduh Struk
      </button>
    </div>
  </div>

  <script>
    // Daftar harga bawaan
    let defaultItems = [
      { name: "Aqua", category: "Minuman", price: 4000 },
      { name: "Teh Botol", category: "Minuman", price: 5000 },
      { name: "Pocari Sweat", category: "Minuman", price: 7000 },
      { name: "Roti Tawar", category: "Makanan", price: 12000 },
      { name: "Chitato", category: "Makanan", price: 8000 },
      { name: "Biskuit Roma", category: "Makanan", price: 6000 },
      { name: "Marlboro", category: "Rokok", price: 30000 },
      { name: "Djarum Super", category: "Rokok", price: 25000 },
      { name: "Sampoerna Mild", category: "Rokok", price: 27000 },
      { name: "Sunsilk Shampo", category: "Kebutuhan Sehari-hari", price: 15000 },
      { name: "Lifebuoy Sabun", category: "Kebutuhan Sehari-hari", price: 5000 },
      { name: "Pepsodent Pasta Gigi", category: "Kebutuhan Sehari-hari", price: 10000 }
    ];

    let items = [];

    // Mengisi dropdown dengan daftar barang bawaan, dikelompokkan berdasarkan kategori
    function populateItemDropdown() {
      const itemSelect = document.getElementById('itemName');
      itemSelect.innerHTML = '<option value="">Pilih Barang atau Masukkan Manual</option>';
      const categories = [...new Set(defaultItems.map(item => item.category))];

      categories.forEach(category => {
        const optgroup = document.createElement('optgroup');
        optgroup.label = category;
        defaultItems
          .filter(item => item.category === category)
          .forEach(item => {
            const option = document.createElement('option');
            option.value = item.name;
            option.textContent = `${item.name} (Rp ${item.price.toLocaleString()})`;
            optgroup.appendChild(option);
          });
        itemSelect.appendChild(optgroup);
      });
    }

    // Tambah barang baru ke daftar bawaan
    function addNewDefaultItem() {
      const name = document.getElementById('newItemName').value;
      const category = document.getElementById('newItemCategory').value;
      const price = parseFloat(document.getElementById('newItemPrice').value);

      if (name && price > 0) {
        if (defaultItems.some(item => item.name.toLowerCase() === name.toLowerCase())) {
          alert('Barang dengan nama ini sudah ada di daftar bawaan!');
          return;
        }
        defaultItems.push({ name, category, price });
        document.getElementById('newItemName').value = '';
        document.getElementById('newItemCategory').value = 'Makanan';
        document.getElementById('newItemPrice').value = '';
        populateItemDropdown();
        alert(`Barang "${name}" berhasil ditambahkan ke daftar bawaan!`);
      } else {
        alert('Harap isi nama dan harga barang dengan benar!');
      }
    }

    // Panggil saat halaman dimuat
    window.onload = populateItemDropdown;

    function addItem() {
      let name = document.getElementById('itemName').value;
      const manualName = document.getElementById('itemNameManual').value;
      const category = document.getElementById('itemCategory').value;
      const price = parseFloat(document.getElementById('itemPrice').value);
      const quantity = parseInt(document.getElementById('itemQuantity').value);

      if (name) {
        const selectedItem = defaultItems.find(item => item.name === name);
        if (selectedItem) {
          name = selectedItem.name;
          document.getElementById('itemCategory').value = selectedItem.category;
          document.getElementById('itemPrice').value = selectedItem.price;
        }
      } else if (manualName) {
        name = manualName;
      }

      if (name && price > 0 && quantity > 0) {
        items.push({ name, category, price, quantity });
        document.getElementById('itemName').value = '';
        document.getElementById('itemNameManual').value = '';
        document.getElementById('itemCategory').value = 'Makanan';
        document.getElementById('itemPrice').value = '';
        document.getElementById('itemQuantity').value = '';
        updateTable();
      } else {
        alert('Harap isi semua kolom dengan benar!');
      }
    }

    function updateTable() {
      const tableBody = document.getElementById('itemTableBody');
      tableBody.innerHTML = '';
      let totalPrice = 0;

      items.forEach((item, index) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td class="border p-2">${item.name}</td>
          <td class="border p-2">${item.category}</td>
          <td class="border p-2">Rp ${item.price.toLocaleString()}</td>
          <td class="border p-2">${item.quantity}</td>
          <td class="border p-2">Rp ${(item.price * item.quantity).toLocaleString()}</td>
          <td class="border p-2">
            <button onclick="deleteItem(${index})" class="bg-red-500 text-white px-2 py-1 rounded btn hover:bg-red-600">
              <i class="fas fa-trash mr-1"></i>Hapus
            </button>
          </td>
        `;
        tableBody.appendChild(row);
        totalPrice += item.price * item.quantity;
      });

      document.getElementById('totalPrice').textContent = totalPrice.toLocaleString();
    }

    function deleteItem(index) {
      items.splice(index, 1);
      updateTable();
    }

    function finishTransaction() {
      if (items.length === 0) {
        alert('Keranjang kosong!');
        return;
      }
      alert('Transaksi selesai! Total: Rp ' + document.getElementById('totalPrice').textContent);
      generateReceipt();
      items = [];
      updateTable();
    }

    function generateReceipt() {
      const now = new Date();
      const dateStr = now.toLocaleString('id-ID', { dateStyle: 'medium', timeStyle: 'short' });
      let receipt = `           Wrg Alifa\n`;
      receipt += `      Jl. Contoh No. 123\n`;
      receipt += `      ${dateStr}\n`;
      receipt += `============================\n`;
      receipt += `Nama Barang      Kategori   Harga   Jml   Total\n`;
      receipt += `----------------------------\n`;

      let totalPrice = 0;
      items.forEach(item => {
        const itemTotal = item.price * item.quantity;
        totalPrice += itemTotal;
        receipt += `${item.name.padEnd(16)} ${item.category.padEnd(10)} ${item.price.toLocaleString().padEnd(7)} ${item.quantity.toString().padEnd(5)} ${itemTotal.toLocaleString()}\n`;
      });

      receipt += `============================\n`;
      receipt += `Total Harga: Rp ${totalPrice.toLocaleString()}\n`;
      receipt += `============================\n`;
      receipt += `Terima Kasih\n`;
      receipt += `Selamat Belanja Kembali!\n`;

      document.getElementById('receiptContent').textContent = receipt;
      document.getElementById('receiptArea').classList.remove('hidden');
    }

    function downloadReceipt() {
      const receiptContent = document.getElementById('receiptContent').textContent;
      const blob = new Blob([receiptContent], { type: 'text/plain' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'struk_wrg_alifa.txt';
      a.click();
      URL.revokeObjectURL(url);
    }

    function printReceipt() {
      if (document.getElementById('receiptArea').classList.contains('hidden')) {
        alert('Silakan selesaikan transaksi terlebih dahulu!');
        return;
      }
      const receiptContent = document.getElementById('receiptContent').textContent;
      const printWindow = window.open('', '_blank');
      printWindow.document.write(`
        <html>
          <head>
            <title>Struk Wrg Alifa</title>
            <style>
              body { font-family: monospace; font-size: 12px; }
              pre { border: 2px solid #1e3a8a; padding: 10px; background: #f8fafc; }
              @media print { body { margin: 0; } }
            </style>
          </head>
          <body><pre>${receiptContent}</pre></body>
        </html>
      `);
      printWindow.document.close();
      printWindow.print();
    }

    // Auto-fill harga dan kategori berdasarkan pilihan barang
    document.getElementById('itemName').addEventListener('change', function() {
      const selectedName = this.value;
      const selectedItem = defaultItems.find(item => item.name === selectedName);
      if (selectedItem) {
        document.getElementById('itemCategory').value = selectedItem.category;
        document.getElementById('itemPrice').value = selectedItem.price;
      } else {
        document.getElementById('itemCategory').value = 'Makanan';
        document.getElementById('itemPrice').value = '';
      }
    });

    // Service Worker untuk PWA (opsional, aktifkan saat hosting)
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/sw.js').catch(err => console.error('Service Worker registration failed:', err));
    }
  </script>
</body>
</html>
