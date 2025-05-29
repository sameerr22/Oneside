<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Real Color Prediction Game</title>
<style>
  body {
    background: #121212; color: #fff; font-family: Arial, sans-serif;
    margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center;
  }
  .box {
    background: #1e1e1e; padding: 15px; border-radius: 10px;
    width: 100%; max-width: 420px; margin-bottom: 20px;
  }
  h1, h3, h4 {
    margin-bottom: 10px;
  }
  input, button {
    width: 100%; padding: 10px; margin-top: 10px;
    border-radius: 5px; border: none; font-size: 16px;
  }
  input {
    background: #2c2c2c; color: #fff;
  }
  button {
    background: #ff5722; color: white; cursor: pointer;
  }
  .colors {
    display: flex; justify-content: space-between; margin-top: 15px;
  }
  .colors button {
    width: 30%;
    padding: 12px 0;
    font-weight: bold;
  }
  #timer {
    font-weight: bold; font-size: 18px; margin-top: 10px;
  }
  table {
    width: 100%; border-collapse: collapse; margin-top: 15px; font-size: 14px;
    background: #2a2a2a;
  }
  th, td {
    border: 1px solid #444; padding: 6px; text-align: center;
  }
  th {
    background: #333;
  }
  #depositInfo {
    font-size: 14px; margin-top: 8px; color: #ccc;
  }
</style>
</head>
<body>
  <h1>Real Color Prediction Game</h1>

  <div class="box" id="loginBox">
    <h3>Login with Mobile</h3>
    <input type="text" id="phone" placeholder="Enter 10-digit mobile number" />
    <button onclick="sendOTP()">Send OTP</button>
    <div id="otpSection" style="display:none;">
      <input type="text" id="otp" placeholder="Enter OTP" />
      <button onclick="verifyOTP()">Verify OTP</button>
    </div>
  </div>

  <div class="box" id="gameBox" style="display:none;">
    <h3>Welcome! Balance: <span id="balance">100</span> Coins</h3>
    <div id="timer">Next round starts in: <span id="countdown">--:--</span></div>
    <input type="number" id="bet" placeholder="Enter bet amount" min="1" />
    <div class="colors">
      <button onclick="chooseColor('Red')">🔴 Red</button>
      <button onclick="chooseColor('Green')">🟢 Green</button>
      <button onclick="chooseColor('Violet')">🟣 Violet</button>
    </div>
    <div id="result" style="margin-top:10px; font-weight:bold;"></div>

    <h4>🧾 Game History</h4>
    <table id="historyTable">
      <thead><tr><th>Time</th><th>Bet</th><th>Choice</th><th>Result</th><th>Status</th></tr></thead>
      <tbody></tbody>
    </table>
  </div>

  <div class="box" id="paymentBox" style="display:none;">
    <h3>💰 Deposit / Withdraw</h3>

    <h4>Deposit Coins</h4>
    <p id="depositInfo">Send payment to UPI: <strong>abc@paytm</strong><br/>After payment, enter amount below and confirm.</p>
    <input type="number" id="amountDeposit" placeholder="Amount to Deposit" min="1" />
    <button onclick="depositCoins()">Confirm Deposit</button>

    <h4 style="margin-top:15px;">Withdraw Coins</h4>
    <input type="number" id="amountWithdraw" placeholder="Amount to Withdraw" min="1" />
    <button onclick="withdrawCoins()">Request Withdraw</button>
  </div>

<script>
  let generatedOTP = '';
  let balance = 100;
  let chosenColor = null;
  let currentBet = 0;
  let gameHistory = [];
  let countdownInterval = null;

  function sendOTP() {
    const phone = document.getElementById('phone').value.trim();
    if (!/^[6-9]\\d{9}$/.test(phone)) {
      alert('Enter valid 10-digit mobile number starting with 6-9');
      return;
    }
    generatedOTP = Math.floor(1000 + Math.random() * 9000).toString();
    alert('Demo OTP sent: ' + generatedOTP);
    document.getElementById('otpSection').style.display = 'block';
  }

  function verifyOTP() {
    const otp = document.getElementById('otp').value.trim();
    if (otp === generatedOTP) {
      document.getElementById('loginBox').style.display = 'none';
      document.getElementById('gameBox').style.display = 'block';
      document.getElementById('paymentBox').style.display = 'block';
      startTimer();
    } else {
      alert('Incorrect OTP');
    }
  }

  function chooseColor(color) {
    const betInput = document.getElementById('bet');
    const bet = parseInt(betInput.value);
    if (isNaN(bet) || bet <= 0 || bet > balance) {
      alert('Enter a valid bet amount within your balance');
      return;
    }
    chosenColor = color;
    currentBet = bet;
    document.getElementById('result').textContent = `You chose ${color}. Waiting for round result...`;
  }

  function getNextRoundTime() {
    const now = new Date();
    const minutes = now.getMinutes();
    const nextRoundMinute = Math.ceil(minutes / 3) * 3;
    let nextRound = new Date(now.getFullYear(), now.getMonth(), now.getDate(), now.getHours(), nextRoundMinute, 0, 0);
    if (nextRound <= now) {
      nextRound = new Date(nextRound.getTime() + 3*60000);
    }
    return nextRound;
  }

  function startTimer() {
    const nextRound = getNextRoundTime();
    updateCountdown(nextRound);
    countdownInterval = setInterval(() => updateCountdown(nextRound), 1000);
  }

  function updateCountdown(nextRound) {
    const now = new Date();
    let diff = Math.floor((nextRound - now) / 1000);
    if (diff <= 0) {
      diff = 0;

