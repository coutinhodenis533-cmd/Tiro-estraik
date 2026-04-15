<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Tiro Estraik - Realismo Sonoro</title>
    <style>
        body, html { margin: 0; padding: 0; width: 100%; height: 100%; background-color: #000; color: #fff; font-family: 'Courier New', Courier, monospace; overflow: hidden; user-select: none; }
        .screen { position: fixed; width: 100%; height: 100%; display: none; flex-direction: column; justify-content: center; align-items: center; background-size: cover; background-position: center; transition: 0.5s; }
        
        /* FILTRO ULTRA REALISTA */
        #game-map { 
            background-image: url('https://images.unsplash.com/photo-1473580044384-7ba9967e16a0?q=80&w=2070&auto=format&fit=crop'); 
            filter: contrast(1.2) brightness(0.9) saturate(1.1);
        }

        #fps-view { position: absolute; bottom: -20px; left: 50%; transform: translateX(-50%); width: 450px; height: 350px; background: url('https://i.ibb.co/5R6K8S0/gun-fps.png') center/contain no-repeat; z-index: 50; pointer-events: none; transition: transform 0.05s; }
        .recoil { transform: translateX(-50%) translateY(30px) rotate(-5deg) scale(1.1) !important; }

        .ctrl-btn { position: absolute; width: 85px; height: 85px; border-radius: 50%; background: rgba(0,0,0,0.7); border: 2px solid #00ffcc; display: flex; justify-content: center; align-items: center; font-weight: bold; z-index: 100; font-size: 12px; box-shadow: 0 0 15px rgba(0,255,204,0.3); active { transform: scale(0.9); } }
        #btn-shoot { bottom: 50px; right: 40px; background: rgba(255, 0, 0, 0.3); width: 110px; height: 110px; border-color: red; }
        #btn-jump { bottom: 50px; left: 40px; }
        #btn-walk { bottom: 150px; left: 40px; border-color: #0088ff; }

        .npc { position: absolute; width: 120px; height: 200px; background: url('https://i.ibb.co/MCm99kz/soldier-npc.png') center/contain no-repeat; transform: scale(0.5); transition: transform 2s; filter: drop-shadow(0 0 10px rgba(0,0,0,0.8)); }
        #luffy { position: absolute; bottom: 80px; left: 50%; width: 280px; height: 400px; background: url('https://i.ibb.co/vxsL6nS/luffy-placeholder.png') center/contain no-repeat; transform: translateX(-50%); display: none; }

        .hud { position: absolute; top: 20px; left: 20px; background: rgba(0,0,0,0.8); padding: 15px; border-left: 5px solid #00ffcc; z-index: 100; backdrop-filter: blur(5px); }
        #flash { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #ffaa00; opacity: 0; pointer-events: none; z-index: 999; }
        
        .walking { animation: walkBob 0.4s infinite alternate; }
        @keyframes walkBob { from { transform: translateY(0); } to { transform: translateY(-15px) rotate(1deg); } }
    </style>
</head>
<body>

    <div id="flash"></div>

    <div id="login-screen" class="screen" style="display:flex; background: radial-gradient(circle, #222, #000);">
        <h1 style="color:#00ffcc; text-shadow: 0 0 20px #00ffcc; letter-spacing: 5px;">TIRO ESTRAIK: REALISM</h1>
        <button class="btn" style="padding:20px 50px; background:#00ffcc; font-weight:bold; border:none; border-radius:50px; cursor:pointer;" onclick="confirmarLogin()">INICIAR OPERAÇÃO</button>
    </div>

    <div id="game-map" class="screen">
        <div class="hud">
            VITALIDADE: <span id="p-hp" style="color:#ff4444;">100</span>%<br>
            ELIMINAÇÕES: <span id="score">0</span>
        </div>

        <div id="luffy"></div>
        <div id="npc-container"></div>
        <div id="fps-view"></div>

        <div id="btn-walk" class="ctrl-btn" ontouchstart="startWalking()" ontouchend="stopWalking()" onmousedown="startWalking()" onmouseup="stopWalking()">ANDAR</div>
        <div id="btn-jump" class="ctrl-btn" onclick="jump()">PULAR</div>
        <div id="btn-shoot" class="ctrl-btn" onclick="shoot()">ATIRAR</div>
    </div>

    <script>
        // Sistema de Áudio (Beeps que simulam realismo sem precisar de arquivos externos pesados)
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        function playSound(freq, type, duration, vol) {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = type;
            osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(vol, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.start();
            osc.stop(audioCtx.currentTime + duration);
        }

        let score = 0; let gameActive = false; let walkInterval;

        function confirmarLogin() {
            audioCtx.resume();
            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('game-map').style.display = 'block';
            gameActive = true;
            setInterval(spawnInimigos, 2500);
        }

        function shoot() {
            if(!gameActive) return;
            // Som de tiro realista (explosão de baixa frequência + ruído)
            playSound(150, 'sawtooth', 0.1, 0.5);
            playSound(50, 'sine', 0.2, 0.8);

            const gun = document.getElementById('fps-view');
            gun.classList.add('recoil');
            setTimeout(() => gun.classList.remove('recoil'), 50);

            const f = document.getElementById('flash');
            f.style.opacity = 0.4;
            setTimeout(() => f.style.opacity = 0, 50);
        }

        function startWalking() {
            if(!gameActive) return;
            document.getElementById('game-map').classList.add('walking');
            walkInterval = setInterval(() => {
                // Som de passo (impacto abafado)
                playSound(60, 'triangle', 0.1, 0.2);
            }, 400);
        }

        function stopWalking() {
            document.getElementById('game-map').classList.remove('walking');
            clearInterval(walkInterval);
        }

        function jump() {
            playSound(300, 'sine', 0.2, 0.3);
            document.getElementById('fps-view').style.transform = "translateX(-50%) translateY(-60px)";
            setTimeout(() => document.getElementById('fps-view').style.transform = "translateX(-50%)", 400);
        }

        function spawnInimigos() {
            if(!gameActive) return;
            const npc = document.createElement('div');
            npc.className = 'npc';
            npc.style.left = Math.random() * 70 + 15 + "%";
            npc.style.bottom = "120px";
            npc.onclick = () => { 
                playSound(200, 'square', 0.1, 0.3); // Som de acerto
                score++; 
                document.getElementById('score').innerText = score; 
                npc.remove(); 
            };
            document.getElementById('npc-container').appendChild(npc);
            setTimeout(() => npc.style.transform = "scale(1.8)", 50);
            setTimeout(() => { if(npc.parentNode) npc.remove(); }, 3500);
        }
    </script>
</body>
</html>



