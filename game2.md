<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>5x5 Slot Machine</title>
<style>
    body {
        background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d); /* Eye-catching gradient */
        color: #fff;
        text-align: center;
        font-family: 'Arial Black', Arial, sans-serif;
        padding-top: 40px;
        background-size: 400% 400%;
        animation: gradientBG 15s ease infinite;
    }

    @keyframes gradientBG {
        0% {background-position:0% 50%}
        50% {background-position:100% 50%}
        100% {background-position:0% 50%}
    }

    /* High-quality 4K-style total display */
    #total-money {
        font-size: 64px;
        font-weight: bold;
        margin-bottom: 20px;
        color: #FFD700; /* gold */
        text-shadow:
            2px 2px 0 #000,
            4px 4px 0 rgba(0,0,0,0.2),
            0 0 20px rgba(255, 215, 0, 0.8);
        letter-spacing: 4px;
        background: linear-gradient(90deg, #ffeb3b, #fdd835, #ffc107);
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        display: inline-block;
    }

    #slot {
        width: fit-content;
        margin: auto;
        padding: 20px;
        border: 3px solid gold;
        border-radius: 12px;
        background: linear-gradient(135deg, #ff512f, #dd2476); /* 4K gradient for slot */
        box-shadow: 0 0 30px rgba(255,255,255,0.3);
    }

    .row {
        display: flex;
        justify-content: space-around;
        margin-bottom: 10px;
    }

    .reel {
        width: 120px;
        height: 120px;
        overflow: hidden;
        border: 3px solid gold;
        border-radius: 10px;
        background: #111;
        box-shadow: 0 0 10px rgba(255, 255, 255, 0.2) inset;
    }

    .reel-inner {
        display: flex;
        flex-direction: column;
        transition: transform 1.5s cubic-bezier(.28,.83,.56,1.2);
    }

    .symbol {
        width: 120px;
        height: 120px;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 100px;
    }

    button {
        background: gold;
        padding: 12px 24px;
        font-size: 18px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        margin-top: 10px;
        box-shadow: 0 0 10px rgba(255,255,0,0.5);
        transition: all 0.3s ease;
    }

    button:hover {
        background: orange;
        transform: scale(1.05);
        box-shadow: 0 0 20px rgba(255,165,0,0.7);
    }

    #message {
        margin-top: 15px;
        font-size: 20px;
    }
</style>
</head>
<body>

<h1>🎰 5x5 Slot Machine</h1>
<div id="total-money">TOTAL: $100</div>
<div id="slot"></div>

<button onclick="spin()">SPIN – $10</button>
<div id="message"></div>

<script>
const symbols = ["🍒","🍋","🔔","⭐","7️⃣"];
let money = 100;

const rows = 5;
const cols = 5;
const reelHeight = 120;
let reels = []; // Store all reel-inner elements

function createGrid() {
    const slot = document.getElementById("slot");
    slot.innerHTML = "";
    reels = [];

    for (let i = 0; i < rows; i++) {
        const rowDiv = document.createElement("div");
        rowDiv.className = "row";
        const rowReels = [];

        for (let j = 0; j < cols; j++) {
            const reel = document.createElement("div");
            reel.className = "reel";

            const inner = document.createElement("div");
            inner.className = "reel-inner";

            for (let k = 0; k < 40; k++) {
                const sym = document.createElement("div");
                sym.className = "symbol";
                sym.innerText = symbols[k % symbols.length];
                inner.appendChild(sym);
            }

            reel.appendChild(inner);
            rowDiv.appendChild(reel);
            rowReels.push(inner);
        }

        slot.appendChild(rowDiv);
        reels.push(rowReels);
    }
}

createGrid();

function updateMoney() {
    document.getElementById("total-money").innerText = "TOTAL: $" + money;
}

function spin() {
    if (money < 10) {
        document.getElementById("message").innerText = "Not enough money!";
        return;
    }

    money -= 10;
    updateMoney();
    document.getElementById("message").innerText = "";

    const stops = [];
    for (let i = 0; i < rows; i++) {
        stops[i] = [];
        for (let j = 0; j < cols; j++) {
            stops[i][j] = Math.floor(Math.random() * 30) + 3;
        }
    }

    for (let i = 0; i < rows; i++) {
        for (let j = 0; j < cols; j++) {
            reels[i][j].style.transition = "transform 1.5s cubic-bezier(.28,.83,.56,1.2)";
            reels[i][j].style.transform = `translateY(-${stops[i][j] * reelHeight}px)`;
        }
    }

    setTimeout(() => {
        const results = [];
        for (let i = 0; i < rows; i++) {
            results[i] = [];
            for (let j = 0; j < cols; j++) {
                results[i][j] = reels[i][j].children[stops[i][j]].innerText;
            }
        }

        const counts = {};
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                const sym = results[i][j];
                counts[sym] = (counts[sym] || 0) + 1;
            }
        }

        const maxMatches = Math.max(...Object.values(counts));
        let msg = "";

        if (maxMatches === 25) {
            money += 500;
            msg = "💎 MEGA JACKPOT! +$500";
        } else if (maxMatches >= 15) {
            money += 200;
            msg = "🔥 BIG WIN! +$200";
        } else if (maxMatches >= 10) {
            money += 60;
            msg = "✨ WIN! +$60";
        } else {
            msg = "❌ No match.";
        }

        updateMoney();
        document.getElementById("message").innerText = msg;

    }, 1550);
}
</script>

</body>
</html>
