<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Dojo Plunder</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; margin: 20px; background: #f4f4f4; }
    .hidden { display: none; }
    .group { margin: 5px; }
    button { margin: 10px; padding: 10px 20px; cursor: pointer; }
    table { margin: 20px auto; border-collapse: collapse; width: 70%; background: #fff; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; }
    .progress-container { width: 100%; background: #eee; border-radius: 5px; overflow: hidden; }
    .progress-bar { height: 20px; text-align: right; padding-right: 5px; color: #fff; font-size: 12px; }
    .green { background: #4caf50; }
    .yellow { background: #ff9800; }
    .red { background: #f44336; }
    #roundTracker { font-size: 20px; margin: 15px; font-weight: bold; }
  </style>
</head>
<body>
  <h1>Dojo Plunder</h1>

  <!-- Step 1: Select number of groups -->
  <div id="setup">
    <label for="groupCount">Number of groups (2–6): </label>
    <input type="number" id="groupCount" min="2" max="6">
    <button onclick="createGroupInputs()">Confirm</button>
    <div id="groupInputs"></div>
    <button id="startGame" class="hidden" onclick="startGame()">Start Game</button>
  </div>

  <!-- Step 2: Game area -->
  <div id="gameArea" class="hidden">
    <div id="roundTracker">Round 1</div>
    <h2>Groups & Points</h2>
    <table id="scoreboard">
      <thead>
        <tr><th>Group</th><th>Points</th><th>Progress</th></tr>
      </thead>
      <tbody></tbody>
    </table>
    <button id="startRound" onclick="startRound()">Start Round</button>
    <div id="challengeArea" class="hidden">
      <p id="challengeText"></p>
      <input type="number" id="betPoints" placeholder="Bet points">
      <button onclick="confirmBet()">Confirm Bet</button>
    </div>
    <div id="timerArea" class="hidden">
      <p id="timer">2:00</p>
      <button onclick="startTimer()">Start Timer</button>
    </div>
    <div id="resultArea" class="hidden">
      <button onclick="resolveChallenge(true)">Succeeded</button>
      <button onclick="resolveChallenge(false)">Failed</button>
    </div>
  </div>

<script>
let groups = [];
let currentChallenger = null;
let currentOpponent = null;
let bet = 0;
let challengedGroups = new Set();
let timerInterval;
let round = 1;

function createGroupInputs() {
  const count = document.getElementById("groupCount").value;
  const container = document.getElementById("groupInputs");
  container.innerHTML = "";
  for (let i = 0; i < count; i++) {
    container.innerHTML += `<div class="group">Group ${i+1} Name: <input type="text" id="group${i}"></div>`;
  }
  document.getElementById("startGame").classList.remove("hidden");
}

function startGame() {
  const count = document.getElementById("groupCount").value;
  groups = [];
  for (let i = 0; i < count; i++) {
    const name = document.getElementById(`group${i}`).value || `Group ${i+1}`;
    groups.push({ name, points: 10 });
  }
  document.getElementById("setup").classList.add("hidden");
  document.getElementById("gameArea").classList.remove("hidden");
  updateScoreboard();
}

function updateScoreboard() {
  const tbody = document.querySelector("#scoreboard tbody");
  tbody.innerHTML = "";
  groups.forEach(g => {
    const percent = Math.max(0, g.points) * 10; // 10 points = 100%
    let colorClass = "green";
    if (g.points <= 3) colorClass = "red";
    else if (g.points <= 7) colorClass = "yellow";

    tbody.innerHTML += `
      <tr>
        <td>${g.name}</td>
        <td>${g.points}</td>
        <td>
          <div class="progress-container">
            <div class="progress-bar ${colorClass}" style="width:${percent}%">${g.points} pts</div>
          </div>
        </td>
      </tr>`;
  });
}

function startRound() {
  if (challengedGroups.size === groups.length) {
    challengedGroups.clear(); // reset for next round
    round++;
    document.getElementById("roundTracker").innerText = `Round ${round}`;
    alert(`All groups challenged! Starting Round ${round}.`);
  }
  currentChallenger = groups[Math.floor(Math.random() * groups.length)];
  while (challengedGroups.has(currentChallenger.name)) {
    currentChallenger = groups[Math.floor(Math.random() * groups.length)];
  }
  challengedGroups.add(currentChallenger.name);
  document.getElementById("challengeText").innerText = `${currentChallenger.name} will challenge...`;
  document.getElementById("challengeArea").classList.remove("hidden");
}

function confirmBet() {
  bet = parseInt(document.getElementById("betPoints").value);
  if (isNaN(bet) || bet <= 0 || bet > currentChallenger.points) {
    alert("Invalid bet amount!");
    return;
  }
  // pick opponent
  do {
    currentOpponent = groups[Math.floor(Math.random() * groups.length)];
  } while (currentOpponent.name === currentChallenger.name);
  document.getElementById("challengeText").innerText = `${currentChallenger.name} bets ${bet} points against ${currentOpponent.name}!`;
  document.getElementById("timerArea").classList.remove("hidden");
}

function startTimer() {
  let time = 120;
  document.getElementById("timer").innerText = "2:00";
  clearInterval(timerInterval);
  timerInterval = setInterval(() => {
    time--;
    let minutes = Math.floor(time / 60);
    let seconds = time % 60;
    document.getElementById("timer").innerText = `${minutes}:${seconds < 10 ? "0" : ""}${seconds}`;
    if (time <= 0) {
      clearInterval(timerInterval);
      alert("Time's up!");
      document.getElementById("resultArea").classList.remove("hidden");
    }
  }, 1000);
}

function resolveChallenge(success) {
  if (success) {
    currentChallenger.points += bet;
    currentOpponent.points -= bet;
  } else {
    currentChallenger.points -= bet;
    currentOpponent.points += bet;
  }
  updateScoreboard();
  document.getElementById("challengeArea").classList.add("hidden");
  document.getElementById("timerArea").classList.add("hidden");
  document.getElementById("resultArea").classList.add("hidden");
}
</script>
</body>
</html>
