<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Spin the Wheel Game</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      font-family: 'Poppins', sans-serif;
      box-sizing: border-box;
    }
    body {
      background: #001e4d;
      color: white;
      text-align: center;
    }
    .app {
      background: #fff;
      color: #001e4d;
      width: 90%;
      max-width: 600px;
      margin: 50px auto;
      border-radius: 10px;
      padding: 30px;
    }
    h1 {
      font-weight: bold;
    }
    #startButton, #spinButton {
      background: #28a745;
      color: white;
      font-weight: bold;
      font-size: 18px;
      padding: 15px 30px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin-top: 20px;
      transition: background 0.3s ease;
    }
    #startButton:hover, #spinButton:hover {
      background: #218838;
    }
    .wheel-container {
      position: relative;
      width: 300px;
      height: 300px;
      margin: 20px auto;
    }
    .wheel {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      border: 5px solid #ccc;
      background: conic-gradient(
        #f39c12 0% 16.67%,
        #3498db 16.67% 33.33%,
        #e74c3c 33.33% 50%,
        #9b59b6 50% 66.67%,
        #2ecc71 66.67% 83.33%,
        #1abc9c 83.33% 100%
      );
      position: absolute;
      top: 0;
      left: 0;
      transform: rotate(0deg);
      transition: transform 3s ease-out;
    }
    .pointer {
      width: 0;
      height: 0;
      border-left: 20px solid transparent;
      border-right: 20px solid transparent;
      border-bottom: 30px solid #ff0000;
      position: absolute;
      top: -25px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 10;
    }
    .question-container {
      padding: 20px;
    }
    .question-container h2 {
      margin-bottom: 20px;
    }
    .answer-option {
      background: #001e4d;
      color: white;
      border: none;
      padding: 15px;
      margin: 10px auto;
      width: 80%;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
      transition: background 0.3s;
    }
    .answer-option:hover {
      background: #004494;
    }
    .hidden {
      display: none;
    }
    .leaderboard-form {
      margin-top: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .leaderboard-form input {
      padding: 10px;
      margin: 10px;
      border-radius: 5px;
      border: 1px solid #ccc;
      width: 80%;
    }
    .leaderboard-table {
      margin: 20px auto;
      width: 80%;
      border-collapse: collapse;
    }
    .leaderboard-table th, .leaderboard-table td {
      padding: 10px;
      border: 1px solid #ccc;
      text-align: center;
    }
  </style>
</head>
<body>
  <!-- Front Page Introduction -->
  <div id="intro-section" class="app">
    <h1>Welcome to the Integrity Booth Game!</h1>
    <p>Test your knowledge and integrity by answering 3 questions correctly.</p>
    <p>Spin the wheel and give it your best shot!</p>
    <button id="startButton">Start</button>
  </div>

  <!-- Spin the Wheel Section -->
  <div id="wheel-section" class="app hidden">
    <h1>Integrity Booth Game!</h1>
    <div class="wheel-container">
      <div class="pointer"></div>
      <div class="wheel" id="wheel"></div>
    </div>
    <button id="spinButton">Spin</button>
  </div>

  <!-- Question Section -->
  <div id="question-section" class="app hidden">
    <div class="question-container">
      <h2 id="question"></h2>
      <div id="answer-buttons"></div>
    </div>
  </div>

  <!-- Leaderboard Section -->
  <div id="final-section" class="app hidden">
    <h1 id="finalMessage"></h1>
    <p id="finalText"></p>
    <form class="leaderboard-form hidden" id="leaderboardForm">
      <input type="text" id="playerName" placeholder="Your name">
      <button type="button" onclick="addHighScore()">Submit</button>
    </form>
    <table class="leaderboard-table hidden" id="leaderboardTable">
      <thead>
        <tr>
          <th>Rank</th>
          <th>Name</th>
          <th>Table Number</th>
        </tr>
      </thead>
      <tbody id="leaderboard"></tbody>
    </table>
    <button onclick="location.reload()">Restart</button>
  </div>

  <script>
    const colors = {
      orange: [
        { question: "Who does the Chief Integrity Officer report to functionally?", answers: [{ text: "PETRONAS Board Risk Committee", correct: true }, { text: "PETRONAS President & Group CEO", correct: false }] },
        { question: "What does IFP stand for?", answers: [{ text: "Integrity Focal Person", correct: true }, { text: "Internal Focal Point", correct: false }] },
        { question: "How many whistleblowing channels are there in PETRONAS?", answers: [{ text: "3", correct: true }, { text: "2", correct: false }] }
      ],
      blue: [
        { question: "What are the five key principles of Adequate Procedures?", answers: [{ text: "T.R.U.S.T", correct: true }, { text: "I.D.E.A.S", correct: false }] },
        { question: "When did Section 17A of the Malaysian Anti-Corruption Commission (MACC) Act 2009 come into force?", answers: [{ text: "1 June 2020", correct: true }, { text: "1 January 2018", correct: false }] },
        { question: "Who is the Chairman of Whistleblowing Committee?", answers: [{ text: "Chief Integrity Officer", correct: true }, { text: "Chairman of Board Risk Committee", correct: false }] }
      ]
    };

    const startButton = document.getElementById("startButton");
    const spinButton = document.getElementById("spinButton");
    const wheel = document.getElementById("wheel");
    const introSection = document.getElementById("intro-section");
    const wheelSection = document.getElementById("wheel-section");
    const questionSection = document.getElementById("question-section");
    const finalSection = document.getElementById("final-section");
    const questionElement = document.getElementById("question");
    const answerButtons = document.getElementById("answer-buttons");
    const leaderboard = document.getElementById("leaderboard");
    const finalMessage = document.getElementById("finalMessage");
    const finalText = document.getElementById("finalText");
    const leaderboardForm = document.getElementById("leaderboardForm");
    const leaderboardTable = document.getElementById("leaderboardTable");

    let usedQuestions = [];
    let correctAnswers = 0;

    startButton.addEventListener("click", () => {
      introSection.classList.add("hidden");
      wheelSection.classList.remove("hidden");
    });

    spinButton.addEventListener("click", () => {
      spinButton.disabled = true;

      const randomAngle = Math.floor(Math.random() * 360) + 1080;
      wheel.style.transform = `rotate(${randomAngle}deg)`;

      setTimeout(() => {
        spinButton.disabled = false;
        wheelSection.classList.add("hidden");
        questionSection.classList.remove("hidden");
        showQuestion();
      }, 3000);
    });

    function getUniqueQuestion() {
      const availableQuestions = Object.keys(colors)
        .flatMap(color => colors[color])
        .filter(q => !usedQuestions.includes(q));
      const randomIndex = Math.floor(Math.random() * availableQuestions.length);
      const question = availableQuestions[randomIndex];
      usedQuestions.push(question);
      return question;
    }

    function showQuestion() {
      resetState();
      const currentQuestion = getUniqueQuestion();
      questionElement.innerText = currentQuestion.question;
      currentQuestion.answers.forEach(answer => {
        const button = document.createElement("button");
        button.innerText = answer.text;
        button.classList.add("answer-option");
        button.onclick = () => {
          if (answer.correct) correctAnswers++;
          if (usedQuestions.length < 3) {
            showQuestion();
          } else {
            showFinalSection();
          }
        };
        answerButtons.appendChild(button);
      });
    }

    function resetState() {
      answerButtons.innerHTML = "";
    }

    function showFinalSection() {
      questionSection.classList.add("hidden");
      finalSection.classList.remove("hidden");

      if (correctAnswers === 3) {
        finalMessage.innerText = "🎉 Congratulations! 🎉";
        finalText.innerText = "You answered all 3 questions correctly! Enter your name for the leaderboard.";
        leaderboardForm.classList.remove("hidden");
        leaderboardTable.classList.remove("hidden");
      } else {
        finalMessage.innerText = "😢 Try Again! 😢";
        finalText.innerText = "You didn't answer all 3 questions correctly. Better luck next time!";
      }
    }

    function addHighScore() {
      const playerName = document.getElementById("playerName").value;
      const row = document.createElement("tr");
      row.innerHTML = `<td>${leaderboard.rows.length + 1}</td><td>${playerName}</td><td>3</td>`;
      leaderboard.appendChild(row);
    }
  </script>
</body>
</html>
