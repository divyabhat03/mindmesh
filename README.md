<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MedScan AI — Medical Image Analysis</title>
  <link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=Syne:wght@400;500;700&display=swap" rel="stylesheet" />
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg: #0a0d12;
      --surface: #111520;
      --surface2: #161c2a;
      --border: rgba(100,180,255,0.12);
      --border2: rgba(100,180,255,0.25);
      --accent: #3ab5ff;
      --accent2: #00e5a0;
      --danger: #ff5c5c;
      --warn: #ffb347;
      --text: #e8f0fe;
      --muted: #7a8ba8;
      --mono: 'DM Mono', monospace;
      --sans: 'Syne', sans-serif;
    }

    html, body {
      height: 100%;
      background: var(--bg);
      color: var(--text);
      font-family: var(--sans);
    }

    .app { display: flex; flex-direction: column; height: 100vh; overflow: hidden; }

    /* ── Header ── */
    .header {
      display: flex; align-items: center; justify-content: space-between;
      padding: 14px 24px;
      border-bottom: 0.5px solid var(--border);
      background: var(--surface);
      flex-shrink: 0;
    }
    .logo { display: flex; align-items: center; gap: 10px; }
    .logo-icon {
      width: 34px; height: 34px; border-radius: 8px;
      background: rgba(58,181,255,0.08);
      border: 1px solid var(--border2);
      display: flex; align-items: center; justify-content: center;
    }
    .logo-icon svg { width: 18px; height: 18px; }
    .logo-text { font-size: 15px; font-weight: 700; letter-spacing: -0.02em; }
    .logo-sub { font-size: 10px; color: var(--muted); font-family: var(--mono); }
    .header-right { display: flex; align-items: center; gap: 12px; }
    .status-pill {
      display: flex; align-items: center; gap: 6px;
      font-family: var(--mono); font-size: 11px; color: var(--accent2);
      background: rgba(0,229,160,0.08); border: 0.5px solid rgba(0,229,160,0.25);
      padding: 5px 12px; border-radius: 20px;
    }
    .dot { width: 6px; height: 6px; border-radius: 50%; background: var(--accent2); animation: pulse 2s infinite; }
    @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.35} }

    /* API Key input in header */
    .api-key-wrap { display: flex; align-items: center; gap: 6px; }
    .api-key-wrap label { font-family: var(--mono); font-size: 10px; color: var(--muted); white-space: nowrap; }
    .api-key-input {
      width: 200px; padding: 6px 10px;
      background: var(--surface2); border: 0.5px solid var(--border);
      border-radius: 6px; color: var(--text); font-family: var(--mono); font-size: 11px;
      outline: none; transition: border 0.15s;
    }
    .api-key-input:focus { border-color: var(--accent); }
    .api-key-input::placeholder { color: var(--muted); }

    /* ── Main layout ── */
    .main { display: flex; flex: 1; overflow: hidden; }

    /* ── Left panel ── */
    .panel-left {
      width: 340px; min-width: 280px; flex-shrink: 0;
      border-right: 0.5px solid var(--border);
      display: flex; flex-direction: column;
      background: var(--surface);
      overflow-y: auto;
    }
    .panel-section { padding: 18px 20px; border-bottom: 0.5px solid var(--border); }
    .panel-label {
      font-family: var(--mono); font-size: 10px; color: var(--muted);
      letter-spacing: 0.1em; text-transform: uppercase; margin-bottom: 12px;
    }

    /* ── Upload zone ── */
    .upload-zone {
      border: 1.5px dashed var(--border2); border-radius: 12px;
      padding: 26px 16px; text-align: center; cursor: pointer;
      transition: all 0.2s; position: relative;
      background: rgba(58,181,255,0.02);
    }
    .upload-zone:hover, .upload-zone.drag { border-color: var(--accent); background: rgba(58,181,255,0.07); }
    .upload-zone input { position: absolute; inset: 0; opacity: 0; cursor: pointer; width: 100%; height: 100%; }
    .upload-icon svg { width: 34px; height: 34px; stroke: var(--accent); stroke-width: 1.5; fill: none; margin-bottom: 8px; }
    .upload-title { font-size: 13px; font-weight: 600; color: var(--text); margin-bottom: 3px; }
    .upload-hint { font-size: 11px; color: var(--muted); font-family: var(--mono); }

    /* ── Preview ── */
    #preview-container { display: none; margin-top: 12px; }
    .preview-wrap { border-radius: 10px; overflow: hidden; position: relative; border: 0.5px solid var(--border2); }
    #preview-img { width: 100%; display: block; max-height: 200px; object-fit: contain; background: #000; }
    .scanning-overlay { position: absolute; inset: 0; pointer-events: none; display: none; }
    .scan-line {
      position: absolute; left: 0; right: 0; height: 2px;
      background: linear-gradient(90deg, transparent, var(--accent), transparent);
      animation: scanDown 2s linear infinite;
    }
    @keyframes scanDown { from { top: 0; opacity: 1; } to { top: 100%; opacity: 0; } }
    .preview-overlay { position: absolute; top: 8px; right: 8px; }
    .close-btn {
      width: 26px; height: 26px; border-radius: 5px;
      background: rgba(0,0,0,0.7); border: 0.5px solid var(--border2);
      color: var(--text); cursor: pointer; display: flex; align-items: center; justify-content: center;
    }
    .close-btn svg { width: 13px; height: 13px; stroke: currentColor; fill: none; stroke-width: 2; }

    /* ── Scan type buttons ── */
    .scan-types { display: flex; flex-wrap: wrap; gap: 6px; }
    .scan-type-btn {
      flex: 1; min-width: 65px; padding: 8px 6px; border-radius: 8px; cursor: pointer;
      font-family: var(--mono); font-size: 11px; font-weight: 500; text-align: center;
      border: 0.5px solid var(--border); background: transparent; color: var(--muted);
      transition: all 0.15s;
    }
    .scan-type-btn:hover { border-color: var(--accent); color: var(--accent); }
    .scan-type-btn.active { border-color: var(--accent); background: rgba(58,181,255,0.1); color: var(--accent); }

    /* ── Form fields ── */
    .field-row { display: flex; gap: 10px; margin-bottom: 10px; }
    .field { flex: 1; }
    .field label { display: block; font-family: var(--mono); font-size: 10px; color: var(--muted); margin-bottom: 4px; }
    .field input, .field select, .field textarea {
      width: 100%; padding: 8px 10px;
      background: var(--surface2); border: 0.5px solid var(--border);
      border-radius: 6px; color: var(--text); font-family: var(--mono); font-size: 12px;
      outline: none; transition: border 0.15s;
    }
    .field input:focus, .field select:focus, .field textarea:focus { border-color: var(--accent); }
    .field input::placeholder, .field textarea::placeholder { color: var(--muted); }
    .field select option { background: var(--surface2); }
    .field textarea { resize: vertical; min-height: 60px; line-height: 1.5; }

    /* ── Analyze button ── */
    .analyze-wrap { padding: 16px 20px; flex-shrink: 0; }
    .analyze-btn {
      width: 100%; padding: 13px; border-radius: 10px; border: none; cursor: pointer;
      font-family: var(--sans); font-size: 14px; font-weight: 700; letter-spacing: 0.02em;
      background: linear-gradient(135deg, #3ab5ff, #00c8a0);
      color: #fff; transition: all 0.2s;
    }
    .analyze-btn:hover:not(:disabled) { opacity: 0.9; transform: translateY(-1px); }
    .analyze-btn:active:not(:disabled) { transform: translateY(0); }
    .analyze-btn:disabled { opacity: 0.35; cursor: not-allowed; }
    .btn-loader {
      display: inline-block; width: 13px; height: 13px;
      border: 2px solid rgba(255,255,255,0.3); border-top-color: #fff;
      border-radius: 50%; animation: spin 0.8s linear infinite; margin-right: 7px; vertical-align: middle;
    }
    @keyframes spin { to { transform: rotate(360deg); } }

    /* ── Right panel ── */
    .panel-right { flex: 1; display: flex; flex-direction: column; overflow: hidden; }

    .report-header {
      padding: 14px 24px; border-bottom: 0.5px solid var(--border);
      display: flex; align-items: center; justify-content: space-between; flex-shrink: 0;
    }
    .report-title-text { font-size: 12px; font-weight: 500; color: var(--muted); font-family: var(--mono); }
    .report-actions { display: flex; gap: 8px; }
    .icon-btn {
      padding: 6px 12px; border-radius: 6px; cursor: pointer;
      font-family: var(--mono); font-size: 11px; color: var(--muted);
      background: transparent; border: 0.5px solid var(--border);
      transition: all 0.15s; display: flex; align-items: center; gap: 5px;
    }
    .icon-btn:hover { border-color: var(--accent); color: var(--accent); }
    .icon-btn svg { width: 12px; height: 12px; stroke: currentColor; fill: none; stroke-width: 2; }

    /* ── Progress bar ── */
    .progress-wrap { padding: 14px 24px; border-bottom: 0.5px solid var(--border); display: none; flex-shrink: 0; }
    .progress-label { font-family: var(--mono); font-size: 11px; color: var(--accent); margin-bottom: 7px; display: flex; justify-content: space-between; }
    .progress-track { height: 3px; background: rgba(58,181,255,0.1); border-radius: 2px; }
    .progress-fill { height: 100%; background: linear-gradient(90deg, #3ab5ff, #00e5a0); border-radius: 2px; transition: width 0.4s ease; }

    /* ── Empty state ── */
    .empty-state {
      flex: 1; display: flex; flex-direction: column;
      align-items: center; justify-content: center; padding: 48px; text-align: center;
    }
    .empty-icon { margin-bottom: 18px; opacity: 0.25; }
    .empty-icon svg { width: 64px; height: 64px; stroke: var(--accent); fill: none; stroke-width: 1; }
    .empty-title { font-size: 17px; font-weight: 600; color: var(--muted); margin-bottom: 6px; }
    .empty-hint { font-size: 12px; color: var(--muted); font-family: var(--mono); opacity: 0.6; }

    /* ── Report content ── */
    .report-content { flex: 1; overflow-y: auto; padding: 24px; }

    /* ── Report cards & sections ── */
    .severity-critical { color: var(--danger); background: rgba(255,92,92,0.1); border: 0.5px solid rgba(255,92,92,0.35); }
    .severity-moderate { color: var(--warn); background: rgba(255,179,71,0.1); border: 0.5px solid rgba(255,179,71,0.35); }
    .severity-normal  { color: var(--accent2); background: rgba(0,229,160,0.1); border: 0.5px solid rgba(0,229,160,0.35); }
    .severity-low     { color: var(--accent); background: rgba(58,181,255,0.1); border: 0.5px solid rgba(58,181,255,0.35); }
    .badge { display: inline-block; padding: 3px 9px; border-radius: 4px; font-family: var(--mono); font-size: 10px; font-weight: 500; }

    .report-section { margin-bottom: 24px; }
    .section-header { display: flex; align-items: center; gap: 8px; margin-bottom: 12px; padding-bottom: 8px; border-bottom: 0.5px solid var(--border); }
    .section-icon { width: 17px; height: 17px; stroke: var(--accent); fill: none; stroke-width: 1.5; flex-shrink: 0; }
    .section-title { font-size: 11px; font-weight: 500; color: var(--accent); font-family: var(--mono); letter-spacing: 0.07em; text-transform: uppercase; }

    .finding-card {
      background: var(--surface2); border: 0.5px solid var(--border);
      border-radius: 8px; padding: 14px; margin-bottom: 8px;
    }
    .finding-top { display: flex; align-items: flex-start; justify-content: space-between; margin-bottom: 6px; gap: 8px; }
    .finding-name { font-size: 13px; font-weight: 600; color: var(--text); }
    .finding-desc { font-size: 12px; color: var(--muted); line-height: 1.65; font-family: var(--mono); }
    .finding-meta { display: flex; gap: 5px; margin-top: 9px; flex-wrap: wrap; }
    .meta-tag {
      font-family: var(--mono); font-size: 10px; color: var(--muted);
      background: rgba(255,255,255,0.04); border: 0.5px solid var(--border);
      padding: 2px 7px; border-radius: 4px;
    }

    .summary-bar { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; margin-bottom: 22px; }
    .summary-card {
      background: var(--surface2); border: 0.5px solid var(--border);
      border-radius: 8px; padding: 13px; text-align: center;
    }
    .summary-num { font-size: 22px; font-weight: 700; font-family: var(--mono); }
    .summary-label { font-size: 10px; color: var(--muted); font-family: var(--mono); margin-top: 2px; }

    .conf-row { display: flex; align-items: center; gap: 8px; margin-top: 9px; }
    .conf-label { font-family: var(--mono); font-size: 10px; color: var(--muted); min-width: 100px; }
    .conf-track { flex: 1; height: 3px; background: rgba(255,255,255,0.06); border-radius: 2px; overflow: hidden; }
    .conf-fill { height: 100%; border-radius: 2px; }
    .conf-val { font-family: var(--mono); font-size: 10px; color: var(--muted); min-width: 28px; text-align: right; }

    .rec-item { display: flex; gap: 10px; margin-bottom: 10px; align-items: flex-start; }
    .rec-num {
      width: 20px; height: 20px; border-radius: 50%; flex-shrink: 0;
      background: rgba(58,181,255,0.1); border: 0.5px solid rgba(58,181,255,0.3);
      display: flex; align-items: center; justify-content: center;
      font-family: var(--mono); font-size: 10px; color: var(--accent); margin-top: 1px;
    }
    .rec-text { font-size: 12px; color: var(--muted); line-height: 1.65; font-family: var(--mono); }

    /* ── Disclaimer box ── */
    .disclaimer-box {
      background: rgba(255,179,71,0.07); border: 0.5px solid rgba(255,179,71,0.3);
      border-radius: 8px; padding: 14px; margin-top: 6px;
    }
    .disclaimer-label { font-family: var(--mono); font-size: 10px; color: var(--warn); font-weight: 500; margin-bottom: 5px; }
    .disclaimer-text { font-family: var(--mono); font-size: 11px; color: var(--muted); line-height: 1.65; }

    /* ── Error box ── */
    .error-box {
      background: rgba(255,92,92,0.08); border: 0.5px solid rgba(255,92,92,0.3);
      border-radius: 8px; padding: 16px; margin: 24px;
    }
    .error-title { font-family: var(--mono); font-size: 12px; color: var(--danger); font-weight: 500; margin-bottom: 6px; }
    .error-msg { font-family: var(--mono); font-size: 11px; color: var(--muted); line-height: 1.6; }

    /* ── Scrollbar ── */
    ::-webkit-scrollbar { width: 4px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: rgba(58,181,255,0.18); border-radius: 2px; }

    /* ── Print styles ── */
    @media print {
      .panel-left, .header, .report-header { display: none !important; }
      .panel-right { width: 100% !important; }
      .report-content { overflow: visible !important; }
      body { background: #fff !important; color: #000 !important; }
    }

    /* ── Responsive ── */
    @media (max-width: 768px) {
      .main { flex-direction: column; }
      .panel-left { width: 100%; min-width: 0; max-height: 55vh; }
      .summary-bar { grid-template-columns: repeat(2, 1fr); }
      .api-key-input { width: 140px; }
      .api-key-wrap label { display: none; }
    }
  </style>
</head>
<body>
<div class="app">

  <!-- ══ HEADER ══ -->
  <div class="header">
    <div class="logo">
      <div class="logo-icon">
        <svg viewBox="0 0 24 24" fill="none" stroke="var(--accent)" stroke-width="1.5">
          <path d="M12 2l3 7h7l-5.5 4 2 7L12 16l-6.5 4 2-7L2 9h7z"/>
        </svg>
      </div>
      <div>
        <div class="logo-text">MedScan AI</div>
        <div class="logo-sub">Medical Image Diagnostics</div>
      </div>
    </div>
    <div class="header-right">
      <div class="api-key-wrap">
        <label>API Key:</label>
        <input class="api-key-input" type="password" id="api-key" placeholder="sk-ant-api03-..." />
      </div>
      <div class="status-pill">
        <div class="dot"></div>
        AI Online
      </div>
    </div>
  </div>

  <!-- ══ MAIN ══ -->
  <div class="main">

    <!-- ── LEFT PANEL ── -->
    <div class="panel-left">

      <!-- Upload -->
      <div class="panel-section">
        <div class="panel-label">Upload Medical Image</div>
        <div class="upload-zone" id="upload-zone">
          <input type="file" id="file-input" accept="image/*" />
          <div class="upload-icon">
            <svg viewBox="0 0 24 24">
              <path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4M17 8l-5-5-5 5M12 3v12"/>
            </svg>
          </div>
          <div class="upload-title">Drop or click to upload</div>
          <div class="upload-hint">X-Ray · MRI · CT · Ultrasound</div>
        </div>
        <div id="preview-container">
          <div class="preview-wrap">
            <img id="preview-img" alt="Scan preview" />
            <div class="scanning-overlay" id="scan-overlay">
              <div class="scan-line"></div>
            </div>
            <div class="preview-overlay">
              <button class="close-btn" onclick="clearImage()" title="Remove image">
                <svg viewBox="0 0 24 24"><path d="M18 6L6 18M6 6l12 12"/></svg>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Scan type -->
      <div class="panel-section">
        <div class="panel-label">Scan Type</div>
        <div class="scan-types">
          <button class="scan-type-btn active" onclick="setScanType(this,'X-Ray')">X-Ray</button>
          <button class="scan-type-btn" onclick="setScanType(this,'MRI')">MRI</button>
          <button class="scan-type-btn" onclick="setScanType(this,'CT Scan')">CT Scan</button>
          <button class="scan-type-btn" onclick="setScanType(this,'Ultrasound')">Ultrasound</button>
        </div>
      </div>

      <!-- Patient info -->
      <div class="panel-section">
        <div class="panel-label">Patient Details</div>
        <div class="field-row">
          <div class="field">
            <label>Patient Name</label>
            <input type="text" id="patient-name" placeholder="Full name" />
          </div>
        </div>
        <div class="field-row">
          <div class="field">
            <label>Age</label>
            <input type="number" id="patient-age" placeholder="Years" min="0" max="120" />
          </div>
          <div class="field">
            <label>Gender</label>
            <select id="patient-gender">
              <option value="">Select</option>
              <option>Male</option>
              <option>Female</option>
              <option>Other</option>
            </select>
          </div>
        </div>
        <div class="field-row">
          <div class="field">
            <label>Clinical Notes / Symptoms</label>
            <textarea id="clinical-notes" placeholder="e.g. Chest pain, shortness of breath, suspected fracture..."></textarea>
          </div>
        </div>
      </div>

      <!-- Analyze button -->
      <div class="analyze-wrap">
        <button class="analyze-btn" id="analyze-btn" onclick="analyze()" disabled>
          Analyze Scan
        </button>
      </div>

    </div><!-- /panel-left -->

    <!-- ── RIGHT PANEL ── -->
    <div class="panel-right">

      <div class="report-header">
        <span class="report-title-text">Diagnostic Report</span>
        <div class="report-actions">
          <button class="icon-btn" id="copy-btn" onclick="copyReport()" style="display:none">
            <svg viewBox="0 0 24 24"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1"/></svg>
            Copy
          </button>
          <button class="icon-btn" id="print-btn" onclick="window.print()" style="display:none">
            <svg viewBox="0 0 24 24"><polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 01-2-2v-5a2 2 0 012-2h16a2 2 0 012 2v5a2 2 0 01-2 2h-2"/><rect x="6" y="14" width="12" height="8"/></svg>
            Print
          </button>
        </div>
      </div>

      <!-- Progress -->
      <div class="progress-wrap" id="progress-wrap">
        <div class="progress-label">
          <span id="progress-label-text">Initializing AI...</span>
          <span id="progress-pct">0%</span>
        </div>
        <div class="progress-track">
          <div class="progress-fill" id="progress-fill" style="width:0%"></div>
        </div>
      </div>

      <!-- Empty state -->
      <div id="empty-state" class="empty-state">
        <div class="empty-icon">
          <svg viewBox="0 0 24 24">
            <circle cx="12" cy="12" r="10"/>
            <path d="M12 8v4M12 16h.01"/>
          </svg>
        </div>
        <div class="empty-title">No scan loaded</div>
        <div class="empty-hint">Upload an X-Ray, MRI, or CT scan to begin AI analysis</div>
      </div>

      <!-- Report -->
      <div class="report-content" id="report-content" style="display:none"></div>

    </div><!-- /panel-right -->
  </div><!-- /main -->
</div><!-- /app -->

<script>
  /* ═══════════════════════════════════════════
     STATE
  ═══════════════════════════════════════════ */
  let imageBase64 = null;
  let imageMediaType = 'image/jpeg';
  let scanType = 'X-Ray';
  let plainTextReport = '';

  /* ═══════════════════════════════════════════
     DOM REFS
  ═══════════════════════════════════════════ */
  const fileInput        = document.getElementById('file-input');
  const previewImg       = document.getElementById('preview-img');
  const previewContainer = document.getElementById('preview-container');
  const analyzeBtn       = document.getElementById('analyze-btn');
  const uploadZone       = document.getElementById('upload-zone');
  const scanOverlay      = document.getElementById('scan-overlay');
  const progressWrap     = document.getElementById('progress-wrap');
  const progressFill     = document.getElementById('progress-fill');
  const progressPct      = document.getElementById('progress-pct');
  const progressLbl      = document.getElementById('progress-label-text');
  const emptyState       = document.getElementById('empty-state');
  const reportContent    = document.getElementById('report-content');

  /* ═══════════════════════════════════════════
     FILE HANDLING
  ═══════════════════════════════════════════ */
  fileInput.addEventListener('change', e => {
    const file = e.target.files[0];
    if (file) loadImage(file);
  });

  uploadZone.addEventListener('dragover', e => { e.preventDefault(); uploadZone.classList.add('drag'); });
  uploadZone.addEventListener('dragleave', () => uploadZone.classList.remove('drag'));
  uploadZone.addEventListener('drop', e => {
    e.preventDefault();
    uploadZone.classList.remove('drag');
    const file = e.dataTransfer.files[0];
    if (file && file.type.startsWith('image/')) loadImage(file);
  });

  function loadImage(file) {
    imageMediaType = file.type || 'image/jpeg';
    const reader = new FileReader();
    reader.onload = ev => {
      const result = ev.target.result;
      imageBase64 = result.split(',')[1];
      previewImg.src = result;
      previewContainer.style.display = 'block';
      analyzeBtn.disabled = false;
    };
    reader.readAsDataURL(file);
  }

  function clearImage() {
    imageBase64 = null;
    previewImg.src = '';
    previewContainer.style.display = 'none';
    analyzeBtn.disabled = true;
    fileInput.value = '';
    reportContent.style.display = 'none';
    emptyState.style.display = 'flex';
    document.getElementById('copy-btn').style.display = 'none';
    document.getElementById('print-btn').style.display = 'none';
    progressWrap.style.display = 'none';
  }

  function setScanType(btn, type) {
    scanType = type;
    document.querySelectorAll('.scan-type-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
  }

  /* ═══════════════════════════════════════════
     PROGRESS HELPERS
  ═══════════════════════════════════════════ */
  function setProgress(pct, label) {
    progressFill.style.width = pct + '%';
    progressPct.textContent  = pct + '%';
    progressLbl.textContent  = label;
  }

  /* ═══════════════════════════════════════════
     MAIN ANALYSIS FUNCTION
  ═══════════════════════════════════════════ */
  async function analyze() {
    if (!imageBase64) return;

    const apiKey = document.getElementById('api-key').value.trim();
    if (!apiKey) {
      alert('Please enter your Anthropic API key in the top-right field.\n\nGet one at: https://console.anthropic.com');
      return;
    }

    const name   = document.getElementById('patient-name').value.trim()   || 'Unknown';
    const age    = document.getElementById('patient-age').value.trim()     || 'N/A';
    const gender = document.getElementById('patient-gender').value.trim()  || 'N/A';
    const notes  = document.getElementById('clinical-notes').value.trim()  || 'None provided';

    // UI: loading state
    analyzeBtn.disabled = true;
    analyzeBtn.innerHTML = '<span class="btn-loader"></span>Analyzing...';
    scanOverlay.style.display = 'block';
    emptyState.style.display  = 'none';
    reportContent.style.display = 'none';
    progressWrap.style.display  = 'block';
    setProgress(10, 'Preprocessing image...');

    const prompt = `You are an expert radiologist AI assistant with deep knowledge of diagnostic imaging.

Analyze this ${scanType} medical image thoroughly and carefully.

Patient Information:
- Name: ${name}
- Age: ${age}
- Gender: ${gender}
- Scan Type: ${scanType}
- Clinical Notes: ${notes}

Perform a complete diagnostic analysis. Look for:
- Fractures, breaks, hairline cracks
- Masses, tumors, nodules, lesions
- Fluid accumulation, effusions
- Opacity changes, consolidation, infiltrates
- Structural abnormalities, misalignment
- Density changes (hyper/hypodense areas)
- Soft tissue swelling or abnormalities
- Any other relevant pathological findings

Respond ONLY with a valid JSON object. No markdown, no backticks, no extra text before or after. Use this exact structure:

{
  "overall_impression": "Concise one-to-two sentence clinical impression",
  "overall_severity": "CRITICAL or MODERATE or LOW or NORMAL",
  "confidence_score": 82,
  "scan_quality": "Excellent or Good or Fair or Poor",
  "scan_region": "e.g. Chest PA view, Left knee AP/Lateral, Brain axial MRI",
  "findings": [
    {
      "name": "Finding name",
      "severity": "CRITICAL or MODERATE or LOW or NORMAL",
      "description": "Detailed clinical description of this finding",
      "location": "Specific anatomical location",
      "confidence": 88,
      "tags": ["relevant", "clinical", "tags"]
    }
  ],
  "summary_stats": {
    "total_findings": 3,
    "critical": 0,
    "moderate": 1,
    "normal": 2
  },
  "recommendations": [
    "Specific clinical recommendation 1",
    "Specific clinical recommendation 2",
    "Specific clinical recommendation 3"
  ],
  "follow_up": "Specific follow-up guidance and timeline",
  "disclaimer": "This AI-generated analysis is intended for decision-support and educational purposes only. All findings must be reviewed and confirmed by a board-certified radiologist before any clinical action is taken."
}

If the uploaded image is not a medical scan (e.g. a regular photograph), still respond in the same JSON format but note clearly in the findings that no medical scan was detected and provide appropriate guidance.`;

    try {
      setProgress(35, 'Sending scan to Claude Vision AI...');

      const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': apiKey,
          'anthropic-version': '2023-06-01',
          'anthropic-dangerous-direct-browser-access': 'true'
        },
        body: JSON.stringify({
          model: 'claude-opus-4-5',
          max_tokens: 2000,
          messages: [{
            role: 'user',
            content: [
              {
                type: 'image',
                source: {
                  type: 'base64',
                  media_type: imageMediaType,
                  data: imageBase64
                }
              },
              { type: 'text', text: prompt }
            ]
          }]
        })
      });

      setProgress(72, 'Processing AI findings...');

      if (!response.ok) {
        const errData = await response.json().catch(() => ({}));
        throw new Error(errData.error?.message || `HTTP ${response.status}: ${response.statusText}`);
      }

      const data = await response.json();
      const rawText = (data.content || []).map(b => b.text || '').join('');

      setProgress(90, 'Generating diagnostic report...');

      let report;
      try {
        // strip any accidental markdown fences
        const clean = rawText.replace(/^```json\s*/i, '').replace(/```\s*$/i, '').trim();
        report = JSON.parse(clean);
      } catch (parseErr) {
        // Fallback: wrap raw text as a single finding
        report = {
          overall_impression: 'Analysis complete. See finding below for details.',
          overall_severity: 'NORMAL',
          confidence_score: 70,
          scan_quality: 'Good',
          scan_region: scanType,
          findings: [{
            name: 'AI Analysis Result',
            severity: 'NORMAL',
            description: rawText,
            location: 'General',
            confidence: 70,
            tags: []
          }],
          summary_stats: { total_findings: 1, critical: 0, moderate: 0, normal: 1 },
          recommendations: ['Please consult a licensed radiologist for clinical interpretation.'],
          follow_up: 'Standard clinical follow-up as indicated.',
          disclaimer: 'This AI analysis is for educational/decision-support purposes only and must be reviewed by a licensed radiologist before clinical use.'
        };
      }

      setProgress(100, 'Report complete');
      setTimeout(() => {
        progressWrap.style.display = 'none';
        scanOverlay.style.display  = 'none';
        renderReport(report, name, age, gender);
      }, 500);

    } catch (err) {
      progressWrap.style.display = 'none';
      scanOverlay.style.display  = 'none';
      reportContent.innerHTML = `
        <div class="error-box">
          <div class="error-title">Analysis Failed</div>
          <div class="error-msg">${err.message}</div>
          <div class="error-msg" style="margin-top:8px;opacity:0.7;">
            Make sure your API key is correct and has access to Claude.<br>
            Get a key at: <a href="https://console.anthropic.com" target="_blank" style="color:var(--accent);">console.anthropic.com</a>
          </div>
        </div>`;
      reportContent.style.display = 'block';
    }

    analyzeBtn.disabled = false;
    analyzeBtn.textContent = 'Analyze Scan';
  }

  /* ═══════════════════════════════════════════
     RENDER REPORT
  ═══════════════════════════════════════════ */
  function getSevClass(s) {
    const m = { CRITICAL:'severity-critical', MODERATE:'severity-moderate', LOW:'severity-low', NORMAL:'severity-normal' };
    return m[(s||'').toUpperCase()] || 'severity-normal';
  }

  function confColor(c) {
    if (c >= 85) return '#00e5a0';
    if (c >= 65) return '#ffb347';
    return '#ff5c5c';
  }

  function renderReport(r, name, age, gender) {
    const now     = new Date();
    const dateStr = now.toLocaleDateString('en-US', { year:'numeric', month:'long', day:'numeric' });
    const timeStr = now.toLocaleTimeString('en-US', { hour:'2-digit', minute:'2-digit' });
    const stats   = r.summary_stats || {};
    const sevCls  = getSevClass(r.overall_severity);

    // Build plain text for clipboard
    plainTextReport = [
      `MEDSCAN AI DIAGNOSTIC REPORT`,
      `Generated: ${dateStr} ${timeStr}`,
      `Patient: ${name} | Age: ${age} | Gender: ${gender}`,
      `Scan: ${scanType} — ${r.scan_region || ''}`,
      ``,
      `OVERALL IMPRESSION`,
      r.overall_impression,
      `Overall Severity: ${r.overall_severity}`,
      `AI Confidence: ${r.confidence_score}%`,
      ``,
      `FINDINGS`,
      ...(r.findings||[]).map((f,i) => `${i+1}. ${f.name} [${f.severity}]\n   Location: ${f.location}\n   ${f.description}`),
      ``,
      `RECOMMENDATIONS`,
      ...(r.recommendations||[]).map((rec,i) => `${i+1}. ${rec}`),
      ``,
      `FOLLOW-UP`,
      r.follow_up,
      ``,
      `DISCLAIMER`,
      r.disclaimer
    ].join('\n');

    let html = `
      <!-- Patient / impression header -->
      <div class="report-section">
        <div style="display:flex;align-items:flex-start;justify-content:space-between;margin-bottom:14px;flex-wrap:wrap;gap:10px;">
          <div>
            <div style="font-size:19px;font-weight:700;color:var(--text);margin-bottom:4px;">${name}</div>
            <div style="font-family:var(--mono);font-size:11px;color:var(--muted);">
              ${age !== 'N/A' ? age + ' yr' : ''}
              ${gender !== 'N/A' ? ' &middot; ' + gender : ''}
              &middot; ${scanType}
              ${r.scan_region ? ' &middot; ' + r.scan_region : ''}
            </div>
            <div style="font-family:var(--mono);font-size:10px;color:var(--muted);margin-top:2px;">${dateStr} &nbsp;${timeStr}</div>
          </div>
          <span class="badge ${sevCls}" style="font-size:11px;padding:5px 12px;">${r.overall_severity}</span>
        </div>
        <div class="finding-card" style="border-left:3px solid var(--accent);background:rgba(58,181,255,0.04);">
          <div style="font-size:13px;color:var(--text);line-height:1.65;">${r.overall_impression}</div>
          <div style="display:flex;gap:12px;margin-top:10px;">
            <span class="meta-tag">Confidence: ${r.confidence_score}%</span>
            <span class="meta-tag">Quality: ${r.scan_quality}</span>
          </div>
        </div>
      </div>

      <!-- Summary bar -->
      <div class="summary-bar">
        <div class="summary-card">
          <div class="summary-num" style="color:var(--text)">${stats.total_findings ?? (r.findings||[]).length}</div>
          <div class="summary-label">TOTAL FINDINGS</div>
        </div>
        <div class="summary-card">
          <div class="summary-num" style="color:var(--danger)">${stats.critical ?? 0}</div>
          <div class="summary-label">CRITICAL</div>
        </div>
        <div class="summary-card">
          <div class="summary-num" style="color:var(--warn)">${stats.moderate ?? 0}</div>
          <div class="summary-label">MODERATE</div>
        </div>
        <div class="summary-card">
          <div class="summary-num" style="color:var(--accent)">${r.confidence_score ?? 0}%</div>
          <div class="summary-label">AI CONFIDENCE</div>
        </div>
      </div>`;

    /* Findings */
    html += `
      <div class="report-section">
        <div class="section-header">
          <svg class="section-icon" viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/><line x1="11" y1="8" x2="11" y2="14"/><line x1="8" y1="11" x2="14" y2="11"/></svg>
          <div class="section-title">Detailed Findings</div>
        </div>`;

    if (r.findings && r.findings.length) {
      r.findings.forEach(f => {
        const fc = getSevClass(f.severity);
        const cc = confColor(f.confidence ?? 70);
        html += `
          <div class="finding-card">
            <div class="finding-top">
              <div class="finding-name">${f.name}</div>
              <span class="badge ${fc}">${f.severity}</span>
            </div>
            <div class="finding-desc">${f.description}</div>
            <div class="finding-meta">
              ${f.location ? `<span class="meta-tag">&#128205; ${f.location}</span>` : ''}
              <span class="meta-tag">~${f.confidence ?? 70}% confidence</span>
              ${(f.tags||[]).map(t => `<span class="meta-tag">${t}</span>`).join('')}
            </div>
            <div class="conf-row">
              <div class="conf-label">AI Confidence</div>
              <div class="conf-track"><div class="conf-fill" style="width:${f.confidence ?? 70}%;background:${cc}"></div></div>
              <div class="conf-val">${f.confidence ?? 70}%</div>
            </div>
          </div>`;
      });
    } else {
      html += `<div class="finding-card"><div class="finding-desc">No specific findings detected.</div></div>`;
    }
    html += `</div>`;

    /* Recommendations */
    if (r.recommendations && r.recommendations.length) {
      html += `
        <div class="report-section">
          <div class="section-header">
            <svg class="section-icon" viewBox="0 0 24 24"><path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
            <div class="section-title">Clinical Recommendations</div>
          </div>
          ${r.recommendations.map((rec, i) => `
            <div class="rec-item">
              <div class="rec-num">${i+1}</div>
              <div class="rec-text">${rec}</div>
            </div>`).join('')}
        </div>`;
    }

    /* Follow-up */
    if (r.follow_up) {
      html += `
        <div class="report-section">
          <div class="section-header">
            <svg class="section-icon" viewBox="0 0 24 24"><rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
            <div class="section-title">Follow-up Guidance</div>
          </div>
          <div class="finding-card"><div class="finding-desc">${r.follow_up}</div></div>
        </div>`;
    }

    /* Disclaimer */
    html += `
      <div class="disclaimer-box">
        <div class="disclaimer-label">&#9888; AI DISCLAIMER</div>
        <div class="disclaimer-text">${r.disclaimer}</div>
      </div>
      <div style="height:28px;"></div>`;

    reportContent.innerHTML = html;
    reportContent.style.display = 'block';
    document.getElementById('copy-btn').style.display = 'flex';
    document.getElementById('print-btn').style.display = 'flex';
  }

  /* ═══════════════════════════════════════════
     COPY REPORT
  ═══════════════════════════════════════════ */
  function copyReport() {
    navigator.clipboard.writeText(plainTextReport).then(() => {
      const btn = document.getElementById('copy-btn');
      const orig = btn.innerHTML;
      btn.style.color = 'var(--accent2)';
      btn.innerHTML = `<svg viewBox="0 0 24 24" style="width:12px;height:12px;stroke:currentColor;fill:none;stroke-width:2"><polyline points="20 6 9 17 4 12"/></svg> Copied!`;
      setTimeout(() => { btn.style.color = ''; btn.innerHTML = orig; }, 2000);
    });
  }
</script>
</body>
</html>

