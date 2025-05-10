<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Controle de Estoque</title>
  <style>
    /* Reset básico */
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: 'Arial', sans-serif;
      background: #f3f3f3;
      color: #333;
      padding: 20px;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      flex-direction: column;
    }

    /* Logo */
    .logo {
      font-size: 36px;
      font-weight: bold;
      color: #007bff;
      margin-bottom: 20px;
      display: flex;
      align-items: center;
    }

    .logo span {
      color: #333;
      margin-left: 10px;
    }

    .container {
      width: 100%;
      max-width: 1200px;
      background: #fff;
      border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
      padding: 40px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    h1 {
      color: #333;
      text-align: center;
      margin-bottom: 30px;
    }

    .form-inline {
      display: flex;
      gap: 20px;
      width: 100%;
      max-width: 600px;
      margin-bottom: 20px;
      justify-content: space-between;
    }

    input[type="text"], input[type="number"] {
      padding: 12px;
      border: 1px solid #ddd;
      border-radius: 6px;
      width: 48%;
      font-size: 16px;
      background: #f9f9f9;
    }

    button {
      padding: 12px 20px;
      background: #007bff;
      color: #fff;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      cursor: pointer;
      transition: background-color 0.3s;
    }

    button:hover {
      background: #0056b3;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }

    th, td {
      padding: 12px;
      border: 1px solid #ddd;
      text-align: center;
    }

    th {
      background-color: #007bff;
      color: white;
    }

    td button {
      padding: 6px 12px;
      border: none;
      background-color: #ff6666;
      color: white;
      border-radius: 6px;
      cursor: pointer;
      transition: background-color 0.3s;
    }

    td button:hover {
      background-color: #e60000;
    }

    .actions {
      display: flex;
      justify-content: space-evenly;
    }

    .modal, .overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.6);
      display: none;
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }

    .modal-content {
      background: #fff;
      padding: 30px;
      border-radius: 12px;
      width: 400px;
      max-width: 100%;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .modal input {
      margin-bottom: 15px;
      width: 100%;
      padding: 10px;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #ddd;
    }

    .modal button {
      background-color: #007bff;
      color: white;
    }

    .modal button:hover {
      background-color: #0056b3;
    }

    .overlay {
      display: none;
    }

  </style>
</head>
<body>
  <!-- Logo -->
  <div class="logo">
    <img src="https://i.imgur.com/FZSsDFQ.png" /> <!-- Se você tiver uma logo em imagem, adicione aqui -->
    <span>Sanches Usinagem</span>
  </div>

  <div class="container">
    <h1>Controle de Estoque</h1>
    
    <div class="form-inline">
      <input type="text" id="item-name" placeholder="Nome do item">
      <input type="number" id="item-quantity" placeholder="Quantidade" min="1">
    </div>
    <button onclick="addItem()">Adicionar Item</button>

    <table>
      <thead>
        <tr>
          <th>Item</th>
          <th>Quantidade</th>
          <th>Data de Adição</th>
          <th>Data da Última Edição</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody id="inventory-table">
        <!-- Conteúdo dinâmico -->
      </tbody>
    </table>
  </div>

  <!-- Modal de edição -->
  <div class="overlay" id="modal-overlay"></div>
  <div class="modal" id="modal">
    <div class="modal-content">
      <h2>Editar Item</h2>
      <input type="text" id="edit-item-name" placeholder="Novo nome">
      <input type="number" id="edit-item-quantity" placeholder="Nova quantidade" min="1">
      <button onclick="saveEdit()">Salvar Alterações</button>
      <button onclick="closeModal()">Cancelar</button>
    </div>
  </div>

  <script>
    let inventory = JSON.parse(localStorage.getItem('inventory')) || [];
    let editingIndex = null;

    function saveInventory() {
      localStorage.setItem('inventory', JSON.stringify(inventory));
    }

    function renderInventory() {
      const table = document.getElementById('inventory-table');
      table.innerHTML = '';
      inventory.forEach((item, index) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${item.name}</td>
          <td>${item.quantity}</td>
          <td>${item.addedDate}</td>
          <td>${item.lastEditedDate}</td>
          <td class="actions">
            <button onclick="removeItem(${index})">Retirar</button>
            <button onclick="editItem(${index})">Editar</button>
          </td>
        `;
        table.appendChild(row);
      });
    }

    function addItem() {
      const name = document.getElementById('item-name').value.trim();
      const quantity = parseInt(document.getElementById('item-quantity').value);
      if (!name || isNaN(quantity) || quantity <= 0) {
        alert('Por favor, preencha os campos corretamente.');
        return;
      }

      const addedDate = new Date().toLocaleString();
      const existingIndex = inventory.findIndex(item => item.name.toLowerCase() === name.toLowerCase());
      if (existingIndex !== -1) {
        inventory[existingIndex].quantity += quantity;
        inventory[existingIndex].lastEditedDate = new Date().toLocaleString(); // Atualiza a data da última edição
      } else {
        inventory.push({ 
          name, 
          quantity, 
          addedDate, 
          lastEditedDate: addedDate 
        });
      }

      saveInventory();
      renderInventory();

      document.getElementById('item-name').value = '';
      document.getElementById('item-quantity').value = '';
    }

    function removeItem(index) {
      const quantityToRemove = parseInt(prompt('Informe a quantidade a ser retirada:'));
      if (isNaN(quantityToRemove) || quantityToRemove <= 0) {
        alert('Por favor, insira uma quantidade válida.');
        return;
      }

      if (inventory[index].quantity >= quantityToRemove) {
        inventory[index].quantity -= quantityToRemove;
        inventory[index].lastEditedDate = new Date().toLocaleString(); // Atualiza a data da última edição
        saveInventory();
        renderInventory();
        alert(`Retirada de ${quantityToRemove} unidades de "${inventory[index].name}" realizada com sucesso!`);
      } else {
        alert('Quantidade insuficiente no estoque.');
      }
    }

    function editItem(index) {
      const item = inventory[index];
      document.getElementById('edit-item-name').value = item.name;
      document.getElementById('edit-item-quantity').value = item.quantity;
      editingIndex = index;

      document.getElementById('modal').style.display = 'block';
      document.getElementById('modal-overlay').style.display = 'block';
    }

    function saveEdit() {
      const newName = document.getElementById('edit-item-name').value.trim();
      const newQuantity = parseInt(document.getElementById('edit-item-quantity').value);

      if (!newName || isNaN(newQuantity) || newQuantity <= 0) {
        alert('Preencha os campos corretamente.');
        return;
      }

      inventory[editingIndex].name = newName;
      inventory[editingIndex].quantity = newQuantity;
      inventory[editingIndex].lastEditedDate = new Date().toLocaleString(); // Atualiza a data da última edição

      saveInventory();
      renderInventory();

      closeModal();
    }

    function closeModal() {
      document.getElementById('modal').style.display = 'none';
      document.getElementById('modal-overlay').style.display = 'none';
    }

    window.onload = renderInventory;
  </script>
</body>
</html>
