<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Buzzer Online 12 Giocatori</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f2f5;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 { color: #333; text-align: center; margin-bottom: 5px; }
        .dashboard {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            text-align: center;
            width: 100%;
            max-width: 500px;
        }
        .stat { font-size: 1.2rem; margin: 10px 0; }
        .highlight { font-weight: bold; color: #007bff; font-size: 1.4rem; }
        .grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
            width: 100%;
            max-width: 800px;
        }
        @media (max-width: 600px) {
            .grid { grid-template-columns: repeat(2, 1fr); }
        }
        .buzzer-btn {
            background-color: #dc3545;
            color: white;
            border: none;
            padding: 25px 10px;
            font-size: 1.1rem;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            transition: transform 0.1s, background-color 0.2s;
            box-shadow: 0 4px #991b1b;
        }
        .buzzer-btn:active { transform: translateY(4px); box-shadow: none; }
        .buzzer-btn.last-clicked { background-color: #28a745; box-shadow: 0 4px #1e7e34; }
        .reset-btn {
            background-color: #6c757d;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }
        .status-pill { font-size: 0.9rem; color: #f59e0b; font-weight: bold; margin-top: 5px; }
        .status-pill.success { color: #28a745; }
        .status-pill.error { color: #dc3545; }
    </style>
</head>
<body>

    <h1>Buzzer Realtime Libero</h1>

    <div class="dashboard">
        <div class="stat">Click Totali: <span id="total-clicks" class="highlight">0</span></div>
        <div class="stat">Ultimo Click: <span id="last-player" class="highlight">-</span></div>
        <button class="reset-btn" id="btn-reset">Azzera per Tutti</button>
        <div class="status-pill" id="network-status">Connessione alla rete...</div>
    </div>

    <div class="grid">
        <button class="buzzer-btn" id="player-0">Marco</button>
        <button class="buzzer-btn" id="player-1">Giulia</button>
        <button class="buzzer-btn" id="player-2">Alessandro</button>
        <button class="buzzer-btn" id="player-3">Sofia</button>
        <button class="buzzer-btn" id="player-4">Francesco</button>
        <button class="buzzer-btn" id="player-5">Emma</button>
        <button class="buzzer-btn" id="player-6">Lorenzo</button>
        <button class="buzzer-btn" id="player-7">Matilde</button>
        <button class="buzzer-btn" id="player-8">Davide</button>
        <button class="buzzer-btn" id="player-9">Chiara</button>
        <button class="buzzer-btn" id="player-10">Riccardo</button>
        <button class="buzzer-btn" id="player-11">Beatrice</button>
    </div>

    <script>
        const nomi = ["Marco", "Giulia", "Alessandro", "Sofia", "Francesco", "Emma", "Lorenzo", "Matilde", "Davide", "Chiara", "Riccardo", "Beatrice"];
        
        // Nuovo server di rete WebSocket pubblico e stabile
        const SERVER_SOCKET = "wss://free.websocket.me/v1/stanza_quiz_fantacanaro_123456"; 

        let socket;
        let clickTotali = 0;
        let ultimoIndiceCliccato = null;

        const displayTotali = document.getElementById('total-clicks');
        const displayUltimo = document.getElementById('last-player');
        const statusText = document.getElementById('network-status');

        function connettiRete() {
            socket = new WebSocket(SERVER_SOCKET);

            socket.onopen = () => {
                statusText.innerText = "🟢 SISTEMA PRONTO ONLINE";
                statusText.className = "status-pill success";
            };

            socket.onmessage = (evento) => {
                try {
                    const dati = JSON.parse(evento.data);
                    
                    if (dati.azione === "click") {
                        clickTotali++;
                        displayTotali.innerText = clickTotali;

                        // Rimuovi colore verde dal vecchio tasto
                        if (ultimoIndiceCliccato !== null) {
                            document.getElementById(`player-${ultimoIndiceCliccato}`).classList.remove('last-clicked');
                        }

                        // Imposta il nuovo tasto verde
                        ultimoIndiceCliccato = dati.id;
                        displayUltimo.innerText = nomi[ultimoIndiceCliccato];
                        document.getElementById(`player-${ultimoIndiceCliccato}`).classList.add('last-clicked');
                    } 
                    else if (dati.azione === "reset") {
                        clickTotali = 0;
                        displayTotali.innerText = "0";
                        displayUltimo.innerText = "-";
                        if (ultimoIndiceCliccato !== null) {
                            document.getElementById(`player-${ultimoIndiceCliccato}`).classList.remove('last-clicked');
                            ultimoIndiceCliccato = null;
                        }
                    }
                } catch (e) {
                    // Ignora messaggi di testo non formattati
                }
            };

            socket.onclose = () => {
                statusText.innerText = "🔴 Rete scollegata. Riconnessione...";
                statusText.className = "status-pill error";
                setTimeout(connettiRete, 1500); 
            };
        }

        // Configura le azioni al click sui pulsanti della griglia
        nomi.forEach((nome, indice) => {
            document.getElementById(`player-${indice}`).onclick = () => {
                if (socket.readyState === WebSocket.OPEN) {
                    socket.send(JSON.stringify({ azione: "click", id: indice }));
                }
            };
        });

        // Configura il pulsante di reset generale
        document.getElementById('btn-reset').onclick = () => {
            if (socket.readyState === WebSocket.OPEN) {
                socket.send(JSON.stringify({ azione: "reset" }));
            }
        };

        // Avvia la connessione di rete all'apertura del sito
        connettiRete();
    </script>
</body>
</html>
