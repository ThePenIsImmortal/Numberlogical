# Numberlogical
6 Numbers, 6 guesses, numbers can repeat any time in any order
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Numberlogical</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for the game, if needed, beyond Tailwind */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 1rem;
        }
        .game-container {
            background-color: #ffffff;
            border-radius: 1.5rem; /* More rounded corners */
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            padding: 2rem;
            max-width: 500px;
            width: 100%;
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
        }
        .digit-box {
            width: 2.5rem; /* Fixed width for digits */
            height: 2.5rem; /* Fixed height for digits */
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            font-size: 1.25rem;
            border-radius: 0.5rem; /* Rounded corners for digit boxes */
            color: #333;
            background-color: #e0e7eb; /* Light gray for default */
            border: 1px solid #cbd5e1; /* Light border */
        }
        .digit-box.green {
            background-color: #10b981; /* Tailwind green-500 */
            color: white;
        }
        .digit-box.yellow {
            background-color: #f59e0b; /* Tailwind yellow-500 */
            color: white;
        }
        .digit-box.black {
            background-color: #4b5563; /* Tailwind gray-600 */
            color: white;
        }
        .message-box {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: #fff;
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            z-index: 1000;
            text-align: center;
            display: none; /* Hidden by default */
            max-width: 90%;
        }
        .overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            z-index: 999;
            display: none; /* Hidden by default */
        }
        .button-style {
            padding: 0.75rem 1.5rem;
            border-radius: 0.75rem;
            font-weight: bold;
            transition: background-color 0.3s ease, transform 0.1s ease;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .button-style:hover {
            transform: translateY(-2px);
        }
        .button-primary {
            background-color: #3b82f6; /* Tailwind blue-500 */
            color: white;
        }
        .button-primary:hover {
            background-color: #2563eb; /* Tailwind blue-600 */
        }
        .button-secondary {
            background-color: #6b7280; /* Tailwind gray-500 */
            color: white;
        }
        .button-secondary:hover {
            background-color: #4b5563; /* Tailwind gray-600 */
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1 class="text-3xl font-extrabold text-center text-gray-800 mb-4">Numberlogical</h1>

        <!-- Secret Number Input Section -->
        <div id="secretInputSection" class="flex flex-col gap-4">
            <p class="text-gray-700 text-center text-lg">Enter your 6-digit secret number:</p>
            <input type="password" id="secretNumberInput" class="w-full p-3 border border-gray-300 rounded-lg text-center text-xl tracking-widest" maxlength="6" pattern="\d{6}" placeholder="******">
            <button id="startGameButton" class="button-style button-primary">Start Game</button>
            <p id="secretInputError" class="text-red-500 text-center text-sm mt-2 hidden">Please enter a valid 6-digit number.</p>
        </div>

        <!-- Game Play Section -->
        <div id="gamePlaySection" class="hidden flex flex-col gap-4">
            <div class="flex flex-col items-center gap-2">
                <p class="text-gray-700 text-lg">Guess the 6-digit number!</p>
                <p class="text-gray-600 text-sm">Guesses left: <span id="guessesLeft" class="font-bold text-blue-600">6</span></p>
            </div>

            <div class="flex gap-2 justify-center mb-4">
                <input type="text" id="guessInput" class="w-full p-3 border border-gray-300 rounded-lg text-center text-xl tracking-widest" maxlength="6" pattern="\d{6}" placeholder="Enter your guess">
                <button id="guessButton" class="button-style button-primary">Guess</button>
            </div>
            <p id="guessError" class="text-red-500 text-center text-sm mt-2 hidden">Please enter a valid 6-digit number.</p>

            <!-- Guesses History Display -->
            <div id="guessesHistory" class="flex flex-col gap-3">
                <!-- Guesses will be dynamically added here -->
            </div>

            <button id="howToPlayButton" class="button-style button-secondary mt-4">How to Play</button>
            <button id="newGameButton" class="button-style button-secondary mt-2 hidden">New Game</button>
        </div>
    </div>

    <!-- Custom Message Box -->
    <div id="overlay" class="overlay"></div>
    <div id="messageBox" class="message-box">
        <p id="messageText" class="text-xl font-semibold mb-4 text-gray-800"></p>
        <button id="closeMessageBox" class="button-style button-primary">OK</button>
    </div>

    <script>
        // Global variables for game state
        let SECRET_NUMBER = '';
        let GUESS_COUNT = 0;
        const MAX_GUESSES = 6;

        // Get DOM elements
        const secretInputSection = document.getElementById('secretInputSection');
        const secretNumberInput = document.getElementById('secretNumberInput');
        const startGameButton = document.getElementById('startGameButton');
        const secretInputError = document.getElementById('secretInputError');

        const gamePlaySection = document.getElementById('gamePlaySection');
        const guessesLeftSpan = document.getElementById('guessesLeft');
        const guessInput = document.getElementById('guessInput');
        const guessButton = document.getElementById('guessButton');
        const guessError = document.getElementById('guessError');
        const guessesHistory = document.getElementById('guessesHistory');
        const newGameButton = document.getElementById('newGameButton');
        const howToPlayButton = document.getElementById('howToPlayButton'); // New element

        const messageBox = document.getElementById('messageBox');
        const messageText = document.getElementById('messageText');
        const closeMessageBox = document.getElementById('closeMessageBox');
        const overlay = document.getElementById('overlay');

        // Game Rules Text
        const GAME_RULES = `
            <h3 class="font-bold text-xl mb-2">How to Play Numberlogical:</h3>
            <ul class="list-disc list-inside text-left space-y-1">
                <li><strong class="font-semibold">Objective:</strong> Guess a 6-digit secret number.</li>
                <li><strong class="font-semibold">Digits:</strong> Can be any number from 0 to 9.</li>
                <li><strong class="font-semibold">Repetition:</strong> Numbers can repeat any number of times.</li>
                <li><strong class="font-semibold">Guesses:</strong> You get 6 rows (guesses) to figure out the code.</li>
                <li><strong class="font-semibold">Feedback:</strong> After each guess, you'll see colored boxes:
                    <ul class="list-circle list-inside ml-4">
                        <li><span class="inline-block w-4 h-4 rounded-full bg-green-500 mr-2"></span><strong class="font-semibold">Green:</strong> Right number, right spot.</li>
                        <li><span class="inline-block w-4 h-4 rounded-full bg-yellow-500 mr-2"></span><strong class="font-semibold">Yellow:</strong> Right number, wrong spot.</li>
                        <li><span class="inline-block w-4 h-4 rounded-full bg-gray-600 mr-2"></span><strong class="font-semibold">Black:</strong> Number is not in the code or all instances of that number are accounted for.</li>
                    </ul>
                </li>
                <li><strong class="font-semibold">Key Rule for Repeats:</strong> When a number appears multiple times in the secret code, and you guess it, feedback prioritizes "green" matches first. Then, for "yellow" matches, it only indicates a "yellow" if there are still instances of that number in the secret code that haven't been matched by a "green" or a previous "yellow" within that guess.
            </ul>
        `;

        /**
         * Displays a custom message box.
         * @param {string} message - The message to display.
         */
        function showMessageBox(message) {
            messageText.innerHTML = message; // Use innerHTML to allow for rich text
            messageBox.style.display = 'block';
            overlay.style.display = 'block';
        }

        /**
         * Hides the custom message box.
         */
        function hideMessageBox() {
            messageBox.style.display = 'none';
            overlay.style.display = 'none';
        }

        /**
         * Validates if the input is a 6-digit number.
         * @param {string} input - The string to validate.
         * @returns {boolean} - True if valid, false otherwise.
         */
        function isValidNumber(input) {
            return /^\d{6}$/.test(input);
        }

        /**
         * Starts a new game. Resets all game state and UI.
         */
        function startNewGame() {
            SECRET_NUMBER = '';
            GUESS_COUNT = 0;
            guessesLeftSpan.textContent = MAX_GUESSES;
            guessInput.value = '';
            guessesHistory.innerHTML = '';
            guessError.classList.add('hidden');
            secretInputError.classList.add('hidden');
            newGameButton.classList.add('hidden');
            guessButton.disabled = false;
            guessInput.disabled = false;
            howToPlayButton.classList.remove('hidden'); // Ensure how to play is visible

            secretInputSection.classList.remove('hidden');
            gamePlaySection.classList.add('hidden');
            secretNumberInput.value = ''; // Clear previous secret
            secretNumberInput.focus();
        }

        /**
         * Compares the guess to the secret number and returns feedback.
         * Implements the Wordle-like logic with repetition handling.
         * @param {string} guess - The player's guess.
         * @param {string} secret - The secret number.
         * @returns {Array<string>} - An array of feedback colors ('green', 'yellow', 'black') for each digit.
         */
        function getFeedback(guess, secret) {
            const feedback = Array(6).fill('black');
            const secretChars = secret.split('');
            const guessChars = guess.split('');

            // Step 1: Mark Greens and remove matched chars from a mutable copy of secret
            const tempSecret = [...secretChars];
            const tempGuess = [...guessChars];

            for (let i = 0; i < 6; i++) {
                if (tempGuess[i] === tempSecret[i]) {
                    feedback[i] = 'green';
                    tempSecret[i] = null; // Mark as used
                    tempGuess[i] = null; // Mark as used
                }
            }

            // Step 2: Mark Yellows with remaining chars
            for (let i = 0; i < 6; i++) {
                if (tempGuess[i] !== null) { // If not already green
                    const secretIndex = tempSecret.indexOf(tempGuess[i]);
                    if (secretIndex !== -1) {
                        feedback[i] = 'yellow';
                        tempSecret[secretIndex] = null; // Mark as used
                    }
                }
            }
            return feedback;
        }


        /**
         * Displays a single guess and its feedback in the history.
         * @param {string} guess - The guessed number.
         * @param {Array<string>} feedback - Array of feedback colors.
         */
        function displayGuess(guess, feedback) {
            const guessRow = document.createElement('div');
            guessRow.className = 'flex items-center gap-4 p-2 bg-gray-50 rounded-lg shadow-sm';

            const guessNumber = document.createElement('span');
            guessNumber.className = 'font-bold text-gray-700 text-lg w-6 text-center';
            guessNumber.textContent = `${GUESS_COUNT}.`;
            guessRow.appendChild(guessNumber);

            const guessDigitsContainer = document.createElement('div');
            guessDigitsContainer.className = 'flex gap-1';
            guess.split('').forEach((digit, index) => {
                const digitBox = document.createElement('div');
                digitBox.className = `digit-box ${feedback[index]}`;
                digitBox.textContent = digit;
                guessDigitsContainer.appendChild(digitBox);
            });
            guessRow.appendChild(guessDigitsContainer);

            guessesHistory.prepend(guessRow); // Add to the top
        }

        /**
         * Handles the "Start Game" button click.
         */
        startGameButton.addEventListener('click', () => {
            const secret = secretNumberInput.value;
            if (isValidNumber(secret)) {
                SECRET_NUMBER = secret;
                secretInputSection.classList.add('hidden');
                gamePlaySection.classList.remove('hidden');
                guessInput.focus();
                guessesLeftSpan.textContent = MAX_GUESSES - GUESS_COUNT;
            } else {
                secretInputError.classList.remove('hidden');
            }
        });

        /**
         * Handles the "Guess" button click.
         */
        guessButton.addEventListener('click', () => {
            const guess = guessInput.value;
            if (!isValidNumber(guess)) {
                guessError.classList.remove('hidden');
                return;
            }
            guessError.classList.add('hidden');

            GUESS_COUNT++;
            guessesLeftSpan.textContent = MAX_GUESSES - GUESS_COUNT;

            const feedback = getFeedback(guess, SECRET_NUMBER);
            displayGuess(guess, feedback);

            // Check for win condition
            const isWin = feedback.every(color => color === 'green');
            if (isWin) {
                showMessageBox(`Congratulations! You guessed the number ${SECRET_NUMBER} in ${GUESS_COUNT} guesses!`);
                guessButton.disabled = true;
                guessInput.disabled = true;
                newGameButton.classList.remove('hidden');
            } else { // Only show new game button if not a win and out of guesses
                if (GUESS_COUNT >= MAX_GUESSES) {
                    showMessageBox(`Game Over! You ran out of guesses. The secret number was ${SECRET_NUMBER}.`);
                    guessButton.disabled = true;
                    guessInput.disabled = true;
                    newGameButton.classList.remove('hidden');
                }
            }


            guessInput.value = ''; // Clear input field
            guessInput.focus();
        });

        /**
         * Handles the "New Game" button click.
         */
        newGameButton.addEventListener('click', startNewGame);

        /**
         * Handles the "How to Play" button click.
         */
        howToPlayButton.addEventListener('click', () => {
            showMessageBox(GAME_RULES);
        });

        /**
         * Handles closing the message box.
         */
        closeMessageBox.addEventListener('click', hideMessageBox);

        // Initialize game on load
        startNewGame();
    </script>
</body>
</html>
