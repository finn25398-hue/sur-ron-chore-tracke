<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>SurRon Chore Tracker</title>
  <style>
    :root {
      --bg: #0f172a;
      --card: #1e293b;
      --text: #e5e7eb;
      --accent: #22c55e;
      --accent2: #38bdf8;
      --danger: #ef4444;
      --glow: 0 0 12px rgba(34,197,94,0.6);
    }

    body {
      font-family: Arial, sans-serif;
      background: radial-gradient(circle at top, #1e293b, #020617);
      color: var(--text);
      padding: 20px;
      max-width: 950px;
      margin: auto;
    }

    h1, h2 {
      text-align: center;
    }

    .box {
      background: linear-gradient(145deg, #1e293b, #020617);
      padding: 16px;
      margin: 15px 0;
      border-radius: 14px;
      box-shadow: 0 0 20px rgba(0,0,0,0.6);
      animation: fadeIn 0.6s ease;
    }

    .goal-box {
      border: 2px solid var(--accent);
      box-shadow: var(--glow);
    }

    .chore {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin: 10px 0;
      gap: 10px;
    }

    button {
      padding: 6px 12px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      background: var(--accent);
      color: #020617;
      font-weight: bold;
      transition: 0.2s ease;
      box-shadow: var(--glow);
    }

    button:hover {
      transform: scale(1.05);
      background: #4ade80;
    }

    button.approve { background: var(--accent2); box-shadow: 0 0 12px rgba(56,189,248,0.7); }
    button.delete { background: var(--danger); box-shadow: 0 0 12px rgba(239,68,68,0.7); }

    input {
      padding: 6px;
      border-radius: 6px;
      border: none;
      outline: none;
      background: #020617;
      color: white;
      border: 1px solid #334155;
    }

    .progress-bar {
      width: 100%;
      background: #020617;
      border-radius: 12px;
      overflow: hidden;
      height: 26px;
      margin-top: 10px;
      border: 1px solid #334155;
    }

    .progress {
      height: 100%;
      background: linear-gradient(90deg, #22c55e, #4ade80);
      width: 0%;
      text-align: center;
      color: #020617;
      font-weight: bold;
      transition: width 0.6s cubic-bezier(.4,1.6,.6,1);
    }

    #total {
      font-size: 22px;
      text-align: center;
      font-weight: bold;
      margin-top: 10px;
      color: #4ade80;
      text-shadow: 0 0 10px rgba(74,222,128,0.7);
    }

    .pop {
      animation: pop 0.4s ease;
    }

    .celebrate {
      animation: celebrate 0.8s ease;
      color: #22c55e;
      text-shadow: 0 0 20px rgba(34,197,94,1);
    }

    @keyframes pop {
      0% { transform: scale(0.6); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    @keyframes celebrate {
      0% { transform: scale(1); }
      50% { transform: scale(1.15); }
      100% { transform: scale(1); }
    }

    .glow-text {
      color: #4ade80;
      text-shadow: 0 0 15px rgba(74,222,128,0.9);
    }
  </style>
</head>
<body>

<h1>⚡ SurRon Chore Tracker</h1>

<div class="box goal-box">
  <h2 class="glow-text">🏍️ SurRon Goal Progress</h2>
  <input id="goalInput" type="number" placeholder="Enter SurRon goal points" />
  <button onclick="setGoal()">Set Goal</button>
  <div class="progress-bar">
    <div id="progress" class="progress">0%</div>
  </div>
</div>

<div id="total">Total Points: 0</div>

<div class="box">
  <h2>➕ Add New Chore</h2>
  <input id="newChoreName" placeholder="Chore name" />
  <input id="newChorePoints" type="number" placeholder="Points" />
  <button onclick="addChore()">Add</button>
</div>

<div class="box">
  <h2>📋 Chores</h2>
  <div id="choreList"></div>
</div>

<div class="box">
  <h2>⏳ Waiting for Parent Approval</h2>
  <div id="pendingList"></div>
</div>

<div class="box">
  <h2>✅ Approved Today</h2>
  <div id="approvedList"></div>
</div>

<div style="text-align:center; margin-top:15px;">
  <button class="delete" onclick="resetDay()">Reset Day</button>
  <button class="delete" onclick="resetAll()">Reset Everything</button>
</div>

<script>
  let chores = JSON.parse(localStorage.getItem("chores")) || [
    { name: "Make bed", points: 10 },
    { name: "Dishes", points: 15 },
    { name: "Vacuum", points: 50 },
    { name: "Laundry", points: 75 }
  ];

  let pending = JSON.parse(localStorage.getItem("pending")) || [];
  let approved = JSON.parse(localStorage.getItem("approved")) || [];
  let total = JSON.parse(localStorage.getItem("total")) || 0;
  let goal = JSON.parse(localStorage.getItem("goal")) || 5000;

  const PARENT_PIN = "1234"; // Change this

  function save() {
    localStorage.setItem("chores", JSON.stringify(chores));
    localStorage.setItem("pending", JSON.stringify(pending));
    localStorage.setItem("approved", JSON.stringify(approved));
    localStorage.setItem("total", JSON.stringify(total));
    localStorage.setItem("goal", JSON.stringify(goal));
  }

  function render() {
    const choreList = document.getElementById("choreList");
    const pendingList = document.getElementById("pendingList");
    const approvedList = document.getElementById("approvedList");

    choreList.innerHTML = "";
    pendingList.innerHTML = "";
    approvedList.innerHTML = "";

    chores.forEach((chore, i) => {
      const div = document.createElement("div");
      div.className = "chore pop";
      div.innerHTML = `
        <span>${chore.name} (${chore.points})</span>
        <div>
          <button onclick="submitChore(${i})">Done</button>
          <button class="delete small" onclick="deleteChore(${i})">X</button>
        </div>
      `;
      choreList.appendChild(div);
    });

    pending.forEach((chore, i) => {
      const div = document.createElement("div");
      div.className = "chore pop";
      div.innerHTML = `
        <span>${chore.name} (${chore.points})</span>
        <button class="approve" onclick="approveChore(${i})">Approve</button>
      `;
      pendingList.appendChild(div);
    });

    approved.forEach(chore => {
      const div = document.createElement("div");
      div.className = "chore pop";
      div.innerHTML = `<span>✅ ${chore.name} (+${chore.points})</span>`;
      approvedList.appendChild(div);
    });

    document.getElementById("total").innerText = "Total Points: " + total;
    updateProgress();
    save();
  }

  function addChore() {
    const name = document.getElementById("newChoreName").value.trim();
    const points = parseInt(document.getElementById("newChorePoints").value);
    if (!name || !points) return alert("Enter chore name and points");
    chores.push({ name, points });
    document.getElementById("newChoreName").value = "";
    document.getElementById("newChorePoints").value = "";
    render();
  }

  function deleteChore(index) {
    if (confirm("Delete this chore?")) {
      chores.splice(index, 1);
      render();
    }
  }

  function submitChore(index) {
    pending.push(chores[index]);
    render();
  }

  function approveChore(index) {
    const pin = prompt("Parent PIN:");
    if (pin !== PARENT_PIN) return alert("Wrong PIN");

    total += pending[index].points;
    approved.push(pending[index]);
    pending.splice(index, 1);

    document.body.classList.add("celebrate");
    setTimeout(() => document.body.classList.remove("celebrate"), 600);

    render();
  }

  function resetDay() {
    if (confirm("Reset today's approvals?")) {
      pending = [];
      approved = [];
      render();
    }
  }

  function resetAll() {
    if (confirm("Reset everything?")) {
      chores = [];
      pending = [];
      approved = [];
      total = 0;
      render();
    }
  }

  function setGoal() {
    const g = parseInt(document.getElementById("goalInput").value);
    if (!g) return alert("Enter a number");
    goal = g;
    render();
  }

  function updateProgress() {
    const percent = Math.min(100, Math.floor((total / goal) * 100));
    const bar = document.getElementById("progress");
    bar.style.width = percent + "%";
    bar.innerText = percent + "%";
  }

  render();
</script>

</body>
</html>

