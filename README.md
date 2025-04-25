
<!DOCTYPE html>
<html>
<head>
  <title>Color Match Game</title>
  <style>
    body { font-family: 'Segoe UI'; background: #f5f5f5; padding: 20px; text-align: center; }
    .question { margin-bottom: 30px; background: #e0e0e0; padding: 15px; border-radius: 10px; }
    .answer { display: none; color: green; font-weight: bold; margin-top: 10px; }
    input { margin-top: 10px; padding: 5px; width: 200px; }
    button { margin-left: 10px; padding: 8px; cursor: pointer; }
    #timer { font-size: 20px; margin-top: 20px; font-weight: bold; }
    .winner { font-size: 24px; color: blue; margin-top: 20px; }
  </style>
</head>
<body>

<h1>Color Match Game - 2 Minutes Challenge!</h1>

<!-- Timer Section -->
<div id="timer">Time Left: 120s</div>

<!-- Game Question and Inputs -->
<div class="question">
  <p>Show an object of the color: <strong>Red!</strong></p>
  <input type="text" placeholder="Your Team Name" id="teamName">
  <input type="text" placeholder="Guess the object..." id="objectGuess">
  <button onclick="submitAnswer()">Submit</button>
  <div class="answer" id="answerResult">🎉 Correct Answer! You are the fastest team!</div>
</div>

<!-- Winner Display -->
<div class="winner" id="winner"></div>

<!-- Firebase Scripts -->
<script src="https://www.gstatic.com/firebasejs/9.1.2/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.1.2/firebase-database.js"></script>

<script>
  // Firebase Configuration
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
  };
  
  const app = firebase.initializeApp(firebaseConfig);
  const database = firebase.database(app);
  
  let timer = 120; // 2 minutes
  let isGameActive = true;
  let winnerDeclared = false;
  
  // Start the countdown timer
  function startTimer() {
    const timerInterval = setInterval(() => {
      if (timer <= 0) {
        clearInterval(timerInterval);
        document.getElementById('timer').innerText = "Time's Up!";
        declareWinner();
      } else {
        document.getElementById('timer').innerText = `Time Left: ${timer}s`;
        timer--;
      }
    }, 1000);
  }
  
  // Start the game when page loads
  window.onload = function() {
    startTimer();
  }

  // Submit the answer
  function submitAnswer() {
    if (!isGameActive || winnerDeclared) return;
    
    const teamName = document.getElementById("teamName").value.trim();
    const objectGuess = document.getElementById("objectGuess").value.trim().toLowerCase();
    
    // Check if the answer is correct
    if (objectGuess === "red object" || objectGuess === "apple" || objectGuess === "rose") {
      const timeTaken = 120 - timer; // Time when answered
      
      // Send the result to Firebase
      const resultData = {
        team: teamName,
        time: timeTaken
      };
      firebase.database().ref('answers').push(resultData);
      
      // Show correct answer and mark the game inactive
      document.getElementById('answerResult').style.display = 'block';
      isGameActive = false;

      // Declare winner if they answered correctly
      declareWinner();
    } else {
      alert("Wrong guess! Try again.");
    }
  }

  // Track the fastest correct answer
  function declareWinner() {
    if (winnerDeclared) return;
    
    // Listen to the database for answers
    const answersRef = firebase.database().ref('answers');
    answersRef.on('value', snapshot => {
      const answers = snapshot.val();
      let fastestTeam = null;
      let fastestTime = null;

      for (let key in answers) {
        const answer = answers[key];
        if (fastestTime === null || answer.time < fastestTime) {
          fastestTeam = answer.team;
          fastestTime = answer.time;
        }
      }

      if (fastestTeam) {
        document.getElementById('winner').innerText = `🏆 Winner: ${fastestTeam} with ${fastestTime}s!`;
        winnerDeclared = true;
      }
    });
  }
</script>

</body>
</html>
