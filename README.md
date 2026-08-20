<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>LogiScan PWA - Control de Entregas</title>
  <meta name="theme-color" content="#2563eb">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- HTML5-QRCode Library -->
  <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
  <!-- Lucide Icons -->
  <script src="https://unpkg.com/lucide@latest"></script>

  <style>
    /* Custom styles & camera overlay */
    #reader video {
      object-fit: cover !important;
      border-radius: 12px;
      width: 100% !important;
    }
    #reader {
      border: none !important;
    }
    #reader__scan_region {
      background: rgba(0,0,0,0.05);
    }
    .badge-success { background-color: #dcfce7; color: #15803d; border: 1px solid #bbf7d0; }
    .badge-danger { background-color: #fee2e2; color: #b91c1c; border: 1px solid #fecaca; }
  </style>
</head>
<body class="bg-slate-50 text-slate-800 font-sans min-h-screen flex flex-col justify-between antialiased selection:bg-blue-500 selection:text-white">

  <!-- Header -->
  <header class="bg-blue-600 text-white shadow-lg sticky top-0 z-30">
    <div class="max-w-md mx-auto px-4 py-3 flex items-center justify-between">
      <div class="flex items-center space-x-2">
        <i data-lucide="package-check" class="w-6 h-6"></i>
        <h1 class="font-bold text-lg tracking-tight">LogiScan PWA</h1>
      </div>
      <div class="flex items-center space-x-2">
        <button id="btn-config" onclick="toggleConfigModal()" class="p-2 rounded-lg hover:bg-blue-700 transition" title="Configuración / Webhook">
          <i data-lucide="settings" class="w-5 h-5"></i>
        </button>
      </div>
    </div>
  </header>

  <!-- Main Container -->
  <main class="max-w-md mx-auto w-full flex-1 p-4 space-y-4">

    <!-- Tab Navigation -->
    <div class="flex bg-slate-200 p-1 rounded-xl font-medium text-sm">
      <button id="tab-scan-btn" onclick="switchTab('scan')" class="flex-1 py-2 rounded-lg bg-white shadow-sm text-blue-600 flex items-center justify-center space-x-1.5 transition">
        <i data-lucide="scan-barcode" class="w-4 h-4"></i>
        <span>Escanear</span>
      </button>
      <button id="tab-list-btn" onclick="switchTab('list')" class="flex-1 py-2 rounded-lg text-slate-600 hover:text-slate-900 flex items-center justify-center space-x-1.5 transition">
        <i data-lucide="list-todo" class="w-4 h-4"></i>
        <span>Registros (<span id="record-count">0</span>)</span>
      </button>
    </div>

    <!-- TAB 1: SCANNER -->
    <div id="tab-scan" class="space-y-4">
      
      <!-- Scanner Container -->
      <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-4 text-center relative overflow-hidden">
        <div id="scanner-wrapper" class="relative">
          <div id="reader" class="w-full rounded-xl overflow-hidden bg-slate-900 min-h-[260px] flex items-center justify-center text-slate-400">
            <div class="text-center p-6 space-y-3">
              <i data-lucide="camera" class="w-12 h-12 mx-auto text-slate-400 stroke-[1.5]"></i>
              <p class="text-sm font-medium">Presiona abajo para activar la cámara</p>
            </div>
          </div>
        </div>

        <!-- Scanner Controls -->
        <div class="mt-4 flex flex-col space-y-2">
          <button id="btn-toggle-camera" onclick="toggleCamera()" class="w-full py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-xl shadow-md flex items-center justify-center space-x-2 transition active:scale-[0.99]">
            <i data-lucide="power" class="w-5 h-5"></i>
            <span id="camera-btn-text">Iniciar Escáner</span>
          </button>

          <button onclick="toggleManualModal()" class="w-full py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 font-medium rounded-xl flex items-center justify-center space-x-1.5 text-sm transition">
            <i data-lucide="keyboard" class="w-4 h-4"></i>
            <span>Ingreso manual de código</span>
          </button>
        </div>
      </div>

      <!-- Quick Info Box -->
      <div class="bg-blue-50 border border-blue-200 rounded-xl p-3 text-xs text-blue-800 flex items-start space-x-2.5">
        <i data-lucide="info" class="w-4 h-4 text-blue-600 flex-shrink-0 mt-0.5"></i>
        <p>Apunta el código de barras o QR. El sistema detectará automáticamente el identificador y desplegará la pantalla de confirmación.</p>
      </div>

    </div>

    <!-- TAB 2: RECORDS LIST -->
    <div id="tab-list" class="hidden space-y-4">
      
      <!-- Filter & Export Toolbar -->
      <div class="bg-white p-3 rounded-xl border border-slate-200 shadow-sm flex items-center justify-between gap-2">
        <select id="filter-status" onchange="renderRecords()" class="bg-slate-100 border-none rounded-lg text-xs font-medium px-2.5 py-2 text-slate-700 focus:ring-2 focus:ring-blue-500">
          <option value="ALL">Todos los registros</option>
          <option value="ENTREGADO">Sólo Entregados</option>
          <option value="NO_ENTREGADO">Sólo Incidencias</option>
        </select>

        <div class="flex items-center space-x-1.5">
          <button onclick="exportToCSV()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-1.5 rounded-lg text-xs font-semibold flex items-center space-x-1 transition shadow-sm">
            <i data-lucide="download" class="w-3.5 h-3.5"></i>
            <span>CSV</span>
          </button>
          <button onclick="confirmClearAll()" class="bg-slate-100 hover:bg-red-100 text-slate-600 hover:text-red-600 px-2.5 py-1.5 rounded-lg text-xs transition">
            <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
          </button>
        </div>
      </div>

      <!-- Records List Container -->
      <div id="records-container" class="space-y-2.5">
        <!-- Dynamic JS Insertion -->
      </div>

    </div>

  </main>

  <!-- MODAL: REGISTRAR RESULTADO DE ESCANEO -->
  <div id="modal-result" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-end sm:items-center justify-center p-0 sm:p-4 transition-opacity">
    <div class="bg-white w-full max-w-md rounded-t-2xl sm:rounded-2xl p-5 space-y-4 shadow-2xl transform transition-all">
      
      <div class="flex justify-between items-center border-b border-slate-100 pb-3">
        <div class="flex items-center space-x-2">
          <div class="p-2 bg-blue-100 text-blue-600 rounded-lg">
            <i data-lucide="barcode" class="w-5 h-5"></i>
          </div>
          <div>
            <span class="text-xs text-slate-400 uppercase font-semibold tracking-wider">Código Escaneado</span>
            <h3 id="scanned-code-display" class="font-mono text-base font-bold text-slate-900 break-all">---</h3>
          </div>
        </div>
        <button onclick="closeResultModal()" class="text-slate-400 hover:text-slate-600 p-1">
          <i data-lucide="x" class="w-5 h-5"></i>
        </button>
      </div>

      <!-- Status Selection Buttons -->
      <div>
        <label class="block text-xs font-semibold text-slate-600 mb-2">Estado de la Entrega</label>
        <div class="grid grid-cols-2 gap-2">
          <button type="button" id="btn-status-delivered" onclick="selectStatus('ENTREGADO')" class="py-3 px-3 rounded-xl border-2 font-semibold text-sm flex items-center justify-center space-x-2 transition border-emerald-500 bg-emerald-50 text-emerald-700">
            <i data-lucide="check-circle" class="w-4 h-4"></i>
            <span>Entregado</span>
          </button>
          <button type="button" id="btn-status-failed" onclick="selectStatus('NO_ENTREGADO')" class="py-3 px-3 rounded-xl border-2 font-semibold text-sm flex items-center justify-center space-x-2 transition border-slate-200 bg-slate-50 text-slate-600 hover:border-red-300">
            <i data-lucide="x-circle" class="w-4 h-4"></i>
            <span>No Entregado</span>
          </button>
        </div>
      </div>

      <!-- Incidencia Reason Dropdown (Hidden when status is ENTREGADO) -->
      <div id="incidencia-group" class="hidden space-y-1.5">
        <label for="reason-select" class="block text-xs font-semibold text-red-600">Motivo de no entrega *</label>
        <select id="reason-select" class="w-full bg-slate-50 border border-slate-300 rounded-xl px-3 py-2.5 text-sm text-slate-800 focus:ring-2 focus:ring-red-500 focus:border-red-500 font-medium">
          <option value="Cliente ausente / Domicilio cerrado">Cliente ausente / Domicilio cerrado</option>
          <option value="Domicilio no localizado / Dirección errónea">Domicilio no localizado / Dirección errónea</option>
          <option value="Paquete dañado / Mal estado">Paquete dañado / Mal estado</option>
          <option value="Rechazado por destinatario (No solicitado)">Rechazado por destinatario (No solicitado)</option>
          <option value="Cobro / Pago rechazado">Cobro / Pago rechazado (Contraentrega)</option>
          <option value="Fuera de horario de atención">Fuera de horario de atención</option>
          <option value="Zona inaccesible / Riesgo">Zona inaccesible / Riesgo</option>
          <option value="Otro motivo">Otro motivo (Especificar abajo)</option>
        </select>
      </div>

      <!-- Notes Input -->
      <div class="space-y-1">
        <label for="input-notes" class="block text-xs font-semibold text-slate-600">Notas / Observaciones (Opcional)</label>
        <textarea id="input-notes" rows="2" placeholder="Ej. Dejado con el vecino, portón blanco, etc." class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-blue-500"></textarea>
      </div>

      <!-- GPS Switch -->
      <div class="flex items-center justify-between bg-slate-50 p-2.5 rounded-xl text-xs text-slate-600">
        <div class="flex items-center space-x-2">
          <i data-lucide="map-pin" class="w-4 h-4 text-slate-500"></i>
          <span>Adjuntar ubicación GPS actual</span>
        </div>
        <input type="checkbox" id="chk-gps" checked class="w-4 h-4 text-blue-600 rounded focus:ring-blue-500">
      </div>

      <!-- Save Button -->
      <button onclick="saveScanRecord()" class="w-full py-3.5 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-xl shadow-lg flex items-center justify-center space-x-2 transition active:scale-[0.99]">
        <i data-lucide="save" class="w-5 h-5"></i>
        <span>Guardar Registro</span>
      </button>

    </div>
  </div>

  <!-- MODAL: INGRESO MANUAL -->
  <div id="modal-manual" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-white w-full max-w-sm rounded-2xl p-5 space-y-4 shadow-2xl">
      <div class="flex justify-between items-center border-b border-slate-100 pb-2">
        <h3 class="font-bold text-slate-900">Ingreso Manual</h3>
        <button onclick="toggleManualModal()" class="text-slate-400 hover:text-slate-600">
          <i data-lucide="x" class="w-5 h-5"></i>
        </button>
      </div>
      <div>
        <label for="manual-code-input" class="block text-xs font-semibold text-slate-600 mb-1">Escribe o pega el código de barras</label>
        <input type="text" id="manual-code-input" placeholder="Ej. 750123456789" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3 font-mono text-base focus:ring-2 focus:ring-blue-500">
      </div>
      <button onclick="submitManualCode()" class="w-full py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-xl shadow transition">
        Procesar Código
      </button>
    </div>
  </div>

  <!-- MODAL: CONFIGURACIÓN / WEBHOOK GOOGLE SHEETS -->
  <div id="modal-config" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-white w-full max-w-md rounded-2xl p-5 space-y-4 shadow-2xl">
      <div class="flex justify-between items-center border-b border-slate-100 pb-2">
        <h3 class="font-bold text-slate-900 flex items-center gap-2">
          <i data-lucide="settings" class="w-5 h-5 text-blue-600"></i>
          <span>Configuración Sincronización</span>
        </h3>
        <button onclick="toggleConfigModal()" class="text-slate-400 hover:text-slate-600">
          <i data-lucide="x" class="w-5 h-5"></i>
        </button>
      </div>
      
      <div class="space-y-3 text-xs text-slate-600">
        <p>Puedes conectar esta app a una hoja de Google Sheets ingresando la URL de tu <strong>Google Apps Script Web App</strong>.</p>
        
        <div>
          <label for="webhook-url-input" class="block font-semibold text-slate-700 mb-1">Webhook URL (Opcional):</label>
          <input type="url" id="webhook-url-input" placeholder="https://script.google.com/macros/s/.../exec" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 font-mono text-xs focus:ring-2 focus:ring-blue-500">
        </div>

        <div class="bg-amber-50 border border-amber-200 rounded-lg p-2.5 text-amber-800">
          <strong>Nota:</strong> Si dejas el Webhook en blanco, todos los datos se guardarán de forma local en tu teléfono y podrás descargarlos como archivo CSV en cualquier momento.
        </div>
      </div>

      <button onclick="saveConfig()" class="w-full py-2.5 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-xl transition">
        Guardar Ajustes
      </button>
    </div>
  </div>

  <!-- Footer -->
  <footer class="bg-white border-t border-slate-200 py-3 text-center text-xs text-slate-400">
    <p>LogiScan PWA • Control de Entregas Offline-Ready</p>
  </footer>

  <!-- APP LOGIC JS -->
  <script>
    // App State
    let records = JSON.parse(localStorage.getItem('logiscan_records') || '[]');
    let webhookUrl = localStorage.getItem('logiscan_webhook') || '';
    let html5Qrcode = null;
    let isScanning = false;
    let currentScannedCode = '';
    let selectedStatus = 'ENTREGADO';

    // Initialize Lucide Icons
    lucide.createIcons();

    // DOM Elements
    const scannedCodeDisplay = document.getElementById('scanned-code-display');
    const btnStatusDelivered = document.getElementById('btn-status-delivered');
    const btnStatusFailed = document.getElementById('btn-status-failed');
    const incidenciaGroup = document.getElementById('incidencia-group');
    const reasonSelect = document.getElementById('reason-select');
    const inputNotes = document.getElementById('input-notes');
    const chkGps = document.getElementById('chk-gps');
    const recordCountEl = document.getElementById('record-count');

    // On Load Initializations
    document.addEventListener('DOMContentLoaded', () => {
      updateRecordCount();
      renderRecords();
      document.getElementById('webhook-url-input').value = webhookUrl;
      registerServiceWorker();
    });

    // Tab Switching
    function switchTab(tab) {
      const scanTab = document.getElementById('tab-scan');
      const listTab = document.getElementById('tab-list');
      const scanBtn = document.getElementById('tab-scan-btn');
      const listBtn = document.getElementById('tab-list-btn');

      if (tab === 'scan') {
        scanTab.classList.remove('hidden');
        listTab.classList.add('hidden');
        scanBtn.className = "flex-1 py-2 rounded-lg bg-white shadow-sm text-blue-600 flex items-center justify-center space-x-1.5 transition";
        listBtn.className = "flex-1 py-2 rounded-lg text-slate-600 hover:text-slate-900 flex items-center justify-center space-x-1.5 transition";
      } else {
        scanTab.classList.add('hidden');
        listTab.classList.remove('hidden');
        listBtn.className = "flex-1 py-2 rounded-lg bg-white shadow-sm text-blue-600 flex items-center justify-center space-x-1.5 transition";
        scanBtn.className = "flex-1 py-2 rounded-lg text-slate-600 hover:text-slate-900 flex items-center justify-center space-x-1.5 transition";
        renderRecords();
      }
    }

    // Toggle Camera
    async function toggleCamera() {
      const cameraBtnText = document.getElementById('camera-btn-text');

      if (isScanning) {
        await stopCamera();
        cameraBtnText.innerText = 'Iniciar Escáner';
      } else {
        startCamera();
      }
    }

    function startCamera() {
      html5Qrcode = new Html5Qrcode("reader");
      const config = { 
        fps: 10, 
        qrbox: { width: 250, height: 180 },
        aspectRatio: 1.0 
      };

      html5Qrcode.start(
        { facingMode: "environment" },
        config,
        onScanSuccess,
        onScanError
      ).then(() => {
        isScanning = true;
        document.getElementById('camera-btn-text').innerText = 'Detener Escáner';
      }).catch(err => {
        alert("Error al acceder a la cámara: " + err + "\nAsegúrate de otorgar permisos de cámara.");
      });
    }

    async function stopCamera() {
      if (html5Qrcode && isScanning) {
        await html5Qrcode.stop();
        html5Qrcode.clear();
        isScanning = false;
        document.getElementById('camera-btn-text').innerText = 'Iniciar Escáner';
      }
    }

    function playBeepSound() {
      try {
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = "sine";
        osc.frequency.value = 880; // A5
        gain.gain.value = 0.1;
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.15);
      } catch (e) {
        console.log("Audio not allowed or supported");
      }
    }

    function onScanSuccess(decodedText, decodedResult) {
      playBeepSound();
      if (navigator.vibrate) navigator.vibrate(100);

      // Temporarily pause camera scan
      if (html5Qrcode && isScanning) {
        stopCamera();
      }

      openResultModal(decodedText);
    }

    function onScanError(errorMessage) {
      // Ignore framing scan errors
    }

    // Result Modal Handling
    function openResultModal(code) {
      currentScannedCode = code;
      scannedCodeDisplay.innerText = code;
      selectStatus('ENTREGADO');
      inputNotes.value = '';
      document.getElementById('modal-result').classList.remove('hidden');
    }

    function closeResultModal() {
      document.getElementById('modal-result').classList.add('hidden');
      currentScannedCode = '';
    }

    function selectStatus(status) {
      selectedStatus = status;
      if (status === 'ENTREGADO') {
        btnStatusDelivered.className = "py-3 px-3 rounded-xl border-2 font-semibold text-sm flex items-center justify-center space-x-2 transition border-emerald-500 bg-emerald-50 text-emerald-700 shadow-sm";
        btnStatusFailed.className = "py-3 px-3 rounded-xl border-2 font-semibold text-sm flex items-center justify-center space-x-2 transition border-slate-200 bg-slate-50 text-slate-600 hover:border-red-300";
        incidenciaGroup.classList.add('hidden');
      } else {
        btnStatusDelivered.className = "py-3 px-3 rounded-xl border-2 font-semibold text-sm flex items-center justify-center space-x-2 transition border-slate-200 bg-slate-50 text-slate-600 hover:border-emerald-300";
        btnStatusFailed.className = "py-3 px-3 rounded-xl border-2 font-semibold text-sm flex items-center justify-center space-x-2 transition border-red-500 bg-red-50 text-red-700 shadow-sm";
        incidenciaGroup.classList.remove('hidden');
      }
    }

    // Save Record Function
    async function saveScanRecord() {
      const now = new Date();
      const record = {
        id: 'REC-' + Date.now(),
        code: currentScannedCode,
        status: selectedStatus,
        reason: selectedStatus === 'NO_ENTREGADO' ? reasonSelect.value : 'N/A',
        notes: inputNotes.value.trim(),
        timestamp: now.toLocaleString('es-MX'),
        isoDate: now.toISOString(),
        location: 'No solicitado'
      };

      // Handle GPS Location if requested
      if (chkGps.checked && navigator.geolocation) {
        try {
          const pos = await getCurrentPositionPromise();
          record.location = `${pos.coords.latitude.toFixed(5)}, ${pos.coords.longitude.toFixed(5)}`;
        } catch (err) {
          record.location = 'No disponible / Permiso denegado';
        }
      }

      // Save locally
      records.unshift(record);
      localStorage.setItem('logiscan_records', JSON.stringify(records));

      // Sync Webhook if set
      if (webhookUrl) {
        sendToWebhook(record);
      }

      updateRecordCount();
      closeResultModal();
      alert(`✅ Registro guardado con éxito (${selectedStatus === 'ENTREGADO' ? 'Entregado' : 'Incidencia'})`);
    }

    function getCurrentPositionPromise() {
      return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(resolve, reject, { timeout: 5000 });
      });
    }

    // Webhook Sender
    function sendToWebhook(data) {
      fetch(webhookUrl, {
        method: 'POST',
        mode: 'no-cors',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      }).catch(err => console.error("Webhook sync error:", err));
    }

    // Manual Entry Modal
    function toggleManualModal() {
      const modal = document.getElementById('modal-manual');
      modal.classList.toggle('hidden');
      if (!modal.classList.contains('hidden')) {
        document.getElementById('manual-code-input').focus();
      }
    }

    function submitManualCode() {
      const codeInput = document.getElementById('manual-code-input');
      const val = codeInput.value.trim();
      if (!val) {
        alert('Ingresa un código válido');
        return;
      }
      toggleManualModal();
      codeInput.value = '';
      openResultModal(val);
    }

    // Config Modal
    function toggleConfigModal() {
      document.getElementById('modal-config').classList.toggle('hidden');
    }

    function saveConfig() {
      const url = document.getElementById('webhook-url-input').value.trim();
      webhookUrl = url;
      localStorage.setItem('logiscan_webhook', url);
      toggleConfigModal();
      alert('Configuración guardada.');
    }

    // Update Counts & Render Records
    function updateRecordCount() {
      recordCountEl.innerText = records.length;
    }

    function renderRecords() {
      const container = document.getElementById('records-container');
      const filter = document.getElementById('filter-status').value;

      let filtered = records;
      if (filter !== 'ALL') {
        filtered = records.filter(r => r.status === filter);
      }

      if (filtered.length === 0) {
        container.innerHTML = `
          <div class="bg-white rounded-xl p-8 text-center text-slate-400 border border-slate-200">
            <i data-lucide="inbox" class="w-10 h-10 mx-auto mb-2 text-slate-300"></i>
            <p class="text-sm font-medium">No hay registros almacenados.</p>
          </div>
        `;
        lucide.createIcons();
        return;
      }

      let html = '';
      filtered.forEach((r) => {
        const isSuccess = r.status === 'ENTREGADO';
        const badgeClass = isSuccess ? 'badge-success' : 'badge-danger';
        const statusText = isSuccess ? 'Entregado' : 'No Entregado';
        const iconName = isSuccess ? 'check-circle' : 'alert-circle';

        html += `
          <div class="bg-white border border-slate-200 rounded-xl p-3.5 shadow-sm space-y-2">
            <div class="flex items-start justify-between">
              <div>
                <span class="font-mono text-xs text-slate-400 block">${r.timestamp}</span>
                <p class="font-mono text-base font-bold text-slate-800 break-all">${r.code}</p>
              </div>
              <span class="px-2.5 py-1 rounded-full text-xs font-semibold flex items-center space-x-1 ${badgeClass}">
                <i data-lucide="${iconName}" class="w-3.5 h-3.5"></i>
                <span>${statusText}</span>
              </span>
            </div>

            ${!isSuccess ? `
              <div class="bg-red-50 border border-red-100 rounded-lg p-2 text-xs text-red-800">
                <strong>Motivo:</strong> ${r.reason}
              </div>
            ` : ''}

            ${r.notes ? `
              <p class="text-xs text-slate-600 bg-slate-50 p-2 rounded-lg border border-slate-100">
                <span class="font-semibold">Nota:</span> ${r.notes}
              </p>
            ` : ''}

            <div class="flex justify-between items-center text-[11px] text-slate-400 pt-1 border-t border-slate-100">
              <span class="flex items-center space-x-1">
                <i data-lucide="map-pin" class="w-3 h-3"></i>
                <span>${r.location}</span>
              </span>
              <button onclick="deleteRecord('${r.id}')" class="text-slate-400 hover:text-red-600">
                <i data-lucide="trash" class="w-3.5 h-3.5"></i>
              </button>
            </div>
          </div>
        `;
      });

      container.innerHTML = html;
      lucide.createIcons();
    }

    function deleteRecord(id) {
      if (confirm('¿Deseas eliminar este registro?')) {
        records = records.filter(r => r.id !== id);
        localStorage.setItem('logiscan_records', JSON.stringify(records));
        updateRecordCount();
        renderRecords();
      }
    }

    function confirmClearAll() {
      if (records.length === 0) return;
      if (confirm('¿Estás seguro de borrar TODOS los registros de la app?')) {
        records = [];
        localStorage.setItem('logiscan_records', JSON.stringify(records));
        updateRecordCount();
        renderRecords();
      }
    }

    // CSV Export
    function exportToCSV() {
      if (records.length === 0) {
        alert('No hay datos para exportar.');
        return;
      }

      let csv = '\uFEFF'; // UTF-8 BOM
      csv += 'ID,Codigo,Estado,Motivo_Incidencia,Notas,Fecha_Hora,Ubicacion_GPS\n';

      records.forEach(r => {
        const cleanCode = `"${r.code.replace(/"/g, '""')}"`;
        const cleanStatus = `"${r.status}"`;
        const cleanReason = `"${r.reason.replace(/"/g, '""')}"`;
        const cleanNotes = `"${(r.notes || '').replace(/"/g, '""')}"`;
        const cleanTime = `"${r.timestamp}"`;
        const cleanLoc = `"${r.location}"`;

        csv += `${r.id},${cleanCode},${cleanStatus},${cleanReason},${cleanNotes},${cleanTime},${cleanLoc}\n`;
      });

      const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.setAttribute('href', url);
      link.setAttribute('download', `Reporte_Entregas_${new Date().toISOString().slice(0,10)}.csv`);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    // Inline PWA Dynamic Manifest & ServiceWorker
    function registerServiceWorker() {
      const manifest = {
        name: "LogiScan PWA - Control de Entregas",
        short_name: "LogiScan",
        start_url: ".",
        display: "standalone",
        background_color: "#2563eb",
        theme_color: "#2563eb",
        icons: [{
          src: "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%232563eb' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'><rect x='2' y='4' width='20' height='16' rx='2'/><path d='M7 8v8M11 8v8M15 8v8M18 8v8'/></svg>",
          sizes: "192x192",
          type: "image/svg+xml"
        }]
      };
      const manifestStr = JSON.stringify(manifest);
      const blob = new Blob([manifestStr], {type: 'application/json'});
      const manifestUrl = URL.createObjectURL(blob);
      const link = document.createElement('link');
      link.rel = 'manifest';
      link.href = manifestUrl;
      document.head.appendChild(link);
    }
  </script>
</body>
</html>
