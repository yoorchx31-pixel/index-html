# index-html 
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>POS El Tridente</title>
    
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>

    <style>
        :root { 
            --cafe: #4e342e; --ocre: #ffb300; --verde: #2e7d32; --fondo: #f4f4f9; 
            --azul: #1976d2; --celeste: #00bcd4; --rojo: #d32f2f; --texto: #333; 
            --card-bg: rgba(255, 255, 255, 0.8); /* Tarjetas con transparencia */
        }
        
        body.dark-mode { 
            --fondo: #121212; --card-bg: rgba(30, 30, 30, 0.8); --texto: #e0e0e0; --cafe: #d7ccc8; 
        }

        body { 
            font-family: 'Segoe UI', sans-serif; 
            background-color: var(--fondo);
            background-image: url('3028.jpg'); /* Tu logo El Tridente */
            background-repeat: no-repeat;
            background-attachment: fixed;
            background-position: center;
            background-size: 80%; 
            margin: 0; 
            padding: 10px; 
            color: var(--texto); 
            transition: 0.3s; 
        }

        /* CAPA DE VISIBILIDAD DEL LOGO */
        body::before {
            content: "";
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background-color: var(--fondo);
            opacity: 0.2; /* SE NOTA MUCHO EL LOGO (0.1 a 0.3 es ideal) */
            z-index: -1;
        }

        .card { 
            background: var(--card-bg); 
            border-radius: 15px; 
            padding: 15px; 
            margin-bottom: 15px; 
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            backdrop-filter: blur(10px); /* Desenfoque para leer mejor sobre el logo */
            border: 1px solid rgba(255,255,255,0.2);
        }

        h2, h3 { color: var(--cafe); margin: 0 0 10px 0; border-bottom: 2px solid var(--ocre); padding-bottom: 5px; }
        input, select { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #555; border-radius: 10px; box-sizing: border-box; font-size: 16px; background: rgba(255,255,255,0.5); color: var(--texto); }
        .grid-botones { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
        .grid-pagos { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 8px; }
        .btn { border: none; border-radius: 10px; padding: 15px; font-weight: bold; width: 100%; cursor: pointer; font-size: 0.9rem; transition: 0.2s; }
        .btn-add { background: var(--ocre); color: #000; }
        .btn-pago { color: white; }
        .btn-merma { background: #607d8b; color: white; }
        .btn-nocturno { background: #333; color: white; position: fixed; top: 10px; right: 10px; width: auto; padding: 10px; z-index: 1000; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th { background: rgba(0,0,0,0.1); padding: 10px; text-align: left; }
        td { padding: 10px; border-bottom: 1px solid rgba(0,0,0,0.05); }
        .tabs { display: flex; gap: 5px; overflow-x: auto; padding-bottom: 5px; }
        .tab-btn { flex: none; padding: 10px 15px; border: none; border-radius: 8px; background: #ddd; cursor: pointer; font-weight: bold; }
        .tab-btn.active { background: var(--cafe); color: white; }
        .item-ticket { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid rgba(0,0,0,0.1); }
    </style>
</head>
<body>

<button class="btn btn-nocturno" onclick="toggleDarkMode()">🌙</button>

<div class="card">
    <h2>🔱 El Tridente - POS</h2>
    <div style="position:relative;">
        <input type="text" id="busqueda" placeholder="🔍 Buscar pan..." onkeyup="buscar()" autocomplete="off">
        <div id="sugerencias" style="display:none; position:absolute; width:100%; background:white; z-index:100; border:1px solid #ccc; border-radius:10px; max-height: 200px; overflow-y: auto; color: #333;"></div>
    </div>
    
    <div style="display:flex; gap:10px;">
        <input type="number" id="cantidad" placeholder="Cant." inputmode="decimal">
        <select id="selectCliente">
            <option value="Mostrador">👤 Mostrador</option>
        </select>
    </div>
    
    <div class="grid-botones">
        <button class="btn btn-add" onclick="agregarAlTicket('venta')">＋ VENTA</button>
        <button class="btn btn-merma" onclick="agregarAlTicket('merma')">🗑️ MERMA</button>
    </div>

    <div id="areaTicket" style="display:none; margin-top:15px; border-top: 2px dashed var(--ocre); padding-top:10px;">
        <div id="listaItems"></div>
        <div style="text-align:right; font-size:1.5rem; font-weight:bold; margin:10px 0;">Total: $<span id="totalActual">0</span></div>
        <div class="grid-pagos">
            <button class="btn btn-pago" style="background:var(--verde);" onclick="finalizarVenta('Efectivo')">💵 Efec.</button>
            <button class="btn btn-pago" style="background:var(--celeste);" onclick="finalizarVenta('Transferencia')">📲 Trans.</button>
            <button class="btn btn-pago" style="background:var(--azul);" onclick="finalizarVenta('Fiado')">📝 Fiado</button>
        </div>
    </div>
</div>

<div class="card">
    <div class="tabs">
        <button class="tab-btn active" onclick="tab(event, 't-stock')">📦 Stock</button>
        <button class="tab-btn" onclick="tab(event, 't-fiados')">🤝 Fiados</button>
        <button class="tab-btn" onclick="tab(event, 't-costos')">⚖️ Costos 🔒</button>
        <button class="tab-btn" onclick="tab(event, 't-balance')">📊 Balance</button>
    </div>

    <div id="t-stock" class="tab-content">
        <table><thead><tr><th>Producto</th><th>Stock</th><th>+</th></tr></thead><tbody id="bodyStock"></tbody></table>
    </div>

    <div id="t-fiados" class="tab-content" style="display:none;">
        <h3>Nuevo Cliente (Para Fiados)</h3>
        <input type="text" id="nuevoCli" placeholder="Nombre completo">
        <input type="text" id="dirCli" placeholder="Dirección">
        <input type="text" id="telCli" placeholder="Teléfono">
        <input type="number" id="diasCli" placeholder="Días de plazo">
        <button class="btn btn-add" onclick="crearCliente()">Guardar Cliente</button>
        <div id="listaFiados" style="margin-top:15px;"></div>
    </div>

    <div id="t-costos" class="tab-content" style="display:none;"><div id="contenidoCostos"></div></div>
    <div id="t-balance" class="tab-content" style="display:none;">
        <div id="resumenBalance" style="font-weight:bold; margin-bottom:10px;"></div>
        <canvas id="chartBalance" style="max-height:200px;"></canvas>
    </div>
</div>

<div class="card">
    <button onclick="toggleSet()" class="btn" style="background:#444; color:white;">⚙️ Ajustes de Productos</button>
    <div id="panelSet" style="display:none; margin-top:10px;">
        <input type="text" id="nP" placeholder="Nombre del pan">
        <input type="number" id="pP" placeholder="Precio Venta">
        <input type="number" id="cP" placeholder="Costo Produc.">
        <input type="number" id="sP" placeholder="Stock Inicial">
        <button class="btn btn-add" onclick="saveProd()">Registrar Producto</button>
        <button class="btn" style="background:var(--rojo); color:white; margin-top:15px;" onclick="resetDB()">⚠️ Borrar Todo</button>
    </div>
    <button class="btn" style="background:#eee; color:#333; margin-top:10px;" onclick="pdf()">📄 Descargar Cierre PDF</button>
</div>

<script>
    let db = JSON.parse(localStorage.getItem('pan_v8_db')) || { prods: [], clientes: [], ventas: [], mermas: [] };
    let ticket = [];
    let seleccionado = null;
    let chartObj = null;
    let accesoCostos = false;

    function save() { localStorage.setItem('pan_v8_db', JSON.stringify(db)); render(); }
    function toggleDarkMode() { document.body.classList.toggle('dark-mode'); }

    function buscar() {
        const t = document.getElementById('busqueda').value.toLowerCase();
        const s = document.getElementById('sugerencias');
        if(!t) { s.style.display='none'; return; }
        const f = db.prods.filter(p => p.nombre.toLowerCase().includes(t));
        s.innerHTML = f.map(p => `<div onclick="pick(${p.id})" style="padding:12px; border-bottom:1px solid #eee; cursor:pointer;">${p.nombre} <b>$${p.precio}</b></div>`).join('');
        s.style.display = 'block';
    }

    function pick(id) {
        seleccionado = db.prods.find(p => p.id === id);
        document.getElementById('busqueda').value = seleccionado.nombre;
        document.getElementById('sugerencias').style.display = 'none';
        document.getElementById('cantidad').focus();
    }

    function agregarAlTicket(tipo) {
        const c = parseFloat(document.getElementById('cantidad').value);
        if(!seleccionado || !c) return alert("Falta producto o cantidad");
        ticket.push({ ...seleccionado, cant: c, tipo: tipo, sub: seleccionado.precio * c });
        document.getElementById('busqueda').value = '';
        document.getElementById('cantidad').value = '';
        seleccionado = null;
        drawTicket();
    }

    function drawTicket() {
        const area = document.getElementById('areaTicket');
        const lista = document.getElementById('listaItems');
        area.style.display = ticket.length ? 'block' : 'none';
        lista.innerHTML = ticket.map((it, i) => `<div class="item-ticket"><span>${it.tipo==='merma'?'[M] ':''}${it.nombre} x ${it.cant}</span><span>$${it.sub.toFixed(2)} <button onclick="ticket.splice(${i},1);drawTicket()" style="background:none; border:none; color:red;">✕</button></span></div>`).join('');
        const total = ticket.reduce((a,b) => a + (b.tipo==='venta' ? b.sub : 0), 0);
        document.getElementById('totalActual').innerText = total.toFixed(2);
    }

    function finalizarVenta(metodo) {
        const cliName = document.getElementById('selectCliente').value;
        if(metodo === 'Fiado' && cliName === 'Mostrador') {
            alert("⚠️ Bloqueado: Debes elegir un cliente para fiar.");
            tab(null, 't-fiados');
            return;
        }
        ticket.forEach(it => {
            const p = db.prods.find(x => x.id === it.id);
            if(p) p.stock_actual -= it.cant;
            if(it.tipo === 'venta') {
                db.ventas.push({ ...it, metodo, cliente: cliName, fecha: new Date().toLocaleDateString(), hora: new Date().getHours() + ":" + new Date().getMinutes() });
                if(metodo === 'Fiado') { const c = db.clientes.find(x => x.nombre === cliName); if(c) c.deuda += it.sub; }
            } else { 
                db.mermas.push({ ...it, fecha: new Date().toLocaleDateString() }); 
            }
        });
        ticket = []; drawTicket(); save(); alert("¡Operación Registrada!");
    }

    function crearCliente() {
        const n = document.getElementById('nuevoCli').value, d = document.getElementById('dirCli').value, t = document.getElementById('telCli').value, p = document.getElementById('diasCli').value;
        if(!n || !d || !t || !p) return alert("Completa todos los campos del cliente.");
        db.clientes.push({ id: Date.now(), nombre: n, direccion: d, telefono: t, plazo: p, deuda: 0 });
        document.getElementById('nuevoCli').value=''; document.getElementById('dirCli').value=''; document.getElementById('telCli').value=''; document.getElementById('diasCli').value='';
        save(); alert("Cliente Creado.");
    }

    function render() {
        document.getElementById('bodyStock').innerHTML = db.prods.map(p => `<tr><td>${p.nombre}</td><td><b>${p.stock_actual}</b></td><td><button onclick="addStock(${p.id})" style="padding:2px 8px;">+</button></td></tr>`).join('');
        const sel = document.getElementById('selectCliente');
        sel.innerHTML = '<option value="Mostrador">👤 Mostrador</option>' + db.clientes.map(c => `<option value="${c.nombre}">${c.nombre}</option>`).join('');
        document.getElementById('listaFiados').innerHTML = db.clientes.map(c => `<div style="padding:10px; background:rgba(0,0,0,0.05); margin-bottom:5px; border-radius:8px;"><b>${c.nombre}</b><br>Deuda: <span style="color:red">$${c.deuda.toFixed(2)}</span> <button onclick="pagarDeuda(${c.id})" style="float:right">Cobrar</button></div>`).join('');
        
        if(accesoCostos) {
            document.getElementById('contenidoCostos').innerHTML = db.prods.map(p => `<div>${p.nombre}: Ganancia $${(p.precio - p.costo).toFixed(2)}</div>`).join('');
        } else {
            document.getElementById('contenidoCostos').innerHTML = '<div style="text-align:center; padding:20px;">🔒 Contenido Protegido</div>';
        }
        actualizarBalance();
    }

    function actualizarBalance() {
        const hoy = new Date().toLocaleDateString();
        const vHoy = db.ventas.filter(v => v.fecha === hoy);
        const total = vHoy.reduce((a,b) => a+b.sub, 0);
        document.getElementById('resumenBalance').innerHTML = `Ventas de Hoy: $${total.toFixed(2)}`;
    }

    function addStock(id) { const q = parseFloat(prompt("Cantidad a sumar:")); if(!isNaN(q)) { db.prods.find(x => x.id === id).stock_actual += q; save(); } }
    function pagarDeuda(id) { const c = db.clientes.find(x => x.id === id); const p = parseFloat(prompt(`Saldo: $${c.deuda}. ¿Cuánto paga?`)); if(!isNaN(p)) { c.deuda -= p; save(); } }
    
    function saveProd() { 
        const n = document.getElementById('nP').value, p = parseFloat(document.getElementById('pP').value), c = parseFloat(document.getElementById('cP').value)||0, s = parseFloat(document.getElementById('sP').value)||0; 
        if(n && p) { db.prods.push({ id: Date.now(), nombre: n, precio: p, costo: c, stock_actual: s }); save(); toggleSet(); } 
    }

    function tab(e, id) {
        if(id === 't-costos' && !accesoCostos) { const p = prompt("Clave de Acceso:"); if(p === "37958601") { accesoCostos = true; render(); } else return; }
        document.querySelectorAll('.tab-content').forEach(x => x.style.display = 'none');
        document.querySelectorAll('.tab-btn').forEach(x => x.classList.remove('active'));
        document.getElementById(id).style.display = 'block';
        if(e) e.target.classList.add('active');
    }

    function toggleSet() { const p = document.getElementById('panelSet'); p.style.display = p.style.display === 'none' ? 'block' : 'none'; }
    function resetDB() { if(confirm("¿Seguro? Se borrará todo.")) { localStorage.clear(); location.reload(); } }
    async function pdf() { 
        const { jsPDF } = window.jspdf; const doc = new jsPDF(); 
        doc.text("Cierre El Tridente - " + new Date().toLocaleDateString(), 10, 10);
        const rows = db.ventas.filter(v => v.fecha === new Date().toLocaleDateString()).map(v => [v.hora, v.nombre, v.metodo, v.sub.toFixed(2)]);
        doc.autoTable({ head: [['Hora', 'Pan', 'Pago', 'Total']], body: rows });
        doc.save("Cierre_Caja.pdf");
    }

    render();
</script>
</body>
</html>
