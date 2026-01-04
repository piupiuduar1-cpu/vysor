// ============================================
// VYSOR WEB DANA AUTOMATOR PRO - DENGAN FITUR DELETE LANGKAH DAN RESIZABLE UI
// ============================================

(function() {
  'use strict';
  
  // Cek apakah sudah ada UI
  if (window.vysorAutomatorUI) {
    console.log("UI sudah ada. Reload halaman untuk membuat baru.");
    return;
  }
  
  // State untuk rekaman
  const recordingState = {
    isRecording: false,
    recordingSteps: [],
    currentStep: null,
    startTime: null,
    playbackSpeed: 1.0,
    selectedStepIndex: -1
  };
  
  // Ukuran default
  const defaultSize = {
    width: 400,
    height: 600,
    minWidth: 300,
    minHeight: 400,
    maxWidth: 800,
    maxHeight: 900
  };
  
  // Muat ukuran dari localStorage jika ada
  let currentSize = {
    width: parseInt(localStorage.getItem('vysorUISizeWidth')) || defaultSize.width,
    height: parseInt(localStorage.getItem('vysorUISizeHeight')) || defaultSize.height
  };
  
  // Pastikan ukuran dalam batas yang wajar
  currentSize.width = Math.max(defaultSize.minWidth, Math.min(defaultSize.maxWidth, currentSize.width));
  currentSize.height = Math.max(defaultSize.minHeight, Math.min(defaultSize.maxHeight, currentSize.height));
  
  // Buat UI container
  const container = document.createElement('div');
  container.id = 'vysor-automator-ui';
  container.style.cssText = `
    position: fixed;
    top: 50px;
    right: 20px;
    width: ${currentSize.width}px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 15px;
    padding: 20px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);
    z-index: 999999;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    color: white;
    border: 2px solid #fff;
    max-height: 85vh;
    overflow-y: auto;
    resize: both;
    overflow: hidden;
  `;
  
  // Header
  const header = document.createElement('div');
  header.innerHTML = `
    <h2 style="margin: 0 0 10px 0; display: flex; align-items: center; gap: 10px;">
      <span style="background: white; color: #667eea; padding: 5px 10px; border-radius: 20px;">📱</span>
      DANA Automator Pro
      <span id="sizeIndicator" style="font-size: 10px; background: rgba(255,255,255,0.3); padding: 2px 6px; border-radius: 10px; margin-left: auto;">
        ${currentSize.width}×${currentSize.height}
      </span>
    </h2>
    <p style="margin: 0 0 15px 0; font-size: 12px; opacity: 0.8;">
      Untuk Vysor Web: ${window.location.hostname}
    </p>
  `;
  
  // Tab System
  const tabContainer = document.createElement('div');
  tabContainer.style.cssText = `
    display: flex;
    gap: 5px;
    margin-bottom: 15px;
    border-bottom: 1px solid rgba(255,255,255,0.2);
    padding-bottom: 10px;
    flex-wrap: wrap;
  `;
  
  const tabs = ['Kirim', 'Rekaman', 'Pengaturan', 'Ukuran'];
  const tabContents = {};
  
  tabs.forEach(tab => {
    const tabButton = document.createElement('button');
    tabButton.textContent = tab;
    tabButton.className = 'tab-btn';
    tabButton.style.cssText = `
      padding: 8px 15px;
      background: transparent;
      border: none;
      color: white;
      cursor: pointer;
      border-radius: 5px;
      font-size: 14px;
      transition: all 0.3s;
      white-space: nowrap;
    `;
    
    tabButton.addEventListener('click', () => {
      // Update active tab
      document.querySelectorAll('.tab-btn').forEach(btn => {
        btn.style.background = 'transparent';
        btn.style.fontWeight = 'normal';
      });
      tabButton.style.background = 'rgba(255,255,255,0.2)';
      tabButton.style.fontWeight = 'bold';
      
      // Show content
      Object.values(tabContents).forEach(content => {
        content.style.display = 'none';
      });
      if (tabContents[tab]) {
        tabContents[tab].style.display = 'block';
      }
    });
    
    tabContainer.appendChild(tabButton);
    
    // Create content for each tab
    const content = document.createElement('div');
    content.id = `tab-${tab.toLowerCase()}`;
    content.style.display = tab === 'Kirim' ? 'block' : 'none';
    tabContents[tab] = content;
  });
  
  // Tab 1: Kirim (Ditingkatkan)
  tabContents['Kirim'].innerHTML = `
    <div style="margin-bottom: 20px;">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
        <label style="font-size: 14px; font-weight: bold;">💰 Pengiriman Dana</label>
        <span id="kirimStatus" style="font-size: 11px; background: rgba(255,255,255,0.2); padding: 2px 8px; border-radius: 10px;">Siap</span>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 10px; margin-bottom: 15px;">
        <div style="margin-bottom: 15px;">
          <label style="display: block; margin-bottom: 5px; font-size: 14px;">
            📝 Tujuan (Nama/Nomor/Email)
          </label>
          <input type="text" id="tujuan" 
                 placeholder="Contoh: 08123456789" 
                 style="width: 100%; padding: 10px; border-radius: 8px; border: none; font-size: 14px; color: #333;">
        </div>
        
        <div style="margin-bottom: 15px;">
          <label style="display: block; margin-bottom: 5px; font-size: 14px;">
            💰 Nominal (Rp)
          </label>
          <input type="number" id="nominal" 
                 placeholder="Contoh: 100000" 
                 style="width: 100%; padding: 10px; border-radius: 8px; border: none; font-size: 16px; color: #333;">
        </div>
        
        <div style="margin-bottom: 15px;">
          <label style="display: block; margin-bottom: 5px; font-size: 14px;">
            🔒 PIN (6 digit)
          </label>
          <input type="password" id="pin" 
                 placeholder="••••••" 
                 maxlength="6"
                 style="width: 100%; padding: 10px; border-radius: 8px; border: none; font-size: 16px; color: #333; letter-spacing: 3px;">
        </div>
        
        <div style="display: flex; align-items: center; gap: 10px; margin-bottom: 15px;">
          <input type="checkbox" id="skipPin" style="transform: scale(1.2);">
          <label for="skipPin" style="font-size: 12px;">Lewati input PIN (jika tidak diperlukan)</label>
        </div>
        
        <div style="display: flex; align-items: center; gap: 10px; margin-bottom: 15px;">
          <input type="checkbox" id="autoConfirm" style="transform: scale(1.2);" checked>
          <label for="autoConfirm" style="font-size: 12px;">Otomatis konfirmasi setelah PIN</label>
        </div>
      </div>
      
      <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px;">
        <button id="btnFindKirim" class="vysor-btn" style="background: #4CAF50;">
          🔍 Cari Kirim
        </button>
        <button id="btnFindTujuan" class="vysor-btn" style="background: #2196F3;">
          👤 Cari Tujuan
        </button>
      </div>
      
      <div style="margin-bottom: 15px;">
        <button id="btnAutoKirimLengkap" class="vysor-btn" style="background: linear-gradient(45deg, #FF416C, #FF4B2B); width: 100%; padding: 15px; font-size: 16px;">
          🚀 Kirim Otomatis (Lengkap)
        </button>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px; margin-bottom: 15px;">
        <label style="display: block; margin-bottom: 5px; font-size: 12px;">Koordinat Manual</label>
        <div style="display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 5px;">
          <input type="number" id="coordX" placeholder="X" style="padding: 8px; border-radius: 5px; border: none;">
          <input type="number" id="coordY" placeholder="Y" style="padding: 8px; border-radius: 5px; border: none;">
          <button id="btnManualClick" class="vysor-btn" style="background: #9C27B0;">
            📍 Klik
          </button>
        </div>
      </div>
      
      <button id="btnCoordFinder" class="vysor-btn" style="background: #607D8B; width: 100%; margin-top: 10px;">
        🎯 Mode Coordinate Finder
      </button>
    </div>
    
    <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px;">
      <label style="display: block; margin-bottom: 5px; font-size: 12px; font-weight: bold;">📋 Langkah Otomatis:</label>
      <div style="font-size: 11px; line-height: 1.4;">
        <div>1. 🔍 Cari tombol "Kirim"</div>
        <div>2. 👤 Cari dan pilih tujuan</div>
        <div>3. 💰 Isi nominal</div>
        <div>4. ➡️ Klik "Lanjut"</div>
        <div>5. 💳 Klik "Bayar"</div>
        <div>6. 🔒 Isi PIN (jika perlu)</div>
        <div>7. ✅ Klik "Konfirmasi"</div>
      </div>
    </div>
  `;
  
  // Tab 2: Rekaman (dengan fitur delete)
  tabContents['Rekaman'].innerHTML = `
    <div style="margin-bottom: 15px;">
      <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px;">
        <button id="btnStartRecording" class="vysor-btn" style="background: #F44336;">
          🔴 Mulai Rekaman
        </button>
        <button id="btnStopRecording" class="vysor-btn" style="background: #FF9800;" disabled>
          ⏹️ Stop Rekaman
        </button>
      </div>
      
      <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px;">
        <button id="btnPlayRecording" class="vysor-btn" style="background: #4CAF50;">
          ▶️ Mainkan
        </button>
        <button id="btnSaveRecording" class="vysor-btn" style="background: #2196F3;">
          💾 Simpan
        </button>
      </div>
      
      <div style="margin-bottom: 15px;">
        <label style="display: block; margin-bottom: 5px; font-size: 12px;">Kecepatan Pemutaran</label>
        <div style="display: flex; align-items: center; gap: 10px;">
          <span style="font-size: 12px;">0.5x</span>
          <input type="range" id="playbackSpeed" min="0.5" max="3" step="0.1" value="1.0" 
                 style="flex: 1;">
          <span style="font-size: 12px;">3x</span>
          <span id="speedValue" style="font-size: 12px; min-width: 30px;">1.0x</span>
        </div>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px; margin-bottom: 10px;">
        <label style="display: block; margin-bottom: 5px; font-size: 12px;">Rekaman Tersimpan</label>
        <select id="savedRecordings" style="width: 100%; padding: 8px; border-radius: 5px; border: none; margin-bottom: 10px; color: #333;">
          <option value="">Pilih rekaman...</option>
        </select>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 5px;">
          <button id="btnLoadRecording" class="vysor-btn" style="background: #9C27B0;">
            📂 Muat
          </button>
          <button id="btnDeleteRecording" class="vysor-btn" style="background: #f44336;">
            🗑️ Hapus
          </button>
        </div>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px; margin-bottom: 10px;">
        <label style="display: block; margin-bottom: 5px; font-size: 12px;">Manajemen Langkah</label>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 5px;">
          <button id="btnAddDelay" class="vysor-btn" style="background: #9C27B0;">
            ⏱️ Tambah Delay
          </button>
          <button id="btnDeleteSelectedStep" class="vysor-btn" style="background: #f44336;">
            🗑️ Hapus Terpilih
          </button>
        </div>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 5px; margin-top: 5px;">
          <button id="btnClearAllSteps" class="vysor-btn" style="background: #795548;">
            🧹 Hapus Semua
          </button>
          <button id="btnEditStep" class="vysor-btn" style="background: #FF9800;">
            ✏️ Edit Terpilih
          </button>
        </div>
      </div>
    </div>
    
    <div style="background: rgba(0,0,0,0.2); padding: 10px; border-radius: 8px; max-height: 200px; overflow-y: auto;">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
        <label style="font-size: 12px; font-weight: bold;">📝 Langkah Rekaman</label>
        <div style="display: flex; align-items: center; gap: 5px;">
          <span id="stepCount" style="font-size: 11px; background: rgba(255,255,255,0.2); padding: 2px 8px; border-radius: 10px;">0 langkah</span>
          <span id="selectedStepInfo" style="font-size: 10px; color: #4CAF50; display: none;">Terpilih: -</span>
        </div>
      </div>
      <div id="recordingStepsList" style="font-size: 11px;">
        <div style="color: rgba(255,255,255,0.6); text-align: center; padding: 10px;">
          Belum ada langkah
        </div>
      </div>
    </div>
  `;
  
  // Tab 3: Pengaturan
  tabContents['Pengaturan'].innerHTML = `
    <div style="margin-bottom: 15px;">
      <label style="display: block; margin-bottom: 5px; font-size: 14px;">
        ⏱️ Delay Antara Langkah (ms)
      </label>
      <input type="number" id="defaultDelay" value="1500" min="500" max="5000"
             style="width: 100%; padding: 10px; border-radius: 8px; border: none; font-size: 14px; color: #333;">
    </div>
    
    <div style="margin-bottom: 15px;">
      <label style="display: block; margin-bottom: 5px; font-size: 14px;">
        🎯 Tolerance Pencarian
      </label>
      <div style="display: flex; align-items: center; gap: 10px;">
        <span style="font-size: 12px;">Low</span>
        <input type="range" id="searchTolerance" min="1" max="10" value="5" 
               style="flex: 1;">
        <span style="font-size: 12px;">High</span>
        <span id="toleranceValue" style="font-size: 12px; min-width: 30px;">5</span>
      </div>
    </div>
    
    <div style="margin-bottom: 15px;">
      <label style="display: block; margin-bottom: 5px; font-size: 14px;">
        📊 Mode Debug
      </label>
      <div style="display: flex; align-items: center; gap: 10px;">
        <input type="checkbox" id="debugMode" style="transform: scale(1.2);">
        <label for="debugMode" style="font-size: 12px;">Tampilkan detail log</label>
      </div>
    </div>
    
    <div style="margin-bottom: 15px;">
      <button id="btnExportConfig" class="vysor-btn" style="background: #2196F3; width: 100%;">
        📤 Export Konfigurasi
      </button>
      <button id="btnImportConfig" class="vysor-btn" style="background: #4CAF50; width: 100%; margin-top: 10px;">
        📥 Import Konfigurasi
      </button>
    </div>
    
    <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px;">
      <label style="display: block; margin-bottom: 5px; font-size: 12px;">Informasi Sistem</label>
      <div style="font-size: 11px; line-height: 1.4;">
        <div>Total Rekaman: <span id="totalRecordings">0</span></div>
        <div>Langkah Tersimpan: <span id="totalSteps">0</span></div>
        <div>Versi: 2.2.0</div>
        <div>Status: <span id="systemStatus" style="color: #4CAF50;">Aktif</span></div>
      </div>
    </div>
  `;
  
  // Tab 4: Ukuran (Baru!)
  tabContents['Ukuran'].innerHTML = `
    <div style="margin-bottom: 20px;">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
        <label style="font-size: 14px; font-weight: bold;">📐 Pengaturan Ukuran UI</label>
        <span id="sizeDisplay" style="font-size: 11px; background: rgba(255,255,255,0.2); padding: 2px 8px; border-radius: 10px;">
          ${currentSize.width} × ${currentSize.height}
        </span>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 15px; border-radius: 10px; margin-bottom: 15px;">
        <div style="margin-bottom: 15px;">
          <label style="display: block; margin-bottom: 5px; font-size: 14px;">
            Lebar (Width) in pixels
          </label>
          <div style="display: flex; gap: 10px; align-items: center;">
            <input type="range" id="widthSlider" min="${defaultSize.minWidth}" max="${defaultSize.maxWidth}" 
                   value="${currentSize.width}" step="10" style="flex: 1;">
            <input type="number" id="widthInput" value="${currentSize.width}" 
                   min="${defaultSize.minWidth}" max="${defaultSize.maxWidth}"
                   style="width: 70px; padding: 8px; border-radius: 5px; border: none; color: #333;">
            <span style="font-size: 12px;">px</span>
          </div>
        </div>
        
        <div style="margin-bottom: 15px;">
          <label style="display: block; margin-bottom: 5px; font-size: 14px;">
            Tinggi (Height) in pixels
          </label>
          <div style="display: flex; gap: 10px; align-items: center;">
            <input type="range" id="heightSlider" min="${defaultSize.minHeight}" max="${defaultSize.maxHeight}" 
                   value="${currentSize.height}" step="10" style="flex: 1;">
            <input type="number" id="heightInput" value="${currentSize.height}" 
                   min="${defaultSize.minHeight}" max="${defaultSize.maxHeight}"
                   style="width: 70px; padding: 8px; border-radius: 5px; border: none; color: #333;">
            <span style="font-size: 12px;">px</span>
          </div>
        </div>
        
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px;">
          <button id="btnApplySize" class="vysor-btn" style="background: #4CAF50;">
            💾 Terapkan
          </button>
          <button id="btnResetSize" class="vysor-btn" style="background: #FF9800;">
            🔄 Reset
          </button>
        </div>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px; margin-bottom: 15px;">
        <label style="display: block; margin-bottom: 5px; font-size: 12px;">Ukuran Preset</label>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 5px;">
          <button class="size-preset" data-width="350" data-height="500" style="background: #2196F3; padding: 8px; border: none; border-radius: 5px; color: white; cursor: pointer;">
            Kecil (350×500)
          </button>
          <button class="size-preset" data-width="400" data-height="600" style="background: #2196F3; padding: 8px; border: none; border-radius: 5px; color: white; cursor: pointer;">
            Default (400×600)
          </button>
          <button class="size-preset" data-width="500" data-height="700" style="background: #2196F3; padding: 8px; border: none; border-radius: 5px; color: white; cursor: pointer;">
            Sedang (500×700)
          </button>
          <button class="size-preset" data-width="600" data-height="800" style="background: #2196F3; padding: 8px; border: none; border-radius: 5px; color: white; cursor: pointer;">
            Besar (600×800)
          </button>
        </div>
      </div>
      
      <div style="background: rgba(255,255,255,0.1); padding: 10px; border-radius: 8px;">
        <label style="display: block; margin-bottom: 5px; font-size: 12px;">Mode Resize</label>
        <div style="font-size: 11px; line-height: 1.4;">
          <div>1. 🔄 Drag sudut kanan bawah untuk resize manual</div>
          <div>2. 📐 Atur ukuran dengan slider di atas</div>
          <div>3. 📱 Gunakan preset untuk ukuran standar</div>
          <div>4. 💾 Ukuran otomatis disimpan di browser</div>
        </div>
      </div>
    </div>
  `;
  
  // Status log
  const status = document.createElement('div');
  status.id = 'status-log';
  status.style.cssText = `
    background: rgba(0,0,0,0.2);
    border-radius: 8px;
    padding: 10px;
    max-height: 150px;
    overflow-y: auto;
    font-size: 12px;
    margin-top: 15px;
  `;
  status.innerHTML = `<div style="color: #4CAF50;">✅ Ready. Pilih aksi di atas.</div>`;
  
  // Recording indicator
  const recordingIndicator = document.createElement('div');
  recordingIndicator.id = 'recording-indicator';
  recordingIndicator.style.cssText = `
    position: absolute;
    top: 10px;
    left: 10px;
    background: #F44336;
    color: white;
    padding: 3px 8px;
    border-radius: 10px;
    font-size: 10px;
    display: none;
    animation: pulse 1s infinite;
  `;
  
  // Progress indicator
  const progressBar = document.createElement('div');
  progressBar.id = 'progress-bar';
  progressBar.style.cssText = `
    position: fixed;
    top: 0;
    left: 0;
    width: 0%;
    height: 3px;
    background: linear-gradient(90deg, #4CAF50, #2196F3);
    z-index: 9999999;
    transition: width 0.3s;
    display: none;
  `;
  
  // Resize handle (sudut kanan bawah)
  const resizeHandle = document.createElement('div');
  resizeHandle.id = 'resize-handle';
  resizeHandle.style.cssText = `
    position: absolute;
    bottom: 5px;
    right: 5px;
    width: 15px;
    height: 15px;
    cursor: nwse-resize;
    opacity: 0.7;
  `;
  resizeHandle.innerHTML = `
    <svg width="15" height="15" viewBox="0 0 15 15" fill="white">
      <path d="M14 14L5 5M14 14H9M14 14V9M5 5L1 1M5 5V10M5 5H10" stroke="white" stroke-width="2" stroke-linecap="round"/>
    </svg>
  `;
  
  // Minimize button
  const minimizeBtn = document.createElement('div');
  minimizeBtn.innerHTML = '−';
  minimizeBtn.style.cssText = `
    position: absolute;
    top: 10px;
    right: 40px;
    font-size: 24px;
    cursor: pointer;
    color: white;
    width: 20px;
    height: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 3px;
  `;
  minimizeBtn.title = 'Minimize';
  
  // Close button
  const closeBtn = document.createElement('div');
  closeBtn.innerHTML = '×';
  closeBtn.style.cssText = `
    position: absolute;
    top: 10px;
    right: 15px;
    font-size: 24px;
    cursor: pointer;
    color: white;
    width: 20px;
    height: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 3px;
  `;
  closeBtn.title = 'Close';
  
  // CSS untuk tombol dan animasi
  const style = document.createElement('style');
  style.textContent = `
    .vysor-btn {
      padding: 12px;
      border: none;
      border-radius: 8px;
      color: white;
      font-weight: bold;
      cursor: pointer;
      transition: all 0.3s;
      font-size: 14px;
    }
    .vysor-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(0,0,0,0.2);
    }
    .vysor-btn:active {
      transform: translateY(0);
    }
    .vysor-btn:disabled {
      opacity: 0.5;
      cursor: not-allowed;
      transform: none !important;
    }
    .log-item {
      padding: 3px 0;
      border-bottom: 1px solid rgba(255,255,255,0.1);
    }
    @keyframes pulse {
      0% { opacity: 1; }
      50% { opacity: 0.5; }
      100% { opacity: 1; }
    }
    .step-item {
      padding: 5px;
      margin-bottom: 3px;
      background: rgba(255,255,255,0.05);
      border-radius: 4px;
      border-left: 3px solid #4CAF50;
      cursor: pointer;
      transition: all 0.2s;
      position: relative;
    }
    .step-item:hover {
      background: rgba(255,255,255,0.1);
    }
    .step-item.selected {
      background: rgba(33, 150, 243, 0.2);
      border-left: 3px solid #2196F3;
    }
    .step-type-click { border-left-color: #2196F3; }
    .step-type-input { border-left-color: #FF9800; }
    .step-type-delay { border-left-color: #9C27B0; }
    .step-actions {
      position: absolute;
      right: 5px;
      top: 5px;
      display: none;
    }
    .step-item:hover .step-actions {
      display: block;
    }
    .step-item.selected .step-actions {
      display: block;
    }
    .step-action-btn {
      background: rgba(0,0,0,0.3);
      border: none;
      color: white;
      border-radius: 3px;
      padding: 2px 5px;
      margin-left: 2px;
      cursor: pointer;
      font-size: 10px;
    }
    .step-action-btn:hover {
      background: rgba(0,0,0,0.5);
    }
    .progress-step {
      display: inline-block;
      width: 20px;
      height: 20px;
      border-radius: 50%;
      background: rgba(255,255,255,0.2);
      margin: 0 2px;
      text-align: center;
      line-height: 20px;
      font-size: 10px;
    }
    .progress-step.active {
      background: #4CAF50;
    }
    .progress-step.completed {
      background: #2196F3;
    }
    #resize-handle:hover {
      opacity: 1;
    }
    .size-preset:hover {
      transform: translateY(-2px);
      box-shadow: 0 3px 10px rgba(0,0,0,0.2);
    }
    .minimized {
      height: 40px !important;
      overflow: hidden !important;
      padding: 10px 20px !important;
    }
    .minimized > *:not(.header-controls) {
      display: none !important;
    }
    .header-controls {
      display: flex;
      align-items: center;
      gap: 10px;
    }
  `;
  
  // Gabungkan semua elemen
  document.body.appendChild(progressBar);
  container.appendChild(resizeHandle);
  container.appendChild(closeBtn);
  container.appendChild(minimizeBtn);
  container.appendChild(recordingIndicator);
  container.appendChild(header);
  container.appendChild(tabContainer);
  Object.values(tabContents).forEach(content => container.appendChild(content));
  container.appendChild(status);
  document.body.appendChild(container);
  document.head.appendChild(style);
  
  // ============================================
  // FUNGSI UTAMA
  // ============================================
  
  let lastFoundElements = [];
  let currentCoordinates = null;
  let clickListeners = [];
  let isProcessing = false;
  let currentStep = 0;
  let isMinimized = false;
  
  // Fungsi untuk menambahkan log
  function addLog(message, type = 'info') {
    const colors = {
      info: '#4CAF50',
      warning: '#FF9800',
      error: '#F44336',
      success: '#4CAF50',
      step: '#2196F3'
    };
    
    const logItem = document.createElement('div');
    logItem.className = 'log-item';
    logItem.innerHTML = `<span style="color: ${colors[type] || '#4CAF50'}">${new Date().toLocaleTimeString()}: ${message}</span>`;
    status.appendChild(logItem);
    status.scrollTop = status.scrollHeight;
    console.log(`[Vysor] ${message}`);
  }
  
  // Fungsi untuk update status kirim
  function updateKirimStatus(statusText, color = '#4CAF50') {
    const statusEl = document.getElementById('kirimStatus');
    if (statusEl) {
      statusEl.textContent = statusText;
      statusEl.style.background = color;
    }
  }
  
  // Fungsi untuk update progress bar
  function updateProgressBar(progress, show = true) {
    const progressEl = document.getElementById('progress-bar');
    if (progressEl) {
      progressEl.style.display = show ? 'block' : 'none';
      progressEl.style.width = `${progress}%`;
    }
  }
  
  // Fungsi untuk update ukuran UI
  function updateUISize(width, height, saveToStorage = true) {
    // Batasi ukuran
    width = Math.max(defaultSize.minWidth, Math.min(defaultSize.maxWidth, width));
    height = Math.max(defaultSize.minHeight, Math.min(defaultSize.maxHeight, height));
    
    // Terapkan ukuran
    container.style.width = `${width}px`;
    
    // Jika tidak minimized, atur tinggi
    if (!isMinimized) {
      container.style.height = `${height}px`;
    }
    
    // Update state
    currentSize.width = width;
    currentSize.height = height;
    
    // Update display
    const sizeIndicator = document.getElementById('sizeIndicator');
    const sizeDisplay = document.getElementById('sizeDisplay');
    
    if (sizeIndicator) {
      sizeIndicator.textContent = `${width}×${height}`;
    }
    
    if (sizeDisplay) {
      sizeDisplay.textContent = `${width} × ${height}`;
    }
    
    // Update input dan slider
    const widthInput = document.getElementById('widthInput');
    const widthSlider = document.getElementById('widthSlider');
    const heightInput = document.getElementById('heightInput');
    const heightSlider = document.getElementById('heightSlider');
    
    if (widthInput) widthInput.value = width;
    if (widthSlider) widthSlider.value = width;
    if (heightInput) heightInput.value = height;
    if (heightSlider) heightSlider.value = height;
    
    // Simpan ke localStorage jika diminta
    if (saveToStorage) {
      localStorage.setItem('vysorUISizeWidth', width);
      localStorage.setItem('vysorUISizeHeight', height);
    }
    
    addLog(`📐 Ukuran UI diubah: ${width}×${height}px`, 'info');
  }
  
  // Fungsi untuk toggle minimize/maximize
  function toggleMinimize() {
    isMinimized = !isMinimized;
    
    if (isMinimized) {
      container.classList.add('minimized');
      minimizeBtn.innerHTML = '+';
      minimizeBtn.title = 'Maximize';
      addLog('📦 UI diminimalkan', 'info');
    } else {
      container.classList.remove('minimized');
      minimizeBtn.innerHTML = '−';
      minimizeBtn.title = 'Minimize';
      // Kembalikan tinggi asli
      container.style.height = `${currentSize.height}px`;
      addLog('📦 UI dimaksimalkan', 'info');
    }
  }
  
  // ============================================
  // FUNGSI PENCARIAN ELEMEN UNTUK ALUR KIRIM
  // ============================================
  
  // Fungsi untuk menemukan elemen berdasarkan teks (fuzzy search)
  function findElementByText(text, partialMatch = true, tolerance = 5) {
    addLog(`🔍 Mencari elemen dengan teks: "${text}"`, 'info');
    
    const allElements = document.querySelectorAll('*');
    const possibleElements = [];
    
    allElements.forEach(el => {
      const elementText = (el.textContent || el.innerText || '').trim();
      if (!elementText) return;
      
      // Check for exact match
      if (elementText === text) {
        possibleElements.push(el);
        return;
      }
      
      // Check for partial match
      if (partialMatch) {
        if (elementText.includes(text) || text.includes(elementText)) {
          possibleElements.push(el);
          return;
        }
        
        // Check for case-insensitive match
        if (elementText.toLowerCase().includes(text.toLowerCase()) || 
            text.toLowerCase().includes(elementText.toLowerCase())) {
          possibleElements.push(el);
          return;
        }
      }
    });
    
    // Juga cari dengan XPath untuk lebih komprehensif
    const xpath = `//*[contains(text(), '${text}')]`;
    const result = document.evaluate(xpath, document, null, XPathResult.ORDERED_NODE_SNAPSHOT_TYPE, null);
    
    for (let i = 0; i < result.snapshotLength; i++) {
      const el = result.snapshotItem(i);
      if (!possibleElements.includes(el) && el.offsetParent !== null) {
        possibleElements.push(el);
      }
    }
    
    // Filter elemen yang terlihat
    const visibleElements = possibleElements.filter(el => {
      return el.offsetParent !== null && 
             el.getBoundingClientRect().width > 0 && 
             el.getBoundingClientRect().height > 0;
    });
    
    if (visibleElements.length > 0) {
      addLog(`✅ Ditemukan ${visibleElements.length} elemen dengan teks "${text}"`, 'success');
      
      // Highlight elemen yang ditemukan
      visibleElements.forEach((el, index) => {
        const originalStyle = el.style.cssText;
        el.style.outline = '3px solid #2196F3';
        el.style.outlineOffset = '2px';
        el.style.transition = 'outline 0.3s';
        
        setTimeout(() => {
          el.style.cssText = originalStyle;
        }, 2000);
      });
      
      return visibleElements;
    } else {
      addLog(`❌ Tidak menemukan elemen dengan teks "${text}"`, 'warning');
      return [];
    }
  }
  
  // Fungsi untuk menemukan input field
  function findInputField(placeholderKeywords = [], type = '') {
    addLog(`🔍 Mencari field input...`, 'info');
    
    // Cari semua input elements
    const allInputs = document.querySelectorAll('input, textarea');
    const possibleInputs = [];
    
    allInputs.forEach(input => {
      // Check by placeholder
      const placeholder = (input.placeholder || '').toLowerCase();
      let placeholderMatch = false;
      
      if (placeholderKeywords.length > 0) {
        placeholderMatch = placeholderKeywords.some(keyword => 
          placeholder.includes(keyword.toLowerCase())
        );
      }
      
      // Check by type
      const inputType = (input.type || '').toLowerCase();
      let typeMatch = !type || inputType === type.toLowerCase();
      
      // Check by label or nearby text
      const parentText = (input.parentElement?.textContent || '').toLowerCase();
      const parentMatch = placeholderKeywords.some(keyword => 
        parentText.includes(keyword.toLowerCase())
      );
      
      if (placeholderMatch || typeMatch || parentMatch) {
        possibleInputs.push(input);
      }
    });
    
    // Juga cari elemen yang mungkin adalah input (div dengan role, etc)
    const possibleDivs = document.querySelectorAll('[contenteditable="true"], [role="textbox"]');
    possibleDivs.forEach(div => {
      possibleInputs.push(div);
    });
    
    if (possibleInputs.length > 0) {
      addLog(`✅ Ditemukan ${possibleInputs.length} field input`, 'success');
      return possibleInputs;
    } else {
      // Fallback: cari semua input yang terlihat
      const visibleInputs = Array.from(allInputs).filter(input => 
        input.offsetParent !== null && 
        input.getBoundingClientRect().width > 0
      );
      
      if (visibleInputs.length > 0) {
        addLog(`ℹ️ Ditemukan ${visibleInputs.length} input field (fallback)`, 'info');
        return visibleInputs;
      }
      
      addLog(`❌ Tidak menemukan field input`, 'warning');
      return [];
    }
  }
  
  // Fungsi untuk klik pada elemen
  function clickElement(element, description = 'Element') {
    if (!element) {
      addLog('❌ Element tidak valid', 'error');
      return false;
    }
    
    try {
      const rect = element.getBoundingClientRect();
      currentCoordinates = {
        x: rect.left + rect.width / 2,
        y: rect.top + rect.height / 2
      };
      
      addLog(`📍 Klik ${description} di (${Math.round(currentCoordinates.x)}, ${Math.round(currentCoordinates.y)})`, 'step');
      
      const mouseEvents = ['mousedown', 'mouseup', 'click'];
      mouseEvents.forEach(eventType => {
        const event = new MouseEvent(eventType, {
          view: window,
          bubbles: true,
          cancelable: true,
          clientX: currentCoordinates.x,
          clientY: currentCoordinates.y
        });
        element.dispatchEvent(event);
      });
      
      const touch = new Touch({
        identifier: Date.now(),
        target: element,
        clientX: currentCoordinates.x,
        clientY: currentCoordinates.y
      });
      
      const touchEvents = ['touchstart', 'touchend'];
      touchEvents.forEach(eventType => {
        const event = new TouchEvent(eventType, {
          touches: eventType === 'touchend' ? [] : [touch],
          changedTouches: [touch],
          bubbles: true
        });
        element.dispatchEvent(event);
      });
      
      showClickIndicator(currentCoordinates.x, currentCoordinates.y);
      
      return true;
    } catch (error) {
      addLog(`❌ Error: ${error.message}`, 'error');
      return false;
    }
  }
  
  // Fungsi untuk mengisi input field
  function fillInputField(inputElement, value, description = 'Field') {
    if (!inputElement) {
      addLog('❌ Input element tidak valid', 'error');
      return false;
    }
    
    try {
      // Focus dulu
      inputElement.focus();
      
      // Clear existing value
      inputElement.value = '';
      
      // Set new value
      inputElement.value = value;
      
      // Trigger events
      ['input', 'change', 'keyup', 'blur'].forEach(eventType => {
        const event = new Event(eventType, { bubbles: true });
        inputElement.dispatchEvent(event);
      });
      
      addLog(`📝 ${description} diisi: "${value}"`, 'step');
      
      // Tampilkan indikator
      const rect = inputElement.getBoundingClientRect();
      showClickIndicator(rect.left + rect.width / 2, rect.top + rect.height / 2, '#FF9800');
      
      return true;
    } catch (error) {
      addLog(`❌ Error mengisi ${description}: ${error.message}`, 'error');
      return false;
    }
  }
  
  // ============================================
  // ALUR KIRIM LENGKAP
  // ============================================
  
  async function processKirimLengkap() {
    if (isProcessing) {
      addLog('⚠️ Proses sedang berjalan', 'warning');
      return;
    }
    
    // Ambil data dari form
    const tujuan = document.getElementById('tujuan').value.trim();
    const nominal = document.getElementById('nominal').value.trim();
    const pin = document.getElementById('pin').value.trim();
    const skipPin = document.getElementById('skipPin').checked;
    const autoConfirm = document.getElementById('autoConfirm').checked;
    const delay = parseInt(document.getElementById('defaultDelay').value) || 1500;
    
    // Validasi
    if (!tujuan) {
      addLog('⚠️ Masukkan tujuan terlebih dahulu', 'warning');
      return;
    }
    
    if (!nominal) {
      addLog('⚠️ Masukkan nominal terlebih dahulu', 'warning');
      return;
    }
    
    if (!skipPin && (!pin || pin.length !== 6)) {
      addLog('⚠️ PIN harus 6 digit', 'warning');
      return;
    }
    
    isProcessing = true;
    currentStep = 0;
    updateKirimStatus('Memulai...', '#FF9800');
    updateProgressBar(10, true);
    
    try {
      addLog('🚀 MEMULAI PROSES KIRIM LENGKAP', 'success');
      addLog(`📋 Data: Tujuan=${tujuan}, Nominal=${nominal}, PIN=${skipPin ? 'Dilewati' : '***'}`, 'info');
      
      // ========== STEP 1: KLIK TOMBOL KIRIM ==========
      updateKirimStatus('Mencari tombol Kirim...', '#2196F3');
      updateProgressBar(20);
      currentStep = 1;
      
      addLog(`\n📌 STEP ${currentStep}: Mencari tombol "Kirim"`, 'step');
      await sleep(delay/2);
      
      const tombolKirimElements = findElementByText('Kirim', true);
      if (tombolKirimElements.length === 0) {
        // Coba varian lain
        const alternativeKirim = ['Send', 'Transfer', 'Kirim Uang', 'Send Money'];
        for (const alt of alternativeKirim) {
          const altElements = findElementByText(alt, true);
          if (altElements.length > 0) {
            if (clickElement(altElements[0], `Tombol ${alt}`)) {
              addLog(`✅ Tombol "${alt}" diklik`, 'success');
              break;
            }
          }
        }
      } else {
        if (clickElement(tombolKirimElements[0], 'Tombol Kirim')) {
          addLog('✅ Tombol "Kirim" diklik', 'success');
        }
      }
      
      await sleep(delay);
      
      // ========== STEP 2: CARI DAN PILIH TUJUAN ==========
      updateKirimStatus('Mencari tujuan...', '#2196F3');
      updateProgressBar(30);
      currentStep = 2;
      
      addLog(`\n📌 STEP ${currentStep}: Mencari dan memilih tujuan`, 'step');
      
      // Cari field untuk input tujuan
      const tujuanInputs = findInputField(['nama', 'kontak', 'tujuan', 'penerima', 'nomor', 'telepon', 'phone', 'search'], 'text');
      
      if (tujuanInputs.length > 0) {
        const tujuanInput = tujuanInputs[0];
        
        // Isi field tujuan
        if (fillInputField(tujuanInput, tujuan, 'Field tujuan')) {
          addLog(`✅ Tujuan "${tujuan}" diisi`, 'success');
          
          // Tunggu hasil pencarian
          await sleep(delay);
          
          // Coba cari kontak yang cocok dalam daftar
          const contactElements = findElementByText(tujuan, true);
          if (contactElements.length > 0) {
            // Cari yang paling mungkin adalah kontak (biasanya di list)
            const likelyContact = contactElements.find(el => {
              const text = (el.textContent || '').toLowerCase();
              return text.includes(tujuan.toLowerCase()) && 
                     !text.includes('kirim') && 
                     !text.includes('send');
            });
            
            if (likelyContact) {
              if (clickElement(likelyContact, `Kontak ${tujuan}`)) {
                addLog(`✅ Kontak "${tujuan}" dipilih`, 'success');
              }
            }
          }
        }
      }
      
      await sleep(delay);
      
      // ========== STEP 3: ISI NOMINAL ==========
      updateKirimStatus('Mengisi nominal...', '#2196F3');
      updateProgressBar(50);
      currentStep = 3;
      
      addLog(`\n📌 STEP ${currentStep}: Mengisi nominal`, 'step');
      
      const nominalInputs = findInputField(['nominal', 'jumlah', 'amount', 'uang', 'rp', 'rupiah'], 'number');
      
      if (nominalInputs.length > 0) {
        const nominalInput = nominalInputs[0];
        if (fillInputField(nominalInput, nominal, 'Field nominal')) {
          addLog(`✅ Nominal Rp ${nominal} diisi`, 'success');
        }
      } else {
        // Fallback: cari input apa saja yang bisa diisi angka
        const allInputs = document.querySelectorAll('input[type="number"], input[type="tel"]');
        if (allInputs.length > 0) {
          const input = allInputs[0];
          if (fillInputField(input, nominal, 'Field nominal (fallback)')) {
            addLog(`✅ Nominal Rp ${nominal} diisi (fallback)`, 'success');
          }
        }
      }
      
      await sleep(delay);
      
      // ========== STEP 4: KLIK LANJUT ==========
      updateKirimStatus('Mencari tombol Lanjut...', '#2196F3');
      updateProgressBar(60);
      currentStep = 4;
      
      addLog(`\n📌 STEP ${currentStep}: Mencari tombol "Lanjut"`, 'step');
      
      const lanjutElements = findElementByText('Lanjut', true);
      if (lanjutElements.length === 0) {
        const alternativeLanjut = ['Next', 'Continue', 'Berikutnya', 'Selanjutnya'];
        for (const alt of alternativeLanjut) {
          const altElements = findElementByText(alt, true);
          if (altElements.length > 0) {
            if (clickElement(altElements[0], `Tombol ${alt}`)) {
              addLog(`✅ Tombol "${alt}" diklik`, 'success');
              break;
            }
          }
        }
      } else {
        if (clickElement(lanjutElements[0], 'Tombol Lanjut')) {
          addLog('✅ Tombol "Lanjut" diklik', 'success');
        }
      }
      
      await sleep(delay);
      
      // ========== STEP 5: KLIK BAYAR ==========
      updateKirimStatus('Mencari tombol Bayar...', '#2196F3');
      updateProgressBar(70);
      currentStep = 5;
      
      addLog(`\n📌 STEP ${currentStep}: Mencari tombol "Bayar"`, 'step');
      
      const bayarElements = findElementByText('Bayar', true);
      if (bayarElements.length === 0) {
        const alternativeBayar = ['Pay', 'Payment', 'Transfer Now', 'Kirim Sekarang'];
        for (const alt of alternativeBayar) {
          const altElements = findElementByText(alt, true);
          if (altElements.length > 0) {
            if (clickElement(altElements[0], `Tombol ${alt}`)) {
              addLog(`✅ Tombol "${alt}" diklik`, 'success');
              break;
            }
          }
        }
      } else {
        if (clickElement(bayarElements[0], 'Tombol Bayar')) {
          addLog('✅ Tombol "Bayar" diklik', 'success');
        }
      }
      
      await sleep(delay);
      
      // ========== STEP 6: ISI PIN (jika diperlukan) ==========
      if (!skipPin) {
        updateKirimStatus('Mengisi PIN...', '#2196F3');
        updateProgressBar(80);
        currentStep = 6;
        
        addLog(`\n📌 STEP ${currentStep}: Mengisi PIN`, 'step');
        
        const pinInputs = findInputField(['pin', 'password', 'kode', 'security'], 'password');
        
        if (pinInputs.length > 0) {
          const pinInput = pinInputs[0];
          if (fillInputField(pinInput, pin, 'Field PIN')) {
            addLog(`✅ PIN diisi (${pin.length} digit)`, 'success');
          }
        } else {
          // Cari input dengan type password atau number untuk PIN
          const passwordInputs = document.querySelectorAll('input[type="password"], input[type="number"][maxlength="6"]');
          if (passwordInputs.length > 0) {
            const pinInput = passwordInputs[0];
            if (fillInputField(pinInput, pin, 'Field PIN (fallback)')) {
              addLog(`✅ PIN diisi (${pin.length} digit)`, 'success');
            }
          }
        }
        
        await sleep(delay);
        
        // ========== STEP 7: KONFIRMASI (jika diaktifkan) ==========
        if (autoConfirm) {
          updateKirimStatus('Mengonfirmasi...', '#2196F3');
          updateProgressBar(90);
          currentStep = 7;
          
          addLog(`\n📌 STEP ${currentStep}: Mengonfirmasi transaksi`, 'step');
          
          const konfirmasiElements = findElementByText('Konfirmasi', true);
          if (konfirmasiElements.length === 0) {
            const alternativeKonfirmasi = ['Confirm', 'Yes', 'Ya', 'OK', 'Setuju', 'Proses'];
            for (const alt of alternativeKonfirmasi) {
              const altElements = findElementByText(alt, true);
              if (altElements.length > 0) {
                if (clickElement(altElements[0], `Tombol ${alt}`)) {
                  addLog(`✅ Tombol "${alt}" diklik`, 'success');
                  break;
                }
              }
            }
          } else {
            if (clickElement(konfirmasiElements[0], 'Tombol Konfirmasi')) {
              addLog('✅ Transaksi dikonfirmasi', 'success');
            }
          }
        }
      }
      
      // ========== SELESAI ==========
      await sleep(delay);
      updateKirimStatus('Selesai!', '#4CAF50');
      updateProgressBar(100);
      
      addLog('\n🎉 PROSES KIRIM LENGKAP SELESAI!', 'success');
      addLog(`📊 Ringkasan: Tujuan=${tujuan}, Nominal=Rp ${nominal}, Langkah=${currentStep}`, 'info');
      
    } catch (error) {
      addLog(`❌ Error dalam proses: ${error.message}`, 'error');
      updateKirimStatus('Error!', '#F44336');
    } finally {
      isProcessing = false;
      setTimeout(() => {
        updateProgressBar(0, false);
      }, 2000);
    }
  }
  
  // ============================================
  // FUNGSI BANTU
  // ============================================
  
  // Helper: sleep function
  function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Fungsi untuk menampilkan indikator klik
  function showClickIndicator(x, y, color = '#FF416C') {
    const indicator = document.createElement('div');
    indicator.style.cssText = `
      position: fixed;
      left: ${x - 20}px;
      top: ${y - 20}px;
      width: 40px;
      height: 40px;
      border: 3px solid ${color};
      border-radius: 50%;
      background: ${color.replace(')', ', 0.2)').replace('rgb', 'rgba')};
      pointer-events: none;
      z-index: 999998;
      animation: vysorPulse 0.5s ease-out;
    `;
    
    const style = document.createElement('style');
    style.textContent = `
      @keyframes vysorPulse {
        0% { transform: scale(0.5); opacity: 1; }
        100% { transform: scale(2); opacity: 0; }
      }
    `;
    document.head.appendChild(style);
    
    document.body.appendChild(indicator);
    setTimeout(() => {
      if (indicator.parentNode) indicator.parentNode.removeChild(indicator);
      if (style.parentNode) style.parentNode.removeChild(style);
    }, 500);
  }
  
  // ============================================
  // FITUR REKAMAN DENGAN DELETE LANGKAH
  // ============================================
  
  // Fungsi untuk memulai rekaman
  function startRecording() {
    if (recordingState.isRecording) {
      addLog('⚠️ Rekaman sudah berjalan', 'warning');
      return;
    }
    
    recordingState.isRecording = true;
    recordingState.recordingSteps = [];
    recordingState.startTime = Date.now();
    recordingState.selectedStepIndex = -1;
    
    // Update UI
    document.getElementById('btnStartRecording').disabled = true;
    document.getElementById('btnStopRecording').disabled = false;
    recordingIndicator.textContent = '🔴 REKAMAN AKTIF';
    recordingIndicator.style.display = 'block';
    
    // Tambahkan event listeners untuk merekam
    setupRecordingListeners();
    
    addLog('🔴 Rekaman dimulai. Mulai melakukan aksi...', 'success');
    updateRecordingStepsDisplay();
  }
  
  // Setup listeners untuk rekaman
  function setupRecordingListeners() {
    // Listener untuk klik
    const clickListener = (e) => {
      if (!recordingState.isRecording) return;
      
      const step = {
        type: 'click',
        timestamp: Date.now() - recordingState.startTime,
        x: e.clientX,
        y: e.clientY,
        element: {
          tagName: e.target.tagName,
          className: e.target.className,
          id: e.target.id,
          text: e.target.textContent?.substring(0, 50)
        }
      };
      
      recordingState.recordingSteps.push(step);
      addLog(`📝 Direkam: Klik di (${step.x}, ${step.y})`, 'info');
      updateRecordingStepsDisplay();
    };
    
    // Listener untuk input
    const inputListener = (e) => {
      if (!recordingState.isRecording) return;
      if (!['INPUT', 'TEXTAREA'].includes(e.target.tagName)) return;
      
      const step = {
        type: 'input',
        timestamp: Date.now() - recordingState.startTime,
        value: e.target.value,
        element: {
          tagName: e.target.tagName,
          className: e.target.className,
          id: e.target.id,
          placeholder: e.target.placeholder,
          type: e.target.type
        }
      };
      
      recordingState.recordingSteps.push(step);
      addLog(`📝 Direkam: Input "${step.value.substring(0, 30)}"`, 'info');
      updateRecordingStepsDisplay();
    };
    
    // Simpan listeners untuk nanti dihapus
    clickListeners = [
      { type: 'click', listener: clickListener },
      { type: 'input', listener: inputListener }
    ];
    
    // Tambahkan listeners
    document.addEventListener('click', clickListener, true);
    document.addEventListener('input', inputListener, true);
  }
  
  // Hapus listeners rekaman
  function removeRecordingListeners() {
    clickListeners.forEach(({ type, listener }) => {
      document.removeEventListener(type, listener, true);
    });
    clickListeners = [];
  }
  
  // Fungsi untuk menghentikan rekaman
  function stopRecording() {
    if (!recordingState.isRecording) {
      addLog('⚠️ Tidak ada rekaman yang berjalan', 'warning');
      return;
    }
    
    recordingState.isRecording = false;
    
    // Update UI
    document.getElementById('btnStartRecording').disabled = false;
    document.getElementById('btnStopRecording').disabled = true;
    recordingIndicator.style.display = 'none';
    
    // Hapus event listeners
    removeRecordingListeners();
    
    const duration = ((Date.now() - recordingState.startTime) / 1000).toFixed(1);
    addLog(`⏹️ Rekaman dihentikan. Durasi: ${duration} detik, ${recordingState.recordingSteps.length} langkah.`, 'success');
    
    // Update daftar rekaman tersimpan
    updateSavedRecordingsList();
  }
  
  // Fungsi untuk menambah delay secara manual
  function addDelayStep() {
    if (!recordingState.isRecording) {
      addLog('⚠️ Rekaman tidak aktif', 'warning');
      return;
    }
    
    const duration = prompt('Masukkan durasi delay (ms):', '1000');
    if (!duration || isNaN(duration)) {
      addLog('❌ Durasi tidak valid', 'error');
      return;
    }
    
    const step = {
      type: 'delay',
      timestamp: Date.now() - recordingState.startTime,
      duration: parseInt(duration)
    };
    
    recordingState.recordingSteps.push(step);
    addLog(`⏱️ Delay ${duration}ms ditambahkan`, 'success');
    updateRecordingStepsDisplay();
  }
  
  // Fungsi untuk menghapus langkah yang dipilih
  function deleteSelectedStep() {
    if (recordingState.selectedStepIndex === -1) {
      addLog('⚠️ Pilih langkah terlebih dahulu', 'warning');
      return;
    }
    
    if (confirm(`Hapus langkah ${recordingState.selectedStepIndex + 1}?`)) {
      recordingState.recordingSteps.splice(recordingState.selectedStepIndex, 1);
      recordingState.selectedStepIndex = -1;
      addLog(`🗑️ Langkah dihapus`, 'success');
      updateRecordingStepsDisplay();
      updateSelectedStepInfo();
    }
  }
  
  // Fungsi untuk menghapus semua langkah
  function clearAllSteps() {
    if (recordingState.recordingSteps.length === 0) {
      addLog('ℹ️ Tidak ada langkah untuk dihapus', 'info');
      return;
    }
    
    if (confirm(`Hapus semua ${recordingState.recordingSteps.length} langkah?`)) {
      recordingState.recordingSteps = [];
      recordingState.selectedStepIndex = -1;
      addLog(`🧹 Semua langkah dihapus`, 'success');
      updateRecordingStepsDisplay();
      updateSelectedStepInfo();
    }
  }
  
  // Fungsi untuk mengedit langkah yang dipilih
  function editSelectedStep() {
    if (recordingState.selectedStepIndex === -1) {
      addLog('⚠️ Pilih langkah terlebih dahulu', 'warning');
      return;
    }
    
    const step = recordingState.recordingSteps[recordingState.selectedStepIndex];
    
    if (step.type === 'click') {
      const newX = prompt(`Koordinat X saat ini: ${step.x}\nMasukkan nilai baru:`, step.x);
      const newY = prompt(`Koordinat Y saat ini: ${step.y}\nMasukkan nilai baru:`, step.y);
      
      if (newX !== null && newY !== null) {
        step.x = parseFloat(newX) || step.x;
        step.y = parseFloat(newY) || step.y;
        addLog(`✏️ Langkah ${recordingState.selectedStepIndex + 1} diperbarui`, 'success');
        updateRecordingStepsDisplay();
      }
    } else if (step.type === 'input') {
      const newValue = prompt(`Nilai input saat ini: "${step.value}"\nMasukkan nilai baru:`, step.value);
      
      if (newValue !== null) {
        step.value = newValue;
        addLog(`✏️ Langkah ${recordingState.selectedStepIndex + 1} diperbarui`, 'success');
        updateRecordingStepsDisplay();
      }
    } else if (step.type === 'delay') {
      const newDuration = prompt(`Durasi delay saat ini: ${step.duration}ms\nMasukkan nilai baru:`, step.duration);
      
      if (newDuration !== null) {
        step.duration = parseInt(newDuration) || step.duration;
        addLog(`✏️ Langkah ${recordingState.selectedStepIndex + 1} diperbarui`, 'success');
        updateRecordingStepsDisplay();
      }
    }
  }
  
  // Update tampilan langkah rekaman
  function updateRecordingStepsDisplay() {
    const container = document.getElementById('recordingStepsList');
    const stepCount = document.getElementById('stepCount');
    
    if (!container) return;
    
    stepCount.textContent = `${recordingState.recordingSteps.length} langkah`;
    
    if (recordingState.recordingSteps.length === 0) {
      container.innerHTML = '<div style="color: rgba(255,255,255,0.6); text-align: center; padding: 10px;">Belum ada langkah</div>';
      return;
    }
    
    container.innerHTML = '';
    recordingState.recordingSteps.forEach((step, index) => {
      const stepEl = document.createElement('div');
      stepEl.className = `step-item step-type-${step.type} ${index === recordingState.selectedStepIndex ? 'selected' : ''}`;
      stepEl.dataset.index = index;
      
      let stepText = '';
      let stepIcon = '';
      
      if (step.type === 'click') {
        stepIcon = '🖱️';
        stepText = `Klik di (${Math.round(step.x)}, ${Math.round(step.y)})`;
        if (step.element.text) {
          stepText += ` - ${step.element.tagName}: "${step.element.text}"`;
        }
      } else if (step.type === 'input') {
        stepIcon = '⌨️';
        stepText = `Input: "${step.value}"`;
        if (step.element.placeholder) {
          stepText += ` [${step.element.placeholder}]`;
        }
      } else if (step.type === 'delay') {
        stepIcon = '⏱️';
        stepText = `Delay: ${step.duration}ms`;
      }
      
      stepEl.innerHTML = `
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <div style="flex: 1;">
            <div style="display: flex; align-items: center; gap: 5px;">
              <span style="font-size: 10px;">${stepIcon}</span>
              <span style="font-size: 10px;">${index + 1}. ${stepText}</span>
            </div>
          </div>
          <div style="font-size: 9px; opacity: 0.7;">${(step.timestamp / 1000).toFixed(1)}s</div>
        </div>
        <div class="step-actions">
          <button class="step-action-btn delete-step" title="Hapus langkah">🗑️</button>
          <button class="step-action-btn edit-step" title="Edit langkah">✏️</button>
        </div>
      `;
      
      // Tambahkan event listener untuk memilih langkah
      stepEl.addEventListener('click', (e) => {
        if (!e.target.classList.contains('step-action-btn')) {
          // Update selected step
          recordingState.selectedStepIndex = index;
          updateRecordingStepsDisplay();
          updateSelectedStepInfo();
        }
      });
      
      container.appendChild(stepEl);
    });
    
    // Tambahkan event delegation untuk tombol delete di setiap langkah
    container.addEventListener('click', (e) => {
      if (e.target.classList.contains('delete-step')) {
        const stepEl = e.target.closest('.step-item');
        const index = parseInt(stepEl.dataset.index);
        
        if (confirm(`Hapus langkah ${index + 1}?`)) {
          recordingState.recordingSteps.splice(index, 1);
          
          // Update selected index jika perlu
          if (recordingState.selectedStepIndex === index) {
            recordingState.selectedStepIndex = -1;
          } else if (recordingState.selectedStepIndex > index) {
            recordingState.selectedStepIndex--;
          }
          
          addLog(`🗑️ Langkah ${index + 1} dihapus`, 'success');
          updateRecordingStepsDisplay();
          updateSelectedStepInfo();
        }
        e.stopPropagation();
      }
      
      if (e.target.classList.contains('edit-step')) {
        const stepEl = e.target.closest('.step-item');
        const index = parseInt(stepEl.dataset.index);
        
        // Select step terlebih dahulu
        recordingState.selectedStepIndex = index;
        updateRecordingStepsDisplay();
        updateSelectedStepInfo();
        
        // Edit step
        editSelectedStep();
        e.stopPropagation();
      }
    });
  }
  
  // Update info langkah terpilih
  function updateSelectedStepInfo() {
    const selectedStepInfo = document.getElementById('selectedStepInfo');
    if (selectedStepInfo) {
      if (recordingState.selectedStepIndex !== -1) {
        selectedStepInfo.textContent = `Terpilih: ${recordingState.selectedStepIndex + 1}`;
        selectedStepInfo.style.display = 'inline';
      } else {
        selectedStepInfo.style.display = 'none';
      }
    }
  }
  
  // Simpan rekaman ke localStorage
  function saveRecording() {
    if (recordingState.recordingSteps.length === 0) {
      addLog('⚠️ Tidak ada rekaman untuk disimpan', 'warning');
      return;
    }
    
    const name = prompt('Berikan nama untuk rekaman ini:', `rekaman_${new Date().toLocaleDateString()}`);
    if (!name) return;
    
    const recording = {
      name,
      steps: recordingState.recordingSteps,
      created: new Date().toISOString(),
      version: '2.2.0'
    };
    
    // Simpan ke localStorage
    const saved = JSON.parse(localStorage.getItem('vysorRecordings') || '{}');
    saved[name] = recording;
    localStorage.setItem('vysorRecordings', JSON.stringify(saved));
    
    addLog(`💾 Rekaman "${name}" disimpan (${recording.steps.length} langkah)`, 'success');
    updateSavedRecordingsList();
  }
  
  // Muat rekaman dari localStorage
  function loadRecording() {
    const select = document.getElementById('savedRecordings');
    const name = select.value;
    
    if (!name) {
      addLog('⚠️ Pilih rekaman terlebih dahulu', 'warning');
      return;
    }
    
    const saved = JSON.parse(localStorage.getItem('vysorRecordings') || '{}');
    const recording = saved[name];
    
    if (!recording) {
      addLog(`❌ Rekaman "${name}" tidak ditemukan`, 'error');
      return;
    }
    
    recordingState.recordingSteps = recording.steps;
    recordingState.selectedStepIndex = -1;
    addLog(`📂 Rekaman "${name}" dimuat (${recording.steps.length} langkah)`, 'success');
    updateRecordingStepsDisplay();
    updateSelectedStepInfo();
  }
  
  // Hapus rekaman
  function deleteRecording() {
    const select = document.getElementById('savedRecordings');
    const name = select.value;
    
    if (!name) {
      addLog('⚠️ Pilih rekaman terlebih dahulu', 'warning');
      return;
    }
    
    if (!confirm(`Hapus rekaman "${name}"?`)) return;
    
    const saved = JSON.parse(localStorage.getItem('vysorRecordings') || '{}');
    delete saved[name];
    localStorage.setItem('vysorRecordings', JSON.stringify(saved));
    
    addLog(`🗑️ Rekaman "${name}" dihapus`, 'success');
    updateSavedRecordingsList();
    document.getElementById('savedRecordings').value = '';
  }
  
  // Mainkan rekaman
  async function playRecording() {
    if (recordingState.recordingSteps.length === 0) {
      addLog('⚠️ Tidak ada langkah untuk dimainkan', 'warning');
      return;
    }
    
    addLog(`▶️ Memainkan rekaman (${recordingState.recordingSteps.length} langkah)...`, 'success');
    
    const speed = recordingState.playbackSpeed;
    let lastTimestamp = 0;
    
    for (let i = 0; i < recordingState.recordingSteps.length; i++) {
      const step = recordingState.recordingSteps[i];
      
      // Hitung delay berdasarkan perbedaan timestamp
      const delay = i === 0 ? 0 : (step.timestamp - lastTimestamp) / speed;
      
      if (delay > 0) {
        await sleep(delay);
      }
      
      // Eksekusi step
      await executeStep(step, i);
      
      lastTimestamp = step.timestamp;
    }
    
    addLog('✅ Rekaman selesai dimainkan', 'success');
  }
  
  // Eksekusi satu langkah
  async function executeStep(step, index) {
    try {
      if (step.type === 'click') {
        addLog(`Langkah ${index + 1}: Klik di (${step.x}, ${step.y})`, 'info');
        await simulateClick(step.x, step.y);
      } else if (step.type === 'input') {
        addLog(`Langkah ${index + 1}: Input "${step.value}"`, 'info');
        await simulateInput(step.element, step.value);
      } else if (step.type === 'delay') {
        addLog(`Langkah ${index + 1}: Delay ${step.duration}ms`, 'info');
        await sleep(step.duration / recordingState.playbackSpeed);
      }
    } catch (error) {
      addLog(`❌ Error pada langkah ${index + 1}: ${error.message}`, 'error');
    }
  }
  
  // Simulasi klik
  async function simulateClick(x, y) {
    const element = document.elementFromPoint(x, y);
    if (!element) {
      throw new Error(`Tidak ada elemen di (${x}, ${y})`);
    }
    
    // Tampilkan indikator
    showClickIndicator(x, y);
    
    // Kirim event
    const mouseEvents = ['mousedown', 'mouseup', 'click'];
    mouseEvents.forEach(eventType => {
      const event = new MouseEvent(eventType, {
        view: window,
        bubbles: true,
        cancelable: true,
        clientX: x,
        clientY: y
      });
      element.dispatchEvent(event);
    });
    
    await sleep(100);
  }
  
  // Simulasi input
  async function simulateInput(elementInfo, value) {
    let element;
    
    // Cari elemen berdasarkan informasi yang direkam
    if (elementInfo.id) {
      element = document.getElementById(elementInfo.id);
    }
    
    if (!element && elementInfo.className) {
      const elements = document.getElementsByClassName(elementInfo.className);
      if (elements.length > 0) element = elements[0];
    }
    
    if (!element) {
      throw new Error(`Elemen tidak ditemukan: ${JSON.stringify(elementInfo)}`);
    }
    
    element.focus();
    element.value = value;
    
    // Trigger events
    ['input', 'change', 'keyup'].forEach(eventType => {
      element.dispatchEvent(new Event(eventType, { bubbles: true }));
    });
    
    await sleep(100);
  }
  
  // Update daftar rekaman tersimpan
  function updateSavedRecordingsList() {
    const select = document.getElementById('savedRecordings');
    const saved = JSON.parse(localStorage.getItem('vysorRecordings') || '{}');
    
    // Simpan nilai yang dipilih
    const selectedValue = select.value;
    
    // Clear options
    select.innerHTML = '<option value="">Pilih rekaman...</option>';
    
    // Add options
    Object.keys(saved).forEach(name => {
      const option = document.createElement('option');
      option.value = name;
      option.textContent = `${name} (${saved[name].steps.length} langkah)`;
      select.appendChild(option);
    });
    
    // Restore selected value if still exists
    if (saved[selectedValue]) {
      select.value = selectedValue;
    }
    
    // Update info
    document.getElementById('totalRecordings').textContent = Object.keys(saved).length;
    document.getElementById('totalSteps').textContent = Object.values(saved).reduce((total, rec) => total + rec.steps.length, 0);
  }
  
  // ============================================
  // EVENT LISTENERS
  // ============================================
  
  // Tab Kirim
  document.getElementById('btnFindKirim').addEventListener('click', () => {
    findElementByText('Kirim', true);
  });
  
  document.getElementById('btnFindTujuan').addEventListener('click', () => {
    const tujuan = document.getElementById('tujuan').value;
    if (tujuan) {
      findElementByText(tujuan, true);
    } else {
      addLog('⚠️ Masukkan tujuan terlebih dahulu', 'warning');
    }
  });
  
  document.getElementById('btnAutoKirimLengkap').addEventListener('click', processKirimLengkap);
  
  document.getElementById('btnManualClick').addEventListener('click', () => {
    const x = parseInt(document.getElementById('coordX').value);
    const y = parseInt(document.getElementById('coordY').value);
    
    if (isNaN(x) || isNaN(y)) {
      addLog('⚠️ Masukkan koordinat X dan Y', 'warning');
      return;
    }
    
    const element = document.elementFromPoint(x, y);
    if (element) {
      clickElement(element, 'Manual');
      addLog(`📍 Klik manual di (${x}, ${y}) pada ${element.tagName}`, 'success');
    } else {
      addLog(`❌ Tidak ada elemen di (${x}, ${y})`, 'error');
    }
  });
  
  document.getElementById('btnCoordFinder').addEventListener('click', () => {
    addLog('🎯 Mode Coordinate Finder aktif. Klik di mana saja untuk melihat koordinat.', 'info');
    
    function handleClick(e) {
      const x = e.clientX;
      const y = e.clientY;
      
      document.getElementById('coordX').value = x;
      document.getElementById('coordY').value = y;
      
      showClickIndicator(x, y);
      addLog(`📍 Klik di: (${x}, ${y})`, 'success');
      
      if (window.coordFinderClicks > 10) {
        document.removeEventListener('click', handleClick);
        addLog('✅ Mode Coordinate Finder dimatikan', 'info');
      }
      window.coordFinderClicks = (window.coordFinderClicks || 0) + 1;
    }
    
    window.coordFinderClicks = 0;
    document.addEventListener('click', handleClick);
    
    setTimeout(() => {
      document.removeEventListener('click', handleClick);
      addLog('⏰ Mode Coordinate Finder dimatikan (timeout)', 'info');
    }, 30000);
  });
  
  // Tab Rekaman - Button baru
  document.getElementById('btnStartRecording').addEventListener('click', startRecording);
  document.getElementById('btnStopRecording').addEventListener('click', stopRecording);
  document.getElementById('btnPlayRecording').addEventListener('click', playRecording);
  document.getElementById('btnSaveRecording').addEventListener('click', saveRecording);
  document.getElementById('btnLoadRecording').addEventListener('click', loadRecording);
  document.getElementById('btnDeleteRecording').addEventListener('click', deleteRecording);
  
  // Tombol manajemen langkah baru
  document.getElementById('btnAddDelay').addEventListener('click', addDelayStep);
  document.getElementById('btnDeleteSelectedStep').addEventListener('click', deleteSelectedStep);
  document.getElementById('btnClearAllSteps').addEventListener('click', clearAllSteps);
  document.getElementById('btnEditStep').addEventListener('click', editSelectedStep);
  
  document.getElementById('playbackSpeed').addEventListener('input', function() {
    recordingState.playbackSpeed = parseFloat(this.value);
    document.getElementById('speedValue').textContent = `${this.value}x`;
  });
  
  // Tab Pengaturan
  document.getElementById('searchTolerance').addEventListener('input', function() {
    document.getElementById('toleranceValue').textContent = this.value;
  });
  
  document.getElementById('btnExportConfig').addEventListener('click', () => {
    const config = {
      recordings: JSON.parse(localStorage.getItem('vysorRecordings') || '{}'),
      settings: {
        defaultDelay: document.getElementById('defaultDelay').value,
        debugMode: document.getElementById('debugMode').checked,
        uiSize: currentSize
      }
    };
    
    const dataStr = JSON.stringify(config, null, 2);
    const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);
    
    const exportFileDefaultName = `vysor-config-${new Date().toISOString().slice(0,10)}.json`;
    
    const linkElement = document.createElement('a');
    linkElement.setAttribute('href', dataUri);
    linkElement.setAttribute('download', exportFileDefaultName);
    linkElement.click();
    
    addLog('📤 Konfigurasi diexport', 'success');
  });
  
  document.getElementById('btnImportConfig').addEventListener('click', () => {
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = '.json';
    
    input.onchange = e => {
      const file = e.target.files[0];
      const reader = new FileReader();
      
      reader.onload = event => {
        try {
          const config = JSON.parse(event.target.result);
          
          if (config.recordings) {
            localStorage.setItem('vysorRecordings', JSON.stringify(config.recordings));
          }
          
          if (config.settings) {
            if (config.settings.defaultDelay) {
              document.getElementById('defaultDelay').value = config.settings.defaultDelay;
            }
            if (config.settings.debugMode !== undefined) {
              document.getElementById('debugMode').checked = config.settings.debugMode;
            }
            if (config.settings.uiSize) {
              updateUISize(config.settings.uiSize.width, config.settings.uiSize.height);
            }
          }
          
          updateSavedRecordingsList();
          addLog('📥 Konfigurasi diimport', 'success');
        } catch (error) {
          addLog(`❌ Error import: ${error.message}`, 'error');
        }
      };
      
      reader.readAsText(file);
    };
    
    input.click();
  });
  
  // Tab Ukuran
  document.getElementById('widthSlider').addEventListener('input', function() {
    document.getElementById('widthInput').value = this.value;
  });
  
  document.getElementById('widthInput').addEventListener('input', function() {
    const width = parseInt(this.value) || defaultSize.width;
    document.getElementById('widthSlider').value = width;
  });
  
  document.getElementById('heightSlider').addEventListener('input', function() {
    document.getElementById('heightInput').value = this.value;
  });
  
  document.getElementById('heightInput').addEventListener('input', function() {
    const height = parseInt(this.value) || defaultSize.height;
    document.getElementById('heightSlider').value = height;
  });
  
  document.getElementById('btnApplySize').addEventListener('click', function() {
    const width = parseInt(document.getElementById('widthInput').value) || defaultSize.width;
    const height = parseInt(document.getElementById('heightInput').value) || defaultSize.height;
    updateUISize(width, height);
  });
  
  document.getElementById('btnResetSize').addEventListener('click', function() {
    updateUISize(defaultSize.width, defaultSize.height);
    addLog('🔄 Ukuran direset ke default', 'success');
  });
  
  // Preset ukuran
  document.querySelectorAll('.size-preset').forEach(preset => {
    preset.addEventListener('click', function() {
      const width = parseInt(this.dataset.width);
      const height = parseInt(this.dataset.height);
      updateUISize(width, height);
    });
  });
  
  // Tombol Close
  closeBtn.addEventListener('click', () => {
    container.remove();
    progressBar.remove();
    window.vysorAutomatorUI = null;
  });
  
  // Tombol Minimize
  minimizeBtn.addEventListener('click', toggleMinimize);
  
  // Resize handle dengan drag
  resizeHandle.addEventListener('mousedown', initResize);
  
  // ============================================
  // FUNGSI RESIZE DRAG
  // ============================================
  
  function initResize(e) {
    e.preventDefault();
    e.stopPropagation();
    
    window.addEventListener('mousemove', resize);
    window.addEventListener('mouseup', stopResize);
    
    function resize(e) {
      const containerRect = container.getBoundingClientRect();
      const newWidth = e.clientX - containerRect.left + 10;
      const newHeight = e.clientY - containerRect.top + 10;
      
      updateUISize(newWidth, newHeight, false);
    }
    
    function stopResize() {
      window.removeEventListener('mousemove', resize);
      window.removeEventListener('mouseup', stopResize);
      
      // Simpan ukuran akhir
      localStorage.setItem('vysorUISizeWidth', currentSize.width);
      localStorage.setItem('vysorUISizeHeight', currentSize.height);
    }
  }
  
  // ============================================
  // INISIALISASI
  // ============================================
  
  function initialize() {
    // Set tab pertama aktif
    document.querySelector('.tab-btn').click();
    
    // Update daftar rekaman
    updateSavedRecordingsList();
    
    // Set kecepatan pemutaran
    const speedInput = document.getElementById('playbackSpeed');
    speedInput.value = recordingState.playbackSpeed;
    document.getElementById('speedValue').textContent = `${recordingState.playbackSpeed}x`;
    
    // Set tolerance
    const toleranceInput = document.getElementById('searchTolerance');
    toleranceInput.value = 5;
    document.getElementById('toleranceValue').textContent = '5';
    
    // Set default values
    document.getElementById('defaultDelay').value = 1500;
    document.getElementById('debugMode').checked = false;
    
    // Set ukuran UI
    updateUISize(currentSize.width, currentSize.height, false);
    
    addLog('✅ Vysor Automator Pro v2.2 siap digunakan!', 'success');
    addLog('📐 Gunakan tab "Ukuran" untuk mengatur ukuran UI', 'info');
    console.log('%c🚀 VYSOR AUTOMATOR PRO v2.2 LOADED', 'color: #764ba2; font-size: 16px; font-weight: bold;');
    console.log('%c📁 Fitur: Kirim lengkap + PIN + Rekaman + Delete Langkah + Resizable UI', 'color: #4CAF50;');
    
    // Simpan referensi global
    window.vysorAutomatorUI = {
      container,
      processKirimLengkap,
      findElementByText,
      clickElement,
      startRecording,
      stopRecording,
      playRecording,
      saveRecording,
      loadRecording,
      deleteSelectedStep,
      clearAllSteps,
      editSelectedStep,
      addLog,
      updateUISize,
      toggleMinimize,
      recordingState
    };
  }
  
  // Jalankan inisialisasi
  initialize();
  
})();
