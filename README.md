<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>WebDevelopment NoteBook</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
  body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #0b0f1a;
    color: white;
  }

  header {
    padding: 20px;
    text-align: center;
    font-size: 36px;
    font-weight: bold;
    background: linear-gradient(90deg, #1e90ff, #00ffff);
    color: black;
  }

  #adminLogin {
    position: fixed;
    top: 10px;
    right: 10px;
    display: flex;
    gap: 5px;
    z-index: 1000;
  }

  #codeBox {
    padding: 6px;
    border-radius: 6px;
    border: none;
    outline: none;
    font-size: 16px;
  }

  #loginBtn {
    padding: 6px 10px;
    border: none;
    background: #00ffff;
    font-weight: bold;
    cursor: pointer;
    font-size: 16px;
  }

  #adminPanel {
    display: none;
    padding: 20px;
    background: #111;
    border-top: 2px solid #00ffff;
  }

  #adminPanel textarea {
    width: 100%;
    height: 120px;
    margin-bottom: 10px;
    font-size: 16px;
    padding: 10px;
    border-radius: 6px;
    border: none;
    outline: none;
  }

  #adminPanel button {
    padding: 12px;
    background: #00ffff;
    border: none;
    cursor: pointer;
    font-weight: bold;
    margin-right: 5px;
    font-size: 16px;
    border-radius: 6px;
  }

  #leaveAdminBtn {
    background: #ff4d4d;
    color: white;
  }

  #content {
    padding: 20px;
  }

  .item {
    background: #1a1f33;
    padding: 15px;
    margin-bottom: 10px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0,255,255,0.2);
    position: relative;
    font-size: 18px;
    word-wrap: break-word;
  }

  .deleteBtn {
    position: absolute;
    top: 8px;
    right: 8px;
    background: #ff4d4d;
    border: none;
    color: white;
    padding: 8px 12px;
    border-radius: 5px;
    cursor: pointer;
    display: none;
    font-size: 14px;
  }

  body.admin .deleteBtn {
    display: block;
  }

  /* Mobile responsive adjustments */
  @media (max-width: 600px) {
    header {
      font-size: 28px;
      padding: 15px;
    }

    #adminLogin {
      top: auto;
      bottom: 10px;
      right: 10px;
      flex-direction: column;
      gap: 8px;
    }

    #codeBox, #loginBtn {
      width: 150px;
      padding: 10px;
      font-size: 16px;
    }

    #adminPanel button {
      width: 100%;
      margin-bottom: 8px;
    }

    .item {
      font-size: 16px;
      padding: 12px;
    }

    .deleteBtn {
      padding: 6px 10px;
      font-size: 14px;
    }
  }

  #pendingRequests {
    margin-top: 10px;
    background: #222;
    padding: 10px;
    border-radius: 6px;
  }

  .request {
    background: #333;
    padding: 8px;
    margin-bottom: 5px;
    border-radius: 4px;
  }

  .request button {
    margin-left: 5px;
    font-size: 14px;
    padding: 5px 8px;
  }

</style>
</head>
<body>

<header>WebDevelopment NoteBook</header>

<div id="adminLogin">
  <input id="codeBox" type="password" placeholder="Admin code">
  <button id="loginBtn">Enter</button>
</div>

<div id="adminPanel">
  <h3>Admin Mode</h3>
  <textarea id="newText" placeholder="Add something to the page..."></textarea>
  <button onclick="addItem()">Add to Page</button>
  <button id="leaveAdminBtn">Leave Admin Mode</button>

  <div id="pendingRequests">
    <h4>Pending Sub-Admin Requests</h4>
    <div id="requestsContainer"></div>
  </div>
</div>

<div id="content"></div>

<script>
  const MASTER_CODE = "4596";
  const SUB_CODES = ["5387","9371","6371","6392","6666"];

  const codeBox = document.getElementById("codeBox");
  const loginBtn = document.getElementById("loginBtn");
  const adminPanel = document.getElementById("adminPanel");
  const leaveAdminBtn = document.getElementById("leaveAdminBtn");
  const content = document.getElementById("content");
  const requestsContainer = document.getElementById("requestsContainer");

  let adminMode = false;
  let currentCode = null; // code of logged-in admin
  let isMaster = false;

  // Stored items: {text: "...", owner: "code"}
  let items = JSON.parse(localStorage.getItem("items") || "[]");
  let pendingRequests = JSON.parse(localStorage.getItem("pendingRequests") || "[]");

  function unlockAdmin() {
    const code = codeBox.value.trim();
    if (code === MASTER_CODE) {
      adminMode = true;
      isMaster = true;
      currentCode = MASTER_CODE;
      adminPanel.style.display = "block";
      document.body.classList.add("admin");
      codeBox.value = "";
      alert("Master admin unlocked");
      renderRequests();
    } else if (SUB_CODES.includes(code)) {
      adminMode = true;
      isMaster = false;
      currentCode = code;
      adminPanel.style.display = "block";
      document.body.classList.add("admin");
      codeBox.value = "";
      alert("Sub-admin unlocked. Requests must be approved by master admin to post.");
    } else {
      alert("Incorrect code");
    }
    refresh();
  }

  leaveAdminBtn.onclick = () => {
    adminMode = false;
    isMaster = false;
    currentCode = null;
    adminPanel.style.display = "none";
    document.body.classList.remove("admin");
  };

  loginBtn.onclick = unlockAdmin;
  codeBox.addEventListener("keydown", e => { if(e.key==="Enter") unlockAdmin(); });

  // Add item function
  function addItem() {
    if (!adminMode || !currentCode) return;

    const text = document.getElementById("newText").value.trim();
    if (!text) return;

    if (isMaster) {
      items.push({text, owner: currentCode});
      saveItems();
      refresh();
      document.getElementById("newText").value = "";
    } else {
      // sub-admin -> send request to master
      pendingRequests.push({text, owner: currentCode});
      saveRequests();
      document.getElementById("newText").value = "";
      alert("Post sent for master approval");
      renderRequests();
    }
  }

  function deleteItem(index) {
    if (!adminMode) return;
    const item = items[index];
    if (isMaster || item.owner === currentCode) {
      items.splice(index,1);
      saveItems();
      refresh();
    } else {
      alert("You can only delete your own posts");
    }
  }

  function saveItems() {
    localStorage.setItem("items", JSON.stringify(items));
  }

  function saveRequests() {
    localStorage.setItem("pendingRequests", JSON.stringify(pendingRequests));
  }

  function refresh() {
    content.innerHTML = "";
    items.forEach((item, index) => {
      const div = document.createElement("div");
      div.className = "item";

      const span = document.createElement("div");
      span.textContent = item.text;

      const del = document.createElement("button");
      del.className = "deleteBtn";
      del.textContent = "Delete";
      del.onclick = () => deleteItem(index);

      div.appendChild(del);
      div.appendChild(span);
      content.appendChild(div);
    });
  }

  // Master renders sub-admin requests
  function renderRequests() {
    if (!isMaster) {
      requestsContainer.innerHTML = "<i>Master admin only</i>";
      return;
    }

    requestsContainer.innerHTML = "";
    pendingRequests.forEach((req, i) => {
      const div = document.createElement("div");
      div.className = "request";
      div.textContent = `[${req.owner}] ${req.text}`;

      const approveBtn = document.createElement("button");
      approveBtn.textContent = "Approve";
      approveBtn.onclick = () => {
        items.push({text:req.text, owner:req.owner});
        pendingRequests.splice(i,1);
        saveItems();
        saveRequests();
        renderRequests();
        refresh();
      };

      const denyBtn = document.createElement("button");
      denyBtn.textContent = "Deny";
      denyBtn.onclick = () => {
        pendingRequests.splice(i,1);
        saveRequests();
        renderRequests();
      };

      div.appendChild(approveBtn);
      div.appendChild(denyBtn);
      requestsContainer.appendChild(div);
    });

    if (pendingRequests.length === 0) {
      requestsContainer.innerHTML = "<i>No pending requests</i>";
    }
  }

  refresh();
  renderRequests();
</script>

</body>
</html>
