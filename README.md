<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Mobile Calculator</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: manipulation;
        }

        body {
            background-color: #000;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
            -webkit-tap-highlight-color: transparent;
        }

        .calculator {
            width: 100%;
            max-width: 500px;
            padding: 10px;
            background-color: #1a1a1a;
            border-radius: 15px;
        }

        .display-container {
            margin-bottom: 15px;
            padding: 15px;
            background-color: #000;
            border-radius: 10px;
        }

        #history {
            font-size: 1.2rem;
            color: #666;
            text-align: right;
            min-height: 1.5rem;
            margin-bottom: 5px;
        }

        #display {
            width: 100%;
            border: none;
            background: transparent;
            color: #fff;
            font-size: 2.5rem;
            text-align: right;
            padding: 5px 0;
        }

        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 8px;
        }

        button {
            padding: 20px;
            font-size: 1.5rem;
            border: none;
            border-radius: 10px;
            background-color: #333;
            color: #fff;
            cursor: pointer;
            transition: background-color 0.2s;
            touch-action: manipulation;
        }

        button:active {
            background-color: #444;
        }

        .operator {
            background-color: #4CAF50;
        }

        .operator.active {
            background-color: #45a049;
            box-shadow: 0 0 10px rgba(76, 175, 80, 0.5);
        }

        .equals {
            background-color: #2196F3;
        }

        .error-message {
            color: #ff4444;
            font-size: 0.9rem;
            text-align: right;
            height: 1rem;
            margin-top: 5px;
        }

        @media (max-width: 480px) {
            .calculator {
                border-radius: 0;
                height: 100vh;
                max-width: none;
                padding: 10px 5px;
            }

            button {
                padding: 15px;
                font-size: 1.3rem;
            }

            #display {
                font-size: 2rem;
            }
        }
    </style>
</head>
<body>
    <div class="calculator">
        <div class="display-container">
            <div id="history"></div>
            <input type="text" id="display" readonly>
            <div id="error" class="error-message"></div>
        </div>
        <div class="buttons">
            <button onclick="clearDisplay()">AC</button>
            <button onclick="deleteLast()">DEL</button>
            <button onclick="addPercent()">%</button>
            <button class="operator" ontouchstart="handleOperatorTouch('/')">/</button>
            <button ontouchstart="appendNumber('7')">7</button>
            <button ontouchstart="appendNumber('8')">8</button>
            <button ontouchstart="appendNumber('9')">9</button>
            <button class="operator" ontouchstart="handleOperatorTouch('*')">×</button>
            <button ontouchstart="appendNumber('4')">4</button>
            <button ontouchstart="appendNumber('5')">5</button>
            <button ontouchstart="appendNumber('6')">6</button>
            <button class="operator" ontouchstart="handleOperatorTouch('-')">-</button>
            <button ontouchstart="appendNumber('1')">1</button>
            <button ontouchstart="appendNumber('2')">2</button>
            <button ontouchstart="appendNumber('3')">3</button>
            <button class="operator" ontouchstart="handleOperatorTouch('+')">+</button>
            <button ontouchstart="appendNumber('0')" style="grid-column: span 2">0</button>
            <button ontouchstart="appendNumber('.')">.</button>
            <button class="equals" ontouchstart="calculate()">=</button>
        </div>
    </div>

    <script>
        let currentNumber = '';
        let previousNumber = '';
        let operation = null;
        let lastTouchTime = 0;
        let lastOperator = null;

        // Обработчик событий клавиатуры
        document.addEventListener('keydown', handleKeyboardInput);

        function showError(message) {
            const errorElement = document.getElementById('error');
            errorElement.textContent = message;
            setTimeout(() => errorElement.textContent = '', 2000);
        }

        function handleKeyboardInput(e) {
            if (e.key >= '0' && e.key <= '9' || e.key === '.') {
                appendNumber(e.key);
            }
            else if (['+', '-', '*', '/'].includes(e.key)) {
                handleOperatorTouch(e.key);
            }
            else if (e.key === 'Enter') {
                e.preventDefault();
                calculate();
            }
            else if (e.key === 'Backspace') {
                deleteLast();
            }
            else if (e.key === 'Escape') {
                clearDisplay();
            }
        }

        function updateDisplay() {
            document.getElementById('display').value = currentNumber || '0';
            document.getElementById('history').textContent = 
                previousNumber ? `${previousNumber} ${operation || ''}` : '';
        }

        function appendNumber(num) {
            if (num === '.' && currentNumber.includes('.')) {
                showError('Decimal point already exists');
                return;
            }
            currentNumber += num;
            updateDisplay();
        }

        function handleOperatorTouch(op) {
            const now = Date.now();
            const isDoubleTap = (now - lastTouchTime) < 300 && op === lastOperator;
            
            lastTouchTime = now;
            lastOperator = op;

            if (currentNumber === '' && previousNumber === '') {
                showError('Enter a number first');
                return;
            }

            if (isDoubleTap) {
                operation = op;
                document.querySelectorAll('.operator').forEach(btn => btn.classList.remove('active'));
                event.target.classList.add('active');
                if (currentNumber === '') {
                    currentNumber = previousNumber;
                }
                calculate();
                previousNumber = currentNumber;
                currentNumber = '';
                return;
            }

            if (currentNumber !== '') {
                previousNumber = currentNumber;
                currentNumber = '';
            }
            
            operation = op;
            updateDisplay();
        }

        function calculate() {
            if (!operation || previousNumber === '') return;
            
            if (currentNumber === '') {
                currentNumber = previousNumber;
            }

            const prev = parseFloat(previousNumber);
            const current = parseFloat(currentNumber);
            let result;

            switch(operation) {
                case '+': result = prev + current; break;
                case '-': result = prev - current; break;
                case '*': result = prev * current; break;
                case '/': result = current === 0 ? NaN : prev / current; break;
            }

            if (isNaN(result)) {
                showError('Invalid operation');
                clearDisplay();
                return;
            }

            currentNumber = result.toString();
            previousNumber = '';
            operation = null;
            document.querySelectorAll('.operator').forEach(btn => btn.classList.remove('active'));
            updateDisplay();
        }

        function addPercent() {
            if (currentNumber === '') return;
            const num = parseFloat(currentNumber);
            currentNumber = (num / 100).toString();
            updateDisplay();
        }

        function clearDisplay() {
            currentNumber = '';
            previousNumber = '';
            operation = null;
            document.querySelectorAll('.operator').forEach(btn => btn.classList.remove('active'));
            updateDisplay();
        }

        function deleteLast() {
            currentNumber = currentNumber.slice(0, -1);
            updateDisplay();
        }
    </script>
</body>
</html>
