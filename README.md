# Despiece
<by GDGG>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Despiece TV</title>
  
  <!-- Configuración PWA para iOS -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="Despiece TV">
  
  <style>
    :root { --bg: #f2f2f7; --card: #ffffff; --primary: #007aff; --secondary: #5856d6; --success: #34c759; --danger: #ff3b30; --border: #e5e5ea; }
    body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: var(--bg); margin: 0; padding: 16px; color: #1c1c1e; }
    h1 { font-size: 24px; margin-bottom: 12px; }
    .top-bar { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; }
    .btn { background: var(--primary); color: white; border: none; padding: 10px 14px; border-radius: 8px; font-weight: 600; font-size: 14px; cursor: pointer; display: inline-flex; align-items: center; justify-content: center; gap: 4px; }
    .btn-secondary { background: #8e8e93; }
    .btn-export { background: var(--secondary); }
    .btn-import { background: var(--success); }
    .btn-danger { background: var(--danger); }
    .search { width: 100%; padding: 10px; border-radius: 10px; border: 1px solid var(--border); box-sizing: border-box; font-size: 16px; margin-bottom: 12px; }
    .card { background: var(--card); border-radius: 12px; padding: 14px; margin-bottom: 10px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
    .card h3 { margin: 0 0 6px 0; font-size: 18px; display: flex; justify-content: space-between; }
    .field { font-size: 14px; margin: 4px 0; color: #3a3a3c; }
    .badge-ok { color: var(--success); font-weight: bold; margin-left: 6px; }
    .modal { display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.4); padding: 20px; overflow-y: auto; z-index: 1000; }
    .modal-content { background: white; border-radius: 14px; padding: 20px; max-width: 500px; margin: 20px auto; }
    label { display: block; font-size: 13px; font-weight: 600; margin-top: 10px; color: #6c6c70; }
    input[type="text"], textarea { width: 100%; padding: 8px; margin-top: 4px; border: 1px solid var(--border); border-radius: 6px; box-sizing: border-box; font-size: 15px; }
    .input-row { display: flex; align-items: center; gap: 8px; margin-top: 4px; }
    .input-row input[type="text"] { flex: 1; margin-top: 0; }
    .checkbox-inline { display: flex; align-items: center; gap: 4px; font-size: 13px; font-weight: 600; color: #3a3a3c; white-space: nowrap; }
    .checkbox-group { display: flex; align-items: center; gap: 8px; margin-top: 12px; }
    .actions { display: flex; gap: 10px; margin-top: 16px; }
  </style>
</head>
<body>

  <h1>Despiece TVs</h1>

  <div class="top-bar">
    <button class="btn" onclick="openModal()">+ Nuevo Despiece</button>
    <button class="btn btn-export" onclick="exportBackup()">📤 Exportar Backup</button>
    <button class="btn btn-import" onclick="document.getElementById('importFile').click()">📥 Importar Backup</button>
    <input type="file" id="importFile" accept=".json" style="display: none;" onchange="importBackup(event)">
  </div>

  <input type="text" id="search" class="search" placeholder="Buscar por marca, modelo o pieza..." oninput="renderTVs()">

  <div id="tv-list"></div>

  <!-- Formulario Modal -->
  <div id="modal" class="modal">
    <div class="modal-content">
      <h2 id="modal-title">Nuevo Despiece</h2>
      <form id="tv-form">
        <input type="hidden" id="tv-id">
        
        <label>Marca *</label>
        <input type="text" id="marca" required>
        
        <label>Modelo *</label>
        <input type="text" id="modelo" required>
        
        <label>Main</label>
        <div class="input-row">
          <input type="text" id="main">
          <label class="checkbox-inline"><input type="checkbox" id="mainOk"> OK</label>
        </div>

        <label>Fuente</label>
        <div class="input-row">
          <input type="text" id="fuente">
          <label class="checkbox-inline"><input type="checkbox" id="fuenteOk"> OK</label>
        </div>

        <label>T-Con</label>
        <div class="input-row">
          <input type="text" id="tcon">
          <label class="checkbox-inline"><input type="checkbox" id="tconOk"> OK</label>
        </div>

        <label>Tiras LED</label>
        <div class="input-row">
          <input type="text" id="tirasLed">
          <label class="checkbox-inline"><input type="checkbox" id="tirasLedOk"> OK</label>
        </div>

        <label>Panel</label>
        <input type="text" id="panel">

        <label>Flex Main / T-Con</label>
        <input type="text" id="flexMainTcon">

        <label>Flex Main / Panel</label>
        <input type="text" id="flexMainPanel">

        <label>Wi-Fi</label>
        <input type="text" id="wifi">

        <label>RF</label>
        <input type="text" id="rf">

        <label>Otros</label>
        <textarea id="otros" rows="2"></textarea>
        
        <div class="checkbox-group">
          <input type="checkbox" id="esPanelCompatible">
          <label for="esPanelCompatible" style="margin: 0;">Panel compatible</label>
        </div>
        
        <label>Modelos compatibles</label>
        <textarea id="modelosCompatibles" rows="2"></textarea>

        <div class="actions">
          <button type="button" class="btn btn-secondary" onclick="closeModal()">Cancelar</button>
          <button type="submit" class="btn">Guardar</button>
        </div>
      </form>
    </div>
  </div>

  <script>
    let db;
    const request = indexedDB.open("TVDatabase", 1);

    request.onupgradeneeded = (e) => {
      db = e.target.result;
      if (!db.objectStoreNames.contains("tvs")) {
        db.createObjectStore("tvs", { keyPath: "id", autoIncrement: true });
      }
    };

    request.onsuccess = (e) => {
      db = e.target.result;
      renderTVs();
    };

    function getTVs(callback) {
      const tx = db.transaction("tvs", "readonly");
      const store = tx.objectStore("tvs");
      const req = store.getAll();
      req.onsuccess = () => callback(req.result);
    }

    function saveTV(tv, callback) {
      const tx = db.transaction("tvs", "readwrite");
      const store = tx.objectStore("tvs");
      store.put(tv);
      tx.oncomplete = callback;
    }

    function deleteTV(id) {
      if (confirm("¿Eliminar este registro?")) {
        const tx = db.transaction("tvs", "readwrite");
        const store = tx.objectStore("tvs");
        store.delete(id);
        tx.oncomplete = renderTVs;
      }
    }

    // --- COPIAS DE SEGURIDAD ---

    function exportBackup() {
      getTVs((tvs) => {
        if (tvs.length === 0) {
          alert("No hay registros para exportar.");
          return;
        }
        const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(tvs, null, 2));
        const downloadAnchor = document.createElement('a');
        const fecha = new Date().toISOString().slice(0, 10);
        downloadAnchor.setAttribute("href", dataStr);
        downloadAnchor.setAttribute("download", `backup_despiece_tv_${fecha}.json`);
        document.body.appendChild(downloadAnchor);
        downloadAnchor.click();
        downloadAnchor.remove();
      });
    }

    function importBackup(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function(e) {
        try {
          const importedTVs = JSON.parse(e.target.result);
          if (!Array.isArray(importedTVs)) throw new Error("Formato inválido");

          if (confirm(`Se van a importar ${importedTVs.length} registros. ¿Deseas continuar?`)) {
            const tx = db.transaction("tvs", "readwrite");
            const store = tx.objectStore("tvs");
            
            importedTVs.forEach(tv => {
              delete tv.id;
              store.add(tv);
            });

            tx.oncomplete = () => {
              alert("¡Copia de seguridad importada con éxito!");
              renderTVs();
              document.getElementById('importFile').value = '';
            };
          }
        } catch (err) {
          alert("Error al leer el archivo. Asegúrate de que es un backup válido (.json).");
        }
      };
      reader.readAsText(file);
    }

    // --- INTERFAZ DE USUARIO ---

    function renderTVs() {
      const query = document.getElementById("search").value.toLowerCase();
      getTVs((tvs) => {
        const list = document.getElementById("tv-list");
        list.innerHTML = "";
        
        const filtered = tvs.filter(tv => 
          (tv.marca + " " + tv.modelo + " " + (tv.main||"") + " " + (tv.fuente||"") + " " + (tv.panel||"") + " " + (tv.modelosCompatibles || ""))
          .toLowerCase().includes(query)
        );

        filtered.reverse().forEach(tv => {
          const card = document.createElement("div");
          card.className = "card";
          card.innerHTML = `
            <h3>${tv.marca} - ${tv.modelo} ${tv.esPanelCompatible ? '✅' : ''}</h3>
            ${tv.main ? `<div class="field"><b>Main:</b> ${tv.main} ${tv.mainOk ? '<span class="badge-ok">✓ Funcional</span>' : ''}</div>` : ''}
            ${tv.fuente ? `<div class="field"><b>Fuente:</b> ${tv.fuente} ${tv.fuenteOk ? '<span class="badge-ok">✓ Funcional</span>' : ''}</div>` : ''}
            ${tv.tcon ? `<div class="field"><b>T-Con:</b> ${tv.tcon} ${tv.tconOk ? '<span class="badge-ok">✓ Funcional</span>' : ''}</div>` : ''}
            ${tv.tirasLed ? `<div class="field"><b>Tiras LED:</b> ${tv.tirasLed} ${tv.tirasLedOk ? '<span class="badge-ok">✓ Funcional</span>' : ''}</div>` : ''}
            ${tv.panel ? `<div class="field"><b>Panel:</b> ${tv.panel}</div>` : ''}
            ${tv.modelosCompatibles ? `<div class="field"><b>Compatibles:</b> ${tv.modelosCompatibles}</div>` : ''}
            <div class="actions" style="margin-top: 10px;">
              <button class="btn btn-secondary" onclick='editTV(${JSON.stringify(tv)})'>Editar</button>
              <button class="btn btn-danger" onclick="deleteTV(${tv.id})">Borrar</button>
            </div>
          `;
          list.appendChild(card);
        });
      });
    }

    function openModal() {
      document.getElementById("tv-form").reset();
      document.getElementById("tv-id").value = "";
      document.getElementById("modal-title").innerText = "Nuevo Despiece";
      document.getElementById("modal").style.display = "block";
    }

    function closeModal() {
      document.getElementById("modal").style.display = "none";
    }

    function editTV(tv) {
      document.getElementById("tv-id").value = tv.id;
      document.getElementById("marca").value = tv.marca;
      document.getElementById("modelo").value = tv.modelo;
      
      document.getElementById("main").value = tv.main || "";
      document.getElementById("mainOk").checked = tv.mainOk || false;

      document.getElementById("fuente").value = tv.fuente || "";
      document.getElementById("fuenteOk").checked = tv.fuenteOk || false;

      document.getElementById("tcon").value = tv.tcon || "";
      document.getElementById("tconOk").checked = tv.tconOk || false;

      document.getElementById("tirasLed").value = tv.tirasLed || "";
      document.getElementById("tirasLedOk").checked = tv.tirasLedOk || false;

      document.getElementById("panel").value = tv.panel || "";
      document.getElementById("flexMainTcon").value = tv.flexMainTcon || "";
      document.getElementById("flexMainPanel").value = tv.flexMainPanel || "";
      document.getElementById("wifi").value = tv.wifi || "";
      document.getElementById("rf").value = tv.rf || "";
      document.getElementById("otros").value = tv.otros || "";
      document.getElementById("esPanelCompatible").checked = tv.esPanelCompatible || false;
      document.getElementById("modelosCompatibles").value = tv.modelosCompatibles || "";
      
      document.getElementById("modal-title").innerText = "Editar Despiece";
      document.getElementById("modal").style.display = "block";
    }

    document.getElementById("tv-form").onsubmit = (e) => {
      e.preventDefault();
      const id = document.getElementById("tv-id").value;
      const tv = {
        marca: document.getElementById("marca").value,
        modelo: document.getElementById("modelo").value,
        
        main: document.getElementById("main").value,
        mainOk: document.getElementById("mainOk").checked,

        fuente: document.getElementById("fuente").value,
        fuenteOk: document.getElementById("fuenteOk").checked,

        tcon: document.getElementById("tcon").value,
        tconOk: document.getElementById("tconOk").checked,

        tirasLed: document.getElementById("tirasLed").value,
        tirasLedOk: document.getElementById("tirasLedOk").checked,

        panel: document.getElementById("panel").value,
        flexMainTcon: document.getElementById("flexMainTcon").value,
        flexMainPanel: document.getElementById("flexMainPanel").value,
        wifi: document.getElementById("wifi").value,
        rf: document.getElementById("rf").value,
        otros: document.getElementById("otros").value,
        esPanelCompatible: document.getElementById("esPanelCompatible").checked,
        modelosCompatibles: document.getElementById("modelosCompatibles").value
      };
      if (id) tv.id = parseInt(id);

      saveTV(tv, () => {
        closeModal();
        renderTVs();
      });
    };
  </script>
</body>
</html>
