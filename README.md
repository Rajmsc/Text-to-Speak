# Text-to-Speak
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Text To Speech</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #000; /* Default to dark mode */
            color: #b3b3b3; /* Text color for dark mode */
            margin: 0;
            padding: 20px;
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        h1, h3 {
            color: #b3b3b3; /* Text color for dark mode */
        }
        #textToSpeak {
            width: 100%;
            height: 200px;
            border-radius: 5px;
            border: 1px solid #444;
            padding: 10px;
            font-size: 16px;
            margin-bottom: 20px;
            background-color: #222;
            color: #b3b3b3; /* Text color for dark mode */
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        button {
            padding: 10px 20px;
            margin: 5px;
            border: none;
            border-radius: 5px;
            background-color: #4CAF50;
            color: white;
            font-size: 16px;
            cursor: pointer;
            transition: background-color 0.3s ease;
            display: inline-block;
        }
        button:hover {
            background-color: #45a049;
        }
        #currentSentence {
            margin-top: 20px;
            padding: 10px;
            border: 2px solid #444;
            border-radius: 5px;
            background-color: #222;
            color: #b3b3b3; /* Text color for dark mode */
            font-size: 18px;
            margin-bottom: 20px;
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        .buttonContainer {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
        }
        #speedControl {
            width: 100px;
            margin: 5px;
        }
        @media screen and (max-width: 600px) {
            button {
                padding: 10px;
                font-size: 14px;
                width: 100%;
                margin: 5px 0;
            }
            #speedControl {
                width: 100%;
            }
        }
        .light-mode {
            background-color: #f0f0f5; /* Light mode background */
            color: #333; /* Text color for light mode */
        }
        .light-mode #textToSpeak, .light-mode #currentSentence {
            background-color: #fff; /* Light mode background */
            color: #333; /* Text color for light mode */
            border: 1px solid #ccc; /* Light mode border */
        }
    </style>
</head>
<body class="dark-mode">
    <div class="headerContainer">
        <h1>Reader</h1>
        <h3 id="estimatedTime"></h3>
    </div>
    <div id="currentSentence"> <span id="currentLine"></span></div>
    <button onclick="startSpeaking()">Play</button>
    <button onclick="pauseSpeaking()">Pause</button>
    <button onclick="forwardOneLine()">Forward</button>
    <button onclick="forwardTenLines()">Forward 10</button>
    <textarea name="textToSpeak" id="textToSpeak" placeholder="Type or paste your text here..." oninput="updateEstimatedReadingTime()"></textarea>
    <div class="buttonContainer">
        <button onclick="forwardFiftyLines()">Forward 50</button>
        <button onclick="forwardHundredLines()">Forward 100</button>
        <label for="speedControl"></label>
        <input type="range" id="speedControl" name="speedControl" min="0.5" max="2" value="1" step="0.1" onchange="changeSpeed()">
        <button onclick="toggleLightMode()">Light</button>
    </div>

    <script>
        // Variables to manage the text and speech
        let textArea = document.getElementById("textToSpeak");
        let lines = [];
        let currentIndex = 0;
        let speechInstance;
        let isPaused = false;
        let speedRate = 1;
        let isLightMode = false;
        let totalSecondsRemaining = 0;

        // Function to start speaking the text
        function startSpeaking() {
            if (isPaused) {
                isPaused = false;
                window.speechSynthesis.resume();
            } else {
                lines = textArea.value.split("\n");
                currentIndex = 0;
                calculateTotalTime();
                speakCurrentLine();
            }
        }

        // Function to speak the current line
        function speakCurrentLine() {
            if (currentIndex < lines.length) {
                const line = lines[currentIndex];
                highlightCurrentLine(line);

                document.getElementById("currentLine").textContent = line;

                speechInstance = new SpeechSynthesisUtterance(line);
                speechInstance.rate = speedRate;
                speechInstance.onend = onLineEnd;
                window.speechSynthesis.speak(speechInstance);
            } else {
                clearHighlight();
                document.getElementById("currentLine").textContent = "Done reading!";
                updateEstimatedReadingTime(0);
            }
        }

        // Function to handle the end of a line
        function onLineEnd() {
            if (!isPaused) {
                currentIndex++;
                totalSecondsRemaining -= calculateLineTime(lines[currentIndex - 1]);
                updateEstimatedReadingTime(totalSecondsRemaining);
                speakCurrentLine();
            }
        }

        // Function to highlight the current line in the textarea
        function highlightCurrentLine(line) {
            const text = textArea.value;
            const startIndex = text.indexOf(line);
            const endIndex = startIndex + line.length;

            textArea.setSelectionRange(startIndex, endIndex);
            textArea.focus();
        }

        // Function to clear the highlight from the textarea
        function clearHighlight() {
            textArea.setSelectionRange(0, 0);
        }

        // Function to pause speaking
        function pauseSpeaking() {
            isPaused = true;
            window.speechSynthesis.pause();
        }

        // Function to move forward one line
        function forwardOneLine() {
            moveForward(1);
        }

        // Function to move forward ten lines
        function forwardTenLines() {
            moveForward(10);
        }

        // Function to move forward fifty lines
        function forwardFiftyLines() {
            moveForward(50);
        }

        // Function to move forward one hundred lines
        function forwardHundredLines() {
            moveForward(100);
        }

        // Function to move forward a specified number of lines
        function moveForward(step) {
            window.speechSynthesis.cancel();
            currentIndex = Math.min(currentIndex + step, lines.length - 1);
            totalSecondsRemaining -= step * calculateAverageLineTime();
            updateEstimatedReadingTime(totalSecondsRemaining);
            speakCurrentLine();
        }

        // Function to change the speaking speed
        function changeSpeed() {
            speedRate = document.getElementById("speedControl").value;
            calculateTotalTime();
            updateEstimatedReadingTime(totalSecondsRemaining);
        }

        // Function to toggle between light and dark mode
        function toggleLightMode() {
            isLightMode = !isLightMode;
            document.body.classList.toggle("light-mode", isLightMode);
        }

        // Function to calculate the time for a line
        function calculateLineTime(line) {
            const words = line.split(' ').length;
            const estimatedTimeInSeconds = words / (200 * speedRate / 60); // Assuming 200 words per minute as average reading speed
            return estimatedTimeInSeconds;
        }

        // Function to calculate the average time for a line
        function calculateAverageLineTime() {
            const totalWords = lines.join(' ').split(' ').length;
            const averageWordsPerLine = totalWords / lines.length;
            const estimatedTimeInSeconds = averageWordsPerLine / (200 * speedRate / 60);
            return estimatedTimeInSeconds;
        }

        // Function to calculate the total time for all lines
        function calculateTotalTime() {
            totalSecondsRemaining = lines.reduce((total, line) => total + calculateLineTime(line), 0);
        }

        // Function to update the estimated reading time
        function updateEstimatedReadingTime(secondsRemaining) {
            const minutes = Math.floor(secondsRemaining / 60);
            const seconds = Math.round(secondsRemaining % 60);
            document.getElementById("estimatedTime").textContent = `Duration: ${minutes} min ${seconds} sec`;
        }

        // Call updateEstimatedReadingTime initially to set the estimated time when the page loads
        updateEstimatedReadingTime(totalSecondsRemaining);
    </script>
</body>
</html>
