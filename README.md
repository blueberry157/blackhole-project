# blackhole-project
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>The Blackhole Project</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #0d1117;
      color: #c9d1d9;
      margin: 0;
      padding: 20px;
    }
    .container {
      max-width: 900px;
      margin: 0 auto;
      background: #161b22;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
    }
    h1 {
      text-align: center;
      color: #58a6ff;
    }
    .auth-buttons {
      display: flex;
      justify-content: center;
      gap: 10px;
      margin-bottom: 20px;
    }
    .auth-buttons button {
      padding: 10px 15px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-weight: bold;
    }
    .login {
      background-color: #238636;
      color: white;
    }
    .logout {
      background-color: #da3633;
      color: white;
    }
    .total {
      text-align: center;
      font-size: 1.4em;
      margin-bottom: 30px;
      color: #f0f6fc;
    }
    .main-nav {
      text-align: center;
      margin-bottom: 20px;
    }
    .main-nav button {
      background-color: #58a6ff;
      color: #0d1117;
      border: none;
      padding: 10px 20px;
      margin: 0 10px;
      border-radius: 5px;
      cursor: pointer;
    }
    .main-nav button:hover {
      background-color: #1f6feb;
    }
  </style>
</head>
<body>
  <div class="container" id="main-page">
    <h1>The Blackhole Project</h1>
    <div class="auth-buttons">
      <button class="login" onclick="signIn()">Sign in with Google</button>
      <button class="logout" onclick="signOut()">Sign Out</button>
    </div>
    <div id="user-info" style="text-align:center; margin-bottom:10px;"></div>
    <div class="total">🌌 Universe Total Quantity Collected: <span id="global-total">000</span></div>
    <div class="main-nav">
      <button onclick="goToBoard()">Go to My Collection Board</button>
    </div>
  </div>

  <div class="container" id="board-page" style="display:none;">
    <h1>Your Collection Board</h1>
    <div class="main-nav">
      <button onclick="goToMain()">Back to Main Page</button>
    </div>
    <table id="collection-table">
      <thead>
        <tr>
          <th>Item Name</th>
          <th>Quantity</th>
          <th>Notes</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><input type="text" placeholder="e.g. Pokemon Card" /></td>
          <td><input type="number" class="qty" value="0" /></td>
          <td><input type="text" placeholder="Holo, Rare..." /></td>
        </tr>
      </tbody>
    </table>
    <button class="add-row" onclick="addRow()">Add Item</button>
    <div class="total" id="total-display">Total Quantity: 0</div>
  </div>

  <script src="https://www.gstatic.com/firebasejs/10.10.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.10.0/firebase-auth.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.10.0/firebase-firestore.js"></script>
  <script>
    let currentUser = null;
    const db = firebase.firestore();
    const auth = firebase.auth();
    const provider = new firebase.auth.GoogleAuthProvider();

    // Add event listener to listen for changes
    window.addEventListener('DOMContentLoaded', () => {
      const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "the-blackhole-project.firebaseapp.com",
        projectId: "the-blackhole-project",
        appId: "YOUR_APP_ID"
      };
      
      firebase.initializeApp(firebaseConfig);
      
      auth.onAuthStateChanged((user) => {
        if (user) {
          currentUser = user;
          document.getElementById("user-info").innerText = `Signed in as: ${user.displayName}`;
          loadGlobalTotal();
        } else {
          document.getElementById("user-info").innerText = "Not signed in";
        }
      });
    });

    function goToBoard() {
      document.getElementById('main-page').style.display = 'none';
      document.getElementById('board-page').style.display = 'block';
    }

    function goToMain() {
      document.getElementById('board-page').style.display = 'none';
      document.getElementById('main-page').style.display = 'block';
      loadGlobalTotal();
    }

    function signIn() {
      auth.signInWithPopup(provider).then((result) => {
        currentUser = result.user;
        document.getElementById("user-info").innerText = `Signed in as: ${currentUser.displayName}`;
        loadGlobalTotal();
      }).catch((error) => {
        console.error(error);
      });
    }

    function signOut() {
      auth.signOut().then(() => {
        document.getElementById("user-info").innerText = "";
        currentUser = null;
      });
    }

    // Function to add a row to the board
    function addRow() {
      const table = document.getElementById('collection-table').getElementsByTagName('tbody')[0];
      const newRow = table.insertRow();
      newRow.innerHTML = `
        <td><input type="text" placeholder="e.g. Pokemon Card" /></td>
        <td><input type="number" class="qty" value="0" /></td>
        <td><input type="text" placeholder="Holo, Rare..." /></td>
      `;
      addListeners(); // Ensure listeners are attached to the new row
      updateTotal(); // Ensure the total is updated immediately after adding a row
    }

    // Function to attach event listeners to the input fields
    function addListeners() {
      const inputs = document.querySelectorAll('.qty');
      inputs.forEach(input => {
        input.removeEventListener('input', updateTotal); // Remove any previous listener
        input.addEventListener('input', updateTotal); // Add the event listener again to each input
      });
    }

    // Function to update the total quantity
    function updateTotal() {
      const quantities = document.querySelectorAll('.qty');
      let total = 0;
      quantities.forEach(input => {
        total += parseInt(input.value) || 0; // Ensure empty fields are counted as 0
      });
      document.getElementById('total-display').textContent = `Total Quantity: ${total}`;
      if (currentUser) {
        db.collection("boards").doc(currentUser.uid).set({ total: total });
      }
    }

    // Function to load the global total
    function loadGlobalTotal() {
      db.collection("boards").get().then((querySnapshot) => {
        let globalTotal = 0;
        querySnapshot.forEach((doc) => {
          globalTotal += doc.data().total || 0;
        });
        document.getElementById("global-total").textContent = globalTotal;
      });
    }
  </script>
</body>
</html>
