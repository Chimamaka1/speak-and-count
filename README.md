<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Speak & Count!</title>
  
  <!-- Web App Metadata for iOS/Android Home Screen -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="Speak & Count">
  <link rel="apple-touch-icon" href="https://fav.farm/⭐">

  <style>
    * {
      box-sizing: border-box;
      user-select: none;
      -webkit-user-select: none;
    }

    body {
      margin: 0;
      padding: 20px;
      font-family: 'Comic Sans MS', 'Chalkboard SE', 'Fredoka', sans-serif;
      background: linear-gradient(135deg, #FF9A9E 0%, #FECFEF 99%, #FECFEF 100%);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-between;
      overflow: hidden;
    }

    h1 {
      color: #fff;
      text-shadow: 2px 3px 0px #ff4757;
      font-size: 2.2rem;
      margin: 10px 0 0 0;
    }

    .mic-button {
      width: 140px;
      height: 140px;
      border-radius: 50%;
      background: #ff4757;
      border: 6px solid #ffffff;
      box-shadow: 0 10px 25px rgba(255, 71, 87, 0.5);
      color: white;
      font-size: 3.5rem;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: transform 0.1s ease, background-color 0.2s ease;
      outline: none;
    }

    .mic-button.listening {
      background: #2ed573;
      animation: pulse 1.2s infinite;
      box-shadow: 0 10px 25px rgba(46, 213, 115, 0.6);
    }

    @keyframes pulse {
      0% { transform: scale(1); }
      50% { transform: scale(1.1); }
      100% { transform: scale(1); }
    }

    .status-text {
      font-size: 1.3rem;
      color: #2f3542;
      font-weight: bold;
      min-height: 30px;
      margin-top: 10px;
    }

    .display-card {
      background: rgba(255, 255, 255, 0.9);
      border-radius: 24px;
      padding: 20px;
      width: 100%;
      max-width: 500px;
      display: flex;
      flex-direction: column;
      align-items: center;
      box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
      min-height: 280px;
    }

    .number-figure {
      font-size: 7rem;
      line-height: 1;
      font-weight: 900;
      color: #3742fa;
      text-shadow: 4px 4px 0px #ffa502;
    }

    .items-grid {
      display: flex;
      flex-wrap: wrap;
      gap: 12px;
      justify-content: center;
      margin-top: 15px;
      max-height: 150px;
      overflow-y: auto;
      width: 100%;
    }

    .item-icon {
      font-size: 2.2rem;
      animation: pop 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }

    @keyframes pop {
      0% { transform: scale(0); }
      100% { transform: scale(1); }
    }

    /* Fallback Quick-Tap Row */
    .quick-bar {
      display: flex;
      gap: 8px;
      overflow-x: auto;
      width: 100%;
      max-width: 500px;
      padding: 10px 0;
    }

    .num-btn {
      flex: 1;
      min-width: 45px;
      height: 45px;
      background: #ffffff;
      border: 2px solid #70a1ff;
      border-radius: 12px;
      font-size: 1.2rem;
      font-weight: bold;
      color: #3742fa;
      cursor: pointer;
    }

    .num-btn:active {
      background: #70a1ff;
      color: white;
    }
  </style>
</head>
<body>

  <h1>Speak & Count! 🎙️</h1>

  <button class="mic-button" id="micBtn" onclick="toggleListening()">🎤</button>
  <div class="status-text" id="statusText">Tap the mic and say a number!</div>

  <div class="display-card">
    <div class="number-figure" id="numberDisplay">5</div>
    <div class="items-grid" id="itemsGrid"></div>
  </div>

  <!-- Fallback row for manual tapping -->
  <div class="quick-bar">
    <button class="num-btn" onclick="setNumber(1)">1</button>
    <button class="num-btn" onclick="setNumber(2)">2</button>
    <button class="num-btn" onclick="setNumber(3)">3</button>
    <button class="num-btn" onclick="setNumber(4)">4</button>
    <button class="num-btn" onclick="setNumber(5)">5</button>
    <button class="num-btn" onclick="setNumber(6)">6</button>
    <button class="num-btn" onclick="setNumber(7)">7</button>
    <button class="num-btn" onclick="setNumber(8)">8</button>
    <button class="num-btn" onclick="setNumber(9)">9</button>
    <button class="num-btn" onclick="setNumber(10)">10</button>
  </div>

  <script>
    let currentNumber = 5;
    let isListening = false;
    let recognition = null;

    // Map common words to digits
    const wordToNumber = {
      'zero': 0, 'one': 1, 'two': 2, 'to': 2, 'too': 2, 'three': 3,
      'four': 4, 'for': 4, 'five': 5, 'six': 6, 'seven': 7, 'eight': 8,
      'ate': 8, 'nine': 9, 'ten': 10, 'eleven': 11, 'twelve': 12
    };

    // Setup Speech Recognition
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

    if (SpeechRecognition) {
      recognition = new SpeechRecognition();
      recognition.continuous = false;
      recognition.interimResults = false;
      recognition.lang = 'en-US';

      recognition.onstart = () => {
        isListening = true;
        document.getElementById('micBtn').classList.add('listening');
        document.getElementById('statusText').innerText = "Listening... Say a number!";
      };

      recognition.onend = () => {
        isListening = false;
        document.getElementById('micBtn').classList.remove('listening');
      };

      recognition.onresult = (event) => {
        const transcript = event.results[0][0].transcript.toLowerCase().trim();
        document.getElementById('statusText').innerText = `You said: "${transcript}"`;
        
        let foundNum = parseInt(transcript);

        if (isNaN(foundNum)) {
          // Check word dictionary
          const words = transcript.split(' ');
          for (let word of words) {
            if (wordToNumber[word] !== undefined) {
              foundNum = wordToNumber[word];
              break;
            }
          }
        }

        if (!isNaN(foundNum) && foundNum >= 0 && foundNum <= 50) {
          setNumber(foundNum);
        } else if (!isNaN(foundNum) && foundNum > 50) {
          document.getElementById('statusText').innerText = "Try a smaller number! (1-50)";
        } else {
          speak("I didn't catch a number. Try again!");
        }
      };
    } else {
      document.getElementById('statusText').innerText = "Voice input isn't supported on this browser. Use the buttons below!";
    }

    function toggleListening() {
      if (!recognition) return;
      if (isListening) {
        recognition.stop();
      } else {
        recognition.start();
      }
    }

    function setNumber(num) {
      currentNumber = num;
      document.getElementById('numberDisplay').innerText = num;
      renderStars(num);
      speak(`${num}!`);
    }

    function renderStars(count) {
      const grid = document.getElementById('itemsGrid');
      grid.innerHTML = '';
      const icons = ['⭐', '🎈', '🍎', '🐱', '🚀'];
      const chosenIcon = icons[count % icons.length];

      // Cap displayed icons at 50 so it doesn't freeze the screen
      const displayCount = Math.min(count, 50);

      for (let i = 0; i < displayCount; i++) {
        const span = document.createElement('span');
        span.className = 'item-icon';
        span.innerText = chosenIcon;
        grid.appendChild(span);
      }
    }

    function speak(text) {
      if ('speechSynthesis' in window) {
        window.speechSynthesis.cancel(); // Stop prior audio
        const utterance = new SpeechSynthesisUtterance(text);
        utterance.pitch = 1.2; // Slightly higher/friendly pitch for kids
        utterance.rate = 0.9;
        window.speechSynthesis.speak(utterance);
      }
    }

    // Initial render
    renderStars(5);
  </script>
</body>
</html>
