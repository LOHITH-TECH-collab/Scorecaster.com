# Scorecaster.com
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Games Scoreboard</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background: #f2f2f2;
    }
    .scoreboard {
      background: white;
      padding: 20px;
      margin: 20px auto;
      border-radius: 12px;
      width: 420px;
      box-shadow: 0px 4px 12px rgba(0,0,0,0.2);
    }
    h1 { margin-bottom: 10px; color: #222; }
    .score { font-size: 18px; margin: 10px 0; }
    button {
      padding: 6px 12px;
      margin: 4px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 14px;
    }
    button:hover { opacity: 0.85; }
    .team { font-weight: bold; margin: 12px 0 6px 0; font-size: 18px; }
    select, input {
      padding: 6px;
      margin: 6px;
      border-radius: 6px;
      border: 1px solid #aaa;
    }
    .player { margin: 6px 0; font-size: 16px; }
    .on-strike { color: green; font-weight: bold; }
    #timer { font-size: 22px; margin: 10px 0; font-weight: bold; }
    ul { text-align: left; }
  </style>
</head>
<body>
  
  <h1> Games Scoreboard</h1>

  <!-- Sport Selector -->
  <label for="sport">Choose Game: </label>
  <select id="sport" onchange="resetScore()">
    <option value="cricket">Cricket</option>
    <option value="kabaddi">Kabaddi</option>
    <option value="khokho">Kho-Kho</option>
  </select>
  <br><br>

  <!-- Team Names -->
  <input type="text" id="teamAName" value="Team A" oninput="updateNames()">
  <input type="text" id="teamBName" value="Team B" oninput="updateNames()">

  <div class="scoreboard">
    <!-- Team A -->
    <div class="team" id="teamALabel">Team A</div>
    <div class="score" id="teamAScore"></div>

    <!-- Team B -->
    <div class="team" id="teamBLabel">Team B</div>
    <div class="score" id="teamBScore"></div>

    <!-- Cricket players -->
    <div id="playersSection">
      <input type="text" id="player1Name" value="Player 1" oninput="updateNames()">
      <span class="player on-strike" id="player1Score">0 (0)</span><br>
      <input type="text" id="player2Name" value="Player 2" oninput="updateNames()">
      <span class="player" id="player2Score">0 (0)</span><br>
      <button onclick="changeStrike()">Change Strike</button>
    </div>

    <!-- Cricket overs -->
    <div id="oversSection" class="score">
      Overs: <span id="overs">0.0</span>
    </div>
    <button id="ballBtn" onclick="addBall()">+Ball</button>
    <button id="removeBallBtn" onclick="removeBall()">-Ball</button>

    <!-- Current Bowler -->
    <div id="bowlerSection" class="score">
      Bowler: <input type="text" id="bowlerName" value="Bowler 1" oninput="updateDisplay()"><br>
      Stats: <span id="bowlerStats">0-0 (0.0)</span><br>
      <button onclick="changeBowler()">Change Bowler</button>
    </div>

    <h3>Batting Order</h3>
    <ul id="battingOrder"></ul>

    <h3>Bowling Figures</h3>
    <ul id="bowlingList"></ul>

    <!-- Kho-Kho timer -->
    <div id="timerSection">
      <div id="timer">00:00</div>
      <button onclick="startTimer()">Start</button>
      <button onclick="stopTimer()">Stop</button>
      <button onclick="resetTimer()">Reset</button>
    </div>

    <!-- Action Buttons -->
    <div id="actionButtons"></div>

    <div class="score">Current Game: <span id="gameName">Cricket</span></div>
    <button style="background:red;color:white;" onclick="resetScore()">Reset</button>
  </div>

  <script>
    let runsA = 0, runsB = 0;
    let wicketsA = 0, wicketsB = 0;
    let balls = 0;

    // players (cricket)
    let player1Runs = 0, player2Runs = 0;
    let player1Balls = 0, player2Balls = 0;
    let striker = 1;
    let batsmanCount = 2;

    // bowler
    let bowlerBalls = 0, bowlerRuns = 0, bowlerWkts = 0;

    // timer (Kho-Kho)
    let timerInterval, seconds = 0;

    function updateDisplay() {
      let sport = document.getElementById('sport').value;

      if (sport === "cricket") {
        document.getElementById('gameName').innerText = "Cricket 🏏";
        document.getElementById('teamAScore').innerText = "Runs: " + runsA + " | Wickets: " + wicketsA;
        document.getElementById('teamBScore').innerText = "Runs: " + runsB + " | Wickets: " + wicketsB;
        document.getElementById('playersSection').style.display = "block";
        document.getElementById('oversSection').style.display = "block";
        document.getElementById('ballBtn').style.display = "inline-block";
        document.getElementById('removeBallBtn').style.display = "inline-block";
        document.getElementById('timerSection').style.display = "none";
        document.getElementById('bowlerSection').style.display = "block";

        // overs
        let overs = Math.floor(balls/6) + "." + (balls%6);
        document.getElementById('overs').innerText = overs;

        // players
        document.getElementById('player1Score').innerText = player1Runs + " (" + player1Balls + ")";
        document.getElementById('player2Score').innerText = player2Runs + " (" + player2Balls + ")";
        document.getElementById('player1Score').classList.remove("on-strike");
        document.getElementById('player2Score').classList.remove("on-strike");
        if (striker === 1) document.getElementById('player1Score').classList.add("on-strike");
        else document.getElementById('player2Score').classList.add("on-strike");

        // bowler stats
        let oversBowled = Math.floor(bowlerBalls/6) + "." + (bowlerBalls%6);
        document.getElementById("bowlerStats").innerText =
          `${bowlerRuns}-${bowlerWkts} (${oversBowled})`;

        // action buttons
        document.getElementById('actionButtons').innerHTML = `
          <button onclick="addRuns(1)">+1 Run</button>
          <button onclick="addRuns(4)">+4</button>
          <button onclick="addRuns(6)">+6</button>
          <button onclick="dotBall()">Dot Ball</button>
          <button onclick="subtractRuns()">-1 Run</button>
          <button onclick="addWicket()">+Wicket</button>
          <button onclick="subtractWicket()">-1 Wicket</button>
          <button onclick="noBall()">No Ball</button>
          <button onclick="wideBall()">Wide</button>
        `;
      } else if (sport === "kabaddi") {
        document.getElementById('gameName').innerText = "Kabaddi 🤼";
        document.getElementById('teamAScore').innerText = "Points: " + runsA;
        document.getElementById('teamBScore').innerText = "Points: " + runsB;
        document.getElementById('playersSection').style.display = "none";
        document.getElementById('oversSection').style.display = "none";
        document.getElementById('ballBtn').style.display = "none";
        document.getElementById('removeBallBtn').style.display = "none";
        document.getElementById('timerSection').style.display = "none";
        document.getElementById('bowlerSection').style.display = "none";

        document.getElementById('actionButtons').innerHTML = `
          <button onclick="addPoints('A')">+1 Team A</button>
          <button onclick="subtractPoints('A')">-1 Team A</button>
          <button onclick="addPoints('B')">+1 Team B</button>
          <button onclick="subtractPoints('B')">-1 Team B</button>
        `;
      } else { // Kho-Kho
        document.getElementById('gameName').innerText = "Kho-Kho 🏃‍♂️";
        document.getElementById('teamAScore').innerText = "Points: " + runsA;
        document.getElementById('teamBScore').innerText = "Points: " + runsB;
        document.getElementById('playersSection').style.display = "none";
        document.getElementById('oversSection').style.display = "none";
        document.getElementById('ballBtn').style.display = "none";
        document.getElementById('removeBallBtn').style.display = "none";
        document.getElementById('timerSection').style.display = "block";
        document.getElementById('bowlerSection').style.display = "none";

        document.getElementById('actionButtons').innerHTML = `
          <button onclick="addPoints('A')">+1 Team A</button>
          <button onclick="subtractPoints('A')">-1 Team A</button>
          <button onclick="addPoints('B')">+1 Team B</button>
          <button onclick="subtractPoints('B')">-1 Team B</button>
        `;
      }
    }

    // Cricket functions
    function addRuns(value) {
      runsA += value;
      balls++;
      bowlerRuns += value;
      bowlerBalls++;
      if (striker === 1) { player1Runs += value; player1Balls++; }
      else { player2Runs += value; player2Balls++; }
      if (value % 2 !== 0) changeStrike();
      checkOverComplete();
      updateDisplay();
    }

    // NEW: Dot Ball
    function dotBall() {
      balls++;
      bowlerBalls++;
      if (striker===1) player1Balls++;
      else player2Balls++;
      checkOverComplete();
      updateDisplay();
    }

    function subtractRuns() { if (runsA > 0) runsA--; updateDisplay(); }

    function addWicket() {
      wicketsA++;  balls++;
      bowlerWkts++;  bowlerBalls++;
      // record outgoing batsman
      let outPlayerName, outPlayerRuns, outPlayerBalls;
      if (striker === 1) {
        outPlayerName = document.getElementById("player1Name").value;
        outPlayerRuns = player1Runs; outPlayerBalls = player1Balls;
      } else {
        outPlayerName = document.getElementById("player2Name").value;
        outPlayerRuns = player2Runs; outPlayerBalls = player2Balls;
      }
      let li = document.createElement("li");
      li.textContent = `${outPlayerName} - ${outPlayerRuns} (${outPlayerBalls}) OUT`;
      document.getElementById("battingOrder").appendChild(li);
      // new batsman
      batsmanCount++;
      let newBatsman = "Player " + batsmanCount;
      if (striker === 1) {
        player1Runs = 0; player1Balls = 0;
        document.getElementById("player1Name").value = newBatsman;
      } else {
        player2Runs = 0; player2Balls = 0;
        document.getElementById("player2Name").value = newBatsman;
      }
      changeStrike();
      checkOverComplete();
      updateDisplay();
    }

    // NEW: Subtract wicket
    function subtractWicket() {
      if (wicketsA > 0) {
        wicketsA--;
        bowlerWkts = Math.max(0, bowlerWkts - 1);
        updateDisplay();
      }
    }

    function addBall() {
      balls++; bowlerBalls++;
      if (striker===1) player1Balls++;
      else player2Balls++;
      checkOverComplete();
      updateDisplay();
    }

    function removeBall() { if (balls > 0) balls--; updateDisplay(); }
    function changeStrike() { striker = (striker === 1) ? 2 : 1; updateDisplay(); }

    // NEW: No Ball
    function noBall() {
      runsA++; bowlerRuns++;
      // no ball => extra legal ball
      updateDisplay();
    }

    // NEW: Wide Ball
    function wideBall() {
      runsA++; bowlerRuns++;
      // wide => no extra legal ball
      updateDisplay();
    }

    // Kabaddi & Kho-Kho
    function addPoints(team) { if (team==='A') runsA++; else runsB++; updateDisplay(); }
    function subtractPoints(team) { if (team==='A' && runsA>0) runsA--; else if (team==='B' && runsB>0) runsB--; updateDisplay(); }

    // Kho-Kho Timer
    function startTimer() {
      if (!timerInterval) {
        timerInterval = setInterval(() => {
          seconds++;
          let min = Math.floor(seconds/60).toString().padStart(2,"0");
          let sec = (seconds%60).toString().padStart(2,"0");
          document.getElementById('timer').innerText = `${min}:${sec}`;
        },1000);
      }
    }
    function stopTimer() { clearInterval(timerInterval); timerInterval = null; }
    function resetTimer() { stopTimer(); seconds=0; document.getElementById('timer').innerText="00:00"; }

    // Change Bowler
    function changeBowler() {
      let name = document.getElementById("bowlerName").value;
      let oversBowled = Math.floor(bowlerBalls/6) + "." + (bowlerBalls%6);
      let li = document.createElement("li");
      li.textContent = `${name} - ${bowlerRuns}-${bowlerWkts} (${oversBowled})`;
      document.getElementById("bowlingList").appendChild(li);
      bowlerBalls = 0; bowlerRuns = 0; bowlerWkts = 0;
      document.getElementById("bowlerName").value = "New Bowler";
      updateDisplay();
    }

    // NEW: Check Over Complete
    function checkOverComplete() {
      if (bowlerBalls > 0 && bowlerBalls % 6 === 0) {
        alert("Over complete! Please change the bowler.");
      }
    }

    // Reset & Names
    function resetScore() {
      runsA=runsB=0; wicketsA=wicketsB=0; balls=0;
      player1Runs=player2Runs=0; player1Balls=player2Balls=0; striker=1;
      batsmanCount = 2;
      bowlerBalls=0; bowlerRuns=0; bowlerWkts=0;
      document.getElementById("battingOrder").innerHTML = "";
      document.getElementById("bowlingList").innerHTML = "";
      resetTimer();
      updateDisplay();
    }

    function updateNames() {
      document.getElementById('teamALabel').innerText = document.getElementById('teamAName').value;
      document.getElementById('teamBLabel').innerText = document.getElementById('teamBName').value;
    }

    updateDisplay();
  </script>
</body>
</html>
