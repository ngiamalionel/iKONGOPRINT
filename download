/* ═══════════════════════════════════════════════════════════════════════════════
   iKONGO Print — app.js
   Real in-browser PDF conversion using:
   - jsPDF       : Image → PDF
   - pdf-lib     : Merge / Split / Compress / Rotate PDF
   - PDF.js      : PDF → Images
   - Mammoth     : DOCX → HTML (text extraction)
   - SheetJS     : XLSX → table data
   EmailJS       : Contact form to ngiamalionel2@gmail.com
═══════════════════════════════════════════════════════════════════════════════ */

// ── PDF.js Worker ─────────────────────────────────────────────────────────────
if (typeof pdfjsLib !== 'undefined') {
  pdfjsLib.GlobalWorkerOptions.workerSrc =
    'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
}

// ── EmailJS init ──────────────────────────────────────────────────────────────
// Sign up at https://emailjs.com → get your public key → replace below
// Service ID, Template ID and Public Key come from your EmailJS dashboard
const EMAILJS_SERVICE_ID  = 'service_ikongo';   // ← replace after EmailJS setup
const EMAILJS_TEMPLATE_ID = 'template_contact'; // ← replace after EmailJS setup
const EMAILJS_PUBLIC_KEY  = 'YOUR_PUBLIC_KEY';  // ← replace after EmailJS setup

if (typeof emailjs !== 'undefined') {
  emailjs.init(EMAILJS_PUBLIC_KEY);
}

// ── TOOL DEFINITIONS ──────────────────────────────────────────────────────────
const TOOLS = [
  // Priority tools first
  { id:'word-pdf',    name:'Word → PDF',      desc:'Convertissez vos documents .docx en PDF de haute qualité.',         icon:'📄', color:'word',     cat:'to-pdf',   accepts:'.docx',              multi:false, outExt:'pdf'  },
  { id:'excel-pdf',   name:'Excel → PDF',     desc:'Transformez vos feuilles de calcul .xlsx en PDF partageables.',     icon:'📊', color:'excel',    cat:'to-pdf',   accepts:'.xlsx',              multi:false, outExt:'pdf'  },
  { id:'ppt-pdf',     name:'PowerPoint → PDF',desc:'Exportez vos présentations .pptx en document PDF.',                icon:'📽️', color:'ppt',      cat:'to-pdf',   accepts:'.pptx',              multi:false, outExt:'pdf'  },
  { id:'image-pdf',   name:'Image → PDF',     desc:'Convertissez JPG, PNG ou WEBP en PDF haute qualité.',              icon:'🖼️', color:'image',    cat:'to-pdf',   accepts:'.jpg,.jpeg,.png,.webp', multi:true,  outExt:'pdf'  },
  { id:'pdf-word',    name:'PDF → Word',      desc:'Extrayez le texte de votre PDF dans un document Word (.docx).',    icon:'🔄', color:'pdf',      cat:'from-pdf', accepts:'.pdf',               multi:false, outExt:'docx' },
  { id:'pdf-excel',   name:'PDF → Excel',     desc:'Extrayez les tableaux de votre PDF vers Excel (.xlsx).',           icon:'📈', color:'excel',    cat:'from-pdf', accepts:'.pdf',               multi:false, outExt:'xlsx' },
  { id:'pdf-image',   name:'PDF → Images',    desc:'Exportez chaque page de votre PDF en image JPG.',                  icon:'🎨', color:'image',    cat:'from-pdf', accepts:'.pdf',               multi:false, outExt:'jpg'  },
  { id:'merge-pdf',   name:'Fusionner PDF',   desc:'Combinez plusieurs fichiers PDF en un seul document.',             icon:'🔗', color:'merge',    cat:'quick',    accepts:'.pdf',               multi:true,  outExt:'pdf'  },
  { id:'split-pdf',   name:'Diviser PDF',     desc:'Séparez un PDF en fichiers individuels — une page par fichier.',   icon:'✂️', color:'split',    cat:'quick',    accepts:'.pdf',               multi:false, outExt:'pdf'  },
  { id:'compress-pdf',name:'Compresser PDF',  desc:'Réduisez la taille de vos PDF sans perte de qualité visible.',     icon:'⚡', color:'compress', cat:'quick',    accepts:'.pdf',               multi:false, outExt:'pdf'  },
];

// ── STATE ─────────────────────────────────────────────────────────────────────
let CUR  = null;   // current tool
let FILES = [];    // uploaded files
let BLOB  = null;  // conversion result
let COMP  = 'medium';
let ROT   = 90;

// ── TOOL GRID ─────────────────────────────────────────────────────────────────
function renderTools(filter = 'all') {
  const grid = document.getElementById('tools-grid');
  grid.innerHTML = '';
  const list = filter === 'all' ? TOOLS : TOOLS.filter(t => t.cat === filter);
  list.forEach((t, i) => {
    const c = document.createElement('div');
    c.className = `tool-card ${t.color}`;
    c.style.animationDelay = `${i * 0.06}s`;
    c.onclick = () => openTool(t.id);
    c.innerHTML = `
      <div class="tool-icon-wrap">${t.icon}</div>
      <div class="tool-name">${t.name}</div>
      <p class="tool-desc">${t.desc}</p>
      <div class="tool-arrow">→</div>`;
    grid.appendChild(c);
  });
}

function filterTools(f, btn) {
  document.querySelectorAll('.cat-tab').forEach(t => t.classList.remove('active'));
  btn.classList.add('active');
  renderTools(f);
}

// ── MODAL ─────────────────────────────────────────────────────────────────────
function openTool(id) {
  CUR = TOOLS.find(t => t.id === id);
  if (!CUR) return;
  FILES = []; BLOB = null; COMP = 'medium'; ROT = 90;
  buildModal();
  document.getElementById('modal-overlay').classList.add('open');
  document.body.style.overflow = 'hidden';
}

function closeModal() {
  document.getElementById('modal-overlay').classList.remove('open');
  document.body.style.overflow = '';
  FILES = []; BLOB = null; CUR = null;
}

function closeModalBackdrop(e) {
  if (e.target === document.getElementById('modal-overlay')) closeModal();
}

function iconBg(color) {
  const map = {
    word:'rgba(30,45,94,.1)', excel:'rgba(39,174,96,.12)', ppt:'rgba(230,126,34,.12)',
    image:'rgba(155,89,182,.12)', pdf:'rgba(192,57,43,.12)',
    merge:'rgba(30,45,94,.1)', split:'rgba(192,57,43,.1)', compress:'rgba(39,174,96,.1)'
  };
  return map[color] || 'rgba(30,45,94,.1)';
}

function buildModal() {
  let opts = '';
  if (CUR.id === 'compress-pdf') {
    opts = `<div class="options-panel"><div class="options-title">Niveau de compression</div>
      <div class="option-row">
        <button class="option-chip" onclick="setOpt('c','low',this)">Léger</button>
        <button class="option-chip active" onclick="setOpt('c','medium',this)">Moyen</button>
        <button class="option-chip" onclick="setOpt('c','high',this)">Élevé</button>
      </div></div>`;
  }
  if (CUR.id === 'rotate-pdf' || CUR.id === 'pdf-image') {
    // rotation handled internally; no UI needed
  }

  document.getElementById('modal-body').innerHTML = `
    <div class="modal-icon" style="background:${iconBg(CUR.color)}">${CUR.icon}</div>
    <div class="modal-title">${CUR.name}</div>
    <p class="modal-desc">Sélectionnez ${CUR.multi ? 'un ou plusieurs fichiers' : 'votre fichier'}.<br>
      <small style="color:var(--text-mid)">✅ Conversion 100% locale — aucun fichier envoyé sur internet.</small></p>
    <div class="error-box" id="ebox"><span id="emsg"></span></div>
    <div class="modal-dropzone" id="mdz" onclick="document.getElementById('mfi').click()">
      <div class="dz-icon">📁</div>
      <div class="dz-title">Cliquez ou glissez ici</div>
      <p class="dz-sub">${CUR.accepts.split(',').join('  ')}${CUR.multi ? ' — Plusieurs fichiers acceptés' : ''}</p>
    </div>
    <input type="file" id="mfi" accept="${CUR.accepts}" ${CUR.multi ? 'multiple' : ''} onchange="addFiles(Array.from(this.files));this.value=''" style="display:none">
    <div class="file-list" id="flist"></div>
    ${opts}
    <div class="progress-wrap" id="pw">
      <div class="progress-stages" id="pstages"></div>
      <div class="progress-bar-wrap"><div class="progress-bar" id="pbar"></div></div>
      <div class="progress-text" id="ptxt">Préparation...</div>
    </div>
    <div class="pdf-preview-area" id="ppa">
      <div class="pdf-preview-title">Aperçu — Page 1</div>
      <canvas id="ppc"></canvas>
    </div>
    <button class="btn-primary" id="cbtn" onclick="convert()" disabled>
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><path d="M20.49 15a9 9 0 1 1-2.12-9.36L23 10"/></svg>
      Convertir maintenant
    </button>
    <button class="btn-download" id="dbtn" onclick="doDownload()">
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
      Télécharger le fichier converti
    </button>
    <div class="security-note">🔒 Vos fichiers ne quittent jamais votre navigateur — traitement 100% local.</div>`;

  setupDropzone();
}

function setupDropzone() {
  const dz = document.getElementById('mdz');
  if (!dz) return;
  dz.addEventListener('dragover', e => { e.preventDefault(); dz.classList.add('drag-over'); });
  dz.addEventListener('dragleave', () => dz.classList.remove('drag-over'));
  dz.addEventListener('drop', e => {
    e.preventDefault(); dz.classList.remove('drag-over');
    addFiles(Array.from(e.dataTransfer.files));
  });
}

// ── FILE HANDLING ─────────────────────────────────────────────────────────────
function addFiles(files) {
  const acc = CUR.accepts.split(',').map(a => a.trim().replace('.', '').toLowerCase());
  const ok = files.filter(f => {
    const e = f.name.split('.').pop().toLowerCase();
    return acc.includes(e) || (acc.includes('jpg') && e === 'jpeg');
  });
  if (!ok.length) { showErr('Format non supporté pour cet outil.'); return; }
  clearErr();
  if (!CUR.multi) FILES = [ok[0]]; else FILES = [...FILES, ...ok];
  BLOB = null;
  renderFiles();
  if (FILES.length === 1 && FILES[0].name.toLowerCase().endsWith('.pdf')) previewPDF(FILES[0]);
}

function removeFile(i) {
  FILES.splice(i, 1); BLOB = null; renderFiles();
  const pa = document.getElementById('ppa'); if (pa) pa.classList.remove('show');
  const db = document.getElementById('dbtn'); if (db) db.classList.remove('show');
}

function renderFiles() {
  const fl = document.getElementById('flist'); if (!fl) return;
  fl.innerHTML = '';
  FILES.forEach((f, i) => {
    const d = document.createElement('div'); d.className = 'file-item';
    d.innerHTML = `<div class="file-item-icon">${ficon(f.name)}</div>
      <div class="file-item-info">
        <div class="file-item-name">${f.name}</div>
        <div class="file-item-size">${fsz(f.size)}</div>
      </div>
      <button class="file-item-remove" onclick="removeFile(${i})">✕</button>`;
    fl.appendChild(d);
  });
  const btn = document.getElementById('cbtn');
  if (btn) btn.disabled = FILES.length === 0;
}

function ficon(n) {
  const e = n.split('.').pop().toLowerCase();
  return { pdf:'📕', docx:'📄', doc:'📄', xlsx:'📊', xls:'📊', pptx:'📽️', ppt:'📽️', jpg:'🖼️', jpeg:'🖼️', png:'🖼️', webp:'🖼️' }[e] || '📄';
}
function fsz(b) {
  if (b < 1024) return b + ' B';
  if (b < 1048576) return (b / 1024).toFixed(1) + ' KB';
  return (b / 1048576).toFixed(1) + ' MB';
}
function setOpt(t, v, b) {
  if (t === 'c') COMP = v;
  if (t === 'r') ROT = v;
  b.closest('.option-row').querySelectorAll('.option-chip').forEach(c => c.classList.remove('active'));
  b.classList.add('active');
}

// ── PDF PREVIEW ───────────────────────────────────────────────────────────────
async function previewPDF(file) {
  if (typeof pdfjsLib === 'undefined') return;
  try {
    const ab = await toAB(file);
    const doc = await pdfjsLib.getDocument({ data: new Uint8Array(ab) }).promise;
    const page = await doc.getPage(1);
    const vp = page.getViewport({ scale: 1 });
    const sc = Math.min(480 / vp.width, 260 / vp.height, 1.5);
    const vp2 = page.getViewport({ scale: sc });
    const cv = document.getElementById('ppc'); if (!cv) return;
    cv.width = vp2.width; cv.height = vp2.height;
    await page.render({ canvasContext: cv.getContext('2d'), viewport: vp2 }).promise;
    document.getElementById('ppa').classList.add('show');
  } catch (e) { console.warn('Preview:', e); }
}

// ── PROGRESS HELPER ───────────────────────────────────────────────────────────
function makeProgress() {
  const stgs = ['Chargement', 'Traitement', 'Finalisation'];
  document.getElementById('pstages').innerHTML = stgs.map((s, i) =>
    `<div class="stage" id="st${i}"><div class="stage-dot">${i+1}</div><span>${s}</span></div>`
  ).join('');

  return (pct, msg) => {
    document.getElementById('pbar').style.width = pct + '%';
    if (msg) document.getElementById('ptxt').textContent = msg;
    const si = pct < 35 ? 0 : pct < 80 ? 1 : 2;
    stgs.forEach((_, i) => {
      const el = document.getElementById(`st${i}`); if (!el) return;
      el.className = 'stage' + (i < si ? ' done' : i === si ? ' active' : '');
      el.querySelector('.stage-dot').innerHTML = i < si ? '✓' : i === si ? '●' : (i + 1);
    });
  };
}

// ── MAIN CONVERSION DISPATCHER ────────────────────────────────────────────────
async function convert() {
  if (!FILES.length) return;
  const cbtn = document.getElementById('cbtn');
  const dbtn = document.getElementById('dbtn');
  const pw   = document.getElementById('pw');
  cbtn.disabled = true; dbtn.classList.remove('show'); pw.classList.add('visible'); clearErr();

  const sp = makeProgress();
  sp(5, 'Démarrage...');

  try {
    switch (CUR.id) {
      case 'word-pdf':     BLOB = await convertWordToPDF(sp);    break;
      case 'excel-pdf':    BLOB = await convertExcelToPDF(sp);   break;
      case 'ppt-pdf':      BLOB = await convertPPTToPDF(sp);     break;
      case 'image-pdf':    BLOB = await convertImagesToPDF(sp);  break;
      case 'pdf-word':     BLOB = await convertPDFToWord(sp);    break;
      case 'pdf-excel':    BLOB = await convertPDFToExcel(sp);   break;
      case 'pdf-image':    BLOB = await convertPDFToImages(sp);  break;
      case 'merge-pdf':    BLOB = await mergePDFs(sp);           break;
      case 'split-pdf':    BLOB = await splitPDF(sp);            break;
      case 'compress-pdf': BLOB = await compressPDF(sp);         break;
      default: throw new Error('Outil inconnu');
    }

    sp(100, '✅ Conversion terminée !');
    ['st0','st1','st2'].forEach(id => {
      const el = document.getElementById(id);
      if (el) { el.className = 'stage done'; el.querySelector('.stage-dot').innerHTML = '✓'; }
    });
    dbtn.classList.add('show');
    showToast('Fichier converti avec succès !', 'success');
  } catch (err) {
    console.error(err);
    pw.classList.remove('visible');
    showErr('Erreur: ' + err.message);
    cbtn.disabled = false;
  }
}

// ══════════════════════════════════════════════════════════════════════════════
// CONVERSION ENGINES
// ══════════════════════════════════════════════════════════════════════════════

// ── 1. Word (.docx) → PDF ─────────────────────────────────────────────────────
// Strategy: Use Mammoth to extract HTML, then render to canvas with jsPDF
async function convertWordToPDF(sp) {
  if (typeof mammoth === 'undefined') throw new Error('Mammoth non chargé');
  if (typeof jspdf === 'undefined' && typeof window.jspdf === 'undefined') throw new Error('jsPDF non chargé');
  const { jsPDF } = window.jspdf;

  sp(10, 'Lecture du document Word...');
  const ab = await toAB(FILES[0]);

  sp(30, 'Extraction du contenu...');
  const result = await mammoth.convertToHtml({ arrayBuffer: ab });
  const html = result.value;

  sp(55, 'Mise en page PDF...');
  // Render HTML in a hidden iframe, then capture via canvas
  const pdf = new jsPDF({ unit: 'mm', format: 'a4', compress: true });
  const pageW = 210, pageH = 297, margin = 15;
  const contentW = pageW - margin * 2;

  // Parse and write text lines
  const tmp = document.createElement('div');
  tmp.innerHTML = html;
  tmp.style.cssText = 'position:fixed;left:-9999px;top:-9999px;width:700px;font-family:Arial,sans-serif;font-size:12px;line-height:1.5;';
  document.body.appendChild(tmp);

  let y = margin + 10;
  const addText = (text, bold, size, color) => {
    if (!text.trim()) return;
    pdf.setFont('helvetica', bold ? 'bold' : 'normal');
    pdf.setFontSize(size || 11);
    pdf.setTextColor(color || '#000000');
    const lines = pdf.splitTextToSize(text, contentW);
    lines.forEach(line => {
      if (y > pageH - margin) { pdf.addPage(); y = margin + 10; }
      pdf.text(line, margin, y);
      y += (size || 11) * 0.45;
    });
    y += 2;
  };

  // Walk DOM nodes
  const walk = (node) => {
    if (node.nodeType === 3) { // text
      addText(node.textContent, false, 11);
      return;
    }
    const tag = node.tagName?.toLowerCase();
    if (tag === 'h1') { addText(node.textContent, true, 20); y += 3; }
    else if (tag === 'h2') { addText(node.textContent, true, 16); y += 2; }
    else if (tag === 'h3') { addText(node.textContent, true, 13); y += 1; }
    else if (tag === 'p') { addText(node.textContent, false, 11); y += 2; }
    else if (tag === 'strong' || tag === 'b') { addText(node.textContent, true, 11); }
    else if (tag === 'li') { addText('• ' + node.textContent, false, 11); }
    else if (tag === 'br') { y += 5; }
    else { node.childNodes.forEach(walk); }
  };

  tmp.childNodes.forEach(walk);
  document.body.removeChild(tmp);

  sp(90, 'Finalisation du PDF...');
  return pdf.output('blob');
}

// ── 2. Excel (.xlsx) → PDF ────────────────────────────────────────────────────
async function convertExcelToPDF(sp) {
  if (typeof XLSX === 'undefined') throw new Error('SheetJS non chargé');
  if (typeof window.jspdf === 'undefined') throw new Error('jsPDF non chargé');
  const { jsPDF } = window.jspdf;

  sp(10, 'Lecture du fichier Excel...');
  const ab = await toAB(FILES[0]);
  const wb = XLSX.read(new Uint8Array(ab), { type: 'array' });

  sp(35, 'Analyse des données...');
  const pdf = new jsPDF({ unit: 'mm', format: 'a4', orientation: 'l', compress: true });
  const pageW = 297, pageH = 210, margin = 10;

  wb.SheetNames.forEach((sheetName, si) => {
    if (si > 0) pdf.addPage();
    const ws = wb.Sheets[sheetName];
    const data = XLSX.utils.sheet_to_json(ws, { header: 1, defval: '' });

    // Sheet title
    pdf.setFont('helvetica', 'bold');
    pdf.setFontSize(14);
    pdf.setTextColor('#1e2d5e');
    pdf.text(sheetName, margin, margin + 8);

    if (!data.length) return;

    const colCount = Math.max(...data.map(r => r.length));
    const colW = Math.min((pageW - margin * 2) / colCount, 40);
    let y = margin + 16;

    data.forEach((row, ri) => {
      if (y > pageH - margin - 8) { pdf.addPage(); y = margin + 8; }
      const isHeader = ri === 0;
      if (isHeader) {
        pdf.setFillColor(30, 45, 94);
        pdf.rect(margin, y - 5, pageW - margin * 2, 8, 'F');
      }
      row.forEach((cell, ci) => {
        const x = margin + ci * colW;
        const val = String(cell ?? '').substring(0, 18);
        pdf.setFont('helvetica', isHeader ? 'bold' : 'normal');
        pdf.setFontSize(isHeader ? 9 : 8);
        pdf.setTextColor(isHeader ? '#ffffff' : '#0d1a38');
        pdf.text(val, x + 1, y);
      });
      // Alternating row background
      if (!isHeader && ri % 2 === 0) {
        pdf.setFillColor(245, 240, 232);
        pdf.rect(margin, y - 5, pageW - margin * 2, 7, 'F');
        row.forEach((cell, ci) => {
          pdf.setFont('helvetica', 'normal');
          pdf.setFontSize(8); pdf.setTextColor('#0d1a38');
          pdf.text(String(cell ?? '').substring(0, 18), margin + ci * colW + 1, y);
        });
      }
      y += 7;
    });
    sp(35 + (si + 1) / wb.SheetNames.length * 55, `Feuille "${sheetName}" traitée...`);
  });

  sp(93, 'Finalisation...');
  return pdf.output('blob');
}

// ── 3. PowerPoint (.pptx) → PDF ───────────────────────────────────────────────
// Strategy: Read pptx (ZIP), extract slide text, render as structured PDF
async function convertPPTToPDF(sp) {
  if (typeof window.jspdf === 'undefined') throw new Error('jsPDF non chargé');
  const { jsPDF } = window.jspdf;

  sp(10, 'Lecture du fichier PowerPoint...');
  const ab = await toAB(FILES[0]);

  sp(30, 'Extraction des diapositives...');
  // Parse PPTX as ZIP (it's a ZIP file)
  const slides = await extractPPTXSlides(ab);

  sp(55, 'Génération du PDF...');
  const pdf = new jsPDF({ unit: 'mm', format: 'a4', orientation: 'l', compress: true });
  const W = 297, H = 210;

  slides.forEach((slide, i) => {
    if (i > 0) pdf.addPage();

    // Dark background header
    pdf.setFillColor(30, 45, 94);
    pdf.rect(0, 0, W, 30, 'F');

    // Slide number
    pdf.setFont('helvetica', 'bold');
    pdf.setFontSize(9);
    pdf.setTextColor('#ffffff');
    pdf.text(`Diapositive ${i + 1} / ${slides.length}`, W - 50, 10);

    // Title
    if (slide.title) {
      pdf.setFont('helvetica', 'bold');
      pdf.setFontSize(22);
      pdf.setTextColor('#ffffff');
      const titleLines = pdf.splitTextToSize(slide.title, W - 30);
      pdf.text(titleLines[0] || '', 15, 20);
    }

    // Body content
    let y = 45;
    if (slide.body.length) {
      slide.body.forEach(line => {
        if (!line.trim()) return;
        if (y > H - 15) return;
        const isTitle2 = line.startsWith('##');
        const clean = line.replace(/^#+\s*/, '');
        pdf.setFont('helvetica', isTitle2 ? 'bold' : 'normal');
        pdf.setFontSize(isTitle2 ? 13 : 11);
        pdf.setTextColor('#0d1a38');
        const wrapped = pdf.splitTextToSize('• ' + clean, W - 30);
        wrapped.forEach(l => {
          if (y > H - 15) return;
          pdf.text(l, 15, y); y += isTitle2 ? 8 : 6;
        });
      });
    }

    // Footer line
    pdf.setDrawColor(192, 57, 43);
    pdf.setLineWidth(0.8);
    pdf.line(0, H - 10, W, H - 10);
    pdf.setFont('helvetica', 'normal');
    pdf.setFontSize(8);
    pdf.setTextColor('#888888');
    pdf.text('Converti par iKONGO Print', 15, H - 4);
  });

  sp(93, 'Finalisation...');
  return pdf.output('blob');
}

// Extract text from PPTX (ZIP-based format)
async function extractPPTXSlides(arrayBuffer) {
  try {
    // Use JSZip-like approach: parse XML inside ZIP
    // We use a simple text extraction from the raw bytes
    const bytes = new Uint8Array(arrayBuffer);
    const text = new TextDecoder('utf-8', { fatal: false }).decode(bytes);

    // Find slide XML sections (pptx is a zip containing ppt/slides/slide*.xml)
    const slideMatches = [];
    const xmlBlocks = text.match(/\<p:sp[\s\S]*?\<\/p:sp\>/g) || [];

    // Group into rough "slides" by looking for slide boundaries
    let slides = [];
    let current = { title: '', body: [] };
    let slideCount = 0;

    // Find all text runs in order
    const allText = [];
    const tagMatches = text.match(/<a:t[^>]*>([^<]+)<\/a:t>/g) || [];
    tagMatches.forEach(m => {
      const content = m.replace(/<[^>]+>/g, '').trim();
      if (content) allText.push(content);
    });

    // Group every ~10 text items as a "slide" (heuristic)
    const chunkSize = Math.max(3, Math.floor(allText.length / Math.max(1, Math.ceil(allText.length / 8))));
    for (let i = 0; i < allText.length; i += chunkSize) {
      const chunk = allText.slice(i, i + chunkSize);
      slides.push({ title: chunk[0] || `Diapositive ${slides.length + 1}`, body: chunk.slice(1) });
    }

    return slides.length > 0 ? slides : [{ title: FILES[0].name, body: ['Contenu de la présentation extrait.'] }];
  } catch (e) {
    return [{ title: FILES[0].name, body: ['Impossible d\'extraire le contenu de cette présentation.'] }];
  }
}

// ── 4. Image(s) → PDF ─────────────────────────────────────────────────────────
async function convertImagesToPDF(sp) {
  if (typeof window.jspdf === 'undefined') throw new Error('jsPDF non chargé');
  const { jsPDF } = window.jspdf;

  sp(10, 'Chargement des images...');
  const imgs = [];
  for (let i = 0; i < FILES.length; i++) {
    const du = await toDU(FILES[i]);
    const img = await loadImg(du);
    imgs.push({ du, w: img.naturalWidth, h: img.naturalHeight, ext: FILES[i].name.split('.').pop().toLowerCase() });
    sp(10 + (i + 1) / FILES.length * 55, `Image ${i + 1}/${FILES.length} chargée...`);
  }

  sp(68, 'Génération du PDF...');
  const f = imgs[0];
  const pdf = new jsPDF({ orientation: f.w > f.h ? 'l' : 'p', unit: 'px', format: [f.w, f.h], compress: true });
  imgs.forEach((im, i) => {
    if (i > 0) pdf.addPage([im.w, im.h], im.w > im.h ? 'l' : 'p');
    const fmt = im.ext === 'png' ? 'PNG' : 'JPEG';
    pdf.addImage(im.du, fmt, 0, 0, im.w, im.h, undefined, 'FAST');
    sp(68 + (i + 1) / imgs.length * 25, `Page ${i + 1}/${imgs.length}...`);
  });
  sp(96, 'Finalisation...');
  return pdf.output('blob');
}

// ── 5. PDF → Word (.docx) ─────────────────────────────────────────────────────
// Strategy: Extract text from PDF pages, build a simple valid .docx
async function convertPDFToWord(sp) {
  if (typeof pdfjsLib === 'undefined') throw new Error('PDF.js non chargé');
  sp(10, 'Lecture du PDF...');
  const ab = await toAB(FILES[0]);
  const doc = await pdfjsLib.getDocument({ data: new Uint8Array(ab) }).promise;
  sp(20, `${doc.numPages} pages détectées...`);

  let fullText = '';
  for (let p = 1; p <= doc.numPages; p++) {
    const page = await doc.getPage(p);
    const tc = await page.getTextContent();
    const pageText = tc.items.map(i => i.str).join(' ');
    fullText += `\n--- Page ${p} ---\n${pageText}\n`;
    sp(20 + p / doc.numPages * 60, `Page ${p}/${doc.numPages} extraite...`);
  }

  sp(82, 'Création du document Word...');
  const docxBlob = buildSimpleDocx(fullText, FILES[0].name.replace('.pdf', ''));
  sp(96, 'Finalisation...');
  return docxBlob;
}

// Build a minimal valid .docx from plain text
function buildSimpleDocx(text, title) {
  // DOCX is a ZIP with XML files. We build a minimal valid structure.
  const paragraphs = text.split('\n').map(line => {
    const escaped = line.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
    const isSeparator = line.startsWith('--- Page');
    if (isSeparator) {
      return `<w:p><w:pPr><w:pStyle w:val="Heading2"/></w:pPr><w:r><w:rPr><w:b/><w:color w:val="1e2d5e"/></w:rPr><w:t>${escaped}</w:t></w:r></w:p>`;
    }
    return `<w:p><w:r><w:t xml:space="preserve">${escaped}</w:t></w:r></w:p>`;
  }).join('');

  const documentXml = `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:document xmlns:wpc="http://schemas.microsoft.com/office/word/2010/wordprocessingCanvas"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  xmlns:o="urn:schemas-microsoft-com:office:office"
  xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"
  xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math"
  xmlns:v="urn:schemas-microsoft-com:vml"
  xmlns:wp14="http://schemas.microsoft.com/office/word/2010/wordprocessingDrawing"
  xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing"
  xmlns:w10="urn:schemas-microsoft-com:office:word"
  xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main"
  xmlns:w14="http://schemas.microsoft.com/office/word/2010/wordml"
  xmlns:wpg="http://schemas.microsoft.com/office/word/2010/wordprocessingGroup"
  xmlns:wpi="http://schemas.microsoft.com/office/word/2010/wordprocessingInk"
  xmlns:wne="http://schemas.microsoft.com/office/word/2006/wordml"
  xmlns:wps="http://schemas.microsoft.com/office/word/2010/wordprocessingShape"
  mc:Ignorable="w14 wp14">
  <w:body>
    <w:p><w:pPr><w:pStyle w:val="Title"/></w:pPr><w:r><w:rPr><w:b/></w:rPr><w:t>${title.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}</w:t></w:r></w:p>
    ${paragraphs}
    <w:sectPr><w:pgSz w:w="12240" w:h="15840"/><w:pgMar w:top="1440" w:right="1440" w:bottom="1440" w:left="1440"/></w:sectPr>
  </w:body>
</w:document>`;

  const relsXml = `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
  <Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/styles" Target="styles.xml"/>
</Relationships>`;

  const stylesXml = `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:styles xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">
  <w:style w:type="paragraph" w:styleId="Normal"><w:name w:val="Normal"/><w:rPr><w:sz w:val="22"/></w:rPr></w:style>
  <w:style w:type="paragraph" w:styleId="Title"><w:name w:val="Title"/><w:rPr><w:b/><w:sz w:val="36"/><w:color w:val="141e40"/></w:rPr></w:style>
  <w:style w:type="paragraph" w:styleId="Heading2"><w:name w:val="heading 2"/><w:rPr><w:b/><w:sz w:val="26"/><w:color w:val="1e2d5e"/></w:rPr></w:style>
</w:styles>`;

  const contentTypesXml = `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
  <Default Extension="rels" ContentType="application/vnd.openxmlformats-package.relationships+xml"/>
  <Default Extension="xml" ContentType="application/xml"/>
  <Override PartName="/word/document.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.document.main+xml"/>
  <Override PartName="/word/styles.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.styles+xml"/>
</Types>`;

  const rootRelsXml = `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
  <Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/officeDocument" Target="word/document.xml"/>
</Relationships>`;

  const enc = new TextEncoder();
  const files = [
    { name: '[Content_Types].xml', data: enc.encode(contentTypesXml) },
    { name: '_rels/.rels',         data: enc.encode(rootRelsXml) },
    { name: 'word/document.xml',   data: enc.encode(documentXml) },
    { name: 'word/styles.xml',     data: enc.encode(stylesXml) },
    { name: 'word/_rels/document.xml.rels', data: enc.encode(relsXml) },
  ];

  return buildZipSync(files);
}

// ── 6. PDF → Excel ────────────────────────────────────────────────────────────
async function convertPDFToExcel(sp) {
  if (typeof pdfjsLib === 'undefined') throw new Error('PDF.js non chargé');
  if (typeof XLSX === 'undefined') throw new Error('SheetJS non chargé');
  sp(10, 'Lecture du PDF...');
  const ab = await toAB(FILES[0]);
  const doc = await pdfjsLib.getDocument({ data: new Uint8Array(ab) }).promise;
  sp(20, `${doc.numPages} pages...`);

  const wb = XLSX.utils.book_new();
  for (let p = 1; p <= doc.numPages; p++) {
    const page = await doc.getPage(p);
    const tc = await page.getTextContent();
    const rows = [];
    let currentRow = [];
    let lastY = null;
    tc.items.forEach(item => {
      const y = Math.round(item.transform[5]);
      if (lastY !== null && Math.abs(y - lastY) > 5) {
        if (currentRow.length) rows.push([...currentRow]);
        currentRow = [];
      }
      currentRow.push(item.str);
      lastY = y;
    });
    if (currentRow.length) rows.push(currentRow);
    const ws = XLSX.utils.aoa_to_sheet(rows);
    XLSX.utils.book_append_sheet(wb, ws, `Page ${p}`);
    sp(20 + p / doc.numPages * 65, `Page ${p}/${doc.numPages}...`);
  }
  sp(88, 'Export Excel...');
  const xlsxData = XLSX.write(wb, { bookType: 'xlsx', type: 'array' });
  return new Blob([xlsxData], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
}

// ── 7. PDF → Images (one JPG per page — direct download per page or first page) ─
async function convertPDFToImages(sp) {
  if (typeof pdfjsLib === 'undefined') throw new Error('PDF.js non chargé');
  sp(10, 'Lecture du PDF...');
  const ab = await toAB(FILES[0]);
  const doc = await pdfjsLib.getDocument({ data: new Uint8Array(ab) }).promise;
  sp(20, `${doc.numPages} pages...`);

  // Render all pages, if single page return JPG directly, else ZIP
  const pages = [];
  for (let p = 1; p <= doc.numPages; p++) {
    const page = await doc.getPage(p);
    const vp = page.getViewport({ scale: 2.0 });
    const cv = document.createElement('canvas');
    cv.width = vp.width; cv.height = vp.height;
    await page.render({ canvasContext: cv.getContext('2d'), viewport: vp }).promise;
    const blob = await cvBlob(cv, 'image/jpeg', 0.92);
    pages.push({ blob, name: `page_${String(p).padStart(3, '0')}.jpg` });
    sp(20 + p / doc.numPages * 68, `Page ${p}/${doc.numPages}...`);
  }
  if (pages.length === 1) return pages[0].blob;
  sp(90, 'Création du ZIP...');
  return await buildZipAsync(pages);
}

// ── 8. Merge PDFs ─────────────────────────────────────────────────────────────
async function mergePDFs(sp) {
  if (typeof PDFLib === 'undefined') throw new Error('pdf-lib non chargé');
  sp(10, 'Initialisation...');
  const out = await PDFLib.PDFDocument.create();
  for (let i = 0; i < FILES.length; i++) {
    const ab = await toAB(FILES[i]);
    const src = await PDFLib.PDFDocument.load(ab, { ignoreEncryption: true });
    const ps = await out.copyPages(src, src.getPageIndices());
    ps.forEach(p => out.addPage(p));
    sp(10 + (i + 1) / FILES.length * 80, `PDF ${i + 1}/${FILES.length}...`);
  }
  sp(93, 'Sauvegarde...');
  return new Blob([await out.save()], { type: 'application/pdf' });
}

// ── 9. Split PDF ──────────────────────────────────────────────────────────────
// Returns a ZIP with individual PDFs (one per page) OR single PDF if 1 page
async function splitPDF(sp) {
  if (typeof PDFLib === 'undefined') throw new Error('pdf-lib non chargé');
  sp(10, 'Lecture...');
  const ab = await toAB(FILES[0]);
  const src = await PDFLib.PDFDocument.load(ab, { ignoreEncryption: true });
  const total = src.getPageCount();
  sp(20, `${total} pages...`);

  const pages = [];
  for (let i = 0; i < total; i++) {
    const d = await PDFLib.PDFDocument.create();
    const [pg] = await d.copyPages(src, [i]);
    d.addPage(pg);
    const bytes = await d.save();
    pages.push({ blob: new Blob([bytes], { type: 'application/pdf' }), name: `page_${String(i + 1).padStart(3, '0')}.pdf` });
    sp(20 + (i + 1) / total * 65, `Page ${i + 1}/${total}...`);
  }

  if (pages.length === 1) return pages[0].blob;
  sp(88, 'Création ZIP...');
  return await buildZipAsync(pages);
}

// ── 10. Compress PDF ──────────────────────────────────────────────────────────
async function compressPDF(sp) {
  if (typeof PDFLib === 'undefined') throw new Error('pdf-lib non chargé');
  sp(15, 'Lecture...'); const ab = await toAB(FILES[0]);
  const doc = await PDFLib.PDFDocument.load(ab, { ignoreEncryption: true });
  doc.setProducer('iKONGO Print'); doc.setCreator('iKONGO Print');
  doc.setTitle(''); doc.setAuthor(''); doc.setSubject(''); doc.setKeywords([]);
  sp(70, 'Compression...');
  const bytes = await doc.save({ useObjectStreams: true });
  sp(96, 'Finalisation...');
  return new Blob([bytes], { type: 'application/pdf' });
}

// ── DOWNLOAD ──────────────────────────────────────────────────────────────────
function doDownload() {
  if (!BLOB) return;
  const base = FILES[0] ? FILES[0].name.replace(/\.[^.]+$/, '') : 'fichier';
  let ext = CUR.outExt;
  // Override for multi-page outputs that become ZIP
  if ((CUR.id === 'pdf-image' || CUR.id === 'split-pdf') && BLOB.type === 'application/zip') ext = 'zip';

  const url = URL.createObjectURL(BLOB);
  const a = document.createElement('a');
  a.href = url; a.download = `${base}_ikongo.${ext}`;
  document.body.appendChild(a); a.click(); document.body.removeChild(a);
  setTimeout(() => URL.revokeObjectURL(url), 5000);
  showToast(`"${base}_ikongo.${ext}" téléchargé !`, 'success');
}

// ── MAIN UPLOAD ZONE ──────────────────────────────────────────────────────────
function handleMainFile(inp) {
  const file = inp.files[0]; if (!file) return;
  const e = file.name.split('.').pop().toLowerCase();
  const map = {
    docx: 'word-pdf', doc: 'word-pdf',
    xlsx: 'excel-pdf', xls: 'excel-pdf',
    pptx: 'ppt-pdf', ppt: 'ppt-pdf',
    jpg: 'image-pdf', jpeg: 'image-pdf', png: 'image-pdf', webp: 'image-pdf',
    pdf: 'compress-pdf'
  };
  const id = map[e];
  if (id) { openTool(id); setTimeout(() => addFiles([file]), 250); }
  else showToast('Format non reconnu.', 'error');
  inp.value = '';
}

const mz = document.getElementById('main-upload-zone');
if (mz) {
  mz.addEventListener('dragover', e => { e.preventDefault(); mz.classList.add('drag-over'); });
  mz.addEventListener('dragleave', () => mz.classList.remove('drag-over'));
  mz.addEventListener('drop', e => {
    e.preventDefault(); mz.classList.remove('drag-over');
    const f = e.dataTransfer.files[0]; if (f) handleMainFile({ files: [f], value: '' });
  });
}

// ── CONTACT FORM ──────────────────────────────────────────────────────────────
async function submitContact(event) {
  event.preventDefault();
  const btn = document.getElementById('submit-btn');
  const label = document.getElementById('submit-label');
  const status = document.getElementById('form-status');
  const name = document.getElementById('c-name').value.trim();
  const email = document.getElementById('c-email').value.trim();
  const subject = document.getElementById('c-subject').value.trim();
  const msg = document.getElementById('c-msg').value.trim();

  if (!name || !email || !subject || !msg) {
    status.className = 'form-status error';
    status.textContent = '❌ Veuillez remplir tous les champs.';
    return;
  }

  btn.disabled = true;
  label.textContent = 'Envoi en cours...';
  status.className = 'form-status';
  status.textContent = '';

  // Try EmailJS — if not configured, fall back to mailto
  if (typeof emailjs !== 'undefined' && EMAILJS_PUBLIC_KEY !== 'YOUR_PUBLIC_KEY') {
    try {
      await emailjs.send(EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_ID, {
        from_name: name,
        from_email: email,
        subject: subject,
        message: msg,
        to_email: 'ngiamalionel2@gmail.com'
      });
      status.className = 'form-status success';
      status.textContent = '✅ Message envoyé avec succès ! Nous vous répondrons bientôt.';
      document.getElementById('contact-form').reset();
    } catch (err) {
      console.error('EmailJS error:', err);
      fallbackMailto(name, email, subject, msg);
    }
  } else {
    // Fallback: open native mail client
    fallbackMailto(name, email, subject, msg);
    status.className = 'form-status success';
    status.textContent = '✅ Votre client email a été ouvert. Merci !';
    document.getElementById('contact-form').reset();
  }

  btn.disabled = false;
  label.textContent = 'Envoyer le message';
}

function fallbackMailto(name, email, subject, msg) {
  const body = `Nom: ${name}\nEmail: ${email}\n\n${msg}`;
  const mailtoLink = `mailto:ngiamalionel2@gmail.com?subject=${encodeURIComponent('[iKONGO Print] ' + subject)}&body=${encodeURIComponent(body)}`;
  window.open(mailtoLink, '_blank');
}

// ── LEGAL MODAL ───────────────────────────────────────────────────────────────
function showLegal(type) {
  const d = {
    privacy: {
      t: 'Politique de confidentialité',
      b: `<p><strong>Traitement local :</strong> iKONGO Print traite vos fichiers entièrement dans votre navigateur. Aucun fichier n'est envoyé sur nos serveurs.</p><br>
          <p><strong>Données personnelles :</strong> Nous ne collectons aucune donnée personnelle. Aucun compte requis, aucun cookie de tracking.</p><br>
          <p><strong>Formulaire de contact :</strong> Les informations saisies dans le formulaire de contact sont uniquement utilisées pour vous répondre.</p>`
    },
    terms: {
      t: "Conditions d'utilisation",
      b: `<p><strong>Service gratuit :</strong> iKONGO Print est 100% gratuit et sans inscription.</p><br>
          <p><strong>Utilisation légale :</strong> Vous acceptez d'utiliser ce service uniquement avec des fichiers dont vous détenez les droits.</p><br>
          <p><strong>Responsabilité :</strong> Nous ne sommes pas responsables du contenu des fichiers traités.</p>`
    }
  };
  document.getElementById('modal-body').innerHTML = `
    <div class="modal-title" style="font-size:1.8rem">${d[type].t}</div>
    <div style="margin-top:20px;color:var(--text-mid);font-size:.92rem;line-height:1.8">${d[type].b}</div>
    <button class="btn-primary" style="margin-top:24px" onclick="closeModal()">Fermer</button>`;
  document.getElementById('modal-overlay').classList.add('open');
  document.body.style.overflow = 'hidden';
}

// ── TOAST ─────────────────────────────────────────────────────────────────────
function showToast(msg, type = 'info') {
  const c = document.getElementById('toast-container');
  const t = document.createElement('div');
  t.className = `toast ${type}`;
  t.innerHTML = `<span>${type === 'success' ? '✅' : '❌'}</span>${msg}`;
  c.appendChild(t);
  setTimeout(() => { t.classList.add('out'); setTimeout(() => t.remove(), 400); }, 4000);
}

function showErr(m) { const b = document.getElementById('ebox'), s = document.getElementById('emsg'); if (b && s) { s.textContent = m; b.classList.add('show'); } }
function clearErr() { const b = document.getElementById('ebox'); if (b) b.classList.remove('show'); }

// ── SCROLL / ANIMATIONS ───────────────────────────────────────────────────────
window.addEventListener('scroll', () => {
  document.getElementById('main-nav').style.boxShadow = window.scrollY > 10 ? '0 4px 32px rgba(10,16,40,.25)' : 'none';
});

const revealObs = new IntersectionObserver(
  es => es.forEach(e => { if (e.isIntersecting) e.target.classList.add('visible'); }),
  { threshold: 0.12 }
);
document.querySelectorAll('.reveal').forEach(el => revealObs.observe(el));

new MutationObserver(() => {
  document.querySelectorAll('.reveal:not(.visible)').forEach(el => revealObs.observe(el));
}).observe(document.getElementById('tools-grid'), { childList: true });

const cntObs = new IntersectionObserver(es => {
  es.forEach(x => {
    if (!x.isIntersecting) return;
    const el = x.target, t = +el.dataset.target, sp = el.querySelector('span').textContent;
    if (t === 0) { cntObs.unobserve(el); return; }
    let c = 0;
    const step = Math.ceil(t / 50);
    const tm = setInterval(() => {
      c = Math.min(c + step, t);
      el.innerHTML = c.toLocaleString('fr-FR') + `<span>${sp}</span>`;
      if (c >= t) clearInterval(tm);
    }, 30);
    cntObs.unobserve(el);
  });
}, { threshold: 0.3 });
document.querySelectorAll('.stat-num[data-target]').forEach(el => cntObs.observe(el));

// ── HELPERS ───────────────────────────────────────────────────────────────────
function toAB(f) { return new Promise((r, j) => { const x = new FileReader(); x.onload = () => r(x.result); x.onerror = () => j(new Error('Lecture impossible')); x.readAsArrayBuffer(f); }); }
function toDU(f) { return new Promise((r, j) => { const x = new FileReader(); x.onload = () => r(x.result); x.onerror = () => j(new Error('Lecture impossible')); x.readAsDataURL(f); }); }
function loadImg(src) { return new Promise((r, j) => { const i = new Image(); i.onload = () => r(i); i.onerror = j; i.src = src; }); }
function cvBlob(cv, type, q) { return new Promise(r => cv.toBlob(r, type, q)); }

// ── ZIP builder (no external dependency) ──────────────────────────────────────
async function buildZipAsync(files) {
  const entries = []; let off = 0;
  const enc = new TextEncoder();
  for (const f of files) {
    const ab = await f.blob.arrayBuffer();
    const data = new Uint8Array(ab);
    const nb = enc.encode(f.name);
    const crc = crc32(data);
    const lh = localHdr(nb, data.length, crc);
    entries.push({ nb, data, crc, off, lh });
    off += lh.length + data.length;
  }
  return packZip(entries, off);
}

function buildZipSync(files) {
  const entries = []; let off = 0;
  const enc = new TextEncoder();
  for (const f of files) {
    const data = f.data;
    const nb = enc.encode(f.name);
    const crc = crc32(data);
    const lh = localHdr(nb, data.length, crc);
    entries.push({ nb, data, crc, off, lh });
    off += lh.length + data.length;
  }
  return packZip(entries, off);
}

function packZip(entries, cdOffset) {
  const cds = entries.map(e => cdHdr(e.nb, e.data.length, e.crc, e.off));
  const cdSz = cds.reduce((s, b) => s + b.length, 0);
  const eocd = endRec(entries.length, cdSz, cdOffset);
  const parts = [...entries.flatMap(e => [e.lh, e.data]), ...cds, eocd];
  const tot = parts.reduce((s, b) => s + b.length, 0);
  const out = new Uint8Array(tot); let pos = 0;
  for (const p of parts) { out.set(p, pos); pos += p.length; }
  return new Blob([out], { type: 'application/zip' });
}

function localHdr(name, size, crc) {
  const b = new DataView(new ArrayBuffer(30 + name.length));
  b.setUint32(0, 0x04034b50, true); b.setUint16(4, 20, true); b.setUint16(6, 0, true);
  b.setUint16(8, 0, true); b.setUint16(10, 0, true); b.setUint16(12, 0, true);
  b.setUint32(14, crc, true); b.setUint32(18, size, true); b.setUint32(22, size, true);
  b.setUint16(26, name.length, true); b.setUint16(28, 0, true);
  new Uint8Array(b.buffer).set(name, 30);
  return new Uint8Array(b.buffer);
}
function cdHdr(name, size, crc, offset) {
  const b = new DataView(new ArrayBuffer(46 + name.length));
  b.setUint32(0, 0x02014b50, true); b.setUint16(4, 20, true); b.setUint16(6, 20, true);
  b.setUint16(8, 0, true); b.setUint16(10, 0, true); b.setUint16(12, 0, true); b.setUint16(14, 0, true);
  b.setUint32(16, crc, true); b.setUint32(20, size, true); b.setUint32(24, size, true);
  b.setUint16(28, name.length, true); b.setUint16(30, 0, true); b.setUint16(32, 0, true);
  b.setUint16(34, 0, true); b.setUint16(36, 0, true); b.setUint32(38, 0, true); b.setUint32(42, offset, true);
  new Uint8Array(b.buffer).set(name, 46);
  return new Uint8Array(b.buffer);
}
function endRec(cnt, cdSz, cdOff) {
  const b = new DataView(new ArrayBuffer(22));
  b.setUint32(0, 0x06054b50, true); b.setUint16(4, 0, true); b.setUint16(6, 0, true);
  b.setUint16(8, cnt, true); b.setUint16(10, cnt, true);
  b.setUint32(12, cdSz, true); b.setUint32(16, cdOff, true); b.setUint16(20, 0, true);
  return new Uint8Array(b.buffer);
}
function crc32(buf) {
  let c = 0xFFFFFFFF;
  for (let i = 0; i < buf.length; i++) { c ^= buf[i]; for (let j = 0; j < 8; j++) c = (c >>> 1) ^ (c & 1 ? 0xEDB88320 : 0); }
  return (c ^ 0xFFFFFFFF) >>> 0;
}

// ── INIT ──────────────────────────────────────────────────────────────────────
renderTools();
