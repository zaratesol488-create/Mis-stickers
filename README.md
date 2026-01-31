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

