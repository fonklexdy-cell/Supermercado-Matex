<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Supermercado MateKen2 Ultra</title>
<style>
    *{ margin:0; padding:0; box-sizing:border-box; font-family:Arial,sans-serif; }
    body{ background:linear-gradient(135deg,#4caf50,#8bc34a); min-height:100vh; display:flex; justify-content:center; align-items:center; padding:15px; }
    .contenedor{ background:white; width:100%; max-width:1000px; border-radius:25px; padding:20px; box-shadow:0 10px 30px rgba(0,0,0,.2); }
    h1{ text-align:center; color:#2e7d32; margin-bottom:15px; }
    .avatar{ font-size:85px; text-align:center; animation:flotar 2s infinite; }
    @keyframes flotar{ 50%{ transform:translateY(-10px); } }
    .panel{ display:flex; gap:10px; flex-wrap:wrap; margin-bottom:15px; }
    .caja{ flex:1; min-width:120px; background:#e8f5e9; padding:12px; border-radius:15px; text-align:center; font-weight:bold; }
    .progreso{ height:20px; background:#ddd; border-radius:20px; overflow:hidden; margin-bottom:15px; }
    .barra{ height:100%; width:0%; background:#4caf50; transition:.5s; }
    .pregunta{ background:#f1f8e9; padding:20px; border-radius:15px; font-size:22px; font-weight:bold; text-align:center; margin-bottom:15px; }
    .pista{ background:#fff8e1; padding:10px; border-radius:10px; text-align:center; margin-bottom:15px; }
    .opcion{ width:100%; padding:15px; border:none; border-radius:12px; margin-bottom:10px; font-size:18px; cursor:pointer; background:#c8e6c9; transition:.3s; }
    .opcion:hover{ transform:scale(1.03); background:#a5d6a7; }
    .boton{ width:100%; padding:15px; border:none; border-radius:15px; background:#2e7d32; color:white; font-size:18px; cursor:pointer; margin-top:10px; }
    .resultado{ display:none; text-align:center; }
    .insignia{ font-size:70px; margin:15px; }
    .detalles-resumen { text-align: left; max-width: 300px; margin: 20px auto; font-size: 18px; line-height: 1.8; }
    .btn-flotante-mateken {
    position: fixed;
    bottom: 25px;
    left: 25px; /* Cambiado a la izquierda */
    background-color: #FFD166;
    color: #073B4C;
    padding: 15px 22px;
    font-size: 1.1rem;
    font-weight: bold;
    text-decoration: none;
    border-radius: 50px;
    box-shadow: 0 6px 15px rgba(0,0,0,0.4);
    display: flex;
    align-items: center;
    gap: 10px;
    transition: transform 0.2s, background-color 0.2s;
    z-index: 9999;
    font-family: sans-serif;
}
.btn-flotante-mateken:hover {
    transform: scale(1.1);
    background-color: #fffde7;
}
</style>
</head>
<body>

<div class="contenedor">
    <h1>🛒 Supermercado MateKen2 Ultra</h1>
    <div id="inicio">
        <div class="avatar">🤖</div>
        <button class="boton" onclick="iniciarAudio(); iniciarJuego();">▶ Empezar Juego</button>
    </div>

    <div id="juego" style="display:none;">
        <div class="panel">
            <div class="caja">⭐ Puntos<br><span id="puntos">0</span></div>
            <div class="caja">📋 Pregunta<br><span id="numero">1</span>/10</div>
            <div class="caja">⏱ Tiempo<br><span id="tiempo">0</span>s</div>
        </div>
        <div class="progreso"><div class="barra" id="barra"></div></div>
        <div class="avatar" id="avatar">🤖</div>
        <div class="pista" id="pista"></div>
        <div class="pregunta" id="pregunta"></div>
        <div id="opciones"></div>
        <button class="boton" onclick="reiniciar()">🔄 Reiniciar</button>
    </div>

    <div id="resultado" class="resultado">
        <h2>🎉 Excelente Trabajo 🎉</h2>
        <div id="insignia" class="insignia"></div>
        <h1 id="rango" style="font-size: 50px; margin: 10px 0; color: #333;"></h1>
        <h1 id="porcentaje" style="color: #2e7d32; font-size: 40px; margin-bottom: 15px;"></h1>
        <div class="detalles-resumen">
            <div>✔️ Correctas: <span id="res-correctas"></span></div>
            <div>❌ Incorrectas: <span id="res-incorrectas"></span></div>
            <div>⏱️ Tiempo: <span id="res-tiempo"></span> segundos</div>
        </div>
        <button class="boton" onclick="reiniciar()">🔄 Jugar Nuevamente</button>
    </div>
</div>

<script>
    let preguntas=[], indice=0, aciertos=0, segundos=0, cronometro, audioCtx;

    // Función de formato estricta para Córdoba Nicaragüense (C$)
    function cordoba(valor) {
        return "C$ " + Math.floor(valor); 
    }

    function iniciarAudio(){
        if(!audioCtx) audioCtx = new(window.AudioContext || window.webkitAudioContext)();
        if(audioCtx.state==="suspended") audioCtx.resume();
    }

    function sonidoCorrecto(){
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.frequency.setValueAtTime(850, audioCtx.currentTime);
        osc.connect(gain); gain.connect(audioCtx.destination);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.3);
        osc.start(); osc.stop(audioCtx.currentTime + 0.3);
    }

    function sonidoError(){
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = "square";
        osc.frequency.setValueAtTime(220, audioCtx.currentTime);
        osc.connect(gain); gain.connect(audioCtx.destination);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.4);
        osc.start(); osc.stop(audioCtx.currentTime + 0.4);
    }

    function aleatorio(min,max){ return Math.floor(Math.random() * (max-min+1))+min; }
    function mezclar(array){ return array.sort(()=>Math.random()-0.5); }

    function generarPreguntas(){
        let banco=[];
        for(let i=0;i<50;i++){
            let c = aleatorio(2,10);
            let p = aleatorio(10,100);
            banco.push({icono:"🍎", pregunta: `Compraste ${c} manzanas a ${cordoba(p)} cada una. ¿Cuánto pagaste?`, respuesta: c * p, pista: "Multiplica cantidad por precio"});
            banco.push({icono:"🍌", pregunta: `Compraste ${c} bananos a ${cordoba(p)} cada uno. ¿Cuál es el total?`, respuesta: c * p, pista: "Multiplica cantidad por precio"});
            banco.push({icono:"🧀", pregunta: `El queso cuesta ${cordoba(p)}. Si pagas con ${cordoba(p*2)}, ¿cuánto es tu vuelto?`, respuesta: p, pista: "Resta el precio al pago"});
        }
        return mezclar(banco).slice(0,10);
    }

    function iniciarJuego(){
        document.getElementById("inicio").style.display="none";
        document.getElementById("resultado").style.display="none";
        document.getElementById("juego").style.display="block";
        preguntas = generarPreguntas();
        indice = 0; aciertos = 0; segundos = 0;
        cronometro = setInterval(()=>{ segundos++; document.getElementById("tiempo").innerText = segundos; },1000);
        mostrarPregunta();
    }

    function mostrarPregunta(){
        let p = preguntas[indice];
        document.getElementById("numero").innerText = indice + 1;
        document.getElementById("pregunta").innerHTML = `${p.icono} ${p.pregunta}`;
        document.getElementById("pista").innerHTML = "💡 " + p.pista;
        document.getElementById("barra").style.width = (indice * 10) + "%";

        let opts = [p.respuesta, p.respuesta+20, p.respuesta-10, p.respuesta+50];
        opts = mezclar([...new Set(opts)]);

        let html = "";
        opts.forEach(v => { html += `<button class="opcion" onclick="verificar(${v})">${cordoba(v)}</button>`; });
        document.getElementById("opciones").innerHTML = html;
    }

    function verificar(valor){
        if(valor === preguntas[indice].respuesta){
            aciertos++; sonidoCorrecto(); document.getElementById("avatar").innerText = "🤖🎉";
        } else { sonidoError(); document.getElementById("avatar").innerText = "🤖😕"; }
        
        document.querySelectorAll(".opcion").forEach(b => b.disabled = true);
        setTimeout(()=>{
            document.getElementById("avatar").innerText = "🤖";
            indice++;
            if(indice >= 10) finalizar(); else mostrarPregunta();
        }, 1000);
    }

    function finalizar(){
        clearInterval(cronometro);
        document.getElementById("juego").style.display="none";
        document.getElementById("resultado").style.display="block";
        document.getElementById("porcentaje").innerText = (aciertos * 10) + "%";
        document.getElementById("res-correctas").innerText = aciertos;
        document.getElementById("res-incorrectas").innerText = 10 - aciertos;
        document.getElementById("res-tiempo").innerText = segundos;
        document.getElementById("rango").innerText = aciertos >= 8 ? "Experto" : "Aprendiz";
        document.getElementById("insignia").innerText = aciertos >= 8 ? "🥇" : "🥉";
    }

    function reiniciar(){ location.reload(); }
</script>
<a href="
https://mateken2.pages.dev/" target="_blank" class="btn-flotante-mateken">
    🤖 Ir a MateKen2
</a>
</body>
</html>
