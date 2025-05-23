<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Expense Splitter</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .student-info {
            max-width: 500px;
            margin: 0 auto 20px;
            padding: 15px;
            background: white;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            text-align: center;
        }
        .chat-box {
            max-width: 500px;
            margin: 0 auto;
            background: white;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        .chat-header {
            background: #4CAF50;
            color: white;
            padding: 15px;
            text-align: center;
            font-weight: bold;
        }
        .chat-messages {
            height: 400px;
            padding: 15px;
            overflow-y: auto;
            background: #f9f9f9;
        }
        .message {
            margin-bottom: 10px;
            padding: 8px 12px;
            border-radius: 5px;
            max-width: 80%;
        }
        .bot-message {
            background: #e5e5ea;
            align-self: flex-start;
        }
        .user-message {
            background: #4CAF50;
            color: white;
            margin-left: auto;
        }
        .input-area {
            display: flex;
            padding: 10px;
            background: white;
            border-top: 1px solid #eee;
        }
        #user-input {
            flex: 1;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        #send-btn {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 8px 15px;
            margin-left: 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        .quick-options {
            display: flex;
            flex-wrap: wrap;
            margin-top: 10px;
        }
        .quick-option {
            background: #e5e5ea;
            padding: 5px 10px;
            margin: 5px;
            border-radius: 15px;
            cursor: pointer;
            font-size: 14px;
        }
        .balance {
            font-weight: bold;
            margin: 5px 0;
        }
        .owed {
            color: #e53935;
        }
        .owes {
            color: #43a047;
        }
    </style>
</head>
<body>
    <!-- Student Information -->
    <div class="student-info">
        <h3>Name: Swati Shanti</h3>
        <p>Reg no. 12314103</p>
        <p>Section: K23PS</p>
        <p>Roll No. 03</p>
    </div>

    <!-- Chat Box -->
    <div class="chat-box">
        <div class="chat-header">
            Simple Expense Splitter
        </div>
        <div class="chat-messages" id="chat-messages">
            <!-- Messages will appear here -->
        </div>
        <div class="input-area">
            <input type="text" id="user-input" placeholder="Type here..." autocomplete="off">
            <button id="send-btn">Send</button>
        </div>
    </div>

    <script>
        // Data storage
        const expenses = [];
        let currentStep = 'start';
        let currentExpense = {};
        let people = [];

        // DOM elements
        const chatMessages = document.getElementById('chat-messages');
        const userInput = document.getElementById('user-input');
        const sendBtn = document.getElementById('send-btn');

        // Start the chat
        window.onload = function() {
            addBotMessage("Welcome to Simple Expense Splitter! Let's split some expenses.");
            askQuestion();
        };

        // Send message when button is clicked or Enter is pressed
        sendBtn.addEventListener('click', sendMessage);
        userInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                sendMessage();
            }
        });

        function sendMessage() {
            const message = userInput.value.trim();
            if (message) {
                addUserMessage(message);
                userInput.value = '';
                processAnswer(message);
            }
        }

        function addUserMessage(text) {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message user-message';
            messageDiv.textContent = text;
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollTop + 500;
        }

        function addBotMessage(text, options = []) {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message bot-message';
            messageDiv.textContent = text;
            chatMessages.appendChild(messageDiv);
            
            if (options.length > 0) {
                const optionsDiv = document.createElement('div');
                optionsDiv.className = 'quick-options';
                
                options.forEach(option => {
                    const optionBtn = document.createElement('div');
                    optionBtn.className = 'quick-option';
                    optionBtn.textContent = option;
                    optionBtn.addEventListener('click', function() {
                        addUserMessage(option);
                        processAnswer(option);
                    });
                    optionsDiv.appendChild(optionBtn);
                });
                
                messageDiv.appendChild(optionsDiv);
            }
            
            chatMessages.scrollTop = chatMessages.scrollTop + 500;
        }

        function askQuestion() {
            if (currentStep === 'start') {
                addBotMessage("Enter names of people splitting expenses (comma separated):");
            } 
            else if (currentStep === 'getTotalAmount') {
                addBotMessage("Enter the total amount to split:");
            }
            else if (currentStep === 'getPayer') {
                addBotMessage(`Who paid this amount? (Enter one name: ${people.join(', ')})`);
            }
            else if (currentStep === 'nextAction') {
                addBotMessage("What would you like to do next?", [
                    "Add another expense",
                    "Show balances",
                    "Start over"
                ]);
            }
        }

        function processAnswer(answer) {
            if (currentStep === 'start') {
                // Get people's names
                people = answer.split(',').map(name => name.trim()).filter(name => name);
                if (people.length < 2) {
                    addBotMessage("Please enter at least 2 names separated by commas.");
                    return;
                }
                currentStep = 'getTotalAmount';
                askQuestion();
            }
            else if (currentStep === 'getTotalAmount') {
                // Get total amount
                const amount = parseFloat(answer);
                if (isNaN(amount) || amount <= 0) {
                    addBotMessage("Please enter a valid positive number.");
                    return;
                }
                currentExpense.totalAmount = amount;
                currentStep = 'getPayer';
                askQuestion();
            }
            else if (currentStep === 'getPayer') {
                // Get who paid
                const payer = people.find(p => p.toLowerCase() === answer.toLowerCase());
                if (!payer) {
                    addBotMessage(`Please enter one of these names: ${people.join(', ')}`);
                    return;
                }
                currentExpense.payer = payer;
                
                // Calculate splits
                const share = currentExpense.totalAmount / people.length;
                const payees = people.filter(p => p !== payer);
                
                // Save the expense
                expenses.push({
                    ...currentExpense,
                    share: share,
                    payees: payees
                });
                
                // Show results
                addBotMessage(`Expense recorded! ${payer} paid $${currentExpense.totalAmount.toFixed(2)}`);
                addBotMessage(`Each person owes $${share.toFixed(2)}`);
                
                if (payees.length > 0) {
                    const owesMessage = payees.map(p => `${p} owes ${payer} $${share.toFixed(2)}`).join('\n');
                    addBotMessage(owesMessage);
                }
                
                // Reset for next expense
                currentExpense = {};
                currentStep = 'nextAction';
                askQuestion();
            }
            else if (currentStep === 'nextAction') {
                if (answer.toLowerCase().includes('add') || answer.toLowerCase().includes('another')) {
                    currentStep = 'getTotalAmount';
                    askQuestion();
                }
                else if (answer.toLowerCase().includes('balance') || answer.toLowerCase().includes('show')) {
                    showBalances();
                    currentStep = 'nextAction';
                    askQuestion();
                }
                else if (answer.toLowerCase().includes('start') || answer.toLowerCase().includes('over')) {
                    expenses.length = 0;
                    people = [];
                    currentStep = 'start';
                    addBotMessage("Let's start over!");
                    askQuestion();
                }
            }
        }

        function showBalances() {
            if (expenses.length === 0) {
                addBotMessage("No expenses recorded yet.");
                return;
            }
            
            // Calculate net balances
            const balances = {};
            people.forEach(person => balances[person] = 0);
            
            expenses.forEach(exp => {
                const share = exp.share;
                balances[exp.payer] += exp.totalAmount;
                
                exp.payees.forEach(payee => {
                    balances[payee] -= share;
                });
            });
            
            // Display balances
            addBotMessage("Current balances:");
            
            people.forEach(person => {
                const balance = balances[person];
                if (balance > 0) {
                    addBotMessage(`${person} should receive $${balance.toFixed(2)}`, [], 'owes');
                } 
                else if (balance < 0) {
                    addBotMessage(`${person} owes $${Math.abs(balance).toFixed(2)}`, [], 'owed');
                } 
                else {
                    addBotMessage(`${person} is settled up`);
                }
            });
        }
    </script>
</body>
</html>
