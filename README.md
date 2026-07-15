<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Plataforma // EN VIVO AVANZADO</title>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');

        :root {
            --neon-blue: #00f3ff;
            --neon-pink: #ff00ea;
            --neon-green: #00ffaa;
            --neon-yellow: #fffb00;
            --bg-dark: #05050a;
            --panel-bg: rgba(5, 5, 15, 0.95);
        }

        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            background-color: var(--bg-dark); color: #fff;
            font-family: 'Share Tech Mono', monospace; overflow: hidden;
        }

        /* --- NAVEGACIÓN Y BUSCADOR (ARRIBA) --- */
        .top-bar {
            position: absolute; top: 0; left: 0; width: 100%;
            background: linear-gradient(to bottom, rgba(0,0,0,0.9), transparent);
            padding: 15px 20px; box-sizing: border-box; z-index: 20;
            display: flex; justify-content: space-between; align-items: flex-start; flex-wrap: wrap; gap: 10px;
        }

        .search-container {
            display: flex; gap: 10px; flex-wrap: wrap; align-items: center;
            background: rgba(0,0,0,0.6); padding: 10px; border: 1px solid var(--neon-blue); border-radius: 5px;
        }

        .search-box {
            background: transparent; border: none; border-bottom: 1px solid var(--neon-blue);
            color: #fff; padding: 5px; font-family: inherit; outline: none; width: 150px;
        }

        .btn-panel {
            background: rgba(0,0,0,0.8); color: var(--neon-pink); border: 1px solid var(--neon-pink);
            padding: 10px 20px; cursor: pointer; font-family: inherit; font-weight: bold; text-transform: uppercase;
            transition: 0.3s; margin-left: 5px;
        }
        .btn-panel:hover { background: var(--neon-pink); color: #000; box-shadow: 0 0 15px var(--neon-pink); }
        
        .top-buttons-right { display: flex; gap: 10px; }

        /* --- CONTENEDOR DE REELS (CENTRO) --- */
        .reels-container {
            width: 100%; height: 100vh; overflow-y: scroll;
            scroll-snap-type: y mandatory; scroll-behavior: smooth; scrollbar-width: none;
        }
        .reels-container::-webkit-scrollbar { display: none; }

        .reel-item {
            position: relative; width: 100%; height: 100vh;
            scroll-snap-align: start; display: flex; justify-content: center; align-items: center; background: #000; overflow: hidden;
        }

        .video-background { position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; z-index: 1; transform: scaleX(-1); }
        .overlay-gradient { position: absolute; bottom: 0; left: 0; width: 100%; height: 60%; background: linear-gradient(to top, rgba(0,0,0,0.95), rgba(0,0,0,0.4), transparent); z-index: 2; pointer-events: none;}

        /* Información del Streamer en el Reel */
        .stream-info { position: absolute; bottom: 20px; left: 15px; z-index: 10; max-width: 60%; }
        .streamer-name { font-size: 18px; font-weight: bold; color: var(--neon-blue); margin-bottom: 5px; display: flex; align-items: center; gap: 10px; }
        .live-badge { background: #ff003c; color: white; font-size: 10px; padding: 3px 6px; border-radius: 3px; animation: pulse 1.5s infinite; }
        .stream-title { font-size: 14px; margin-bottom: 10px; line-height: 1.4; text-shadow: 1px 1px 2px #000;}
        
        .ai-supervisor-badge {
            display: inline-block; background: rgba(0, 255, 170, 0.2); border: 1px solid var(--neon-green);
            color: var(--neon-green); font-size: 10px; padding: 2px 5px; margin-bottom: 5px;
        }

        /* --- CARRUSEL DE REGALOS --- */
        .gift-carousel-container {
            position: absolute; bottom: 85px; left: 0; width: 100%; height: 50px;
            z-index: 12; pointer-events: none; overflow: hidden; white-space: nowrap;
        }
        .gift-track { display: flex; align-items: center; position: absolute; left: 100%; height: 100%; gap: 30px; }
        
        .gift-alert {
            display: inline-flex; align-items: center; background: rgba(0, 0, 0, 0.7);
            border: 1px solid var(--neon-pink); border-radius: 30px; padding: 5px 15px;
            color: #fff; font-size: 14px; text-shadow: 1px 1px 2px #000;
            box-shadow: 0 0 10px var(--neon-pink); white-space: nowrap;
            animation: moveLeft 12s linear forwards;
        }
        .gift-alert b { color: var(--neon-yellow); margin-right: 5px;}
        .gift-alert img { width: 40px; height: 40px; margin-left: 10px; object-fit: contain; animation: pulseGlow 1.5s infinite; }

        @keyframes moveLeft { 0% { transform: translateX(0); } 100% { transform: translateX(-200vw); } }

        /* --- CHAT INTERACTIVO --- */
        .chat-container {
            position: absolute; bottom: 140px; left: 15px; z-index: 10; width: 280px; max-height: 200px;
            display: flex; flex-direction: column; justify-content: flex-end; mask-image: linear-gradient(to top, black 70%, transparent 100%); -webkit-mask-image: linear-gradient(to top, black 70%, transparent 100%);
        }
        .chat-messages {
            overflow-y: auto; max-height: 160px; padding-bottom: 10px; display: flex; flex-direction: column; gap: 8px; scrollbar-width: none;
        }
        .chat-messages::-webkit-scrollbar { display: none; }
        
        .chat-msg { font-size: 13px; line-height: 1.3; text-shadow: 1px 1px 2px #000; word-wrap: break-word; animation: slideInLeft 0.3s ease-out;}
        .chat-user { font-weight: bold; color: var(--neon-pink); margin-right: 5px; }
        .chat-user.admin-tag { color: var(--neon-yellow); }

        .chat-input-wrapper {
            display: flex; gap: 5px; background: rgba(0,0,0,0.6); border: 1px solid #555; border-radius: 20px; padding: 5px 10px; align-items: center;
        }
        .chat-input-wrapper input {
            background: transparent; border: none; color: #fff; font-family: inherit; font-size: 12px; width: 100%; outline: none;
        }
        .chat-input-wrapper button {
            background: none; border: none; color: var(--neon-blue); cursor: pointer; font-weight: bold; font-family: inherit; font-size: 12px; transition: 0.2s;
        }
        .chat-input-wrapper button:hover { color: var(--neon-green); }

        /* Botones de Acción (Reel) */
        .action-buttons { position: absolute; bottom: 30px; right: 15px; z-index: 10; display: flex; flex-direction: column; gap: 20px; align-items: center; }
        .action-btn { background: rgba(0,0,0,0.6); border: 1px solid var(--neon-blue); border-radius: 50%; width: 45px; height: 45px; display: flex; justify-content: center; align-items: center; color: var(--neon-blue); cursor: pointer; transition: 0.2s; font-size: 20px; user-select: none;}
        .action-btn:hover { background: var(--neon-blue); color: #000; transform: scale(1.1); }
        .btn-superlike { border-color: var(--neon-yellow); color: var(--neon-yellow); font-size: 22px; animation: pulseGlow 2s infinite; }
        .btn-superlike:hover { background: var(--neon-yellow); color: #000; box-shadow: 0 0 15px var(--neon-yellow); }
        .btn-donate { border-color: var(--neon-green); color: var(--neon-green); animation: float 3s ease-in-out infinite; }
        .btn-donate:hover { background: var(--neon-green); color: #000; }
        .btn-label { font-size: 11px; margin-top: 5px; text-shadow: 1px 1px 2px #000; text-align: center; font-weight: bold;}

        /* Animaciones Flotantes de Likes */
        .floating-heart {
            position: absolute; bottom: 80px; right: 30px; font-size: 24px; color: var(--neon-pink);
            pointer-events: none; z-index: 15; animation: floatUp 2s ease-out forwards;
            text-shadow: 0 0 10px var(--neon-pink);
        }

        /* --- PANEL LATERAL DERECHO Y ADMIN --- */
        .right-panel {
            position: fixed; top: 0; right: -400px; width: 350px; height: 100vh;
            background: var(--panel-bg); border-left: 2px solid var(--neon-blue);
            z-index: 100; transition: right 0.4s ease-in-out; padding: 20px; box-sizing: border-box;
            display: flex; flex-direction: column; gap: 20px; box-shadow: -10px 0 30px rgba(0,0,0,0.8);
            overflow-y: auto;
        }
        .right-panel.open { right: 0; }
        .right-panel.admin { border-left-color: var(--neon-yellow); }
        
        .panel-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--neon-blue); padding-bottom: 10px;}
        .panel-header.admin-header { border-bottom-color: var(--neon-yellow); }
        .close-btn { color: var(--neon-pink); cursor: pointer; font-size: 20px; font-weight: bold; }

        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; margin-bottom: 5px; color: #aaa; font-size: 12px; }
        .form-group input { width: 100%; padding: 10px; box-sizing: border-box; background: #000; border: 1px solid #555; color: #fff; font-family: inherit; }
        .form-group input:focus { border-color: var(--neon-blue); outline: none; }
        .form-group input.admin-input:focus { border-color: var(--neon-yellow); }

        .auth-section, .obs-section, .register-section, .recover-section, .admin-login-section, .admin-dashboard-section, .tienda-section { 
            background: rgba(0,0,0,0.5); padding: 15px; border: 1px dashed #555; display: none; 
        }
        #seccion-login { display: block; } 

        .btn-full { width: 100%; padding: 10px; background: transparent; border: 1px solid var(--neon-blue); color: var(--neon-blue); cursor: pointer; font-family: inherit; text-transform: uppercase; font-weight: bold; margin-top: 10px; transition: 0.3s;}
        .btn-full:hover { background: var(--neon-blue); color: #000; }
        .btn-full.cam-btn { border-color: var(--neon-green); color: var(--neon-green); }
        .btn-full.cam-btn:hover { background: var(--neon-green); color: #000; }
        .btn-full.recharge-btn { border-color: var(--neon-yellow); color: var(--neon-yellow); background: rgba(255, 251, 0, 0.1); }
        .btn-full.recharge-btn:hover { background: var(--neon-yellow); color: #000; }
        
        .btn-link { background: none; border: none; color: #aaa; text-decoration: underline; cursor: pointer; font-family: inherit; font-size: 12px; margin-top: 10px; width: 100%; text-align: center;}
        .btn-link:hover { color: var(--neon-pink); }

        .btn-admin { border-color: var(--neon-yellow); color: var(--neon-yellow); }
        .btn-admin:hover { background: var(--neon-yellow); color: #000; }

        .admin-padlock {
            position: fixed; bottom: 15px; left: 15px; z-index: 100;
            font-size: 20px; cursor: pointer; animation: padlockGlow 2s infinite alternate;
            background: rgba(0,0,0,0.6); padding: 5px 10px; border-radius: 5px; border: 1px solid #555;
        }

        .token-balance-display { text-align: center; font-size: 24px; color: var(--neon-green); font-weight: bold; margin: 10px 0; text-shadow: 0 0 10px var(--neon-green); }
        .gift-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px; max-height: 400px; overflow-y: auto; padding-right: 5px; scrollbar-width: thin; scrollbar-color: var(--neon-pink) #000;}
        .gift-card { background: rgba(0,0,0,0.8); border: 1px solid #444; border-radius: 8px; padding: 15px 5px; text-align: center; cursor: pointer; transition: 0.3s; }
        .gift-card:hover { border-color: var(--neon-pink); box-shadow: 0 0 10px rgba(255, 0, 234, 0.3); transform: translateY(-2px);}
        .gift-card img { width: 60px; height: 60px; object-fit: contain; margin-bottom: 5px; filter: drop-shadow(0 0 5px rgba(255,255,255,0.2)); }
        .gift-name { font-size: 12px; font-weight: bold; margin-bottom: 5px; color: #fff; }
        .gift-price { color: var(--neon-yellow); font-size: 14px; font-weight: bold; }

        /* KEYFRAMES */
        @keyframes pulse { 50% { opacity: 0.5; } }
        @keyframes pulseGlow { 50% { box-shadow: 0 0 15px var(--neon-yellow); } }
        @keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-5px); } }
        @keyframes floatUp { 0% { transform: translateY(0) scale(1) rotate(0deg); opacity: 1; } 100% { transform: translateY(-300px) scale(1.5) rotate(20deg); opacity: 0; } }
        @keyframes slideInLeft { from { transform: translateX(-20px); opacity: 0;} to { transform: translateX(0); opacity: 1;} }
        @keyframes padlockGlow { 0% { text-shadow: 0 0 5px var(--neon-yellow), 0 0 10px var(--neon-yellow); border-color: #555; } 100% { text-shadow: 0 0 15px var(--neon-yellow), 0 0 20px var(--neon-yellow); border-color: var(--neon-yellow); } }
    </style>
</head>
<body>
<div class="admin-padlock" onclick="abrirAdmin()">🔒</div>

<div class="top-bar">
    <div class="search-container">
        <span style="color: var(--neon-blue); font-weight: bold;">[ BÚSQUEDA ]</span>
        <input type="text" class="search-box" placeholder="Buscar canal...">
    </div>

    <div class="top-buttons-right">
        <button class="btn-panel" style="border-color: var(--neon-yellow); color: var(--neon-yellow);" onclick="togglePanel('tienda')">[ 🎁 TIENDA ]</button>
        <button class="btn-panel" onclick="togglePanel('usuario')">[ MI CUENTA / ESTUDIO ]</button>
    </div>
</div>

<div class="reels-container">
    <div class="reel-item" id="main-reel">
        <video id="stream-video" class="video-background" loop muted autoplay playsinline crossorigin="anonymous">
            <source src="https://www.w3schools.com/html/mov_bbb.mp4" type="video/mp4">
        </video>
        <div class="overlay-gradient"></div>

        <div class="gift-carousel-container">
            <div class="gift-track" id="gift-track-container"></div>
        </div>

        <div class="chat-container">
            <div class="chat-messages" id="chat-box">
                <div class="chat-msg"><span class="chat-user admin-tag">Sistema:</span> ¡Bienvenidos al directo! Comparte y envía regalos.</div>
            </div>
            <div class="chat-input-wrapper">
                <input type="text" id="chat-input" placeholder="Comenta en vivo..." onkeypress="handleEnter(event)">
                <button onclick="enviarMensaje()">ENVIAR</button>
            </div>
        </div>

        <div class="stream-info">
            <div class="ai-supervisor-badge">✓ P2P WEBRTC DIRECTO</div>
            <div class="streamer-name">@VentasPro <span class="live-badge">EN VIVO</span></div>
            <div class="stream-title" id="ui-stream-title">Presentando nuevos productos 🚀</div>
        </div>

        <div class="action-buttons">
            <div onclick="darLike()">
                <div class="action-btn">♡</div>
                <div class="btn-label" id="counter-likes">1200</div>
            </div>
            <div onclick="darSuperLike()">
                <div class="action-btn btn-superlike">🌟</div>
                <div class="btn-label" style="color: var(--neon-yellow);">SUPER</div>
            </div>
            <div onclick="togglePanel('tienda')">
                <div class="action-btn btn-donate">🎁</div>
                <div class="btn-label" style="color: var(--neon-green);">Regalar</div>
            </div>
        </div>
    </div>
</div>

<div class="right-panel" id="panel-derecho">
    <div class="panel-header" id="panel-header">
        <h3 id="panel-title" style="margin: 0; color: var(--neon-blue);">CENTRO DE CONTROL</h3>
        <div class="close-btn" onclick="cerrarPanel()">X</div>
    </div>

    <div class="tienda-section" id="seccion-tienda">
        <h4 style="color: var(--neon-pink); margin-top: 0; text-align: center;">TIENDA DE REGALOS VIP</h4>
        <div style="background: rgba(0,0,0,0.8); border: 1px solid var(--neon-green); padding: 15px; border-radius: 8px;">
            <div style="font-size: 12px; color: #aaa; text-align: center;">TU SALDO ACTUAL</div>
            <div class="token-balance-display">🪙 <span id="user-tokens-display">0</span></div>
        </div>
        <div class="gift-grid" id="gift-grid-container"></div>
    </div>

    <div class="auth-section" id="seccion-login">
        <h4 style="color: #fff; margin-top: 0;">ACCESO</h4>
        <div class="form-group"><input type="email" id="email-login" placeholder="usuario@correo.com"></div>
        <div class="form-group"><input type="password" id="pass-login" placeholder="Contraseña"></div>
        <button class="btn-full" onclick="iniciarSesion()">INGRESAR</button>
        <button class="btn-link" onclick="cambiarVista('seccion-registro')">Regístrate aquí</button>
    </div>

    <div class="register-section" id="seccion-registro">
        <h4 style="color: var(--neon-pink); margin-top: 0;">NUEVO USUARIO</h4>
        <div class="form-group"><input type="email" id="email-reg" placeholder="nuevo@correo.com"></div>
        <div class="form-group"><input type="password" id="pass-reg" placeholder="Crea tu contraseña"></div>
        <div class="form-group"><input type="text" id="secret-reg" placeholder="Palabra secreta para recuperar"></div>
        <button class="btn-full" style="border-color: var(--neon-pink); color: var(--neon-pink);" onclick="registrarUsuario()">REGISTRARME</button>
        <button class="btn-link" onclick="cambiarVista('seccion-login')">Volver al inicio</button>
    </div>

    <div class="obs-section" id="seccion-obs">
        <h4 style="color: var(--neon-green); margin-top: 0;">✓ AUTENTICADO</h4>
        
        <hr style="border: 0; border-top: 1px dashed #555; margin: 15px 0;">
        <h5 style="color: var(--neon-pink); margin: 0 0 10px 0;">🎥 MODO CREADOR (TRANSMITIR)</h5>
        <p style="font-size: 11px; color: #ccc;">Transmitir directo a todos los dispositivos (P2P WebRTC).</p>
        <button class="btn-full cam-btn" id="btn-iniciar-cam" onclick="iniciarCamaraDispositivo()">INICIAR MI CÁMARA</button>
        <button class="btn-full" id="btn-detener-cam" style="border-color: #ff003c; color: #ff003c; display:none;" onclick="detenerCamaraDispositivo()">DETENER TRANSMISIÓN</button>

        <hr style="border: 0; border-top: 1px dashed #555; margin: 20px 0 15px 0;">
        <h5 style="color: var(--neon-blue); margin: 0 0 10px 0;">👀 MODO ESPECTADOR (VER)</h5>
        <p style="font-size: 11px; color: #ccc;">Conectarse directamente al video del Streamer.</p>
        <button class="btn-full" style="border-color: var(--neon-blue); color: var(--neon-blue);" onclick="conectarStreamWebRTC()">▶️ VER CÁMARA DEL STREAMER</button>

        <button class="btn-full" style="border-color: #555; color: #aaa; margin-top: 30px;" onclick="cerrarSesionUsuario()">CERRAR SESIÓN</button>
    </div>

    <div class="admin-login-section" id="seccion-admin-login">
        <h4 style="color: var(--neon-yellow); margin-top: 0;">🔒 ACCESO MAESTRO</h4>
        <input type="email" id="email-admin" class="form-group admin-input" placeholder="correo@admin.com" style="width: 100%; padding: 10px; margin-bottom:10px;">
        <input type="password" id="pass-admin" class="form-group admin-input" placeholder="••••••••" style="width: 100%; padding: 10px; margin-bottom:10px;">
        <button class="btn-full btn-admin" onclick="iniciarSesionAdmin()">ACCEDER</button>
    </div>

    <div class="admin-dashboard-section" id="seccion-admin-dashboard">
        <h4 style="color: var(--neon-yellow); margin-top: 0;">PANEL SUPERVISOR</h4>
        <button class="btn-full btn-admin" onclick="cerrarSesionAdmin()">CERRAR SESIÓN MAESTRA</button>
    </div>
</div>

<script>
    function saveDB(table, data) { localStorage.setItem(`sql_db_${table}`, JSON.stringify(data)); }
    let dbUsers = JSON.parse(localStorage.getItem('sql_db_users')) || [{ email: "ventasonlinenet168@gmail.com", password: "Leomas4409405.", role: "admin", tokenBalance: 99999, secretQuestion: "admin" }];
    let usuarioActual = null;
    let dbGifts = JSON.parse(localStorage.getItem('sql_db_gifts')) || [{ id: 1, name: "Rosa de Neón", price: 10, imageUrl: "https://cdn-icons-png.flaticon.com/512/3050/3050417.png" }, { id: 2, name: "Corona Real", price: 200, imageUrl: "https://cdn-icons-png.flaticon.com/512/6941/6941697.png" }];

    function abrirPanel() { document.getElementById('panel-derecho').classList.add('open'); }
    function cerrarPanel() { document.getElementById('panel-derecho').classList.remove('open'); document.getElementById('panel-derecho').classList.remove('admin'); }
    
    function togglePanel(tipo) {
        cerrarPanel();
        setTimeout(() => {
            if(tipo === 'usuario') {
                document.getElementById('panel-title').innerText = "ESTUDIO";
                if (usuarioActual) cambiarVista('seccion-obs'); else cambiarVista('seccion-login');
                abrirPanel();
            } else if (tipo === 'tienda') {
                document.getElementById('panel-title').innerText = "TIENDA VIP";
                if (!usuarioActual) { alert("Inicia Sesión para usar la tienda."); cambiarVista('seccion-login'); } 
                else { cargarTienda(); cambiarVista('seccion-tienda'); }
                abrirPanel();
            }
        }, 100);
    }
    
    function abrirAdmin() { cambiarVista('seccion-admin-login'); document.getElementById('panel-derecho').classList.add('admin'); abrirPanel(); }
    function cambiarVista(id) { ['seccion-login', 'seccion-registro', 'seccion-obs', 'seccion-admin-login', 'seccion-admin-dashboard', 'seccion-tienda'].forEach(s => document.getElementById(s).style.display = 'none'); document.getElementById(id).style.display = 'block'; }

    function iniciarSesion() {
        const email = document.getElementById('email-login').value;
        const pass = document.getElementById('pass-login').value;
        const user = dbUsers.find(u => u.email === email && u.password === pass);
        if(user) { usuarioActual = user; cambiarVista('seccion-obs'); } else alert("Credenciales incorrectas");
    }
    function registrarUsuario() {
        const email = document.getElementById('email-reg').value; const pass = document.getElementById('pass-reg').value;
        if(dbUsers.find(u => u.email === email)) return alert("El usuario ya existe");
        dbUsers.push({ email, password: pass, role: "user", tokenBalance: 500 }); saveDB('users', dbUsers);
        alert("¡Registrado! Inicia sesión."); cambiarVista('seccion-login');
    }
    function cerrarSesionUsuario() { usuarioActual = null; detenerCamaraDispositivo(); cambiarVista('seccion-login'); }

    function cargarTienda() {
        if(!usuarioActual.tokenBalance) usuarioActual.tokenBalance = 500;
        document.getElementById('user-tokens-display').innerText = usuarioActual.tokenBalance;
        const grid = document.getElementById('gift-grid-container'); grid.innerHTML = '';
        dbGifts.forEach(g => { grid.innerHTML += `<div class="gift-card" onclick="comprarRegalo(${g.id})"><img src="${g.imageUrl}"><div class="gift-name">${g.name}</div><div class="gift-price">🪙 ${g.price}</div></div>`; });
    }

    function handleEnter(e) { if(e.key === 'Enter') enviarMensaje(); }
    
    function enviarMensaje() {
        const input = document.getElementById('chat-input');
        if(input.value.trim() !== "") { 
            const nombre = usuarioActual ? usuarioActual.email.split('@')[0] : "Invitado";
            const esAdmin = usuarioActual ? (usuarioActual.role === 'admin') : false;

            if(window.enviarMensajeFirebase) {
                window.enviarMensajeFirebase(nombre, input.value, esAdmin);
            } else {
                window.agregarMensajeChatDOM(nombre, input.value, esAdmin);
            }
            input.value = ""; 
        }
    }

    window.agregarMensajeChatDOM = function(u, t, a) {
        const b = document.getElementById('chat-box');
        b.innerHTML += `<div class="chat-msg"><span class="chat-user ${a?'admin-tag':''}">${u}:</span> ${t}</div>`;
        b.scrollTop = b.scrollHeight;
    };

    function comprarRegalo(id) {
        const gift = dbGifts.find(g => g.id === id);
        if(usuarioActual.tokenBalance >= gift.price) {
            usuarioActual.tokenBalance -= gift.price; cargarTienda();
            saveDB('users', dbUsers);

            const nombre = usuarioActual.email.split('@')[0];
            
            if(window.enviarRegaloFirebase) {
                window.enviarRegaloFirebase(nombre, gift.name, gift.imageUrl);
            } else {
                window.mostrarAlertaRegaloDOM(nombre, gift.name, gift.imageUrl);
            }
        } else alert("Sin tokens suficientes. ¡Recarga en la billetera!");
    }

    window.mostrarAlertaRegaloDOM = function(nombre, nombreRegalo, imgRegalo) {
        const track = document.getElementById('gift-track-container');
        const alertDiv = document.createElement('div'); alertDiv.className = 'gift-alert';
        alertDiv.innerHTML = `<b>${nombre}</b> envió ${nombreRegalo} <img src="${imgRegalo}">`;
        track.appendChild(alertDiv); setTimeout(() => alertDiv.remove(), 12000);
    };

    let totalLikes = 1200;
    function darLike() { totalLikes++; document.getElementById('counter-likes').innerText = totalLikes; animarCorazon(); }
    function darSuperLike() { totalLikes+=50; document.getElementById('counter-likes').innerText = totalLikes; animarCorazon(5); document.getElementById('chat-input').value = "¡SUPER LIKE 🌟!"; enviarMensaje(); }
    function animarCorazon(cant = 1) {
        const reel = document.getElementById('main-reel');
        for(let i=0; i<cant; i++) setTimeout(() => {
            const h = document.createElement('div'); h.className = 'floating-heart'; h.innerText = ['♡','💖','🔥'][Math.floor(Math.random()*3)];
            h.style.right = (20 + Math.random() * 40) + 'px'; reel.appendChild(h); setTimeout(() => h.remove(), 2000);
        }, i*100);
    }
</script>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.16.0/firebase-app.js";
    import { getFirestore, collection, doc, setDoc, onSnapshot, addDoc, updateDoc, serverTimestamp, query, orderBy, limit } from "https://www.gstatic.com/firebasejs/12.16.0/firebase-firestore.js";

    const firebaseConfig = {
        apiKey: "AIzaSyAPSmj3q7acuRlykh9Kw87h_9BQqjycf_U",
        authDomain: "camenvivosuperlike.firebaseapp.com",
        projectId: "camenvivosuperlike",
        storageBucket: "camenvivosuperlike.firebasestorage.app",
        messagingSenderId: "1085729702867",
        appId: "1:1085729702867:web:13acca5f20115bbf5be526"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    const servers = { iceServers: [{ urls: ['stun:stun1.l.google.com:19302', 'stun:stun2.l.google.com:19302'] }] };

    window.streamLocalGlobal = null;
    window.conexionesWebRTC = {}; 

    // --- SISTEMA DE CHAT Y REGALOS GLOBAL ---
    window.enviarMensajeFirebase = async function(u, t, a) {
        await addDoc(collection(db, 'salas_live', 'canal_principal', 'mensajes'), {
            usuario: u, texto: t, admin: a, timestamp: serverTimestamp()
        });
    };

    const mensajesRef = collection(db, 'salas_live', 'canal_principal', 'mensajes');
    const qMensajes = query(mensajesRef, orderBy("timestamp", "asc"), limit(50));
    onSnapshot(qMensajes, snapshot => {
        snapshot.docChanges().forEach(change => {
            if (change.type === "added") {
                const data = change.doc.data();
                window.agregarMensajeChatDOM(data.usuario, data.texto, data.admin);
            }
        });
    });

    window.enviarRegaloFirebase = async function(u, r, img) {
        await addDoc(collection(db, 'salas_live', 'canal_principal', 'regalos'), {
            usuario: u, regalo: r, img: img, timestamp: serverTimestamp()
        });
    };

    const regalosRef = collection(db, 'salas_live', 'canal_principal', 'regalos');
    const qRegalos = query(regalosRef, orderBy("timestamp", "asc"), limit(10));
    let cargaInicialRegalos = true;
    onSnapshot(qRegalos, snapshot => {
        if(cargaInicialRegalos) { cargaInicialRegalos = false; return; }
        snapshot.docChanges().forEach(change => {
            if (change.type === "added") {
                const data = change.doc.data();
                window.mostrarAlertaRegaloDOM(data.usuario, data.regalo, data.img);
            }
        });
    });

    // =========================================================
    // ⚙️ MOTOR DE CÁMARA CORREGIDO Y DIRECTO
    // =========================================================

    // --- MODO CREADOR (HOST) ---
    window.iniciarCamaraDispositivo = async function() {
        
        // 1. Verificación básica: ¿Soporta la cámara el navegador?
        if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
            alert("⚠️ TU NAVEGADOR BLOQUEÓ LA CÁMARA.\n\nPara transmitir, debes alojar este archivo en un servidor seguro (HTTPS) o usar 'localhost'. No funcionará si lo abres con doble clic (file:///).");
            return;
        }

        try {
            // 2. Invocación nativa, directa y sin adaptadores antiguos
            window.streamLocalGlobal = await navigator.mediaDevices.getUserMedia({ 
                video: { facingMode: "user" }, 
                audio: true 
            });
            
            document.getElementById('stream-video').srcObject = window.streamLocalGlobal;
            document.getElementById('stream-video').play();

            document.getElementById('btn-iniciar-cam').style.display = 'none';
            document.getElementById('btn-detener-cam').style.display = 'block';
            document.getElementById('ui-stream-title').innerText = "🔴 TRANSMITIENDO P2P (Mundial)";
            cerrarPanel();

            const roomRef = doc(db, 'salas_live', 'canal_principal');
            await setDoc(roomRef, { enVivo: true, creador: "Admin" });

            const espectadoresCollection = collection(roomRef, 'espectadores');
            onSnapshot(espectadoresCollection, snapshot => {
                snapshot.docChanges().forEach(async change => {
                    const espectadorId = change.doc.id;
                    const data = change.doc.data();

                    if (change.type === 'added') {
                        const pc = new RTCPeerConnection(servers);
                        window.conexionesWebRTC[espectadorId] = pc;

                        window.streamLocalGlobal.getTracks().forEach(track => pc.addTrack(track, window.streamLocalGlobal));

                        pc.onicecandidate = event => {
                            if (event.candidate) {
                                addDoc(collection(db, 'salas_live', 'canal_principal', 'espectadores', espectadorId, 'creadorCandidates'), event.candidate.toJSON());
                            }
                        };

                        onSnapshot(collection(db, 'salas_live', 'canal_principal', 'espectadores', espectadorId, 'espectadorCandidates'), snap => {
                            snap.docChanges().forEach(c => {
                                if (c.type === 'added') pc.addIceCandidate(new RTCIceCandidate(c.doc.data()));
                            });
                        });
                    }

                    if ((change.type === 'added' || change.type === 'modified') && data.offer) {
                        const pc = window.conexionesWebRTC[espectadorId];
                        if (pc && !pc.currentRemoteDescription) {
                            await pc.setRemoteDescription(new RTCSessionDescription(data.offer));
                            const answer = await pc.createAnswer();
                            await pc.setLocalDescription(answer);
                            await updateDoc(change.doc.ref, { answer: { type: answer.type, sdp: answer.sdp } });
                        }
                    }
                });
            });

        } catch (err) { 
            // 3. Sistema inteligente para atrapar errores de cámara
            if (err.name === 'NotAllowedError' || err.name === 'PermissionDeniedError') {
                alert("🚫 PERMISO DENEGADO.\n\nEl navegador bloqueó la cámara. Sigue estos pasos:\n1. Asegúrate de estar en una conexión segura (HTTPS o localhost).\n2. Haz clic en el candado 🔒 junto a la barra de direcciones.\n3. Permite explícitamente el acceso a 'Cámara' y 'Micrófono'.\n4. Recarga la página.");
            } else if (err.name === 'NotFoundError' || err.name === 'DevicesNotFoundError') {
                alert("📱 ERROR DE HARDWARE:\nNo se detectó ninguna cámara o micrófono conectado a este dispositivo.");
            } else if (err.name === 'NotReadableError' || err.name === 'TrackStartError') {
                alert("⚠️ CÁMARA OCUPADA:\nOtra aplicación (como Zoom, WhatsApp, o el sistema operativo) está usando la cámara en este momento. Ciérrala e intenta de nuevo.");
            } else {
                alert("❌ Error desconocido al acceder a la cámara: " + err.message);
            }
        }
    };

    window.detenerCamaraDispositivo = async function() {
        if (window.streamLocalGlobal) { window.streamLocalGlobal.getTracks().forEach(t => t.stop()); window.streamLocalGlobal = null; }
        document.getElementById('stream-video').srcObject = null;
        document.getElementById('btn-iniciar-cam').style.display = 'block'; document.getElementById('btn-detener-cam').style.display = 'none';
        document.getElementById('ui-stream-title').innerText = "Transmisión finalizada";
        await setDoc(doc(db, 'salas_live', 'canal_principal'), { enVivo: false });
    };

    // --- MODO ESPECTADOR (VIEWER) ---
    window.conectarStreamWebRTC = async function() {
        cerrarPanel();
        document.getElementById('ui-stream-title').innerText = "Conectando al Streamer...";

        const pc = new RTCPeerConnection(servers);

        pc.ontrack = event => {
            const videoEl = document.getElementById('stream-video');
            videoEl.srcObject = event.streams[0];
            videoEl.play();
            document.getElementById('ui-stream-title').innerText = "🔴 VIENDO EN DIRECTO (P2P)";
        };

        const roomRef = doc(db, 'salas_live', 'canal_principal');
        const espectadorRef = await addDoc(collection(roomRef, 'espectadores'), {});

        pc.onicecandidate = event => {
            if (event.candidate) {
                addDoc(collection(espectadorRef, 'espectadorCandidates'), event.candidate.toJSON());
            }
        };

        const offer = await pc.createOffer({ offerToReceiveVideo: true, offerToReceiveAudio: true });
        await pc.setLocalDescription(offer);
        await updateDoc(espectadorRef, { offer: { type: offer.type, sdp: offer.sdp } });

        onSnapshot(espectadorRef, snapshot => {
            const data = snapshot.data();
            if (!pc.currentRemoteDescription && data && data.answer) {
                pc.setRemoteDescription(new RTCSessionDescription(data.answer));
            }
        });

        onSnapshot(collection(espectadorRef, 'creadorCandidates'), snap => {
            snap.docChanges().forEach(c => {
                if (c.type === 'added') pc.addIceCandidate(new RTCIceCandidate(c.doc.data()));
            });
        });
    };
</script>
</body>
</html>
