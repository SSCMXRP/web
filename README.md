import { useState, useEffect } from "react";

// ════════════════════════════════════════════════════════
//  CONFIG — Solo el dueño del sitio
// ════════════════════════════════════════════════════════
const OWNERS = [
  { placa: "MX-06", password: "kracccea2026", nombre: "kracccea", titulo: "Director General de Asuntos Internos" },
];
const isOwner = (placa) => OWNERS.some(o => o.placa === placa);

// ════════════════════════════════════════════════════════
//  STORAGE
// ════════════════════════════════════════════════════════
const S = {
  get: (k, def = []) => { try { const v = localStorage.getItem(k); return v ? JSON.parse(v) : def; } catch { return def; } },
  set: (k, v) => { try { localStorage.setItem(k, JSON.stringify(v)); } catch {} },
};

const initStore = () => {
  if (!localStorage.getItem("ssc_elements")) S.set("ssc_elements", []);
  if (!localStorage.getItem("ssc_admins"))   S.set("ssc_admins", []);
  if (!localStorage.getItem("ssc_reports"))  S.set("ssc_reports", []);
  if (!localStorage.getItem("ssc_pending"))  S.set("ssc_pending", []);
  if (!localStorage.getItem("ssc_adminreqs"))S.set("ssc_adminreqs", []);
};

const now = () => new Date().toLocaleString("es-MX", { timeZone: "America/Mexico_City" });

// ════════════════════════════════════════════════════════
//  ICONS
// ════════════════════════════════════════════════════════
const Ic = {
  shield:  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:18,height:18}}><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>,
  user:    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:18,height:18}}><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>,
  star:    <svg viewBox="0 0 24 24" fill="currentColor" style={{width:15,height:15}}><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>,
  trash:   <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:15,height:15}}><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/></svg>,
  plus:    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:17,height:17}}><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>,
  logout:  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:17,height:17}}><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>,
  edit:    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:15,height:15}}><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></svg>,
  check:   <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.5" style={{width:15,height:15}}><polyline points="20 6 9 17 4 12"/></svg>,
  file:    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:18,height:18}}><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>,
  users:   <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:18,height:18}}><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>,
  warn:    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:15,height:15}}><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>,
  clock:   <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:15,height:15}}><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>,
  key:     <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:15,height:15}}><path d="M21 2l-2 2m-7.61 7.61a5.5 5.5 0 1 1-7.778 7.778 5.5 5.5 0 0 1 7.777-7.777zm0 0L15.5 7.5m0 0l3 3L22 7l-3-3m-3.5 3.5L19 4"/></svg>,
  back:    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:16,height:16}}><polyline points="15 18 9 12 15 6"/></svg>,
  ban:     <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" style={{width:15,height:15}}><circle cx="12" cy="12" r="10"/><line x1="4.93" y1="4.93" x2="19.07" y2="19.07"/></svg>,
};

// ════════════════════════════════════════════════════════
//  CSS
// ════════════════════════════════════════════════════════
const CSS = `
@import url('https://fonts.googleapis.com/css2?family=Rajdhani:wght@400;500;600;700&family=Share+Tech+Mono&family=Barlow:wght@300;400;500;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}

:root{
  --navy:#08111f;--panel:#0c1628;--card:#101e35;--card2:#13243e;
  --border:#1a2e4a;--accent:#1a6aff;--green:#00c47a;--red:#ff3b55;
  --gold:#f0c040;--text:#bdd0ee;--muted:#4d6a90;--white:#deeaff;
  --orange:#ff8c42;
}

body{background:var(--navy);color:var(--text);font-family:'Barlow',sans-serif;min-height:100vh}
body::before{content:'';position:fixed;inset:0;pointer-events:none;z-index:9999;
  background:repeating-linear-gradient(0deg,transparent,transparent 3px,rgba(0,0,0,.03) 3px,rgba(0,0,0,.03) 4px)}

h1,h2,h3,h4{font-family:'Rajdhani',sans-serif;letter-spacing:.05em}
.app{display:flex;flex-direction:column;min-height:100vh}

/* ── Topbar ── */
.topbar{display:flex;align-items:center;justify-content:space-between;padding:0 24px;height:60px;
  background:var(--panel);border-bottom:1px solid var(--border);position:sticky;top:0;z-index:100}
.topbar-brand{display:flex;align-items:center;gap:10px}
.topbar-logo{height:38px;width:38px;object-fit:contain;border-radius:50%;filter:drop-shadow(0 0 8px rgba(240,192,64,.35))}
.topbar-title{font-family:'Rajdhani',sans-serif;font-size:17px;font-weight:700;color:var(--white)}
.topbar-sub{font-size:10px;color:var(--muted);letter-spacing:.12em;text-transform:uppercase}
.topbar-right{display:flex;align-items:center;gap:12px}

.placa-badge{font-family:'Share Tech Mono',monospace;font-size:11px;padding:3px 9px;border-radius:4px;
  background:rgba(26,106,255,.15);border:1px solid rgba(26,106,255,.3);color:var(--accent)}
.role-badge{font-size:10px;padding:3px 8px;border-radius:3px;font-weight:700;letter-spacing:.08em;text-transform:uppercase}
.rb-owner{background:rgba(240,192,64,.12);border:1px solid rgba(240,192,64,.3);color:var(--gold)}
.rb-admin{background:rgba(0,196,122,.12);border:1px solid rgba(0,196,122,.3);color:var(--green)}
.rb-elem {background:rgba(26,106,255,.12);border:1px solid rgba(26,106,255,.3);color:var(--accent)}

/* ── Buttons ── */
.btn{display:inline-flex;align-items:center;gap:6px;padding:8px 15px;border-radius:5px;border:none;cursor:pointer;
  font-family:'Barlow',sans-serif;font-size:13px;font-weight:500;transition:all .15s;white-space:nowrap}
.btn:disabled{opacity:.35;cursor:not-allowed}
.btn-primary{background:var(--accent);color:#fff}
.btn-primary:hover:not(:disabled){background:#3a7bff;box-shadow:0 0 16px rgba(26,106,255,.4)}
.btn-green{background:var(--green);color:#001a0e}
.btn-green:hover:not(:disabled){box-shadow:0 0 16px rgba(0,196,122,.35)}
.btn-red{background:rgba(255,59,85,.12);border:1px solid rgba(255,59,85,.28);color:var(--red)}
.btn-red:hover:not(:disabled){background:rgba(255,59,85,.22)}
.btn-ghost{background:rgba(255,255,255,.04);border:1px solid var(--border);color:var(--text)}
.btn-ghost:hover:not(:disabled){background:rgba(255,255,255,.08)}
.btn-gold{background:rgba(240,192,64,.12);border:1px solid rgba(240,192,64,.28);color:var(--gold)}
.btn-gold:hover:not(:disabled){background:rgba(240,192,64,.22)}
.btn-orange{background:rgba(255,140,66,.12);border:1px solid rgba(255,140,66,.28);color:var(--orange)}
.btn-orange:hover:not(:disabled){background:rgba(255,140,66,.22)}
.btn-sm{padding:5px 10px;font-size:12px}

/* ── Main ── */
.main{flex:1;padding:28px 28px;max-width:1200px;margin:0 auto;width:100%}

/* ── Login ── */
.login-wrap{min-height:100vh;display:flex;align-items:center;justify-content:center;
  background:var(--navy);background-image:radial-gradient(ellipse 70% 50% at 50% -10%,rgba(26,106,255,.15) 0%,transparent 70%)}
.login-box{width:370px;padding:40px 32px;background:var(--panel);border:1px solid var(--border);
  border-radius:12px;box-shadow:0 24px 60px rgba(0,0,0,.55)}
.login-logo-wrap{text-align:center;margin-bottom:24px}
.login-logo{width:80px;height:80px;object-fit:contain;margin:0 auto 10px;display:block;
  filter:drop-shadow(0 0 14px rgba(240,192,64,.4))}

/* ── Fields ── */
.field{margin-bottom:15px}
.field label{display:block;font-size:10px;font-weight:700;letter-spacing:.12em;text-transform:uppercase;color:var(--muted);margin-bottom:5px}
.field input,.field select,.field textarea{width:100%;padding:9px 12px;border-radius:5px;
  background:rgba(255,255,255,.04);border:1px solid var(--border);color:var(--white);
  font-family:'Barlow',sans-serif;font-size:13px;outline:none;transition:border .12s}
.field input:focus,.field select:focus,.field textarea:focus{border-color:var(--accent)}
.field textarea{resize:vertical;min-height:75px}
.field select option{background:var(--panel)}
.field input::placeholder{color:var(--muted)}

.err{background:rgba(255,59,85,.1);border:1px solid rgba(255,59,85,.25);color:var(--red);
  border-radius:5px;padding:8px 12px;font-size:12px;margin-bottom:14px;display:flex;align-items:center;gap:6px}
.suc{background:rgba(0,196,122,.1);border:1px solid rgba(0,196,122,.25);color:var(--green);
  border-radius:5px;padding:8px 12px;font-size:12px;margin-bottom:14px}

.divider{height:1px;background:var(--border);margin:18px 0}
.link-btn{background:none;border:none;cursor:pointer;color:var(--accent);font-size:13px;text-decoration:underline;font-family:'Barlow',sans-serif}

/* ── Cards ── */
.menu-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:16px;margin-top:20px}
.menu-card{background:var(--card);border:1px solid var(--border);border-radius:10px;padding:30px 22px;
  cursor:pointer;transition:all .18s;display:flex;flex-direction:column;align-items:center;gap:10px;text-align:center;position:relative}
.menu-card:hover{border-color:var(--accent);transform:translateY(-2px);box-shadow:0 10px 28px rgba(0,0,0,.35)}
.menu-card-icon{font-size:36px}
.menu-card-title{font-family:'Rajdhani',sans-serif;font-size:18px;font-weight:700;color:var(--white)}
.menu-card-desc{font-size:12px;color:var(--muted);line-height:1.5}
.card-badge{position:absolute;top:10px;right:10px;background:var(--red);color:#fff;
  border-radius:20px;font-size:10px;font-weight:700;padding:2px 7px;min-width:20px;text-align:center}

/* ── Section header ── */
.sec-hdr{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px;gap:10px;flex-wrap:wrap}
.sec-hdr h2{font-size:20px;color:var(--white);display:flex;align-items:center;gap:8px}

/* ── Table ── */
.tbl-wrap{background:var(--card);border:1px solid var(--border);border-radius:8px;overflow:hidden}
table{width:100%;border-collapse:collapse}
th{background:rgba(255,255,255,.03);padding:10px 14px;text-align:left;font-size:10px;font-weight:700;
  letter-spacing:.1em;text-transform:uppercase;color:var(--muted);border-bottom:1px solid var(--border)}
td{padding:11px 14px;font-size:13px;border-bottom:1px solid rgba(26,46,74,.5);vertical-align:middle}
tr:last-child td{border-bottom:none}
tr:hover td{background:rgba(255,255,255,.015)}
.mono{font-family:'Share Tech Mono',monospace;font-size:12px}

/* ── Modal ── */
.modal-ov{position:fixed;inset:0;background:rgba(0,0,0,.72);backdrop-filter:blur(4px);
  display:flex;align-items:center;justify-content:center;z-index:500;padding:16px}
.modal{background:var(--panel);border:1px solid var(--border);border-radius:10px;width:100%;
  max-width:500px;max-height:90vh;overflow-y:auto;padding:28px;box-shadow:0 28px 70px rgba(0,0,0,.6)}
.modal h3{font-size:18px;color:var(--white);margin-bottom:18px;display:flex;align-items:center;gap:8px}

/* ── Profile ── */
.profile-card{background:var(--card);border:1px solid var(--border);border-radius:10px;padding:24px}
.profile-hdr{display:flex;align-items:center;gap:18px;margin-bottom:20px;flex-wrap:wrap}
.avatar{width:64px;height:64px;border-radius:50%;background:linear-gradient(135deg,var(--accent),#0038b8);
  display:flex;align-items:center;justify-content:center;font-family:'Rajdhani',sans-serif;
  font-weight:700;font-size:22px;color:#fff;box-shadow:0 0 20px rgba(26,106,255,.3);flex-shrink:0}
.avatar-gold{background:linear-gradient(135deg,#c8950a,var(--gold));box-shadow:0 0 20px rgba(240,192,64,.3)}
.profile-name{font-size:20px;font-weight:700;color:var(--white)}
.profile-placa{font-family:'Share Tech Mono',monospace;font-size:13px;color:var(--accent)}
.profile-rank{font-size:12px;color:var(--muted);margin-top:2px}

.info-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.info-item label{font-size:10px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--muted);display:block;margin-bottom:3px}
.info-item span{font-size:13px;color:var(--white)}

/* ── Records ── */
.rec-list{display:flex;flex-direction:column;gap:9px;margin-top:10px}
.rec{background:rgba(255,255,255,.03);border:1px solid var(--border);border-radius:6px;
  padding:11px 14px;display:flex;justify-content:space-between;align-items:flex-start;gap:10px}
.rec.sancion{border-left:3px solid var(--red)}
.rec.elogio {border-left:3px solid var(--green)}
.rec.nota   {border-left:3px solid var(--accent)}
.rec-meta{font-size:11px;color:var(--muted);margin-top:3px}

.tag{font-size:10px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;padding:2px 7px;border-radius:3px}
.tag-sancion{background:rgba(255,59,85,.15);color:var(--red);border:1px solid rgba(255,59,85,.25)}
.tag-elogio {background:rgba(0,196,122,.15);color:var(--green);border:1px solid rgba(0,196,122,.25)}
.tag-nota   {background:rgba(26,106,255,.15);color:var(--accent);border:1px solid rgba(26,106,255,.25)}
.tag-pend   {background:rgba(255,140,66,.15);color:var(--orange);border:1px solid rgba(255,140,66,.25)}
.tag-aprov  {background:rgba(0,196,122,.15);color:var(--green);border:1px solid rgba(0,196,122,.25)}
.tag-rechaz {background:rgba(255,59,85,.15);color:var(--red);border:1px solid rgba(255,59,85,.25)}

/* ── Chips ── */
.chip{display:inline-flex;align-items:center;gap:4px;font-size:10px;font-weight:700;padding:3px 8px;border-radius:20px}
.chip-active {background:rgba(0,196,122,.1);color:var(--green);border:1px solid rgba(0,196,122,.22)}
.chip-baja   {background:rgba(255,59,85,.1);color:var(--red);border:1px solid rgba(255,59,85,.22)}
.chip-suspend{background:rgba(240,192,64,.1);color:var(--gold);border:1px solid rgba(240,192,64,.22)}

/* ── Tabs ── */
.tabs{display:flex;gap:3px;background:rgba(255,255,255,.04);border-radius:6px;padding:3px;margin-bottom:18px}
.tab{flex:1;padding:7px;border-radius:4px;border:none;cursor:pointer;font-family:'Barlow',sans-serif;
  font-size:12px;font-weight:500;transition:all .12s;background:none;color:var(--muted)}
.tab.active{background:var(--accent);color:#fff}

/* ── Stats ── */
.stats-row{display:grid;grid-template-columns:repeat(auto-fill,minmax(150px,1fr));gap:12px;margin-bottom:24px}
.stat{background:var(--card);border:1px solid var(--border);border-radius:8px;padding:16px 18px}
.stat-val{font-family:'Rajdhani',sans-serif;font-size:28px;font-weight:700;color:var(--white)}
.stat-lbl{font-size:10px;color:var(--muted);text-transform:uppercase;letter-spacing:.1em;margin-top:1px}

/* ── Empty ── */
.empty{text-align:center;padding:40px 20px;color:var(--muted);font-size:13px}

/* ── Toast ── */
.toast{position:fixed;bottom:24px;right:24px;z-index:999;background:var(--panel);border:1px solid var(--border);
  border-radius:7px;padding:10px 16px;font-size:13px;color:var(--white);box-shadow:0 8px 28px rgba(0,0,0,.45);
  animation:slideIn .2s ease}
@keyframes slideIn{from{transform:translateY(16px);opacity:0}to{transform:translateY(0);opacity:1}}

/* ── Search ── */
.search-wrap{position:relative;max-width:300px;margin-bottom:14px}
.search-wrap input{padding-left:32px}
.search-wrap::before{content:'🔍';position:absolute;left:9px;top:50%;transform:translateY(-50%);font-size:12px;pointer-events:none}

/* ── Pending card ── */
.pend-card{background:var(--card2);border:1px solid var(--border);border-radius:8px;padding:18px;margin-bottom:12px}
.pend-card-hdr{display:flex;align-items:center;justify-content:space-between;gap:10px;flex-wrap:wrap;margin-bottom:12px}

/* ── Admin auth list ── */
.auth-item{display:flex;align-items:center;justify-content:space-between;padding:10px 14px;
  background:var(--card2);border:1px solid var(--border);border-radius:7px;margin-bottom:8px;gap:10px;flex-wrap:wrap}

@media(max-width:640px){.main{padding:14px}.topbar{padding:0 12px}.info-grid{grid-template-columns:1fr}}
`;

// ════════════════════════════════════════════════════════
//  TOAST
// ════════════════════════════════════════════════════════
function Toast({ msg, onClose }) {
  useEffect(() => { const t = setTimeout(onClose, 3200); return () => clearTimeout(t); }, []);
  return <div className="toast">{msg}</div>;
}

// ════════════════════════════════════════════════════════
//  MODAL
// ════════════════════════════════════════════════════════
function Modal({ title, onClose, children, wide }) {
  return (
    <div className="modal-ov" onClick={e => e.target === e.currentTarget && onClose()}>
      <div className="modal" style={wide ? { maxWidth: 640 } : {}}>
        <h3>{title}</h3>
        {children}
        <div style={{ marginTop: 10 }}>
          <button className="btn btn-ghost btn-sm" onClick={onClose}>Cancelar</button>
        </div>
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════
//  LOGO (base64 placeholder — se reemplaza con la imagen real)
//  Para usar el logo real, ponemos la URL de la imagen subida
// ════════════════════════════════════════════════════════
// Usamos el logo como imagen externa en el artifact
const LOGO_URL = "https://i.imgur.com/placeholder.png"; // Se usará emoji fallback
const LogoImg = ({ cls, size = 38 }) => (
  <div style={{
    width: size, height: size, borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center",
    background: "linear-gradient(135deg,#c8950a,#f0c040)",
    boxShadow: "0 0 14px rgba(240,192,64,.4)", fontSize: size * 0.35, flexShrink: 0,
    fontFamily: "'Rajdhani',sans-serif", fontWeight: 700, color: "#08111f", letterSpacing: ".02em"
  }}>🛡️</div>
);

// ════════════════════════════════════════════════════════
//  MAIN APP
// ════════════════════════════════════════════════════════
export default function App() {
  const [session, setSession] = useState(null);
  const [view,    setView]    = useState("home");
  const [toast,   setToast]   = useState(null);

  const [elements,  setElements]  = useState([]);
  const [admins,    setAdmins]    = useState([]);
  const [reports,   setReports]   = useState([]);
  const [pending,   setPending]   = useState([]); // solicitudes de registro pendientes
  const [adminReqs, setAdminReqs] = useState([]); // placas autorizadas como admin (formato: [{placa,nombre,addedAt,addedBy}])

  useEffect(() => {
    initStore();
    setElements(S.get("ssc_elements"));
    setAdmins(S.get("ssc_admins"));
    setReports(S.get("ssc_reports"));
    setPending(S.get("ssc_pending"));
    setAdminReqs(S.get("ssc_adminreqs"));
  }, []);

  const saveEl  = d => { setElements(d);  S.set("ssc_elements",  d); };
  const saveAdm = d => { setAdmins(d);    S.set("ssc_admins",    d); };
  const saveRep = d => { setReports(d);   S.set("ssc_reports",   d); };
  const savePend= d => { setPending(d);   S.set("ssc_pending",   d); };
  const saveAReq= d => { setAdminReqs(d); S.set("ssc_adminreqs", d); };

  const toast$ = msg => setToast(msg);
  const nav    = v  => setView(v);

  // ── Login ──
  const handleLogin = ({ placa, password }) => {
    const p = placa.trim().toUpperCase();
    // Owner
    const own = OWNERS.find(o => o.placa === p && o.password === password);
    if (own) { setSession({ placa: p, role: "owner", nombre: own.nombre, titulo: own.titulo }); setView("home"); return true; }
    // Admin
    const adm = admins.find(a => a.placa === p && a.password === password);
    if (adm) { setSession({ placa: p, role: "admin", nombre: adm.nombre }); setView("home"); return true; }
    // Element (solo si está aprobado)
    const el = elements.find(e => e.placa === p && e.password === password);
    if (el) { setSession({ placa: p, role: "element", nombre: el.nombre }); setView("home"); return true; }
    return false;
  };

  // ── Registro de elemento (va a pendientes) ──
  const handleRegister = (data) => {
    const p = data.placa.toUpperCase();
    if (elements.find(e => e.placa === p)) return "Esa placa ya está registrada y activa.";
    if (pending.find(e => e.placa === p))  return "Ya tienes una solicitud pendiente con esa placa.";
    savePend([...pending, { ...data, placa: p, status: "pendiente", solicitadoEn: now(), records: [] }]);
    return null;
  };

  const pendCount = pending.filter(p => p.status === "pendiente").length;
  const isAdmin = session && (session.role === "owner" || session.role === "admin");

  if (!session) {
    return (
      <>
        <style>{CSS}</style>
        <LoginView onLogin={handleLogin} onRegister={handleRegister} />
      </>
    );
  }

  return (
    <>
      <style>{CSS}</style>
      <div className="app">
        <Topbar session={session} view={view} onLogout={() => { setSession(null); setView("home"); }} onBack={view !== "home" ? () => setView("home") : null} />
        <div className="main">
          {view === "home"      && <HomeView session={session} onNav={nav} elements={elements} admins={admins} adminReqs={adminReqs} pending={pending} pendCount={pendCount} />}
          {view === "elementos" && isAdmin && <ElementosView session={session} elements={elements} onSave={saveEl} toast={toast$} />}
          {view === "admins"    && session.role === "owner" && <AdminsView admins={admins} adminReqs={adminReqs} onSaveAdmins={saveAdm} onSaveReqs={saveAReq} toast={toast$} session={session} />}
          {view === "registros" && isAdmin && <RegistrosView pending={pending} onSave={savePend} onApprove={(el) => { saveEl([...elements, el]); }} toast={toast$} session={session} />}
          {view === "reportes"  && <ReportesView session={session} reports={reports} onSave={saveRep} toast={toast$} />}
          {view === "miperfil"  && <MiPerfilView session={session} elements={elements} />}
        </div>
        {toast && <Toast msg={toast} onClose={() => setToast(null)} />}
      </div>
    </>
  );
}

// ════════════════════════════════════════════════════════
//  TOPBAR
// ════════════════════════════════════════════════════════
function Topbar({ session, view, onLogout, onBack }) {
  const rl = { owner: "Dueño del Sitio", admin: "Administrador", element: "Elemento" };
  const rc = { owner: "rb-owner", admin: "rb-admin", element: "rb-elem" };
  return (
    <div className="topbar">
      <div className="topbar-brand">
        {onBack
          ? <button onClick={onBack} className="btn btn-ghost btn-sm" style={{ padding: "5px 10px" }}>{Ic.back} Volver</button>
          : <LogoImg size={38} />
        }
        {!onBack && (
          <div>
            <div className="topbar-title">SSC CDMX</div>
            <div className="topbar-sub">Sistema Interno — Roleplay MX</div>
          </div>
        )}
      </div>
      <div className="topbar-right">
        <span className="placa-badge">{session.placa}</span>
        <span className={`role-badge ${rc[session.role]}`}>{rl[session.role]}</span>
        <button className="btn btn-ghost btn-sm" onClick={onLogout}>{Ic.logout} Salir</button>
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════
//  LOGIN
// ════════════════════════════════════════════════════════
function LoginView({ onLogin, onRegister }) {
  const [mode, setMode] = useState("login");
  const [placa, setPlaca] = useState("");
  const [pass,  setPass]  = useState("");
  const [err,   setErr]   = useState("");
  const [ok,    setOk]    = useState("");
  const [nombre, setNombre]    = useState("");
  const [rango,  setRango]     = useState("Policía");
  const [adsc,   setAdsc]      = useState("");
  const [pass2,  setPass2]     = useState("");
  const RANGOS = ["Policía","Cabo","Sargento","Suboficial","Oficial","Subinspector","Inspector","Subcomisario","Comisario","Director"];

  const doLogin = () => {
    setErr("");
    if (!placa || !pass) { setErr("Completa todos los campos."); return; }
    if (!onLogin({ placa, password: pass })) setErr("Placa o contraseña incorrectos, o tu cuenta aún no está aprobada.");
  };
  const doReg = () => {
    setErr(""); setOk("");
    if (!placa || !nombre || !pass || !pass2) { setErr("Completa todos los campos."); return; }
    if (pass !== pass2) { setErr("Las contraseñas no coinciden."); return; }
    if (pass.length < 6) { setErr("Contraseña: mínimo 6 caracteres."); return; }
    const e = onRegister({ placa, nombre, rango, adscripcion: adsc, password: pass });
    if (e) { setErr(e); return; }
    setOk("Solicitud enviada. Espera que un administrador apruebe tu cuenta.");
    setMode("login"); setPlaca(""); setPass(""); setNombre(""); setAdsc("");
  };

  return (
    <div className="login-wrap">
      <div className="login-box">
        <div className="login-logo-wrap">
          <LogoImg size={72} />
          <div style={{ marginTop: 10, fontFamily: "'Rajdhani',sans-serif", fontSize: 22, fontWeight: 700, color: "var(--white)", textAlign: "center" }}>SSC CDMX</div>
          <div style={{ fontSize: 11, color: "var(--muted)", textAlign: "center", letterSpacing: ".1em", textTransform: "uppercase" }}>Sistema Interno · Roleplay MX</div>
        </div>
        {err && <div className="err">{Ic.warn} {err}</div>}
        {ok  && <div className="suc">{ok}</div>}

        {mode === "login" ? (<>
          <div className="field"><label>Número de Placa</label>
            <input value={placa} onChange={e => setPlaca(e.target.value.toUpperCase())} placeholder="MX-XX" onKeyDown={e => e.key === "Enter" && doLogin()} /></div>
          <div className="field"><label>Contraseña</label>
            <input type="password" value={pass} onChange={e => setPass(e.target.value)} placeholder="••••••••" onKeyDown={e => e.key === "Enter" && doLogin()} /></div>
          <button className="btn btn-primary" style={{ width: "100%", justifyContent: "center" }} onClick={doLogin}>Iniciar sesión</button>
          <div className="divider" />
          <div style={{ textAlign: "center", fontSize: 12, color: "var(--muted)" }}>
            ¿Eres nuevo?{" "}<button className="link-btn" onClick={() => { setMode("reg"); setErr(""); }}>Solicitar registro</button>
          </div>
        </>) : (<>
          <div style={{ fontSize: 12, color: "var(--muted)", marginBottom: 14, lineHeight: 1.5 }}>
            Tu solicitud quedará pendiente hasta que un administrador la apruebe.
          </div>
          <div className="field"><label>Placa</label><input value={placa} onChange={e => setPlaca(e.target.value.toUpperCase())} placeholder="MX-XX" /></div>
          <div className="field"><label>Nombre</label><input value={nombre} onChange={e => setNombre(e.target.value)} placeholder="Nombre en el RP" /></div>
          <div className="field"><label>Rango</label>
            <select value={rango} onChange={e => setRango(e.target.value)}>{RANGOS.map(r => <option key={r}>{r}</option>)}</select></div>
          <div className="field"><label>Adscripción / Sector</label><input value={adsc} onChange={e => setAdsc(e.target.value)} placeholder="Sector Cuauhtémoc..." /></div>
          <div className="field"><label>Contraseña</label><input type="password" value={pass} onChange={e => setPass(e.target.value)} placeholder="Mín. 6 caracteres" /></div>
          <div className="field"><label>Confirmar contraseña</label><input type="password" value={pass2} onChange={e => setPass2(e.target.value)} /></div>
          <button className="btn btn-green" style={{ width: "100%", justifyContent: "center" }} onClick={doReg}>{Ic.plus} Enviar solicitud</button>
          <div className="divider" />
          <div style={{ textAlign: "center", fontSize: 12, color: "var(--muted)" }}>
            ¿Ya tienes cuenta?{" "}<button className="link-btn" onClick={() => { setMode("login"); setErr(""); }}>Iniciar sesión</button>
          </div>
        </>)}
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════
//  HOME
// ════════════════════════════════════════════════════════
function HomeView({ session, onNav, elements, admins, adminReqs, pending, pendCount }) {
  const isOwn = session.role === "owner";
  const isAdm = session.role === "owner" || session.role === "admin";

  const cards = [
    isOwn && { id: "admins",    icon: "⭐", title: "Gestión de Admins",     desc: "Autoriza placas con acceso administrativo." },
    isAdm && { id: "registros", icon: "📋", title: "Registros",              desc: "Solicitudes de registro pendientes.",         badge: pendCount || null },
    isAdm && { id: "elementos", icon: "🛡️", title: "Asuntos Internos",       desc: "Expedientes, sanciones y gestión del personal." },
              { id: "reportes", icon: "📄", title: "Reportes",                desc: "Crea o consulta reportes de incidentes." },
              { id: "miperfil", icon: "👤", title: "Mi Perfil",               desc: "Consulta tu expediente personal." },
  ].filter(Boolean);

  return (
    <>
      <div>
        <h2 style={{ fontSize: 24, color: "var(--white)" }}>Bienvenido, {session.nombre}</h2>
        <p style={{ color: "var(--muted)", fontSize: 13, marginTop: 2 }}>SSC CDMX — Sistema Interno · Roleplay MX</p>
      </div>

      {isOwn && (
        <div className="stats-row" style={{ marginTop: 20 }}>
          {[
            { v: elements.length, l: "Elementos" },
            { v: admins.length,   l: "Admins" },
            { v: elements.filter(e => e.status === "activo").length, l: "Activos" },
            { v: pendCount,       l: "Pend. Aprob." },
          ].map((s, i) => (
            <div key={i} className="stat"><div className="stat-val">{s.v}</div><div className="stat-lbl">{s.l}</div></div>
          ))}
        </div>
      )}

      <div className="menu-grid">
        {cards.map(c => (
          <div key={c.id} className="menu-card" onClick={() => onNav(c.id)}>
            {c.badge ? <span className="card-badge">{c.badge}</span> : null}
            <div className="menu-card-icon">{c.icon}</div>
            <div className="menu-card-title">{c.title}</div>
            <div className="menu-card-desc">{c.desc}</div>
          </div>
        ))}
      </div>
    </>
  );
}

// ════════════════════════════════════════════════════════
//  GESTIÓN DE ADMINS (solo owner)
//  Dos secciones:
//    1. Lista de admins autorizados (adminReqs) — el owner pone la placa y ya
//    2. Lista de cuentas admin activas (con contraseña)
// ════════════════════════════════════════════════════════
function AdminsView({ admins, adminReqs, onSaveAdmins, onSaveReqs, toast, session }) {
  const [tab, setTab]     = useState("auth");
  const [modal, setModal] = useState(false);
  // auth form
  const [aPlaca,  setAPlaca]  = useState("");
  const [aNombre, setANombre] = useState("");
  // admin account form
  const [dPlaca,  setDPlaca]  = useState("");
  const [dNombre, setDNombre] = useState("");
  const [dPass,   setDPass]   = useState("");
  const [err, setErr] = useState("");

  const addAuth = () => {
    setErr("");
    const p = aPlaca.trim().toUpperCase();
    if (!p || !aNombre) { setErr("Completa placa y nombre."); return; }
    if (adminReqs.find(a => a.placa === p)) { setErr("Esa placa ya está autorizada."); return; }
    onSaveReqs([...adminReqs, { placa: p, nombre: aNombre, addedAt: now(), addedBy: session.placa }]);
    toast("Placa autorizada: " + p); setAPlaca(""); setANombre(""); setModal(false);
  };

  const removeAuth = (placa) => {
    if (!confirm("¿Quitar autorización a " + placa + "?")) return;
    onSaveReqs(adminReqs.filter(a => a.placa !== placa));
    toast("Autorización revocada.");
  };

  const addAdmin = () => {
    setErr("");
    const p = dPlaca.trim().toUpperCase();
    if (!p || !dNombre || !dPass) { setErr("Completa todos los campos."); return; }
    if (admins.find(a => a.placa === p)) { setErr("Esa placa ya tiene cuenta admin."); return; }
    onSaveAdmins([...admins, { placa: p, nombre: dNombre, password: dPass, createdAt: now() }]);
    toast("Cuenta admin creada: " + p); setDPlaca(""); setDNombre(""); setDPass("");
  };

  const removeAdmin = (placa) => {
    if (!confirm("¿Eliminar cuenta admin de " + placa + "?")) return;
    onSaveAdmins(admins.filter(a => a.placa !== placa));
    toast("Admin eliminado.");
  };

  return (
    <>
      <div className="sec-hdr">
        <h2>{Ic.star} Gestión de Administradores</h2>
        {tab === "auth" && <button className="btn btn-gold" onClick={() => { setModal(true); setErr(""); }}>{Ic.plus} Autorizar Placa</button>}
      </div>

      <div className="tabs">
        <button className={`tab ${tab === "auth" ? "active" : ""}`} onClick={() => setTab("auth")}>Placas Autorizadas ({adminReqs.length})</button>
        <button className={`tab ${tab === "cuentas" ? "active" : ""}`} onClick={() => setTab("cuentas")}>Cuentas Admin ({admins.length})</button>
      </div>

      {tab === "auth" && (<>
        <p style={{ fontSize: 12, color: "var(--muted)", marginBottom: 14 }}>
          Las placas aquí listadas están autorizadas por el Dueño del Sitio para tener acceso administrativo. Crea su cuenta en la pestaña "Cuentas Admin".
        </p>
        {adminReqs.length === 0 ? <div className="tbl-wrap"><div className="empty">No hay placas autorizadas aún.</div></div> : (
          adminReqs.map(a => (
            <div key={a.placa} className="auth-item">
              <div>
                <span className="mono" style={{ color: "var(--gold)", marginRight: 10 }}>{a.placa}</span>
                <span style={{ fontSize: 13, color: "var(--white)" }}>{a.nombre}</span>
                <div style={{ fontSize: 11, color: "var(--muted)", marginTop: 3 }}>Autorizado el {a.addedAt} · por {a.addedBy}</div>
              </div>
              <button className="btn btn-red btn-sm" onClick={() => removeAuth(a.placa)}>{Ic.ban} Revocar</button>
            </div>
          ))
        )}
      </>)}

      {tab === "cuentas" && (<>
        <div style={{ background: "var(--card2)", border: "1px solid var(--border)", borderRadius: 8, padding: 18, marginBottom: 18 }}>
          <div style={{ fontSize: 13, fontWeight: 600, color: "var(--white)", marginBottom: 14 }}>Crear cuenta de administrador</div>
          {err && <div className="err">{Ic.warn} {err}</div>}
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr auto", gap: 10, alignItems: "end" }}>
            <div className="field" style={{ margin: 0 }}><label>Placa</label><input value={dPlaca} onChange={e => setDPlaca(e.target.value)} placeholder="MX-XX" /></div>
            <div className="field" style={{ margin: 0 }}><label>Nombre</label><input value={dNombre} onChange={e => setDNombre(e.target.value)} placeholder="Nombre RP" /></div>
            <div className="field" style={{ margin: 0 }}><label>Contraseña</label><input type="password" value={dPass} onChange={e => setDPass(e.target.value)} /></div>
            <button className="btn btn-green" style={{ height: 38 }} onClick={addAdmin}>{Ic.check} Crear</button>
          </div>
        </div>
        <div className="tbl-wrap">
          {admins.length === 0 ? <div className="empty">Sin cuentas admin creadas.</div> : (
            <table>
              <thead><tr><th>Placa</th><th>Nombre</th><th>Creado</th><th></th></tr></thead>
              <tbody>
                {admins.map(a => (
                  <tr key={a.placa}>
                    <td><span className="mono">{a.placa}</span></td>
                    <td style={{ color: "var(--white)" }}>{a.nombre}</td>
                    <td style={{ fontSize: 11, color: "var(--muted)" }}>{a.createdAt}</td>
                    <td><button className="btn btn-red btn-sm" onClick={() => removeAdmin(a.placa)}>{Ic.trash} Eliminar</button></td>
                  </tr>
                ))}
              </tbody>
            </table>
          )}
        </div>
      </>)}

      {modal && (
        <Modal title={<>{Ic.key} Autorizar Placa</>} onClose={() => setModal(false)}>
          {err && <div className="err">{Ic.warn} {err}</div>}
          <p style={{ fontSize: 12, color: "var(--muted)", marginBottom: 14 }}>Agrega la placa y nombre del elemento que autorizas como administrador.</p>
          <div className="field"><label>Placa</label><input value={aPlaca} onChange={e => setAPlaca(e.target.value.toUpperCase())} placeholder="MX-XX" /></div>
          <div className="field"><label>Nombre</label><input value={aNombre} onChange={e => setANombre(e.target.value)} /></div>
          <button className="btn btn-gold" style={{ marginTop: 6 }} onClick={addAuth}>{Ic.check} Confirmar</button>
        </Modal>
      )}
    </>
  );
}

// ════════════════════════════════════════════════════════
//  REGISTROS (solicitudes pendientes)
// ════════════════════════════════════════════════════════
const RANGOS = ["Policía","Cabo","Sargento","Suboficial","Oficial","Subinspector","Inspector","Subcomisario","Comisario","Director"];

function RegistrosView({ pending, onSave, onApprove, toast, session }) {
  const [filter, setFilter] = useState("pendiente");
  const [editing, setEditing] = useState(null); // placa que se edita
  const [editData, setEditData] = useState({});

  const shown = pending.filter(p => filter === "todos" ? true : p.status === filter);

  const startEdit = (p) => { setEditing(p.placa); setEditData({ ...p }); };

  const approveReq = (p) => {
    const d = editing === p.placa ? editData : p;
    // Mueve de pending a elements
    const updated = pending.map(x => x.placa === p.placa ? { ...x, status: "aprobado", aprobadoEn: now(), aprobadoPor: session.placa } : x);
    onSave(updated);
    // Crea el elemento activo
    onApprove({ ...d, status: "activo", createdAt: now(), createdBy: session.placa, records: d.records || [] });
    toast("Registro aprobado: " + d.placa);
    setEditing(null);
  };

  const rejectReq = (p) => {
    if (!confirm("¿Rechazar solicitud de " + p.placa + "?")) return;
    onSave(pending.map(x => x.placa === p.placa ? { ...x, status: "rechazado", rechazadoEn: now() } : x));
    toast("Solicitud rechazada.");
  };

  const statusTag = { pendiente: "tag-pend", aprobado: "tag-aprov", rechazado: "tag-rechaz" };

  return (
    <>
      <div className="sec-hdr">
        <h2>{Ic.clock} Registros de Personal</h2>
        <div style={{ display: "flex", gap: 6 }}>
          {["pendiente","aprobado","rechazado","todos"].map(f => (
            <button key={f} className={`btn btn-sm ${filter === f ? "btn-primary" : "btn-ghost"}`} onClick={() => setFilter(f)} style={{ textTransform: "capitalize" }}>{f}</button>
          ))}
        </div>
      </div>

      {shown.length === 0 ? (
        <div className="tbl-wrap"><div className="empty">No hay solicitudes en este estado.</div></div>
      ) : (
        shown.map(p => (
          <div key={p.placa} className="pend-card">
            <div className="pend-card-hdr">
              <div style={{ display: "flex", alignItems: "center", gap: 10, flexWrap: "wrap" }}>
                <span className="mono" style={{ color: "var(--gold)" }}>{p.placa}</span>
                <span style={{ color: "var(--white)", fontWeight: 600 }}>{p.nombre}</span>
                <span className={`tag ${statusTag[p.status]}`}>{p.status}</span>
              </div>
              <div style={{ fontSize: 11, color: "var(--muted)" }}>Solicitud: {p.solicitadoEn}</div>
            </div>

            {editing === p.placa ? (
              // Edición inline
              <div>
                <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10 }}>
                  <div className="field" style={{ margin: 0 }}><label>Placa</label><input value={editData.placa} onChange={e => setEditData({ ...editData, placa: e.target.value.toUpperCase() })} /></div>
                  <div className="field" style={{ margin: 0 }}><label>Nombre</label><input value={editData.nombre} onChange={e => setEditData({ ...editData, nombre: e.target.value })} /></div>
                  <div className="field" style={{ margin: 0 }}><label>Rango</label>
                    <select value={editData.rango} onChange={e => setEditData({ ...editData, rango: e.target.value })}>{RANGOS.map(r => <option key={r}>{r}</option>)}</select></div>
                  <div className="field" style={{ margin: 0 }}><label>Adscripción</label><input value={editData.adscripcion || ""} onChange={e => setEditData({ ...editData, adscripcion: e.target.value })} /></div>
                </div>
                <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
                  <button className="btn btn-green btn-sm" onClick={() => approveReq(p)}>{Ic.check} Aprobar</button>
                  <button className="btn btn-ghost btn-sm" onClick={() => setEditing(null)}>Cancelar</button>
                </div>
              </div>
            ) : (
              <div>
                <div className="info-grid" style={{ marginBottom: 12 }}>
                  <div className="info-item"><label>Rango</label><span>{p.rango}</span></div>
                  <div className="info-item"><label>Adscripción</label><span>{p.adscripcion || "—"}</span></div>
                </div>
                {p.status === "pendiente" && (
                  <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
                    <button className="btn btn-primary btn-sm" onClick={() => startEdit(p)}>{Ic.edit} Revisar y aprobar</button>
                    <button className="btn btn-green btn-sm" onClick={() => approveReq(p)}>{Ic.check} Aprobar directo</button>
                    <button className="btn btn-red btn-sm" onClick={() => rejectReq(p)}>{Ic.ban} Rechazar</button>
                  </div>
                )}
                {p.status !== "pendiente" && (
                  <div style={{ fontSize: 11, color: "var(--muted)" }}>
                    {p.status === "aprobado"  ? `Aprobado el ${p.aprobadoEn} por ${p.aprobadoPor}` : `Rechazado el ${p.rechazadoEn}`}
                  </div>
                )}
              </div>
            )}
          </div>
        ))
      )}
    </>
  );
}

// ════════════════════════════════════════════════════════
//  ELEMENTOS (Asuntos Internos)
// ════════════════════════════════════════════════════════
function ElementosView({ session, elements, onSave, toast }) {
  const [search, setSearch] = useState("");
  const [selected, setSelected] = useState(null);
  const [modal, setModal] = useState(false);

  const filtered = elements.filter(e =>
    e.placa.includes(search.toUpperCase()) ||
    e.nombre.toLowerCase().includes(search.toLowerCase())
  );

  const stCls = { activo: "chip-active", baja: "chip-baja", suspendido: "chip-suspend" };
  const stLbl = { activo: "Activo", baja: "Baja", suspendido: "Suspendido" };

  if (selected) {
    return (
      <ExpedienteView
        element={selected}
        session={session}
        onBack={() => setSelected(null)}
        onSave={(upd) => { const d = elements.map(e => e.placa === upd.placa ? upd : e); onSave(d); setSelected(upd); toast("Guardado."); }}
        onDelete={() => {
          if (!confirm("¿Eliminar elemento " + selected.placa + "? Su acceso quedará bloqueado.")) return;
          onSave(elements.filter(e => e.placa !== selected.placa));
          setSelected(null); toast("Elemento eliminado. Su cuenta ya no tiene acceso.");
        }}
      />
    );
  }

  return (
    <>
      <div className="sec-hdr">
        <h2>{Ic.users} Personal SSC</h2>
        <button className="btn btn-primary" onClick={() => setModal(true)}>{Ic.plus} Crear Elemento</button>
      </div>
      <div className="search-wrap">
        <input value={search} onChange={e => setSearch(e.target.value)} placeholder="Buscar placa o nombre..." />
      </div>
      <div className="tbl-wrap">
        {filtered.length === 0 ? <div className="empty">Sin resultados.</div> : (
          <table>
            <thead><tr><th>Placa</th><th>Nombre</th><th>Rango</th><th>Adscripción</th><th>Estado</th><th>Registros</th><th></th></tr></thead>
            <tbody>
              {filtered.map(e => (
                <tr key={e.placa}>
                  <td><span className="mono">{e.placa}</span></td>
                  <td style={{ color: "var(--white)", fontWeight: 500 }}>{e.nombre}</td>
                  <td style={{ fontSize: 12, color: "var(--muted)" }}>{e.rango}</td>
                  <td style={{ fontSize: 12 }}>{e.adscripcion || "—"}</td>
                  <td><span className={`chip ${stCls[e.status] || "chip-active"}`}>⬤ {stLbl[e.status] || "Activo"}</span></td>
                  <td style={{ fontSize: 12, color: "var(--muted)" }}>{e.records?.length || 0}</td>
                  <td><button className="btn btn-ghost btn-sm" onClick={() => setSelected(e)}>{Ic.file} Ver</button></td>
                </tr>
              ))}
            </tbody>
          </table>
        )}
      </div>

      {modal && (
        <Modal title={<>{Ic.plus} Crear Elemento</>} onClose={() => setModal(false)}>
          <CrearElementoForm
            onCancel={() => setModal(false)}
            onCreate={(data) => {
              if (elements.find(e => e.placa === data.placa)) return "Placa ya registrada.";
              onSave([...elements, { ...data, records: [], createdAt: now(), createdBy: session.placa, status: "activo" }]);
              toast("Elemento creado: " + data.placa); setModal(false); return null;
            }}
          />
        </Modal>
      )}
    </>
  );
}

function CrearElementoForm({ onCreate, onCancel }) {
  const [d, setD] = useState({ placa: "", nombre: "", rango: "Policía", adscripcion: "", password: "" });
  const [err, setErr] = useState("");
  const set = (k, v) => setD(p => ({ ...p, [k]: v }));
  const submit = () => {
    if (!d.placa || !d.nombre || !d.password) { setErr("Placa, nombre y contraseña son obligatorios."); return; }
    const e = onCreate({ ...d, placa: d.placa.toUpperCase() });
    if (e) setErr(e);
  };
  return (<>
    {err && <div className="err">{Ic.warn} {err}</div>}
    <div className="field"><label>Placa</label><input value={d.placa} onChange={e => set("placa", e.target.value)} placeholder="MX-XX" /></div>
    <div className="field"><label>Nombre</label><input value={d.nombre} onChange={e => set("nombre", e.target.value)} /></div>
    <div className="field"><label>Rango</label><select value={d.rango} onChange={e => set("rango", e.target.value)}>{RANGOS.map(r => <option key={r}>{r}</option>)}</select></div>
    <div className="field"><label>Adscripción</label><input value={d.adscripcion} onChange={e => set("adscripcion", e.target.value)} /></div>
    <div className="field"><label>Contraseña inicial</label><input type="password" value={d.password} onChange={e => set("password", e.target.value)} /></div>
    <button className="btn btn-primary" style={{ marginTop: 6 }} onClick={submit}>{Ic.check} Crear</button>
  </>);
}

// ════════════════════════════════════════════════════════
//  EXPEDIENTE
// ════════════════════════════════════════════════════════
function ExpedienteView({ element, session, onBack, onSave, onDelete }) {
  const [tab, setTab] = useState("info");
  const [el, setEl] = useState(element);
  // Sanción modal
  const [recModal, setRecModal] = useState(false);
  const [rType, setRType] = useState("sancion");
  const [rDesc, setRDesc] = useState("");
  const [rAutor, setRAutor] = useState(session.nombre + " (" + session.placa + ")");

  const update = (upd) => { setEl(upd); onSave(upd); };

  const addRecord = () => {
    if (!rDesc) return;
    update({ ...el, records: [...(el.records || []), { id: Date.now(), type: rType, desc: rDesc, autor: rAutor, fecha: now() }] });
    setRecModal(false); setRDesc("");
  };
  const delRecord = (id) => {
    if (!confirm("¿Eliminar este registro?")) return;
    update({ ...el, records: el.records.filter(r => r.id !== id) });
  };

  const tagCls = { sancion: "tag-sancion", elogio: "tag-elogio", nota: "tag-nota" };
  const recCls = { sancion: "sancion", elogio: "elogio", nota: "nota" };
  const stCls  = { activo: "chip-active", baja: "chip-baja", suspendido: "chip-suspend" };
  const sanciones = (el.records || []).filter(r => r.type === "sancion").length;

  return (
    <>
      <button className="btn btn-ghost btn-sm" style={{ marginBottom: 14 }} onClick={onBack}>{Ic.back} Volver al listado</button>
      <div className="profile-card">
        <div className="profile-hdr">
          <div className="avatar">{el.nombre?.[0] || "?"}</div>
          <div style={{ flex: 1 }}>
            <div className="profile-name">{el.nombre}</div>
            <div className="profile-placa">{el.placa}</div>
            <div className="profile-rank">{el.rango} · {el.adscripcion || "Sin adscripción"}</div>
            <div style={{ marginTop: 6, display: "flex", gap: 8, flexWrap: "wrap", alignItems: "center" }}>
              <span className={`chip ${stCls[el.status] || "chip-active"}`}>⬤ {el.status || "Activo"}</span>
              {sanciones > 0 && <span className="chip chip-baja">⚠ {sanciones} sanción{sanciones > 1 ? "es" : ""}</span>}
            </div>
          </div>
          <div style={{ display: "flex", gap: 7, flexWrap: "wrap", alignItems: "flex-start", flexShrink: 0 }}>
            <select value={el.status || "activo"} onChange={e => update({ ...el, status: e.target.value })}
              style={{ padding: "5px 8px", borderRadius: 5, background: "var(--card2)", border: "1px solid var(--border)", color: "var(--text)", fontSize: 12, fontFamily: "Barlow,sans-serif", cursor: "pointer" }}>
              <option value="activo">Activo</option>
              <option value="suspendido">Suspendido</option>
              <option value="baja">Baja</option>
            </select>
            <button className="btn btn-red btn-sm" onClick={onDelete}>{Ic.trash} Eliminar</button>
          </div>
        </div>

        <div className="tabs">
          {[["info","📋 Info"],["sanciones",`⚠ Sanciones (${sanciones})`],["registros",`📁 Todos los registros (${el.records?.length||0})`]].map(([t,l])=>(
            <button key={t} className={`tab ${tab===t?"active":""}`} onClick={()=>setTab(t)}>{l}</button>
          ))}
        </div>

        {tab === "info"      && <EditableInfo el={el} onSave={update} />}
        {tab === "sanciones" && <RecordsPanel records={(el.records||[]).filter(r=>r.type==="sancion")} onDel={delRecord} onAdd={()=>{setRType("sancion");setRecModal(true)}} tagCls={tagCls} recCls={recCls} label="sanción" />}
        {tab === "registros" && <RecordsPanel records={el.records||[]} onDel={delRecord} onAdd={()=>setRecModal(true)} tagCls={tagCls} recCls={recCls} showAdd />}
      </div>

      {recModal && (
        <Modal title={<>{Ic.file} Nuevo Registro</>} onClose={() => setRecModal(false)}>
          <div className="field"><label>Tipo</label>
            <select value={rType} onChange={e => setRType(e.target.value)}>
              <option value="sancion">Sanción Administrativa</option>
              <option value="elogio">Elogio / Reconocimiento</option>
              <option value="nota">Nota Interna</option>
            </select>
          </div>
          <div className="field"><label>Descripción</label><textarea value={rDesc} onChange={e => setRDesc(e.target.value)} placeholder="Detalla el registro..." /></div>
          <div className="field"><label>Registrado por</label><input value={rAutor} onChange={e => setRAutor(e.target.value)} /></div>
          <button className="btn btn-primary" style={{ marginTop: 6 }} onClick={addRecord}>{Ic.check} Guardar</button>
        </Modal>
      )}
    </>
  );
}

function RecordsPanel({ records, onDel, onAdd, tagCls, recCls, label = "registro", showAdd = false }) {
  return (<>
    <div style={{ marginBottom: 12 }}>
      <button className="btn btn-primary btn-sm" onClick={onAdd}>{Ic.plus} Agregar {label}</button>
    </div>
    {records.length === 0
      ? <div className="empty">Sin {label}s en el expediente.</div>
      : <div className="rec-list">
          {[...records].reverse().map(r => (
            <div key={r.id} className={`rec ${recCls[r.type]}`}>
              <div style={{ flex: 1 }}>
                <div style={{ display: "flex", gap: 7, alignItems: "center", marginBottom: 5 }}>
                  <span className={`tag ${tagCls[r.type]}`}>{r.type}</span>
                </div>
                <div style={{ fontSize: 13, color: "var(--white)", lineHeight: 1.45 }}>{r.desc}</div>
                <div className="rec-meta">Por: {r.autor} — {r.fecha}</div>
              </div>
              <button className="btn btn-red btn-sm" style={{ flexShrink: 0 }} onClick={() => onDel(r.id)}>{Ic.trash}</button>
            </div>
          ))}
        </div>
    }
  </>);
}

function EditableInfo({ el, onSave }) {
  const [editing, setEditing] = useState(false);
  const [d, setD] = useState({ ...el });
  const set = (k, v) => setD(p => ({ ...p, [k]: v }));
  const save = () => { onSave(d); setEditing(false); };
  const F = ({ label, k, opts }) => (
    <div className="info-item">
      <label>{label}</label>
      {editing
        ? opts
          ? <select value={d[k]||""} onChange={e=>set(k,e.target.value)} style={{padding:"7px 10px",borderRadius:5,background:"var(--card2)",border:"1px solid var(--border)",color:"var(--white)",fontSize:12,fontFamily:"Barlow,sans-serif",width:"100%"}}>{opts.map(o=><option key={o}>{o}</option>)}</select>
          : <input value={d[k]||""} onChange={e=>set(k,e.target.value)} style={{padding:"7px 10px",borderRadius:5,background:"var(--card2)",border:"1px solid var(--border)",color:"var(--white)",fontSize:12,fontFamily:"Barlow,sans-serif",width:"100%",outline:"none"}} />
        : <span>{el[k]||"—"}</span>
      }
    </div>
  );
  return (<>
    <div style={{ marginBottom: 14 }}>
      {editing
        ? <><button className="btn btn-green btn-sm" onClick={save}>{Ic.check} Guardar</button>{" "}<button className="btn btn-ghost btn-sm" onClick={()=>setEditing(false)}>Cancelar</button></>
        : <button className="btn btn-ghost btn-sm" onClick={()=>setEditing(true)}>{Ic.edit} Editar</button>
      }
    </div>
    <div className="info-grid">
      <F label="Nombre"       k="nombre" />
      <F label="Placa"        k="placa" />
      <F label="Rango"        k="rango"  opts={RANGOS} />
      <F label="Adscripción"  k="adscripcion" />
      <F label="Turno"        k="turno"  opts={["Matutino","Vespertino","Nocturno","Mixto"]} />
      <F label="CURP / ID"    k="curp" />
    </div>
    {editing && <div className="field" style={{ marginTop: 14 }}><label>Notas generales</label><textarea value={d.notas||""} onChange={e=>set("notas",e.target.value)} /></div>}
    {!editing && el.notas && <div style={{ marginTop: 14 }}><div style={{ fontSize: 10, fontWeight: 700, letterSpacing: ".1em", textTransform: "uppercase", color: "var(--muted)", marginBottom: 5 }}>Notas generales</div><p style={{ fontSize: 13, lineHeight: 1.5 }}>{el.notas}</p></div>}
  </>);
}

// ════════════════════════════════════════════════════════
//  MI PERFIL
// ════════════════════════════════════════════════════════
function MiPerfilView({ session, elements }) {
  const isOwn = session.role === "owner";
  const own = OWNERS.find(o => o.placa === session.placa);
  const el = elements.find(e => e.placa === session.placa);
  const stCls = { activo: "chip-active", baja: "chip-baja", suspendido: "chip-suspend" };
  const tagCls = { sancion: "tag-sancion", elogio: "tag-elogio", nota: "tag-nota" };
  const recCls = { sancion: "sancion", elogio: "elogio", nota: "nota" };

  if (isOwn) return (
    <div className="profile-card">
      <div className="profile-hdr">
        <div className="avatar avatar-gold">{Ic.star}</div>
        <div>
          <div className="profile-name">{own?.nombre || session.nombre}</div>
          <div className="profile-placa">{session.placa}</div>
          <div className="profile-rank">{own?.titulo || "Dueño del Sitio"}</div>
          <div style={{ marginTop: 6 }}><span className="role-badge rb-owner">{Ic.star} Dueño del Sitio</span></div>
        </div>
      </div>
      <p style={{ fontSize: 13, color: "var(--muted)" }}>Cuenta con acceso total al sistema. Gestiona admins, registros y expedientes desde el menú principal.</p>
    </div>
  );

  if (!el) return (
    <div className="profile-card">
      <div className="empty">Tu perfil no está en el sistema o aún está pendiente de aprobación.</div>
    </div>
  );

  return (
    <div className="profile-card">
      <div className="profile-hdr">
        <div className="avatar">{el.nombre?.[0]}</div>
        <div>
          <div className="profile-name">{el.nombre}</div>
          <div className="profile-placa">{el.placa}</div>
          <div className="profile-rank">{el.rango} · {el.adscripcion || "Sin adscripción"}</div>
          <div style={{ marginTop: 6 }}><span className={`chip ${stCls[el.status]||"chip-active"}`}>⬤ {el.status||"Activo"}</span></div>
        </div>
      </div>
      <div className="info-grid" style={{ marginBottom: 18 }}>
        <div className="info-item"><label>Turno</label><span>{el.turno||"—"}</span></div>
        <div className="info-item"><label>CURP / ID</label><span>{el.curp||"—"}</span></div>
      </div>
      <div style={{ fontSize: 11, fontWeight: 700, letterSpacing: ".1em", textTransform: "uppercase", color: "var(--muted)", marginBottom: 10 }}>
        Mi Expediente ({el.records?.length||0} registros)
      </div>
      {(!el.records||el.records.length===0)
        ? <div className="empty">Sin registros en tu expediente.</div>
        : <div className="rec-list">
            {[...el.records].reverse().map(r=>(
              <div key={r.id} className={`rec ${recCls[r.type]}`}>
                <div>
                  <div style={{display:"flex",gap:7,alignItems:"center",marginBottom:5}}>
                    <span className={`tag ${tagCls[r.type]}`}>{r.type}</span>
                  </div>
                  <div style={{fontSize:13,color:"var(--white)",lineHeight:1.45}}>{r.desc}</div>
                  <div className="rec-meta">Por: {r.autor} — {r.fecha}</div>
                </div>
              </div>
            ))}
          </div>
      }
    </div>
  );
}

// ════════════════════════════════════════════════════════
//  REPORTES
// ════════════════════════════════════════════════════════
function ReportesView({ session, reports, onSave, toast }) {
  const [modal, setModal] = useState(false);
  const [tipo, setTipo] = useState("Incidente");
  const [desc, setDesc] = useState("");
  const [inv,  setInv]  = useState("");
  const isAdmin = session.role === "owner" || session.role === "admin";
  const shown = isAdmin ? reports : reports.filter(r => r.autorPlaca === session.placa);

  const create = () => {
    if (!desc) return;
    onSave([{ id: Date.now(), tipo, desc, involucrados: inv, autorPlaca: session.placa, autorNombre: session.nombre, fecha: now() }, ...reports]);
    toast("Reporte creado."); setModal(false); setDesc(""); setInv("");
  };
  const del = (id) => { if (!confirm("¿Eliminar reporte?")) return; onSave(reports.filter(r => r.id !== id)); toast("Eliminado."); };
  const canDel = r => isAdmin || r.autorPlaca === session.placa;

  return (<>
    <div className="sec-hdr">
      <h2>{Ic.file} Reportes</h2>
      <button className="btn btn-primary" onClick={() => setModal(true)}>{Ic.plus} Nuevo Reporte</button>
    </div>
    {shown.length === 0
      ? <div className="tbl-wrap"><div className="empty">Sin reportes aún.</div></div>
      : <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
          {shown.map(r => (
            <div key={r.id} className="pend-card">
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", gap: 10 }}>
                <div style={{ flex: 1 }}>
                  <div style={{ display: "flex", gap: 8, alignItems: "center", marginBottom: 6, flexWrap: "wrap" }}>
                    <span className="tag tag-nota">{r.tipo}</span>
                    <span style={{ fontSize: 11, color: "var(--muted)" }}>{r.fecha}</span>
                  </div>
                  <p style={{ fontSize: 13, color: "var(--white)", lineHeight: 1.5 }}>{r.desc}</p>
                  {r.involucrados && <div className="rec-meta" style={{ marginTop: 5 }}>Involucrados: {r.involucrados}</div>}
                  <div className="rec-meta">Por: {r.autorNombre} ({r.autorPlaca})</div>
                </div>
                {canDel(r) && <button className="btn btn-red btn-sm" onClick={() => del(r.id)}>{Ic.trash}</button>}
              </div>
            </div>
          ))}
        </div>
    }
    {modal && (
      <Modal title={<>{Ic.file} Nuevo Reporte</>} onClose={() => setModal(false)}>
        <div className="field"><label>Tipo</label>
          <select value={tipo} onChange={e => setTipo(e.target.value)}>
            {["Incidente","Informe de patrullaje","Queja ciudadana","Uso de fuerza","Accidente","Otro"].map(t=><option key={t}>{t}</option>)}
          </select>
        </div>
        <div className="field"><label>Descripción</label><textarea value={desc} onChange={e=>setDesc(e.target.value)} placeholder="Detalla el reporte..." style={{minHeight:90}} /></div>
        <div className="field"><label>Elementos involucrados (opcional)</label><input value={inv} onChange={e=>setInv(e.target.value)} placeholder="Placas o nombres" /></div>
        <button className="btn btn-primary" style={{ marginTop: 6 }} onClick={create}>{Ic.check} Crear Reporte</button>
      </Modal>
    )}
  </>);
}
