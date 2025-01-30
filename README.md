<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Dark Mode Calculator</title>
    <style>
        body {
            background-color: #000;
            margin: 0;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            font-family: Arial, sans-serif;
        }
        
        .calculator {
            width: 320px;
            margin: 20px auto;
            padding: 20px;
            border-radius: 15px;
            background-color: #2d2d2d;
            box-shadow: 0 0 20px rgba(0, 150, 255, 0.2);
        }
        
        .display-container {
            margin-bottom: 15px;
            padding: 15px;
            background: #1a1a1a;
            border-radius: 10px;
            min-height: 80px;
            border: 1px solid #3d3d3d;
        }
        
        #history {
            height: 40px;
            font-size: 16px;
            color: #888;
            text-align: right;
            margin-bottom: 5px;
            overflow: hidden;
        }
        
        #current-operation {
            height: 25px;
            font-size: 18px;
            color: #666;
            text-align: right;
            margin-bottom: 5px;
        }
        
        #display {
            width: 100%;
            height: 50px;
            padding: 5px;
            font-size: 28px;
            text-align: right;
            border: none;
            background: transparent;
            color: #fff;
            font-weight: 300;
        }
        
        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 8px;
        }
        
        button {
            padding: 18px;
            font-size: 20px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            background-color: #3d3d3d;
            color: #fff;
            transition: all 0.2s;
        }
        
        button:hover {
            background-color: #4d4d4d;
        }
        
        .equals {
            background-color: #2196F3;
        }
        
        .operator.active {
            background-color: #4CAF50 !important;
            box-shadow: 0 0 10px rgba(76, 175, 80, 0.3);
        }
        
        .error-message {
            color: #ff4444;
            font-size: 14px;
            text-align: right;
            height: 20px;
            margin-top: 5px;
        }
    </style>
</head>
<body>
    <div class="calculator">
        <div class="display-container">
            <div id="history"></div>
            <div id="current-operation"></div>
            <input type="text" id="display" readonly>
            <div id="error" class="error-message"></div>
        </div>
        <div class="buttons">
            <button onclick="clearDisplay()">AC</button>
            <button onclick="deleteLast()">DEL</button>
            <button onclick="addPercent()">%</button>
            <button class="operator" onclick="handleOperatorClick('/')">/</button>
            <button onclick="appendNumber('7')">7</button>
            <button onclick="appendNumber('8')">8</button>
            <button onclick="appendNumber('9')">9</button>
            <button class="operator" onclick="handleOperatorClick('*')">×</button>
            <button onclick="appendNumber('4')">4</button>
            <button onclick="appendNumber('5')">5</button>
            <button onclick="appendNumber('6')">6</button>
            <button class="operator" onclick="handleOperatorClick('-')">-</button>
            <button onclick="appendNumber('1')">1</button>
            <button onclick="appendNumber('2')">2</button>
            <button onclick="appendNumber('3')">3</button>
            <button class="operator" onclick="handleOperatorClick('+')">+</button>
            <button onclick="appendNumber('0')" style="grid-column: span 2">0</button>
            <button onclick="appendNumber('.')">.</button>
            <button onclick="calculate()" class="equals">=</button>
        </div>
    </div>

    <script>
        let currentNumber = '';
        let previousNumber = '';
        let operation = null;
        let stickyOperation = false;
        let lastOperatorClickTime = 0;

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
                handleOperatorClick(e.key);
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
            document.getElementById('current-operation').textContent = 
                operation ? (stickyOperation ? `[${operation}] ${currentNumber}` : ` ${currentNumber}`) : '';
        }

        function appendNumber(num) {
            if (num === '.' && currentNumber.includes('.')) {
                showError('Точка уже добавлена');
                return;
            }
            currentNumber += num;
            updateDisplay();
        }

        function handleOperatorClick(op) {
            const now = Date.now();
            const isDoubleClick = (now - lastOperatorClickTime) < 300 && op === operation;
            lastOperatorClickTime = now;

            if (currentNumber === '' && previousNumber === '') {
                showError('Введите число');
                return;
            }

            if (isDoubleClick) {
                stickyOperation = true;
                document.querySelectorAll('.operator').forEach(btn => btn.classList.remove('active'));
                document.querySelector(`button[onclick="handleOperatorClick('${op}')"]`).classList.add('active');
            } else {
                if (currentNumber !== '') {
                    previousNumber = currentNumber;
                    currentNumber = '';
                }
                stickyOperation = false;
            }

            operation = op;
            updateDisplay();
        }

        function calculate() {
            if (!operation || previousNumber === '') {
                showError('Выберите операцию');
                return;
            }
            
            if (currentNumber === '') {
                if (stickyOperation) {
                    showError('Введите число');
                    return;
                }
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
                showError('Недопустимая операция');
                clearDisplay();
                return;
            }

            previousNumber = result.toString();
            currentNumber = stickyOperation ? '' : result.toString();
            
            if (!stickyOperation) {
                operation = null;
                document.querySelectorAll('.operator').forEach(btn => btn.classList.remove('active'));
            }
            
            updateDisplay();
        }

        function addPercent() {
            if (currentNumber === '') {
                showError('Введите число');
                return;
            }
            const num = parseFloat(currentNumber);
            currentNumber = (num / 100).toString();
            updateDisplay();
        }

        function clearDisplay() {
            currentNumber = '';
            previousNumber = '';
            operation = null;
            stickyOperation = false;
            document.querySelectorAll('.operator').forEach(btn => btn.classList.remove('active'));
            updateDisplay();
        }

        function deleteLast() {
            if (currentNumber.length > 0) {
                currentNumber = currentNumber.slice(0, -1);
                updateDisplay();
            }
        }
    </script>
</body>
</html>
