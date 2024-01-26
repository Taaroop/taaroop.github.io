<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            font-family: 'Georgia', serif;
            text-align: center;
            margin-top: 50px;
        }
        #game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #total-items-container {
            margin-top: 20px;
            font-size: 32px;
        }
        #buttons-container {
            margin-top: 20px;
        }
        button {
            padding: 15px 30px;
            font-size: 24px;
            margin: 0 15px;
            cursor: pointer;
        }
        #result {
            margin-top: 20px;
            font-size: 32px;
        }
        #input-container {
            margin-top: 40px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        label {
            margin: 15px 0;
            font-size: 24px;
        }
        input {
            padding: 10px;
            font-size: 18px;
        }
    </style>
</head>
<body>

<div id="game-container">
    <div id="total-items-container"></div>

    <div id="input-container">
        <label for="total-items">Enter the total number of items:</label>
        <input type="number" id="total-items" min="1" required>
        
        <label for="option1">Enter option 1:</label>
        <input type="number" id="option1" min="1" required>

        <label for="option2">Enter option 2:</label>
        <input type="number" id="option2" min="1" required>

        <br>

        <button onclick="startGame()">Start Game</button>
    </div>

    <div id="game-content" style="display: none;">
        <div id="buttons-container">
            <button id="button1" onclick="playerTurn(1)"></button>
            <button id="button2" onclick="playerTurn(2)"></button>
        </div>

        <div id="result" style="display: none;"></div>
    </div>
</div>

<script>
    let totalItems;
    let option1;
    let option2;
    let playerTurnFlag;
    let botLastMove;

    function play(x, y, n) {
        let a = Math.min(x, y);
        let b = Math.max(x, y);
        
        let li = [];
        let status = "W";
        
        for (let i = 0; i < b; i++) {
            if (i % a === 0) {
                status = (status === "W") ? "L" : "W";
            }
            li.push((i === b - 1) ? "W" : status);
        }
        
        return (n < b) ? li[n] : ((play(x, y, n - x) === "L" || play(x, y, n - y) === "L") ? "W" : "L");
    }

    function startGame() {
        totalItems = parseInt(document.getElementById('total-items').value);
        option1 = parseInt(document.getElementById('option1').value);
        option2 = parseInt(document.getElementById('option2').value);

        if (option1 < 0 || option2 < 0 || totalItems < 0 || option1 > totalItems || option2 > totalItems) {
            alert("Invalid parameters. Game terminated.");
            return;
        }

        document.getElementById('input-container').style.display = 'none';
        document.getElementById('game-content').style.display = 'block';

        updateTotalItems();

        if (play(option1, option2, totalItems) === "L") {
            document.getElementById('result').innerText = "Okay, you go first!";
            playerTurnFlag = true;
        } else {
            const botChoice = play(option1, option2, totalItems - option1) === "L" ? option1 : option2;
            totalItems -= botChoice;

            updateTotalItems();

            document.getElementById('result').innerText = `Okay, I go first!\nI take ${botChoice} item(s). Items remaining: ${totalItems}`;
            document.getElementById('result').style.display = 'block';
            botLastMove = botChoice;
            playerTurnFlag = true;
        }

        document.getElementById('buttons-container').style.display = 'block';
        updateButtonLabels();
        checkValidMoves();
    }

    function updateTotalItems() {
        document.getElementById('total-items-container').innerText = `Items remaining: ${totalItems}`;
    }

    function updateButtonLabels() {
        document.getElementById('button1').innerText = option1;
        document.getElementById('button2').innerText = option2;
    }

    function playerTurn(playerChoice) {
        if (playerTurnFlag) {
            const choice = (playerChoice === 1) ? option1 : option2;

            totalItems -= choice;
            updateTotalItems();

            makeBotMove();
        }
    }

    function makeBotMove() {
        document.getElementById('buttons-container').style.display = 'none';
        document.getElementById('result').innerText = "The bot is thinking...";
        document.getElementById('result').style.display = 'block';

        setTimeout(() => {
            const botChoice = play(option1, option2, totalItems - option1) === "L" ? option1 : option2;
            totalItems -= botChoice;

            updateTotalItems();

            document.getElementById('result').innerText = `I take ${botChoice} item(s). Items remaining: ${totalItems}`;
            document.getElementById('result').style.display = 'block';
            botLastMove = botChoice;

            checkValidMoves();

            if (totalItems <= 0) {
                setTimeout(() => {
                    document.getElementById('result').innerText = `I take ${botLastMove} item(s). Items remaining: ${totalItems}\nYou have no valid moves! The bot won!`;
                    document.getElementById('result').style.display = 'block';
                }, 1000);
            } else {
                document.getElementById('buttons-container').style.display = 'block';
                disableButtonsIfNegative();
            }
        }, 1000);
    }

    function checkValidMoves() {
        if (totalItems <= Math.min(option1, option2)) {
            document.getElementById('button1').disabled = true;
            document.getElementById('button2').disabled = true;
            document.getElementById('result').innerText = `I take ${botLastMove} item(s). Items remaining: ${totalItems}\nYou have no valid moves! The bot won!`;
            document.getElementById('result').style.display = 'block';
            document.getElementById('buttons-container').style.display = 'none';
        }
    }

    function disableButtonsIfNegative() {
        document.getElementById('button1').disabled = totalItems - option1 < 0;
        document.getElementById('button2').disabled = totalItems - option2 < 0;
    }
</script>

</body>
</html>
