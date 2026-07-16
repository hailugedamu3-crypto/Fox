<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="am" lang="am">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>የቴሌግራም O/X ጨዋታ ቦት (ከAI እና ሰርች ጋር)</title>
    <style type="text/css">
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .container {
            background: #fff;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 550px;
            box-sizing: border-box;
        }
        h2 {
            text-align: center;
            color: #0088cc;
            margin-top: 0;
        }
        label {
            font-weight: bold;
            display: block;
            margin-top: 15px;
            margin-bottom: 5px;
            color: #333;
        }
        input {
            width: 100%;
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #ccc;
            box-sizing: border-box;
            font-size: 14px;
        }
        .btn-group {
            display: flex;
            gap: 15px;
            margin-top: 20px;
        }
        button {
            flex: 1;
            padding: 12px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            color: white;
            transition: background 0.2s;
        }
        .btn-start { background-color: #28a745; }
        .btn-start:hover { background-color: #218838; }
        .btn-stop { background-color: #dc3545; display: none; }
        .btn-stop:hover { background-color: #c82333; }
        
        .status {
            margin-top: 15px;
            padding: 10px;
            border-radius: 8px;
            text-align: center;
            font-weight: bold;
            font-size: 14px;
        }
        .status-offline { background-color: #f8d7da; color: #721c24; }
        .status-online { background-color: #d4edda; color: #155724; }
        
        .log-box {
            margin-top: 20px;
            background: #1e1e1e;
            color: #00ff00;
            padding: 15px;
            border-radius: 8px;
            height: 200px;
            overflow-y: auto;
            font-family: 'Courier New', Courier, monospace;
            font-size: 13px;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>O/X ጨዋታ ቦት (ከAI እና ሰርች ጋር)</h2>
    
    <label for="botToken">የእርስዎ ቦት ቶክን (Bot Token)፦</label>
    <input type="text" id="botToken" value="8064323092:AAGmkmtHntLbktprgngmV7a2BDtkn7y1J7Y" />

    <div class="btn-group">
        <button id="startBtn" class="btn-start" onclick="startBot()">ቦቱን አስነሳ (Start Bot)</button>
        <button id="stopBtn" class="btn-stop" onclick="stopBot()">ቦቱን አቁም (Stop Bot)</button>
    </div>

    <div id="statusBox" class="status status-offline">ቦቱ አልተነሳም (Offline)</div>
    
    <label>የክትትል ሰሌዳ (Live Server Log)፦</label>
    <div id="logBox" class="log-box">እንቅስቃሴዎች እዚህ ይታያሉ...</div>
</div>

<script type="text/javascript">
    //<![CDATA[
    let isRunning = false;
    let botToken = "";
    let lastUpdateId = 0;
    let pollingInterval = null;

    // የጨዋታዎችን ሁኔታ በሜሞሪ መያዣ
    let games = {};

    // የተጫዋቾች ነጥብ መቆጣጠሪያ
    let scores = JSON.parse(localStorage.getItem('xo_scores') || '{}');

    function logMessage(text) {
        const logBox = document.getElementById('logBox');
        const time = new Date().toLocaleTimeString();
        logBox.innerHTML += `<br />[${time}] ${text}`;
        logBox.scrollTop = logBox.scrollHeight;
    }

    async function startBot() {
        botToken = document.getElementById('botToken').value.trim();
        if (!botToken) {
            alert("እባክዎ መጀመሪያ የቦት ቶክን ያስገቡ!");
            return;
        }

        isRunning = true;
        document.getElementById('startBtn').style.display = 'none';
        document.getElementById('stopBtn').style.display = 'block';
        
        const statusBox = document.getElementById('statusBox');
        statusBox.className = "status status-online";
        statusBox.innerText = "የጨዋታ ቦቱ በመስራት ላይ ነው (Online)";
        
        document.getElementById('logBox').innerHTML = "";
        logMessage("የጨዋታ ቦቱ መሥራት ጀምሯል።");
        logMessage("ትዕዛዞች፦ /play (ከሰው ጋር ለመጫወት)፣ /playbot (ከቦት AI ጋር ለመጫወት)፣ /top (መሪ ሰሌዳ)");

        pollingInterval = setInterval(fetchUpdates, 1500);
    }

    async function fetchUpdates() {
        if (!isRunning) return;

        const url = "https://api.telegram.org/bot" + botToken + "/getUpdates?offset=" + (lastUpdateId + 1) + "&timeout=1";

        try {
            const response = await fetch(url);
            const data = await response.json();

            if (data.ok && data.result.length > 0) {
                for (let update of data.result) {
                    lastUpdateId = update.update_id;
                    
                    // 1. መደበኛ መልእክቶችን ማስተናገድ
                    if (update.message) {
                        await handleMessage(update.message);
                    }
                    
                    // 2. የቁልፍ ንክኪዎችን (Inline Clicks) ማስተናገድ
                    if (update.callback_query) {
                        await handleCallbackQuery(update.callback_query);
                    }

                    // 3. የሰርች (Inline Query) ጥያቄዎችን ማስተናገድ [1.1.3]
                    if (update.inline_query) {
                        await handleInlineQuery(update.inline_query);
                    }
                }
            }
        } catch (err) {
            logMessage("ስህተት፦ ከሰርቨር ጋር መገናኘት አልተቻለም።");
        }
    }

    async function handleMessage(message) {
        const chatId = message.chat.id;
        const text = message.text ? message.text.trim() : "";
        const userId = message.from.id;
        const userName = message.from.first_name || "ተጠቃሚ";

        if (text === "/start") {
            logMessage(`ተጠቃሚ [${userName}] ቦቱን አስነሳ።`);
            await sendReply(chatId, `ሰላም ${userName}! 👋\nእንኳን ወደ O/X ጨዋታ ቦት በደህና መጡ!\n\n<b>የመጫወቻ ትዕዛዞች፦</b>\n1. **/play** ➡️ በግሩፕ ውስጥ ከሰው ጋር ለመጫወት\n2. **/playbot** ➡️ ከቦት AI ጋር ብቻዎን ለመጫወት 🤖\n3. **/top** ➡️ ምርጥ ተጫዋቾችን ለማየት\n\n<b>በሰርች መጫወት፦</b>\nበማንኛውም ግሩፕ ውስጥ ሆነው የቦቱን ስም ሲጽፉ (ለምሳሌ <code>@bot_username</code>) ጨዋታውን በቀጥታ መጀመር ይችላሉ!`);
        } 
        else if (text.startsWith("/playbot")) {
            logMessage(`ተጠቃሚ [${userName}] ከቦት ጋር መጫወት ፈለገ።`);
            await startNewGame(chatId, userId, userName, true);
        }
        else if (text.startsWith("/play")) {
            logMessage(`አዲስ የሰው ለሰው ጨዋታ ተጠየቀ።`);
            await startNewGame(chatId, userId, userName, false);
        }
        else if (text.startsWith("/top")) {
            await sendLeaderboard(chatId);
        }
    }

    // አዲስ ጨዋታ መጀመር (ከሰው ወይም ከቦት ጋር)
    async function startNewGame(chatId, userId, userName, isAgainstBot) {
        const initialBoard = ["➖", "➖", "➖", "➖", "➖", "➖", "➖", "➖", "➖"];
        let text = "";
        
        if (isAgainstBot) {
            text = `❌ <b>ከቦት (AI) ጋር ጨዋታ ተጀምሯል!</b> 🤖\n\n<b>ተጫዋች X</b>፦ ${userName}\n<b>ተጫዋች O (AI)</b>፦ Bot AI 🤖\n\n👉 <b>የእርስዎ ተራ ነው! (❌)</b>`;
        } else {
            text = `❌ <b>O/X ጨዋታ ተጀምሯል!</b> ⭕\n\n<b>ተጫዋች X</b>፦ ${userName}\n<b>ተጫዋች O</b>፦ በመጠባበቅ ላይ...\n\nለመቀላቀል ከታች ካሉት ምልክቶች አንዱን ይጫኑ!`;
        }
        
        const markup = getBoardMarkup(initialBoard);

        const url = "https://api.telegram.org/bot" + botToken + "/sendMessage";
        const payload = {
            chat_id: chatId,
            text: text,
            parse_mode: "HTML",
            reply_markup: markup
        };

        try {
            const response = await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            
            if (result.ok) {
                const messageId = result.result.message_id;
                const gameKey = chatId + "_" + messageId;
                
                games[gameKey] = {
                    board: initialBoard,
                    playerX: userId,
                    playerX_name: userName,
                    playerO: isAgainstBot ? "bot" : null,
                    playerO_name: isAgainstBot ? "Bot AI 🤖" : null,
                    turn: "X",
                    active: true
                };
            }
        } catch (e) {}
    }

    // የሰርች (Inline Query) ጥያቄ ሲመጣ [1.1.3]
    async function handleInlineQuery(inlineQuery) {
        const queryId = inlineQuery.id;
        logMessage(`የሰርች ጥያቄ መጣ (Inline Query)።`);

        const initialBoard = ["➖", "➖", "➖", "➖", "➖", "➖", "➖", "➖", "➖"];
        const markup = getBoardMarkup(initialBoard);

        const url = "https://api.telegram.org/bot" + botToken + "/answerInlineQuery";
        const payload = {
            inline_query_id: queryId,
            results: [
                {
                    type: "article",
                    id: "play_xo_game",
                    title: "❌ O/X ጨዋታ ⭕",
                    description: "በግሩፕ ውስጥ የቲክ-ታክ-ቶ ጨዋታ ለመጀመር እዚህ ይጫኑ!",
                    input_message_content: {
                        message_text: `❌ <b>O/X ጨዋታ ተጀምሯል!</b> ⭕\n\nለመቀላቀል ከታች ካሉት ምልክቶች አንዱን ይጫኑ!`,
                        parse_mode: "HTML"
                    },
                    reply_markup: markup
                }
            ],
            cache_time: 0
        };

        try {
            await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
        } catch (e) {}
    }

    // የቁልፍ ንክኪዎችን መቆጣጠር
    async function handleCallbackQuery(callback) {
        const callbackData = callback.data;
        const clickerId = callback.from.id;
        const clickerName = callback.from.first_name || "ተጫዋች";
        
        const inlineMessageId = callback.inline_message_id;
        let gameKey = "";
        let isInline = false;

        // የሰርች መልእክት ከሆነ በ inline_message_id እንመዘግበዋለን [1.2.7]
        if (inlineMessageId) {
            gameKey = "inline_" + inlineMessageId;
            isInline = true;
        } else {
            gameKey = callback.message.chat.id + "_" + callback.message.message_id;
        }

        if (!callbackData.startsWith("xo:")) return;

        const index = parseInt(callbackData.split(":")[1]);
        
        // በሰርች የመጣ ጨዋታ ከሆነና በሜሞሪ ከሌለ አዲስ እንመዘግበዋለን
        if (!games[gameKey]) {
            games[gameKey] = {
                board: ["➖", "➖", "➖", "➖", "➖", "➖", "➖", "➖", "➖"],
                playerX: clickerId,
                playerX_name: clickerName,
                playerO: null,
                playerO_name: null,
                turn: "X",
                active: true
            };
        }

        const game = games[gameKey];

        if (!game || !game.active) {
            await answerCallback(callback.id, "ይህ ጨዋታ አልቋል ወይም አልተገኘም!");
            return;
        }

        // 1. ሁለተኛውን ተጫዋች (O) መመዝገብ
        if (game.playerO === null) {
            if (clickerId === game.playerX) {
                await answerCallback(callback.id, "ከራስዎ ጋር መጫወት አይችሉም! ሌላ ሰው ይጠብቁ።");
                return;
            }
            game.playerO = clickerId;
            game.playerO_name = clickerName;
        }

        // 2. የተራ ማረጋገጫ
        if (game.turn === "X" && clickerId !== game.playerX) {
            await answerCallback(callback.id, "የእርስዎ ተራ አይደለም! (የ X ተራ ነው)");
            return;
        }
        if (game.turn === "O" && clickerId !== game.playerO) {
            await answerCallback(callback.id, "የእርስዎ ተራ አይደለም! (የ O ተራ ነው)");
            return;
        }

        // 3. ምልክቱ የተያዘ መሆኑን ማረጋገጥ
        if (game.board[index] !== "➖") {
            await answerCallback(callback.id, "ይህ ቦታ ተይዟል!");
            return;
        }

        // 4. ምልክቱን ማስቀመጥ
        game.board[index] = game.turn === "X" ? "❌" : "⭕";

        // 5. አሸናፊ ማረጋገጥ
        let winnerSymbol = checkWinner(game.board);
        if (winnerSymbol) {
            await finishGame(callback, gameKey, winnerSymbol, game, isInline, inlineMessageId);
            return;
        }

        // 6. ከቦት ጋር ከሆነ ቦቱ አውቶማቲክ እንዲጫወት ማድረግ
        if (game.playerO === "bot") {
            logMessage("ቦቱ እያሰላ ነው (AI Move)...");
            const botMove = getSmartBotMove(game.board);
            if (botMove !== -1) {
                game.board[botMove] = "⭕";
            }
            
            winnerSymbol = checkWinner(game.board);
            if (winnerSymbol) {
                await finishGame(callback, gameKey, winnerSymbol, game, isInline, inlineMessageId);
                return;
            }
            
            const statusText = `❌ <b>ተጫዋች X</b>፦ ${game.playerX_name}\n⭕ <b>ተጫዋች O (AI)</b>፦ Bot AI 🤖\n\n👉 <b>የእርስዎ ተራ ነው! (❌)</b>`;
            await updateGameMessage(callback.message?.chat.id, callback.message?.message_id, inlineMessageId, statusText, getBoardMarkup(game.board));
            await answerCallback(callback.id, "እርምጃዎ ተመዝግቧል!");
        } else {
            // ከሰው ጋር ከሆነ ተራ መቀየር
            game.turn = game.turn === "X" ? "O" : "X";
            const nextPlayerName = game.turn === "X" ? game.playerX_name : game.playerO_name;
            const statusText = `❌ <b>ተጫዋች X</b>፦ ${game.playerX_name}\n⭕ <b>ተጫዋች O</b>፦ ${game.playerO_name}\n\n👉 <b>የተራ ተጫዋች</b>፦ <b>${nextPlayerName} (${game.turn === 'X' ? '❌' : '⭕'})</b> ተራ ነው!`;
            
            await updateGameMessage(callback.message?.chat.id, callback.message?.message_id, inlineMessageId, statusText, getBoardMarkup(game.board));
            await answerCallback(callback.id, "እርምጃዎ ተመዝግቧል!");
        }
    }

    // ጨዋታውን ማጠናቀቅና ውጤት ማሳየት
    async function finishGame(callback, gameKey, winnerSymbol, game, isInline, inlineMessageId) {
        game.active = false;
        let statusText = "";
        
        if (winnerSymbol === "Draw") {
            statusText = `<b>የጨዋታው ውጤት</b>፦ 🤝 አቻ ወጥቷል!\n\n❌ ${game.playerX_name} vs ⭕ ${game.playerO_name}`;
        } else {
            const winnerName = winnerSymbol === "❌" ? game.playerX_name : game.playerO_name;
            const winnerId = winnerSymbol === "❌" ? game.playerX : game.playerO;
            
            statusText = `<b>የጨዋታው ውጤት</b>፦ 🎉 <b>${winnerName} (${winnerSymbol}) አሸንፏል!</b> 🏆\n\n❌ ${game.playerX_name} vs ⭕ ${game.playerO_name}`;
            
            // ለቦት ድል ነጥብ አንመዘግብም
            if (winnerId !== "bot") {
                saveWin(winnerId, winnerName);
            }
        }
        
        await updateGameMessage(callback.message?.chat.id, callback.message?.message_id, inlineMessageId, statusText, getBoardMarkup(game.board));
        await answerCallback(callback.id, "ጨዋታው ተጠናቋል!");
    }

    // የቦቱ አርቴፊሻል ኢንተለጀንስ (Smart Bot AI Move Logic)
    function getSmartBotMove(board) {
        const winPatterns = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6]
        ];

        // 1. ቦቱ ማሸነፍ የሚችልበት ቀዳዳ ካለ ይፈልጋል
        for (let pattern of winPatterns) {
            let countO = 0, countEmpty = 0, emptyIdx = -1;
            for (let i of pattern) {
                if (board[i] === "⭕") countO++;
                else if (board[i] === "➖") { countEmpty++; emptyIdx = i; }
            }
            if (countO === 2 && countEmpty === 1) return emptyIdx;
        }

        // 2. ተቃዋሚ (ሰው) ሊያሸንፍ ከሆነ ይዘጋል (ይከላከላል)
        for (let pattern of winPatterns) {
            let countX = 0, countEmpty = 0, emptyIdx = -1;
            for (let i of pattern) {
                if (board[i] === "❌") countX++;
                else if (board[i] === "➖") { countEmpty++; emptyIdx = i; }
            }
            if (countX === 2 && countEmpty === 1) return emptyIdx;
        }

        // 3. መሃሉ ባዶ ከሆነ መሃሉን ይይዛል
        if (board[4] === "➖") return 4;

        // 4. ዝም ብሎ ባዶ ቦታ ላይ በደፈናው ያስቀምጣል
        let emptySpaces = [];
        for (let i = 0; i < board.length; i++) {
            if (board[i] === "➖") emptySpaces.push(i);
        }
        if (emptySpaces.length > 0) {
            return emptySpaces[Math.floor(Math.random() * emptySpaces.length)];
        }
        return -1;
    }

    function checkWinner(board) {
        const winPatterns = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6]
        ];

        for (let pattern of winPatterns) {
            if (board[pattern[0]] !== "➖" && board[pattern[0]] === board[pattern[1]] && board[pattern[0]] === board[pattern[2]]) {
                return board[pattern[0]];
            }
        }

        if (!board.includes("➖")) {
            return "Draw";
        }
        return null;
    }

    function saveWin(userId, userName) {
        if (!scores[userId]) {
            scores[userId] = { name: userName, wins: 0 };
        }
        scores[userId].wins += 1;
        scores[userId].name = userName;
        localStorage.setItem('xo_scores', JSON.stringify(scores));
    }

    async function sendLeaderboard(chatId) {
        const sortedScores = Object.values(scores).sort((a, b) => b.wins - a.wins);
        let leadText = "🏆 <b>የ O/X ጨዋታ ምርጥ ተጫዋቾች (Leaderboard)</b> 🏆\n\n";
        
        if (sortedScores.length === 0) {
            leadText += "እስካሁን ያሸነፈ የለም! ለመጫወት /play ይበሉ።";
        } else {
            for (let i = 0; i < Math.min(10, sortedScores.length); i++) {
                const medal = i === 0 ? "🥇" : i === 1 ? "🥈" : i === 2 ? "🥉" : "👤";
                leadText += `${medal} <b>${sortedScores[i].name}</b> ➡️ <b>${sortedScores[i].wins} ድሎች</b>\n`;
            }
        }
        await sendReply(chatId, leadText);
    }

    function getBoardMarkup(board) {
        return {
            inline_keyboard: [
                [
                    { text: board[0], callback_data: "xo:0" },
                    { text: board[1], callback_data: "xo:1" },
                    { text: board[2], callback_data: "xo:2" }
                ],
                [
                    { text: board[3], callback_data: "xo:3" },
                    { text: board[4], callback_data: "xo:4" },
                    { text: board[5], callback_data: "xo:5" }
                ],
                [
                    { text: board[6], callback_data: "xo:6" },
                    { text: board[7], callback_data: "xo:7" },
                    { text: board[8], callback_data: "xo:8" }
                ]
            ]
        };
    }

    // መልእክት ማሻሻያ (ለመደበኛ መልእክትም ሆነ ለሰርች መልእክት ይሠራል) [1.2.7]
    async function updateGameMessage(chatId, messageId, inlineMessageId, text, markup) {
        const url = "https://api.telegram.org/bot" + botToken + "/editMessageText";
        const payload = {
            text: text,
            parse_mode: "HTML",
            reply_markup: markup
        };

        if (inlineMessageId) {
            payload.inline_message_id = inlineMessageId;
        } else {
            payload.chat_id = chatId;
            payload.message_id = messageId;
        }

        try {
            await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
        } catch (e) {}
    }

    async function answerCallback(callbackId, text) {
        const url = "https://api.telegram.org/bot" + botToken + "/answerCallbackQuery";
        const payload = {
            callback_query_id: callbackId,
            text: text,
            show_alert: false
        };
        try {
            await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
        } catch (e) {}
    }

    async function sendReply(chatId, replyText) {
        const url = "https://api.telegram.org/bot" + botToken + "/sendMessage";
        const payload = {
            chat_id: chatId,
            text: replyText,
            parse_mode: "HTML"
        };
        try {
            await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
        } catch (err) {}
    }

    function stopBot() {
        isRunning = false;
        clearInterval(pollingInterval);
        
        document.getElementById('startBtn').style.display = 'block';
        document.getElementById('stopBtn').style.display = 'none';
        
        const statusBox = document.getElementById('statusBox');
        statusBox.className = "status status-offline";
        statusBox.innerText = "ቦቱ አልተነሳም (Offline)";
        
        logMessage("የጨዋታ ቦቱ ቆሟል።");
    }
    //]]>
</script>

</body>
</html>
