<!DOCTYPE html>
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
    :root { --bg: #f2f2f7; --card: #ffffff; --primary: #007aff; --secondary: #5856d6; --success: #34c759; --danger: #ff3b30; --wallapop: #13c1a3; --border: #e5e5ea; }
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
    .badge-ok { color: var(--success); font-weight: bold; margin-left: 4px; }
    .badge-combo { background: #ff9500; color: white; padding: 2px 6px; border-radius: 4px; font-size: 11px; font-weight: bold; margin-left: 6px; }
    .badge-noref { color: #8e8e93; font-style: italic; font-size: 12px; margin-left: 4px; }
    .badge-wallapop { background: var(--wallapop); color: white; padding: 2px 6px; border-radius: 4px; font-size: 11px; font-weight: bold; margin-left: 4px; }
    .badge-no-disp { color: var(--danger); font-size: 12px; margin-left: 4px; font-weight: bold; }
    .modal { display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.4); padding: 20px; overflow-y: auto; z-index: 1000; }
    .modal-content { background: white; border-radius: 14px; padding: 20px; max-width: 500px; margin: 20px auto; }
    label { display: block; font-size: 13px; font-weight: 600; margin-top: 10px; color: #6c6c70; }
    input[type="text"], textarea { width: 100%; padding: 8px; margin-top: 4px; border: 1px solid var(--border); border-radius: 6px; box-sizing: border-box; font-size: 15px; }
    .input-row { display: flex; align-items: center; gap: 8px; margin-top: 4px; flex-wrap: wrap; }
    .input-row input[type="text"] { flex: 1; min-width: 120px; margin-top: 0; }
    .sub-options { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 4px; background: #f8f8f8; padding: 6px 10px; border-radius: 6px; border: 1px solid #f0f0f0; }
    .checkbox-inline { display: flex; align-items: center; gap: 4px; font-size: 12px; font-weight: 600; color: #3a3a3c; white-space: nowrap; }
    .combo-divider { background: #e5e5ea; padding: 8px; border-radius: 8px; margin-top: 8px; text-align: center; }
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
        <input type="text" id="main">
        <div class="sub-options">
          <label class="checkbox-inline"><input type="checkbox" id="mainOk"> OK</label>
          <label class="checkbox-inline"><input type="checkbox" id="mainNoRef"> Sin ref.</label>
          <label class="checkbox-inline"><input type="checkbox" id="mainWallapop"> Wallapop</label>
          <label class="checkbox-inline"><input type="checkbox" id="mainDisponible" checked> Disponible</label>
        </div>

        <div class="combo-divider">
          <label class="checkbox-inline" style="justify-content: center; font-size: 13px; color: #ff9500;">
            <input type="checkbox" id="esCombo"> 🔗 Main + Fuente en Combo
          </label>
        </div>

        <label>Fuente</label>
        <input type="text" id="fuente">
        <div class="sub-options">
          <label class="checkbox-inline"><input type="checkbox" id="fuenteOk"> OK</label>
          <label class="checkbox-inline"><input type="checkbox" id="fuenteNoRef"> Sin ref.</label>
          <label class="checkbox-inline"><input type="checkbox" id="fuenteWallapop"> Wallapop</label>
          <label class="checkbox-inline"><input type="checkbox" id="fuenteDisponible" checked> Disponible</label>
        </div>

        <label>T-Con</label>
        <input type="text" id="tcon">
        <div class="sub-options">
          <label class="checkbox-inline"><input type="checkbox" id="tconOk"> OK</label>
          <label class="checkbox-inline"><input type="checkbox" id="tconNoRef"> Sin ref.</label>
          <label class="checkbox-inline"><input type="checkbox" id="tconWallapop"> Wallapop</label>
          <label class="checkbox-inline"><input type="checkbox" id="tconDisponible" checked> Disponible</label>
        </div>

        <label>Tiras LED</label>
        <input type="text" id="tirasLed">
        <div class="sub-options">
          <label class="checkbox-inline"><input type="checkbox" id="tirasLedOk"> OK</label>
          <label class="checkbox-inline"><input type="checkbox" id="tirasLedNoRef"> Sin ref.</label>
          <label class="checkbox-inline"><input type="checkbox" id="tirasLedWallapop"> Wallapop</label>
          <label class="checkbox-inline"><input type="checkbox" id="tirasLedDisponible" checked> Disponible</label>
        </div>

        <label>Panel</label>
        <div class="input-row">
          <input type="text" id="panel">
          <label class="checkbox-inline"><input type="checkbox" id="panelNoRef"> Sin ref.</label>
        </div>

        <label>Flex Main / T-Con</label>
        <div class="input-row">
          <input type="text" id="flexMainTcon">
          <label class="checkbox-inline"><input type="checkbox" id="flexMainTconNoRef"> Sin ref.</label>
        </div>

        <label>Flex Main / Panel</label>
        <div class="input-row">
          <input type="text" id="flexMainPanel">
          <label class="checkbox-inline"><input type="checkbox" id="flexMainPanelNoRef"> Sin ref.</label>
        </div>

        <label>Wi-Fi</label>
        <div class="input-row">
          <input type="text" id="wifi">
          <label class="checkbox-inline"><input type="checkbox" id="wifiNoRef"> Sin ref.</label>
        </div>

        <label>RF</label>
        <div class="input-row">
          <input type="text" id="rf">
          <label class="checkbox-inline"><input type="checkbox" id="rfNoRef"> Sin ref.</label>
        </div>

        <label>Botonera</label>
        <div class="input-row">
          <input type="text" id="botonera">
          <label class="checkbox-inline"><input type="checkbox" id="botoneraNoRef"> Sin ref.</label>
        </div>

        <div class="checkbox-group">
          <input type="checkbox" id="tienePeana">
          <label for="tienePeana" style="margin: 0; font-size: 14px;">Peana / Patas disponibles</label>
        </div>

        <label>Otros</label>
        <div class="input-row">
          <textarea id="otros" rows="2"></textarea>
          <label class="checkbox-inline"><input type="checkbox" id="otrosNoRef"> Sin ref.</label>
        </div>
        
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

  <!-- Modal Backup para iOS -->
  <div id="backup-modal" class="modal">
    <div class="modal-content">
      <h2>Backup de tu Base de Datos</h2>
      <p style="font-size: 13px; color: #6c6c70;">Copia el siguiente texto o guárdalo como archivo para transferirlo:</p>
      <textarea id="backup-text" rows="8" readonly onclick="this.select()"></textarea>
      <div class="actions">
        <button class="btn btn-secondary" onclick="document.getElementById('backup-modal').style.display='none'">Cerrar</button>
        <button class="btn" onclick="copyBackupText()">📋 Copiar Texto</button>
      </div>
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

    // --- COPIAS DE SEGURIDAD Y FUSIÓN SIN DUPLICADOS ---

    function exportBackup() {
      getTVs((tvs) => {
        if (tvs.length === 0) {
          alert("No hay registros para exportar.");
          return;
        }
        
        const jsonStr = JSON.stringify(tvs, null, 2);
        
        try {
          const blob = new Blob([jsonStr], { type: "application/json" });
          const url = URL.createObjectURL(blob);
          const fecha = new Date().toISOString().slice(0, 10);
          const downloadAnchor = document.createElement('a');
          downloadAnchor.href = url;
          downloadAnchor.download = `backup_despiece_tv_${fecha}.json`;
          document.body.appendChild(downloadAnchor);
          downloadAnchor.click();
          downloadAnchor.remove();
          
          if (window.navigator.standalone) {
            showBackupModal(jsonStr);
          }
        } catch (e) {
          showBackupModal(jsonStr);
        }
      });
    }

    function showBackupModal(jsonStr) {
      document.getElementById("backup-text").value = jsonStr;
      document.getElementById("backup-modal").style.display = "block";
    }

    function copyBackupText() {
      const textarea = document.getElementById("backup-text");
      textarea.select();
      textarea.setSelectionRange(0, 99999);
      navigator.clipboard.writeText(textarea.value).then(() => {
        alert("¡Texto copiado al portapapeles!");
      }).catch(() => {
        document.execCommand("copy");
        alert("¡Texto copiado al portapapeles!");
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

          getTVs((existingTVs) => {
            if (confirm(`Se han encontrado ${importedTVs.length} registros en el archivo. ¿Deseas unificarlos sin duplicar coincidencia de Marca y Modelo?`)) {
              const tx = db.transaction("tvs", "readwrite");
              const store = tx.objectStore("tvs");

              importedTVs.forEach(importedItem => {
                // Buscar si la TV ya existe por Marca + Modelo (ignorando mayúsculas/minúsculas)
                const existing = existingTVs.find(curr => 
                  curr.marca.trim().toLowerCase() === importedItem.marca.trim().toLowerCase() && 
                  curr.modelo.trim().toLowerCase() === importedItem.modelo.trim().toLowerCase()
                );

                if (existing) {
                  // Si existe, actualiza sus datos conservando la ID original
                  importedItem.id = existing.id;
                  store.put(importedItem);
                } else {
                  // Si es nueva, la añade eliminando cualquier ID previa para autoincrementar
                  delete importedItem.id;
                  store.add(importedItem);
                }
              });

              tx.oncomplete = () => {
                alert("¡Base de datos importada e unificada correctamente!");
                renderTVs();
                document.getElementById('importFile').value = '';
              };
            }
          });
        } catch (err) {
          alert("Error al leer el archivo. Asegúrate de que es un backup válido (.json).");
        }
      };
      reader.readAsText(file);
    }

    // --- INTERFAZ DE USUARIO ---

    function formatField(label, val, ok, noRef, wallapop, disponible) {
      if (!val && !noRef) return '';
      let text = val ? val : 'Disponible';
      let badges = '';
      if (ok) badges += '<span class="badge-ok">✓ Funcional</span>';
      if (noRef) badges += '<span class="badge-noref">(Sin ref.)</span>';
      if (wallapop) badges += '<span class="badge-wallapop">Wallapop</span>';
      if (disponible === false) badges += '<span class="badge-no-disp">(No disp.)</span>';
      return `<div class="field"><b>${label}:</b> ${text} ${badges}</div>`;
    }

    function renderTVs() {
      const query = document.getElementById("search").value.toLowerCase();
      getTVs((tvs) => {
        const list = document.getElementById("tv-list");
        list.innerHTML = "";
        
        const filtered = tvs.filter(tv => 
          (tv.marca + " " + tv.modelo + " " + (tv.main||"") + " " + (tv.fuente||"") + " " + (tv.panel||"") + " " + (tv.botonera||"") + " " + (tv.modelosCompatibles || ""))
          .toLowerCase().includes(query)
        );

        filtered.reverse().forEach(tv => {
          const card = document.createElement("div");
          card.className = "card";
          card.innerHTML = `
            <h3>${tv.marca} - ${tv.modelo} ${tv.esPanelCompatible ? '✅' : ''} ${tv.esCombo ? '<span class="badge-combo">COMBO</span>' : ''}</h3>
            ${formatField('Main', tv.main, tv.mainOk, tv.mainNoRef, tv.mainWallapop, tv.mainDisponible)}
            ${formatField('Fuente', tv.fuente, tv.fuenteOk, tv.fuenteNoRef, tv.fuenteWallapop, tv.fuenteDisponible)}
            ${formatField('T-Con', tv.tcon, tv.tconOk, tv.tconNoRef, tv.tconWallapop, tv.tconDisponible)}
            ${formatField('Tiras LED', tv.tirasLed, tv.tirasLedOk, tv.tirasLedNoRef, tv.tirasLedWallapop, tv.tirasLedDisponible)}
            ${formatField('Panel', tv.panel, false, tv.panelNoRef)}
            ${formatField('Flex Main/T-Con', tv.flexMainTcon, false, tv.flexMainTconNoRef)}
            ${formatField('Flex Main/Panel', tv.flexMainPanel, false, tv.flexMainPanelNoRef)}
            ${formatField('Wi-Fi', tv.wifi, false, tv.wifiNoRef)}
            ${formatField('RF', tv.rf, false, tv.rfNoRef)}
            ${formatField('Botonera', tv.botonera, false, tv.botoneraNoRef)}
            ${tv.tienePeana ? '<div class="field"><b>Peana / Patas:</b> Disponible</div>' : ''}
            ${formatField('Otros', tv.otros, false, tv.otrosNoRef)}
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
      
      document.getElementById("mainDisponible").checked = true;
      document.getElementById("fuenteDisponible").checked = true;
      document.getElementById("tconDisponible").checked = true;
      document.getElementById("tirasLedDisponible").checked = true;

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
      document.getElementById("mainNoRef").checked = tv.mainNoRef || false;
      document.getElementById("mainWallapop").checked = tv.mainWallapop || false;
      document.getElementById("mainDisponible").checked = tv.mainDisponible !== undefined ? tv.mainDisponible : true;

      document.getElementById("esCombo").checked = tv.esCombo || false;

      document.getElementById("fuente").value = tv.fuente || "";
      document.getElementById("fuenteOk").checked = tv.fuenteOk || false;
      document.getElementById("fuenteNoRef").checked = tv.fuenteNoRef || false;
      document.getElementById("fuenteWallapop").checked = tv.fuenteWallapop || false;
      document.getElementById("fuenteDisponible").checked = tv.fuenteDisponible !== undefined ? tv.fuenteDisponible : true;

      document.getElementById("tcon").value = tv.tcon || "";
      document.getElementById("tconOk").checked = tv.tconOk || false;
      document.getElementById("tconNoRef").checked = tv.tconNoRef || false;
      document.getElementById("tconWallapop").checked = tv.tconWallapop || false;
      document.getElementById("tconDisponible").checked = tv.tconDisponible !== undefined ? tv.tconDisponible : true;

      document.getElementById("tirasLed").value = tv.tirasLed || "";
      document.getElementById("tirasLedOk").checked = tv.tirasLedOk || false;
      document.getElementById("tirasLedNoRef").checked = tv.tirasLedNoRef || false;
      document.getElementById("tirasLedWallapop").checked = tv.tirasLedWallapop || false;
      document.getElementById("tirasLedDisponible").checked = tv.tirasLedDisponible !== undefined ? tv.tirasLedDisponible : true;

      document.getElementById("panel").value = tv.panel || "";
      document.getElementById("panelNoRef").checked = tv.panelNoRef || false;

      document.getElementById("flexMainTcon").value = tv.flexMainTcon || "";
      document.getElementById("flexMainTconNoRef").checked = tv.flexMainTconNoRef || false;

      document.getElementById("flexMainPanel").value = tv.flexMainPanel || "";
      document.getElementById("flexMainPanelNoRef").checked = tv.flexMainPanelNoRef || false;

      document.getElementById("wifi").value = tv.wifi || "";
      document.getElementById("wifiNoRef").checked = tv.wifiNoRef || false;

      document.getElementById("rf").value = tv.rf || "";
      document.getElementById("rfNoRef").checked = tv.rfNoRef || false;

      document.getElementById("botonera").value = tv.botonera || "";
      document.getElementById("botoneraNoRef").checked = tv.botoneraNoRef || false;

      document.getElementById("tienePeana").checked = tv.tienePeana || false;

      document.getElementById("otros").value = tv.otros || "";
      document.getElementById("otrosNoRef").checked = tv.otrosNoRef || false;

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
        mainNoRef: document.getElementById("mainNoRef").checked,
        mainWallapop: document.getElementById("mainWallapop").checked,
        mainDisponible: document.getElementById("mainDisponible").checked,

        esCombo: document.getElementById("esCombo").checked,

        fuente: document.getElementById("fuente").value,
        fuenteOk: document.getElementById("fuenteOk").checked,
        fuenteNoRef: document.getElementById("fuenteNoRef").checked,
        fuenteWallapop: document.getElementById("fuenteWallapop").checked,
        fuenteDisponible: document.getElementById("fuenteDisponible").checked,

        tcon: document.getElementById("tcon").value,
        tconOk: document.getElementById("tconOk").checked,
        tconNoRef: document.getElementById("tconNoRef").checked,
        tconWallapop: document.getElementById("tconWallapop").checked,
        tconDisponible: document.getElementById("tconDisponible").checked,

        tirasLed: document.getElementById("tirasLed").value,
        tirasLedOk: document.getElementById("tirasLedOk").checked,
        tirasLedNoRef: document.getElementById("tirasLedNoRef").checked,
        tirasLedWallapop: document.getElementById("tirasLedWallapop").checked,
        tirasLedDisponible: document.getElementById("tirasLedDisponible").checked,

        panel: document.getElementById("panel").value,
        panelNoRef: document.getElementById("panelNoRef").checked,

        flexMainTcon: document.getElementById("flexMainTcon").value,
        flexMainTconNoRef: document.getElementById("flexMainTconNoRef").checked,

        flexMainPanel: document.getElementById("flexMainPanel").value,
        flexMainPanelNoRef: document.getElementById("flexMainPanelNoRef").checked,

        wifi: document.getElementById("wifi").value,
        wifiNoRef: document.getElementById("wifiNoRef").checked,

        rf: document.getElementById("rf").value,
        rfNoRef: document.getElementById("rfNoRef").checked,

        botonera: document.getElementById("botonera").value,
        botoneraNoRef: document.getElementById("botoneraNoRef").checked,

        tienePeana: document.getElementById("tienePeana").checked,

        otros: document.getElementById("otros").value,
        otrosNoRef: document.getElementById("otrosNoRef").checked,

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

