let products = [];
let totalBill = 0;
let dailyIncome = 0;
let expenseAmount = 0;

function showTab(tabId) {
  document.querySelectorAll('.tab-content').forEach(tab => tab.classList.remove('active-tab'));
  document.getElementById(tabId).classList.add('active-tab');
}

document.getElementById('productForm').addEventListener('submit', function (event) {
  event.preventDefault();
  const name = document.getElementById('productName').value;
  const price = parseFloat(document.getElementById('productPrice').value);
  const image = document.getElementById('productImage').files[0];

  const reader = new FileReader();
  reader.onload = function () {
    products.push({ name, price, image: reader.result, quantity: 1 });
    renderProducts();
    renderProductList();
  };
  reader.readAsDataURL(image);

  document.getElementById('productForm').reset();
});

function renderProducts() {
  const container = document.getElementById('productContainer');
  container.innerHTML = '';
  products.forEach((p, i) => {
    const div = document.createElement('div');
    div.className = 'product';
    div.innerHTML = `
      <img src="${p.image}" onclick="addToBill(${i})">
      <p>${p.name}</p>
      <p>${p.price.toFixed(2)} MVR</p>
    `;
    container.appendChild(div);
  });
}

function renderProductList() {
  const list = document.getElementById('productList');
  list.innerHTML = '';
  products.forEach((p, i) => {
    const li = document.createElement('li');
    li.innerHTML = `${p.name} - ${p.price.toFixed(2)} MVR 
      <button onclick="editProduct(${i})">Edit</button>
      <button onclick="deleteProduct(${i})">Delete</button>`;
    list.appendChild(li);
  });
}

function addToBill(index) {
  totalBill += products[index].price;
  dailyIncome += products[index].price;
  products[index].quantity++;
  document.getElementById('totalBill').textContent = totalBill.toFixed(2) + ' MVR';
  updateBudget();
}

function updateBudget() {
  document.getElementById('dailyIncome').textContent = dailyIncome.toFixed(2) + ' MVR';
  document.getElementById('netIncome').textContent = (dailyIncome - expenseAmount).toFixed(2) + ' MVR';
}

function saveBill() {
  const customerName = document.getElementById('customerName').value.trim();
  if (!customerName) {
    alert("Please enter customer name.");
    return;
  }
  if (totalBill === 0) {
    alert("The bill is empty.");
    return;
  }

  const { jsPDF } = window.jspdf;
  const doc = new jsPDF('p', 'mm', 'a4');
  const margin = 10;
  const pageWidth = doc.internal.pageSize.width - 2 * margin;
  const pageHeight = doc.internal.pageSize.height - 2 * margin;
  let y = margin + 10;

  doc.setFontSize(16);
  doc.setFont("helvetica", "bold");
  const titleWidth = doc.getTextWidth('VIEW SPOT');
  doc.text('VIEW SPOT', margin + (pageWidth - titleWidth) / 2, y);

  y += 20;
  doc.setFontSize(12);
  doc.text('Bill Details', margin, y);
  y += 10;

  doc.setFontSize(10);
  doc.text('No.', margin + 2, y);
  doc.text('Description', margin + 18, y);
  doc.text('Quantity', margin + 80, y);
  doc.text('Rate (MVR)', margin + 110, y);
  doc.text('Amount (MVR)', margin + 140, y);
  y += 6;
  doc.line(margin, y, pageWidth, y);
  y += 10;

  let index = 1;
  let calculatedTotal = 0;

  products.forEach(p => {
    if (p.quantity > 0) {
      doc.text(index.toString(), margin + 2, y);
      doc.text(p.name, margin + 18, y);
      doc.text(p.quantity.toString(), margin + 80, y);
      doc.text(p.price.toFixed(2), margin + 110, y);
      const amount = (p.quantity * p.price).toFixed(2);
      doc.text(amount, margin + 140, y);
      calculatedTotal += parseFloat(amount);
      index++;
      y += 8;

      if (y > pageHeight - 40) {
        doc.addPage();
        y = margin + 10;
      }
    }
  });

  y += 10;
  doc.setFontSize(12);
  doc.text(`Total: ${calculatedTotal.toFixed(2)} MVR`, margin + 120, y);

  y += 12;
  doc.line(margin, y, pageWidth, y);
  y += 10;

  doc.setFontSize(9);
  doc.text('Transfer Details:', 20, y); y += 10;
  doc.text('Account Name: FATH.H.H.MOHAMED', 20, y); y += 6;
  doc.text('Account Number: 7705329041101', 20, y); y += 6;
  doc.text('Viber Number: 9323601', 20, y); y += 25;

  doc.setFontSize(10);
  doc.text('Thank you for your purchase!', 85, y);

  doc.save(`${customerName}_bill.pdf`);

  totalBill = 0;
  products.forEach(p => p.quantity = 0);
  document.getElementById('totalBill').textContent = '0.00 MVR';
  document.getElementById('customerName').value = '';
}

document.getElementById('expenseForm').addEventListener('submit', function (e) {
  e.preventDefault();
  expenseAmount = parseFloat(document.getElementById('expenseAmount').value) || 0;
  updateBudget();
});

function resetIncome() {
  expenseAmount = 0;
  dailyIncome = 0;
  totalBill = 0;
  products.forEach(p => p.quantity = 0);
  updateBudget();
  document.getElementById('totalBill').textContent = '0.00 MVR';
  document.getElementById('dailyIncome').textContent = '0.00 MVR';
  document.getElementById('netIncome').textContent = '0.00 MVR';
}

function editProduct(index) {
  const product = products[index];
  document.getElementById('productName').value = product.name;
  document.getElementById('productPrice').value = product.price;

  document.getElementById('productForm').onsubmit = function (event) {
    event.preventDefault();
    product.name = document.getElementById('productName').value;
    product.price = parseFloat(document.getElementById('productPrice').value);
    renderProducts();
    renderProductList();
    this.reset();
  };
}

function deleteProduct(index) {
  products.splice(index, 1);
  renderProducts();
  renderProductList();
}
