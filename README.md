<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Sticker Lab Pro v3.3</title>
  <style>
    :root { --accent: #f472b6; --bg: #0f172a; --panel: #1e293b; --cyan: #22d3ee; }
    body { background: var(--bg); color: white; font-family: sans-serif; margin: 0; display: flex; flex-direction: column; height: 100vh; overflow: hidden; }
    header { padding: 10px; text-align: center; background: rgba(0,0,0,0.2); border-bottom: 1px solid #334155; }
    main { flex: 1; display: flex; align-items: center; justify-content: center; padding: 10px; position: relative; overflow: hidden; }
    .canvas-box { position: relative; width: 100%; max-width: 350px; aspect-ratio: 1/1; background: white; border-radius: 20px; box-shadow: 0 0 25px rgba(34, 211, 238, 0.4); touch-action: none; }
    #mainCanvas { width: 100%; height: 100%; border-radius: 20px; display: block; }
    .controls { background: var(--panel); padding: 15px; border-top-left-radius: 30px; border-top-right-radius: 30px; display: flex; flex-direction: column; gap: 10px; overflow-y: auto; box-shadow: 0 -5px 15px rgba(0,0,0,0.3); }
    input, select { background: var(--bg); border: 1px solid #475569; color: white; padding: 12px; border-radius: 12px; width: 100%; font-size: 16px; box-sizing: border-box; }
    .btn-grid { display: grid; grid-template-columns: repeat(6, 1fr); gap: 5px; }
    button { border: none; padding: 12px; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; active { transform: scale(0.95); } }
    .btn-gen { background: var(--accent); color: white; font-size: 16px; }
    .btn-acc { background: rgba(255,255,255,0.1); color: white; font-size: 20px; padding: 8px; }
    .btn-del { background: #ef4444; color: white; font-size: 14px; }
    .btn-save { background: #10b981; color: white; padding: 15px; font-size: 16px; margin-top: 5px; }
    #loader { position: absolute; background: rgba(255,255,255,0.95); color: #000; padding: 20px; border-radius: 20px; font-weight: bold; display: none; z-index: 100; text-align: center; box-shadow: 0 0 20px rgba(0,0,0,0.5); }
    #galeria { display: none; position: absolute; top: 60px; left: 10px; right: 10px; background: var(--panel); z-index: 200; padding: 15px; border-radius: 15px; border: 2px solid var(--cyan); overflow-x: auto; white-space: nowrap; }
    .thumb { width: 70px; height: 70px; border-radius: 10px; margin-right: 10px; border: 2px solid transparent; object-fit: cover; background: #fff; }
  </style>
</head>
<body>

  <header>
    <h2 style="margin:0; font-size:1.2rem; color:var(--accent)">STICKER LAB <span style="color:white">PRO</span></h2>
    <button onclick="toggleGaleria()" style="background:none; color:var(--cyan); font-size:13px; text-decoration:underline; border:none; padding:5px;">📂 Mis Diseños Guardados</button>
  </header>

  <div id="galeria"></div>

  <main>
    <div id="loader">🎨 CREANDO MAGIA...<br><small>Espera 5-8 segundos</small></div>
    <div class="canvas-box" id="cBox">
      <canvas id="mainCanvas"></canvas>
    </div>
  </main>

  <section class="controls">
    <div style="display:flex; gap:5px">
      <input type="text" id="prompt" placeholder="Ej: Gato ninja, helado rosa...">
      <select id="styleMode" style="width:130px">
        <option value="sticker style, white background, high resolution">Sticker</option>
        <option value="anime style, cute, high quality">Anime</option>
        <option value="3d render, clay style, white background">3D Clay</option>
      </select>
    </div>
    
    <button class="btn-gen" id="btnMain" onclick="crearSticker()">✨ GENERAR IMAGEN</button>

    <input type="text" id="bubbleText" placeholder="Texto de la burbuja..." oninput="actualizarBurbuja()">
    
    <div class="btn-grid">
      <button class="btn-acc" onclick="addObj('💭')">💭</button>
      <button class="btn-acc" onclick="addObj('👑')">👑</button>
      <button class="btn-acc" onclick="addObj('🕶️')">🕶️</button>
      <button class="btn-acc" onclick="addObj('🔥')">🔥</button>
      <button class="btn-acc" onclick="addObj('❤️')">❤️</button>
      <button class="btn-del" onclick="borrarUltimo()">🗑️</button>
    </div>
    
    <button class="btn-save" onclick="guardar()">💾 GUARDAR EN GALERÍA</button>
  </section>

  <script>
    const canvas = document.getElementById('mainCanvas'), ctx = canvas.getContext('2d');
    let baseImg = new Image();
    baseImg.crossOrigin = "anonymous";
    let objetos = [], seleccionado = null, offset = {x: 0, y: 0}, historial = [];

    function initCanvas() {
      canvas.width = 512; canvas.height = 512;
      dibujarTodo();
    }
    window.onload = initCanvas;

    async function crearSticker() {
      const p = document.getElementById('prompt').value; 
      if (!p) return alert("Escribe algo primero");
      const loader = document.getElementById('loader');
      const btn = document.getElementById('btnMain');
      loader.style.display = "block";
      btn.disabled = true;
      const seed = Math.floor(Math.random() * 999999);
      const style = document.getElementById('styleMode').value;
      const url = `https://image.pollinations.ai/prompt/${encodeURIComponent(p + ", " + style)}?width=512&height=512&nologo=true&seed=${seed}`;
      
      baseImg.src = url;
      baseImg.onload = () => {
        loader.style.display = "none";
        btn.disabled = false;
        if(!historial.includes(url)) {
          historial.unshift(url);
          if(historial.length > 10) historial.pop();
          actualizarGaleria();
        }
        dibujarTodo();
      };
    }

    function toggleGaleria() { 
      const g = document.getElementById('galeria');
      g.style.display = g.style.display === 'none' ? 'flex' : 'none';
    }

    function actualizarGaleria() {
      const g = document.getElementById('galeria');
      g.innerHTML = "";
      historial.forEach(url => {
        const t = document.createElement('img');
        t.src = url; t.className = "thumb";
        t.onclick = () => { baseImg.src = url; toggleGaleria(); dibujarTodo(); };
        g.appendChild(t);
      });
    }

    function addObj(e) { 
      objetos.push({ t: e, x: 256, y: 256, s: 100, texto: "" }); 
      dibujarTodo(); 
    }

    function borrarUltimo() {
      objetos.pop();
      dibujarTodo();
    }

    function actualizarBurbuja() {
      const txt = document.getElementById('bubbleText').value;
      let b = objetos.find(o => o.t === '💭');
      if (b) { b.texto = txt; } 
      else if(txt) { objetos.push({ t: '💭', x: 256, y: 150, s: 120, texto: txt }); }
      dibujarTodo();
    }

    function dibujarTodo() {
      ctx.fillStyle = "white";
      ctx.fillRect(0, 0, 512, 512);
      if(baseImg.complete && baseImg.naturalWidth > 0) {
        ctx.drawImage(baseImg, 0, 0, 512, 512);
      }
      objetos.forEach(o => {
        ctx.font = `${o.s}px Arial`;
        ctx.textAlign = "center"; ctx.textBaseline = "middle"; 
        ctx.fillText(o.t, o.x, o.y);
        if(o.t === '💭' && o.texto) {
          ctx.font = `bold ${o.s/5}px Arial`; ctx.fillStyle = "black";
          ctx.fillText(o.texto, o.x, o.y - 5, o.s * 0.8);
        }
      });
    }

    // Movimiento táctil mejorado
    canvas.ontouchstart = (e) => {
      const r = canvas.getBoundingClientRect();
      const x = (e.touches[0].clientX - r.left) * (512 / r.width);
      const y = (e.touches[0].clientY - r.top) * (512 / r.height);
      seleccionado = objetos.findLast(o => Math.abs(o.x - x) < 60 && Math.abs(o.y - y) < 60);
      if(seleccionado) { offset.x = x - seleccionado.x; offset.y = y - seleccionado.y; }
    };

    canvas.ontouchmove = (e) => {
      if(!seleccionado) return;
      e.preventDefault();
      const r = canvas.getBoundingClientRect();
      seleccionado.x = (e.touches[0].clientX - r.left) * (512 / r.width) - offset.x;
      seleccionado.y = (e.touches[0].clientY - r.top) * (512 / r.height) - offset.y;
      dibujarTodo();
    };

    // SISTEMA DE GUARDADO INFALIBLE PARA APKS
    async function guardar() {
      try {
        const dataUrl = canvas.toDataURL("image/png");
        
        // 1. Intentar compartir (Ideal para WhatsApp/Instagram)
        const blob = await (await fetch(dataUrl)).blob();
        const file = new File([blob], 'mi-sticker.png', { type: 'image/png' });

        if (navigator.share && navigator.canShare && navigator.canShare({ files: [file] })) {
          await navigator.share({ files: [file], title: 'Mi Sticker', text: 'Creado en Sticker Lab' });
        } else {
          // 2. Si falla compartir, forzar descarga
          const link = document.createElement('a');
          link.href = dataUrl;
          link.download = `sticker-${Date.now()}.png`;
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
          
          // 3. Aviso visual para el usuario de la App
          alert("Imagen enviada a descargas. Si no aparece, mantén presionada la imagen para guardarla.");
          
          // 4. Último recurso: Abrir en grande
          setTimeout(() => {
            const win = window.open();
            if(win) win.document.write(`<img src="${dataUrl}" style="width:100%">`);
          }, 500);
        }
      } catch (err) {
        alert("Error al procesar la imagen. Intenta de nuevo.");
      }
    }
  </script>
</body>
</html>

