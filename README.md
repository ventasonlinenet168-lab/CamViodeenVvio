// 2. Invocación nativa, directa y sin adaptadores antiguos
window.streamLocalGlobal = await navigator.mediaDevices.getUserMedia({ 
    video: { facingMode: "user" }, 
    audio: true 
});
            
document.getElementById('stream-video').srcObject = window.streamLocalGlobal;
document.getElementById('stream-video').play();
