[cumulative_planning_search.html](https://github.com/user-attachments/files/26280264/cumulative_planning_search.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cumulative Planning Search — UK Residential Monitor</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Barlow+Condensed:wght@300;400;500;600&family=Courier+Prime:ital,wght@0,400;0,700;1,400&family=Barlow:wght@300;400;500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet-draw@1.0.4/dist/leaflet.draw.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-draw@1.0.4/dist/leaflet.draw.js"></script>
<script src="https://unpkg.com/@turf/turf@6.5.0/turf.min.js"></script>
<style>
/* ─────────────────────────────────────────────
   DESIGN: Survey document / planning inspector's desk
   Cream paper · ink blacks · red survey lines · courier mono accents
   ───────────────────────────────────────────── */
:root {
  --paper:    #f4f1e8;
  --paper2:   #ece9df;
  --paper3:   #e4e1d6;
  --ink:      #1e1d19;
  --ink2:     #3a3930;
  --muted:    #7a7869;
  --rule:     rgba(30,29,25,0.11);
  --rule2:    rgba(30,29,25,0.06);
  --red:      #bf3b1a;
  --red-l:    rgba(191,59,26,0.1);
  --blue:     #1a4e8c;
  --blue-l:   rgba(26,78,140,0.1);
  --green:    #2d6b3e;
  --green-l:  rgba(45,107,62,0.1);
  --amber:    #b06b00;
  --amber-l:  rgba(176,107,0,0.1);
  --panel-l:  300px;
  --panel-r:  360px;
  --top:      46px;
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html,body{height:100%;overflow:hidden}
body{font-family:'Barlow',sans-serif;background:var(--paper);color:var(--ink);font-size:13px;line-height:1.5}

/* ── TOPBAR ── */
.topbar{
  position:fixed;top:0;left:0;right:0;height:var(--top);
  background:var(--ink);z-index:2000;
  display:flex;align-items:center;padding:0 16px;gap:12px;
  border-bottom:1px solid rgba(255,255,255,0.07);
}
.tb-logo{
  font-family:'Barlow Condensed',sans-serif;
  font-size:16px;font-weight:600;letter-spacing:0.06em;
  color:#f4f1e8;text-transform:uppercase;white-space:nowrap;
}
.tb-logo span{color:var(--red);font-weight:300;font-style:italic;}
.tb-div{width:1px;height:18px;background:rgba(255,255,255,0.12);}
.tb-sub{font-family:'Courier Prime',monospace;font-size:10px;color:rgba(244,241,232,0.4);letter-spacing:0.06em;text-transform:uppercase;}
.tb-spacer{flex:1;}
.tb-stats{display:flex;gap:20px;}
.tb-stat{display:flex;flex-direction:column;align-items:flex-end;}
.tb-stat-val{font-family:'Courier Prime',monospace;font-size:15px;color:#f4f1e8;line-height:1;}
.tb-stat-lbl{font-size:9px;color:rgba(244,241,232,0.35);letter-spacing:0.08em;text-transform:uppercase;margin-top:1px;}
.tb-status{display:flex;align-items:center;gap:7px;font-family:'Courier Prime',monospace;font-size:10px;color:rgba(244,241,232,0.45);letter-spacing:0.04em;}
.st-dot{width:7px;height:7px;border-radius:50%;background:#444;}
.st-dot.idle{background:#555;}
.st-dot.working{background:#e8a030;animation:pulse 0.9s infinite;}
.st-dot.ok{background:#4a9e60;}
.st-dot.err{background:var(--red);}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:0.25}}

/* ── LAYOUT ── */
.shell{display:flex;height:100vh;padding-top:var(--top);}

/* ── LEFT PANEL ── */
.panel-l{
  width:var(--panel-l);flex-shrink:0;
  background:var(--paper);border-right:1px solid var(--rule);
  display:flex;flex-direction:column;overflow:hidden;z-index:100;
}
.panel-scroll{flex:1;overflow-y:auto;overflow-x:hidden;}
.panel-scroll::-webkit-scrollbar{width:3px;}
.panel-scroll::-webkit-scrollbar-thumb{background:var(--rule);border-radius:2px;}

/* ── RIGHT PANEL ── */
.panel-r{
  width:var(--panel-r);flex-shrink:0;
  background:var(--paper);border-left:1px solid var(--rule);
  display:flex;flex-direction:column;overflow:hidden;z-index:100;
  transform:translateX(var(--panel-r));
  transition:transform 0.3s cubic-bezier(0.4,0,0.2,1);
}
.panel-r.open{transform:translateX(0);}

/* ── MAP ── */
.map-wrap{flex:1;position:relative;overflow:hidden;}
#map{width:100%;height:100%;}

/* ── SECTION HEADERS ── */
.sec{padding:14px 16px;border-bottom:1px solid var(--rule2);}
.sec:last-child{border-bottom:none;}
.sec-label{
  font-family:'Courier Prime',monospace;font-size:9px;font-weight:700;
  text-transform:uppercase;letter-spacing:0.12em;color:var(--muted);
  margin-bottom:10px;display:flex;align-items:center;gap:8px;
}
.sec-label::after{content:'';flex:1;height:1px;background:var(--rule);}

/* ── DRAW TOOLS ── */
.tool-grid{display:grid;grid-template-columns:1fr 1fr;gap:5px;}
.tbtn{
  display:flex;flex-direction:column;align-items:center;gap:4px;
  padding:9px 6px;border:1px solid var(--rule);border-radius:4px;
  background:var(--paper2);cursor:pointer;transition:all 0.12s;
  font-family:'Barlow',sans-serif;color:var(--ink2);
}
.tbtn:hover{border-color:var(--ink2);background:var(--paper3);}
.tbtn.active{border-color:var(--red);background:var(--red-l);color:var(--red);}
.tbtn svg{width:16px;height:16px;stroke-width:1.5;}
.tbtn-lbl{font-size:9px;font-weight:500;letter-spacing:0.03em;text-align:center;line-height:1.2;}

/* ── SITE STATS ── */
.site-stats{display:grid;grid-template-columns:1fr 1fr;gap:5px;margin-top:8px;}
.stat-box{
  background:var(--paper2);border:1px solid var(--rule);border-radius:4px;
  padding:7px 9px;
}
.stat-lbl{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);letter-spacing:0.08em;text-transform:uppercase;margin-bottom:1px;}
.stat-val{font-family:'Barlow Condensed',sans-serif;font-size:20px;font-weight:300;color:var(--ink);line-height:1;}
.stat-unit{font-size:10px;color:var(--muted);}

/* ── LPA BADGES ── */
.lpa-list{display:flex;flex-direction:column;gap:4px;}
.lpa-badge{
  display:flex;align-items:center;gap:8px;
  padding:6px 9px;background:var(--paper2);border:1px solid var(--rule);border-radius:4px;
}
.lpa-dot{width:6px;height:6px;border-radius:50%;background:var(--blue);flex-shrink:0;}
.lpa-name{flex:1;font-size:11px;font-weight:500;color:var(--ink);}
.lpa-url{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);}
.lpa-open{
  font-size:9px;color:var(--blue);text-decoration:none;
  padding:1px 5px;border:1px solid rgba(26,78,140,0.25);border-radius:2px;
  transition:background 0.12s;white-space:nowrap;
}
.lpa-open:hover{background:var(--blue-l);}

/* ── FORM CONTROLS ── */
.f-grid{display:grid;grid-template-columns:1fr 1fr;gap:6px;}
.f-item{display:flex;flex-direction:column;gap:3px;}
.f-lbl{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);letter-spacing:0.06em;text-transform:uppercase;}
input[type="number"],input[type="text"],select{
  width:100%;padding:6px 8px;
  border:1px solid var(--rule);border-radius:3px;
  background:var(--paper2);color:var(--ink);
  font-family:'Courier Prime',monospace;font-size:11px;
  outline:none;transition:border-color 0.12s;
  -moz-appearance:textfield;
}
input:focus,select:focus{border-color:var(--red);}
input[type="number"]::-webkit-inner-spin-button{-webkit-appearance:none;}
select{cursor:pointer;}

.trow{display:flex;align-items:center;justify-content:space-between;margin-top:6px;}
.trow-lbl{font-size:11px;color:var(--ink2);}
.tog{position:relative;width:32px;height:17px;cursor:pointer;flex-shrink:0;}
.tog input{opacity:0;width:0;height:0;}
.tog-track{
  position:absolute;inset:0;border-radius:99px;
  background:var(--paper3);border:1px solid var(--rule);transition:background 0.2s;
}
.tog input:checked~.tog-track{background:var(--red);border-color:var(--red);}
.tog-thumb{
  position:absolute;top:2px;left:2px;width:11px;height:11px;
  border-radius:50%;background:white;transition:transform 0.18s;pointer-events:none;
}
.tog input:checked~.tog-thumb{transform:translateX(15px);}

/* ── SEARCH BUTTON ── */
.search-btn{
  width:100%;padding:10px;background:var(--ink);color:var(--paper);
  border:none;border-radius:4px;
  font-family:'Barlow Condensed',sans-serif;font-size:14px;font-weight:500;
  letter-spacing:0.08em;text-transform:uppercase;cursor:pointer;
  transition:all 0.15s;display:flex;align-items:center;justify-content:center;gap:8px;
}
.search-btn:hover{background:var(--red);}
.search-btn:disabled{opacity:0.35;cursor:default;pointer-events:none;}
.search-btn .spin{width:12px;height:12px;border:2px solid rgba(244,241,232,0.25);border-top-color:#f4f1e8;border-radius:50%;animation:spin 0.7s linear infinite;}
@keyframes spin{to{transform:rotate(360deg)}}

/* ── PROGRESS LOG ── */
.prog-log{
  display:none;padding:8px 12px;
  background:#1e1d19;border-top:1px solid rgba(255,255,255,0.06);
  max-height:110px;overflow-y:auto;
  font-family:'Courier Prime',monospace;font-size:10px;line-height:1.9;
}
.prog-log.on{display:block;}
.prog-log::-webkit-scrollbar{width:2px;}
.prog-log::-webkit-scrollbar-thumb{background:#444;}
.log-line{display:flex;gap:8px;}
.log-t{color:#e8a030;flex-shrink:0;}
.log-ok{color:#4a9e60;flex-shrink:0;}
.log-err{color:var(--red);flex-shrink:0;}
.log-txt{color:rgba(244,241,232,0.55);}

/* ── RESULTS PANEL ── */
.panel-tabs{display:flex;border-bottom:1px solid var(--rule);flex-shrink:0;}
.p-tab{
  flex:1;padding:9px 8px;text-align:center;cursor:pointer;
  font-family:'Courier Prime',monospace;font-size:9px;font-weight:700;
  text-transform:uppercase;letter-spacing:0.1em;color:var(--muted);
  border-bottom:2px solid transparent;transition:all 0.15s;
}
.p-tab:hover{color:var(--ink);}
.p-tab.on{color:var(--red);border-bottom-color:var(--red);}
.tab-pane{display:none;flex:1;flex-direction:column;overflow:hidden;}
.tab-pane.on{display:flex;}

/* ── RESULTS LIST ── */
.res-header{display:flex;align-items:flex-start;justify-content:space-between;margin-bottom:10px;}
.res-count{font-family:'Barlow Condensed',sans-serif;font-size:28px;font-weight:300;color:var(--ink);line-height:1;}
.res-sub{font-size:10px;color:var(--muted);margin-top:2px;}
.res-sort{display:flex;gap:4px;flex-wrap:wrap;}
.sort-btn{
  font-family:'Courier Prime',monospace;font-size:8px;padding:3px 7px;
  border:1px solid var(--rule);border-radius:2px;background:transparent;
  color:var(--muted);cursor:pointer;transition:all 0.12s;letter-spacing:0.04em;text-transform:uppercase;
}
.sort-btn:hover{border-color:var(--ink);color:var(--ink);}
.sort-btn.on{border-color:var(--red);color:var(--red);background:var(--red-l);}

.app-list{display:flex;flex-direction:column;gap:4px;}
.app-card{
  border:1px solid var(--rule);border-radius:4px;overflow:hidden;
  cursor:pointer;transition:all 0.12s;background:var(--paper);
}
.app-card:hover{border-color:rgba(30,29,25,0.28);box-shadow:0 2px 8px rgba(0,0,0,0.07);}
.app-card.sel{border-color:var(--red);border-width:1.5px;}
.card-top{display:flex;align-items:stretch;gap:0;}
.card-stripe{width:3px;flex-shrink:0;}
.cs-blue{background:var(--blue);}
.cs-red{background:var(--red);}
.cs-green{background:var(--green);}
.cs-amber{background:var(--amber);}
.card-body{flex:1;padding:8px 9px 6px;min-width:0;}
.card-ref{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);margin-bottom:2px;letter-spacing:0.02em;}
.card-desc{font-size:11px;color:var(--ink);line-height:1.35;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden;}
.card-chips{display:flex;gap:4px;flex-wrap:wrap;margin-top:5px;padding:0 9px 7px 12px;}
.chip{
  font-family:'Courier Prime',monospace;font-size:8px;padding:2px 5px;
  border-radius:2px;letter-spacing:0.02em;
}
.ch-units{background:var(--blue-l);color:var(--blue);}
.ch-ha{background:var(--green-l);color:var(--green);}
.ch-status-pend{background:var(--amber-l);color:var(--amber);}
.ch-status-appr{background:var(--green-l);color:var(--green);}
.ch-status-ref{background:var(--red-l);color:var(--red);}
.ch-dist{background:var(--rule2);color:var(--muted);}
.ch-lpa{background:var(--paper3);color:var(--muted);}
.ch-unquant{background:rgba(176,107,0,0.12);color:var(--amber);}
.card-portal{
  display:inline-block;font-size:9px;color:var(--blue);text-decoration:none;
  padding:1px 5px;border:1px solid rgba(26,78,140,0.25);border-radius:2px;
  margin-left:4px;transition:background 0.12s;
}
.card-portal:hover{background:var(--blue-l);}

/* ── PDF VIEWER ── */
.pdf-header{display:flex;align-items:center;justify-content:space-between;padding:10px 14px;border-bottom:1px solid var(--rule);flex-shrink:0;}
.pdf-title{font-size:11px;font-weight:500;color:var(--ink);line-height:1.3;}
.pdf-sub{font-size:9px;color:var(--muted);margin-top:1px;}
.pdf-close{background:none;border:none;cursor:pointer;color:var(--muted);font-size:18px;line-height:1;padding:0 2px;}
.pdf-close:hover{color:var(--ink);}
.pdf-body{flex:1;display:flex;flex-direction:column;overflow:hidden;}
.pdf-docs{padding:8px 12px;border-bottom:1px solid var(--rule2);display:flex;flex-direction:column;gap:4px;max-height:160px;overflow-y:auto;}
.pdf-doc-btn{
  display:flex;align-items:center;gap:8px;padding:6px 8px;
  border:1px solid var(--rule);border-radius:3px;background:var(--paper2);
  cursor:pointer;transition:all 0.12s;text-align:left;width:100%;
}
.pdf-doc-btn:hover{border-color:var(--blue);background:var(--blue-l);}
.pdf-doc-btn.on{border-color:var(--red);background:var(--red-l);}
.pdf-doc-icon{font-size:14px;flex-shrink:0;}
.pdf-doc-name{font-size:10px;font-weight:500;color:var(--ink);flex:1;line-height:1.3;}
.pdf-doc-type{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);}
.pdf-frame-wrap{flex:1;position:relative;background:var(--paper3);}
.pdf-frame-wrap iframe{width:100%;height:100%;border:none;}
.pdf-empty{
  display:flex;flex-direction:column;align-items:center;justify-content:center;
  gap:10px;height:100%;color:var(--muted);text-align:center;padding:20px;
}
.pdf-empty-icon{font-size:32px;opacity:0.3;}
.pdf-empty-txt{font-size:11px;line-height:1.5;}
.pdf-note{
  padding:8px 12px;background:var(--amber-l);border-top:1px solid rgba(176,107,0,0.15);
  font-size:10px;color:var(--amber);line-height:1.5;flex-shrink:0;
}
.pdf-note strong{font-weight:700;}

/* ── MAP OVERLAY ── */
.map-legend{
  position:absolute;bottom:24px;left:14px;z-index:500;
  background:rgba(244,241,232,0.93);border:1px solid var(--rule);
  border-radius:4px;padding:9px 11px;min-width:150px;
  backdrop-filter:blur(6px);pointer-events:none;
}
.leg-title{font-family:'Courier Prime',monospace;font-size:8px;font-weight:700;text-transform:uppercase;letter-spacing:0.1em;color:var(--muted);margin-bottom:7px;}
.leg-row{display:flex;align-items:center;gap:7px;margin-bottom:4px;font-size:10px;color:var(--ink);}
.leg-swatch{width:20px;height:10px;border-radius:2px;flex-shrink:0;}
.ls-site{background:var(--red-l);border:2px solid var(--red);}
.ls-buf{background:rgba(26,78,140,0.08);border:2px dashed var(--blue);}
.ls-std{background:var(--blue-l);border:1.5px solid var(--blue);}
.ls-maj{background:var(--red-l);border:1.5px solid var(--red);}
.ls-lg{background:var(--green-l);border:1.5px solid var(--green);}

.draw-banner{
  position:absolute;top:14px;left:50%;transform:translateX(-50%);
  background:var(--ink);color:var(--paper);
  font-family:'Courier Prime',monospace;font-size:11px;letter-spacing:0.04em;
  padding:7px 16px;border-radius:99px;z-index:800;
  white-space:nowrap;pointer-events:none;opacity:0;transition:opacity 0.25s;
}
.draw-banner.on{opacity:1;}

.map-ctrl{
  position:absolute;top:14px;right:14px;z-index:500;
  display:flex;flex-direction:column;gap:5px;
}
.mc-btn{
  display:flex;align-items:center;gap:6px;padding:6px 10px;
  background:rgba(244,241,232,0.93);border:1px solid var(--rule);border-radius:4px;
  font-family:'Courier Prime',monospace;font-size:9px;color:var(--ink2);cursor:pointer;
  transition:all 0.12s;white-space:nowrap;letter-spacing:0.04em;text-transform:uppercase;
  backdrop-filter:blur(6px);
}
.mc-btn:hover{border-color:var(--ink2);}
.mc-btn.on{border-color:var(--red);color:var(--red);background:rgba(191,59,26,0.06);}
.mc-btn svg{width:12px;height:12px;}

/* ── EMPTY STATES ── */
.empty-state{
  display:flex;flex-direction:column;align-items:center;justify-content:center;
  padding:30px 16px;text-align:center;gap:9px;color:var(--muted);
}
.es-icon{
  width:44px;height:44px;border:1px solid var(--rule);border-radius:50%;
  display:flex;align-items:center;justify-content:center;
  font-family:'Courier Prime',monospace;font-size:18px;color:var(--muted);opacity:0.5;
}
.es-title{font-family:'Barlow Condensed',sans-serif;font-size:15px;font-weight:400;color:var(--ink2);}
.es-sub{font-size:11px;line-height:1.5;}

/* ── STEP INDICATOR ── */
.steps{display:flex;flex-direction:column;gap:4px;}
.step{display:flex;align-items:flex-start;gap:8px;padding:5px 7px;border-radius:3px;transition:background 0.15s;}
.step.on{background:var(--rule2);}
.step.done{opacity:0.45;}
.step-n{font-family:'Courier Prime',monospace;font-size:9px;color:var(--muted);width:14px;flex-shrink:0;margin-top:1px;}
.step.on .step-n{color:var(--red);}
.step-t{font-size:11px;color:var(--ink2);line-height:1.4;}

/* ── NOTIFICATION ── */
.notif{
  position:fixed;bottom:20px;right:20px;z-index:9999;
  background:var(--ink);color:var(--paper);
  padding:9px 14px;border-radius:4px;font-size:11px;max-width:300px;line-height:1.5;
  box-shadow:0 4px 20px rgba(0,0,0,0.2);
  transform:translateY(60px);opacity:0;transition:all 0.25s;pointer-events:none;
}
.notif.on{transform:translateY(0);opacity:1;}
.notif.warn{background:#7a5500;}
.notif.err{background:var(--red);}
.notif.ok{background:var(--green);}

/* ── LEAFLET OVERRIDES ── */
.leaflet-container{font-family:'Barlow',sans-serif;}
.leaflet-popup-content-wrapper{
  background:var(--paper);border:1px solid var(--rule);
  border-radius:4px;box-shadow:0 4px 20px rgba(0,0,0,0.12);color:var(--ink);
}
.leaflet-popup-tip{background:var(--paper);}
.leaflet-popup-content{margin:10px 12px;}
.leaflet-bar a{background:var(--paper) !important;color:var(--muted) !important;border-color:var(--rule) !important;}
.leaflet-bar a:hover{color:var(--ink) !important;}
.leaflet-draw-toolbar a{background-color:var(--paper) !important;}
.popup-ref{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);margin-bottom:3px;}
.popup-desc{font-size:11px;font-weight:500;color:var(--ink);line-height:1.4;margin-bottom:6px;}
.popup-chips{display:flex;gap:3px;flex-wrap:wrap;margin-bottom:7px;}
.popup-addr{font-size:10px;color:var(--muted);margin-bottom:6px;}
.popup-links{display:flex;gap:5px;flex-wrap:wrap;}
.popup-btn{font-size:10px;color:var(--blue);text-decoration:none;padding:3px 7px;border:1px solid rgba(26,78,140,0.3);border-radius:2px;cursor:pointer;background:none;}
.popup-btn:hover{background:var(--blue-l);}
.popup-btn.red{color:var(--red);border-color:rgba(191,59,26,0.3);}
.popup-btn.red:hover{background:var(--red-l);}

/* ── EXPORT BAR ── */
.export-bar{
  display:none;padding:8px 14px;border-top:1px solid var(--rule);
  flex-shrink:0;display:none;align-items:center;gap:6px;flex-wrap:wrap;
}
.export-bar.on{display:flex;}
.exp-lbl{font-family:'Courier Prime',monospace;font-size:8px;text-transform:uppercase;letter-spacing:0.08em;color:var(--muted);margin-right:2px;}
.exp-btn{
  display:flex;align-items:center;gap:4px;padding:4px 8px;
  border:1px solid var(--rule);border-radius:2px;background:transparent;
  font-family:'Courier Prime',monospace;font-size:9px;color:var(--ink2);
  cursor:pointer;transition:all 0.12s;letter-spacing:0.03em;text-transform:uppercase;
}
.exp-btn:hover{border-color:var(--ink);color:var(--ink);}
.exp-btn svg{width:10px;height:10px;}
</style>
</head>
<body>

<!-- TOPBAR -->
<header class="topbar">
  <div class="tb-logo">Cumulative <span>Planning</span> Search</div>
  <div class="tb-div"></div>
  <div class="tb-sub">UK Residential Development Monitor</div>
  <div class="tb-spacer"></div>
  <div class="tb-stats">
    <div class="tb-stat"><div class="tb-stat-val" id="stTotal">—</div><div class="tb-stat-lbl">Results</div></div>
    <div class="tb-stat"><div class="tb-stat-val" id="stUnits">—</div><div class="tb-stat-lbl">Total units</div></div>
    <div class="tb-stat"><div class="tb-stat-val" id="stLPAs">—</div><div class="tb-stat-lbl">LPAs</div></div>
  </div>
  <div class="tb-div"></div>
  <div class="tb-status"><div class="st-dot idle" id="stDot"></div><span id="stTxt">Ready</span></div>
</header>

<!-- DRAW BANNER -->
<div class="draw-banner" id="drawBanner">Click to place vertices — double-click to close polygon</div>

<div class="shell">

<!-- ═══ LEFT PANEL ═══ -->
<aside class="panel-l">
  <div class="panel-scroll">

    <!-- WORKFLOW -->
    <div class="sec">
      <div class="sec-label">Workflow</div>
      <div class="steps">
        <div class="step on" id="stp1"><div class="step-n">01</div><div class="step-t">Draw site boundary polygon</div></div>
        <div class="step" id="stp2"><div class="step-n">02</div><div class="step-t">LPA(s) detected automatically</div></div>
        <div class="step" id="stp3"><div class="step-n">03</div><div class="step-t">5km buffer from polygon edge</div></div>
        <div class="step" id="stp4"><div class="step-n">04</div><div class="step-t">Planning databases queried</div></div>
        <div class="step" id="stp5"><div class="step-n">05</div><div class="step-t">Site plans retrieved &amp; listed</div></div>
      </div>
    </div>

    <!-- DRAW -->
    <div class="sec">
      <div class="sec-label">Site Boundary</div>
      <div class="tool-grid">
        <button class="tbtn active" id="btnDraw" onclick="startDraw()">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor"><polygon points="3,21 3,3 21,12"/><line x1="12" y1="5" x2="19" y2="5"/><line x1="19" y1="5" x2="21" y2="12"/><line x1="21" y1="12" x2="12" y2="19"/><line x1="12" y1="19" x2="3" y2="21"/></svg>
          <div class="tbtn-lbl">Draw polygon</div>
        </button>
        <button class="tbtn" onclick="document.getElementById('fileIn').click()">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
          <div class="tbtn-lbl">Upload GeoJSON/KML</div>
        </button>
        <button class="tbtn" onclick="clearAll()">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14H6L5 6"/><path d="M10 11v6"/><path d="M14 11v6"/></svg>
          <div class="tbtn-lbl">Clear site</div>
        </button>
        <button class="tbtn" onclick="fitSite()">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor"><path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"/></svg>
          <div class="tbtn-lbl">Fit to site</div>
        </button>
      </div>
      <input type="file" id="fileIn" accept=".geojson,.json,.kml" style="display:none" onchange="loadFile(event)">
      <div id="siteStats" style="display:none">
        <div class="site-stats">
          <div class="stat-box"><div class="stat-lbl">Site area</div><div class="stat-val" id="siteHa">—</div><div class="stat-unit">hectares</div></div>
          <div class="stat-box"><div class="stat-lbl">Buffer radius</div><div class="stat-val" id="bufKmDisp">5.0</div><div class="stat-unit">km from edge</div></div>
        </div>
      </div>
    </div>

    <!-- LPA DETECTION -->
    <div class="sec" id="lpaSection" style="display:none">
      <div class="sec-label">Local Planning Authorities</div>
      <div class="lpa-list" id="lpaList"></div>
    </div>

    <!-- FILTERS -->
    <div class="sec">
      <div class="sec-label">Search Parameters</div>
      <div class="f-grid">
        <div class="f-item"><div class="f-lbl">Buffer (km)</div><input type="number" id="bufKm" value="5" min="0.5" max="25" step="0.5" onchange="redrawBuffer()"></div>
        <div class="f-item"><div class="f-lbl">Years back</div><input type="number" id="yrsBack" value="5" min="1" max="20" step="1"></div>
        <div class="f-item"><div class="f-lbl">Min units (OR)</div><input type="number" id="minUnits" value="10" min="0" step="1"></div>
        <div class="f-item"><div class="f-lbl">Min area ha (OR)</div><input type="number" id="minHa" value="0.5" min="0" step="0.1"></div>
      </div>
      <div class="trow"><span class="trow-lbl">Include refused applications</span><label class="tog"><input type="checkbox" id="togRefused" checked><div class="tog-track"></div><div class="tog-thumb"></div></label></div>
      <div class="trow"><span class="trow-lbl">Show search buffer on map</span><label class="tog"><input type="checkbox" id="togBuffer" checked onchange="toggleBuf()"><div class="tog-track"></div><div class="tog-thumb"></div></label></div>
    </div>

    <!-- SEARCH -->
    <div class="sec">
      <button class="search-btn" id="searchBtn" onclick="runSearch()" disabled>
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        Search Planning Databases
      </button>
      <div style="margin-top:7px;font-size:10px;color:var(--muted);text-align:center;line-height:1.5" id="srchHint">Draw a site boundary to enable search</div>
    </div>

  </div><!-- /panel-scroll -->

  <!-- PROGRESS LOG -->
  <div class="prog-log" id="progLog"></div>
</aside>

<!-- ═══ MAP ═══ -->
<div class="map-wrap">
  <div id="map"></div>

  <!-- Map controls -->
  <div class="map-ctrl">
    <button class="mc-btn on" id="mcBase" onclick="cycleBase()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="18" height="18" rx="2"/><path d="M3 9h18M3 15h18M9 3v18M15 3v18"/></svg>
      Street map
    </button>
    <button class="mc-btn" id="mcSat" onclick="toggleSat()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><circle cx="12" cy="12" r="3"/><path d="M12 2v3M12 19v3M4.22 4.22l2.12 2.12M17.66 17.66l2.12 2.12M2 12h3M19 12h3M4.22 19.78l2.12-2.12M17.66 6.34l2.12-2.12"/></svg>
      Satellite
    </button>
    <button class="mc-btn" onclick="openResults()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/></svg>
      Results panel
    </button>
    <button class="mc-btn" onclick="fitAll()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"/></svg>
      Fit all
    </button>
  </div>

  <!-- Legend -->
  <div class="map-legend">
    <div class="leg-title">Legend</div>
    <div class="leg-row"><div class="leg-swatch ls-site"></div>Site boundary</div>
    <div class="leg-row"><div class="leg-swatch ls-buf"></div>5km buffer</div>
    <div class="leg-row"><div class="leg-swatch ls-std"></div>Residential scheme</div>
    <div class="leg-row"><div class="leg-swatch ls-maj"></div>EIA / Major</div>
    <div class="leg-row"><div class="leg-swatch ls-lg"></div>Large ≥0.5ha</div>
  </div>
</div>

<!-- ═══ RIGHT PANEL (Results + PDF) ═══ -->
<aside class="panel-r" id="panelR">
  <div class="panel-tabs">
    <div class="p-tab on" id="tabRes" onclick="switchTab('res')">Results</div>
    <div class="p-tab" id="tabPDF" onclick="switchTab('pdf')">Site Plans</div>
  </div>

  <!-- RESULTS TAB -->
  <div class="tab-pane on" id="paneRes" style="flex-direction:column;overflow:hidden;">
    <div style="flex:1;overflow-y:auto;padding:14px;" id="resScroll">
      <div class="empty-state">
        <div class="es-icon">◎</div>
        <div class="es-title">No results yet</div>
        <div class="es-sub">Draw a site boundary and run a search to find qualifying planning applications within the 5km buffer.</div>
      </div>
    </div>
    <div class="export-bar" id="expBar">
      <span class="exp-lbl">Export</span>
      <button class="exp-btn" onclick="exportCSV()"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2"/><path d="M3 9h18M3 15h18M9 3v18"/></svg>CSV</button>
      <button class="exp-btn" onclick="exportGeoJSON()"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="1 6 1 22 8 18 16 22 23 18 23 2 16 6 8 2 1 6"/></svg>GeoJSON</button>
      <button class="exp-btn" onclick="exportKML()"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 2h10l-4 6 4 6H3l4-6z"/></svg>KML</button>
    </div>
  </div>

  <!-- PDF TAB -->
  <div class="tab-pane" id="panePDF" style="flex-direction:column;overflow:hidden;">
    <div class="pdf-header">
      <div>
        <div class="pdf-title" id="pdfTitle">No application selected</div>
        <div class="pdf-sub" id="pdfSub">Select a result to view its planning documents</div>
      </div>
      <button class="pdf-close" onclick="clearPDF()">×</button>
    </div>
    <div class="pdf-body">
      <div class="pdf-docs" id="pdfDocList">
        <div style="font-size:10px;color:var(--muted);padding:4px 0;">No documents loaded.</div>
      </div>
      <div class="pdf-frame-wrap" id="pdfFrameWrap">
        <div class="pdf-empty">
          <div class="pdf-empty-icon">📄</div>
          <div class="pdf-empty-txt">Select an application from the results list, then choose a document above to view it alongside the map.</div>
        </div>
      </div>
      <div class="pdf-note" id="pdfNote" style="display:none">
        <strong>Georeferencing:</strong> This document is shown alongside the map for visual assessment. Red-line boundaries extracted from PDFs require manual registration — compare grid references or known features.
      </div>
    </div>
  </div>
</aside>

</div><!-- /shell -->

<div class="notif" id="notif"></div>

<script>
'use strict';
// ══════════════════════════════════════════════════════════════
// STATE
// ══════════════════════════════════════════════════════════════
const S = {
  site: null,       // Turf feature
  buffer: null,     // Turf feature
  siteL: null,      // Leaflet layer
  bufL: null,       // Leaflet layer
  resL: null,       // Leaflet layer group
  results: [],
  lpas: [],
  markers: [],
  sortKey: 'dist',
  baseCycle: 0,
  satOn: false,
  panelOpen: false,
};

// ══════════════════════════════════════════════════════════════
// IDOX PORTAL DIRECTORY — ~50 UK councils
// ══════════════════════════════════════════════════════════════
const PORTALS = {
  'North Yorkshire':           'https://public.selby.gov.uk/online-applications',
  'Craven':                    'https://publicaccess.cravendc.gov.uk/online-applications',
  'Leeds':                     'https://publicaccess.leeds.gov.uk/online-applications',
  'Bradford':                  'https://planning.bradford.gov.uk/online-applications',
  'Sheffield':                 'https://planningapps.sheffield.gov.uk/online-applications',
  'Wakefield':                 'https://planning.wakefield.gov.uk/online-applications',
  'Kirklees':                  'https://www.kirklees.gov.uk/planning',
  'Calderdale':                'https://calderdale.gov.uk/planning',
  'Harrogate':                 'https://uniformonline.harrogate.gov.uk/online-applications',
  'Hambleton':                 'https://planning.hambleton.gov.uk/online-applications',
  'Scarborough':               'https://planning.scarborough.gov.uk/online-applications',
  'Richmondshire':             'https://planningregister.richmondshire.gov.uk/online-applications',
  'Manchester':                'https://www.manchester.gov.uk/planning',
  'Salford':                   'https://publicaccess.salford.gov.uk/online-applications',
  'Trafford':                  'https://planning.trafford.gov.uk/online-applications',
  'Stockport':                 'https://planning.stockport.gov.uk/online-applications',
  'Tameside':                  'https://planning.tameside.gov.uk/online-applications',
  'Oldham':                    'https://planning.oldham.gov.uk/online-applications',
  'Rochdale':                  'https://rochdale.gov.uk/planning',
  'Bury':                      'https://planning.bury.gov.uk/online-applications',
  'Bolton':                    'https://planning.bolton.gov.uk/online-applications',
  'Wigan':                     'https://planning.wigan.gov.uk/online-applications',
  'Birmingham':                'https://eplanning.birmingham.gov.uk/Northgate/PlanningExplorer',
  'Coventry':                  'https://planningapps.coventry.gov.uk/online-applications',
  'Wolverhampton':             'https://planningapps.wolverhampton.gov.uk/online-applications',
  'Dudley':                    'https://www.dudley.gov.uk/planning',
  'Sandwell':                  'https://www.sandwell.gov.uk/planning',
  'Walsall':                   'https://planning.walsall.gov.uk/online-applications',
  'Solihull':                  'https://planning.solihull.gov.uk/online-applications',
  'Bristol':                   'https://pa.bristol.gov.uk/online-applications',
  'Bath and North East Somerset': 'https://www.bathnes.gov.uk/services/planning',
  'South Gloucestershire':     'https://developments.southglos.gov.uk/online-applications',
  'Gloucester':                'https://planning.gloucester.gov.uk/online-applications',
  'Cheltenham':                'https://planning.cheltenham.gov.uk/online-applications',
  'Oxford':                    'https://www.oxford.gov.uk/info/20214/search_for_planning_applications',
  'South Oxfordshire':         'https://publicaccess.southoxon.gov.uk/online-applications',
  'Vale of White Horse':       'https://publicaccess.whitehorsedc.gov.uk/online-applications',
  'Cambridge':                 'https://applications.greatercambridgeplanning.org/online-applications',
  'South Cambridgeshire':      'https://applications.greatercambridgeplanning.org/online-applications',
  'Norwich':                   'https://planning.norwich.gov.uk/online-applications',
  'Broadland':                 'https://planning.broadland.gov.uk/online-applications',
  'Westminster':               'https://idoxpa.westminster.gov.uk/online-applications',
  'Camden':                    'https://planning.camden.gov.uk/online-applications',
  'Hackney':                   'https://planningscrutiny.hackney.gov.uk/online-applications',
  'Tower Hamlets':             'https://development.towerhamlets.gov.uk/online-applications',
  'Southwark':                 'https://planning.southwark.gov.uk/online-applications',
  'Lambeth':                   'https://planning.lambeth.gov.uk/online-applications',
  'Lewisham':                  'https://planning.lewisham.gov.uk/online-applications',
  'Greenwich':                 'https://planning.royalgreenwich.gov.uk/online-applications',
  'Islington':                 'https://www.islington.gov.uk/planning',
  'Haringey':                  'https://www.planning.haringey.gov.uk/online-applications',
  'Enfield':                   'https://planning.enfield.gov.uk/online-applications',
  'Barnet':                    'https://barnet.gov.uk/planning',
  'Ealing':                    'https://pam.ealing.gov.uk/online-applications',
  'Hounslow':                  'https://planningandbuilding.hounslow.gov.uk/online-applications',
  'Kingston upon Thames':      'https://maps.kingston.gov.uk/online-applications',
  'Wandsworth':                'https://planning.wandsworth.gov.uk/online-applications',
  'Merton':                    'https://planning.merton.gov.uk/online-applications',
  'Croydon':                   'https://planning.croydon.gov.uk/online-applications',
  'Bromley':                   'https://searchapplications.bromley.gov.uk/online-applications',
  'Bexley':                    'https://pa.bexley.gov.uk/online-applications',
};

// ══════════════════════════════════════════════════════════════
// MAP INIT
// ══════════════════════════════════════════════════════════════
const TILES = [
  { name:'Street map',  url:'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', attr:'© OpenStreetMap, © CARTO' },
  { name:'OS style',    url:'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',               attr:'© OpenStreetMap' },
  { name:'Dark',        url:'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',   attr:'© OpenStreetMap, © CARTO' },
];

const map = L.map('map',{ center:[52.9,-1.5], zoom:7 });
let tileLayer = L.tileLayer(TILES[0].url,{ attribution:TILES[0].attr, maxZoom:19 }).addTo(map);
let satLayer  = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',{ attribution:'© Esri', maxZoom:19, opacity:0 });

S.resL = L.layerGroup().addTo(map);
const drawn = new L.FeatureGroup().addTo(map);

// Leaflet Draw
const drawCtrl = new L.Control.Draw({
  draw:{
    polygon:{
      allowIntersection:false,
      shapeOptions:{ color:'#bf3b1a', fillColor:'#bf3b1a', fillOpacity:0.1, weight:2.5 }
    },
    rectangle:false,polyline:false,circle:false,circlemarker:false,marker:false
  },
  edit:{ featureGroup:drawn }
});
map.addControl(drawCtrl);

map.on(L.Draw.Event.DRAWSTART, ()=>{ showBanner(); });
map.on(L.Draw.Event.DRAWSTOP, ()=>{ hideBanner(); });
map.on(L.Draw.Event.CREATED, e=>{ drawn.clearLayers(); drawn.addLayer(e.layer); processSite(e.layer.toGeoJSON().geometry); });
map.on(L.Draw.Event.EDITED,  e=>{ e.layers.eachLayer(l=>processSite(l.toGeoJSON().geometry)); });
map.on(L.Draw.Event.DELETED, ()=>{ clearAll(); });

// ══════════════════════════════════════════════════════════════
// SITE PROCESSING
// ══════════════════════════════════════════════════════════════
async function processSite(geom) {
  if (S.siteL) map.removeLayer(S.siteL);
  if (S.bufL)  map.removeLayer(S.bufL);

  let feat;
  try {
    if (geom.type === 'Polygon')      feat = turf.polygon(geom.coordinates);
    else if (geom.type === 'MultiPolygon') feat = turf.multiPolygon(geom.coordinates);
    else if (geom.type === 'Feature') feat = geom;
    else { notify('Unsupported geometry type','err'); return; }
  } catch(e) { notify('Geometry error: '+e.message,'err'); return; }

  S.site = feat;

  // Area
  const ha = (turf.area(feat)/10000).toFixed(2);
  document.getElementById('siteHa').textContent = ha;
  document.getElementById('siteStats').style.display = 'block';

  // Site boundary layer
  S.siteL = L.geoJSON(feat,{ style:{ color:'#bf3b1a', weight:2.5, fillColor:'#bf3b1a', fillOpacity:0.1 }}).addTo(map);

  // Buffer
  drawBuffer();

  // LPA detection
  setStatus('working','Detecting LPA…');
  await detectLPAs(feat);

  stepDone(1); stepActive(2); stepDone(2); stepActive(3); stepDone(3);
  document.getElementById('searchBtn').disabled = false;
  document.getElementById('srchHint').textContent = 'LPAs detected — click to search';
  setStatus('idle','Site ready');
  notify('Site boundary set. '+S.lpas.length+' LPA(s) detected.','ok');
}

function drawBuffer() {
  if (!S.site) return;
  if (S.bufL) map.removeLayer(S.bufL);
  const km = parseFloat(document.getElementById('bufKm').value)||5;
  document.getElementById('bufKmDisp').textContent = km.toFixed(1);
  S.buffer = turf.buffer(S.site, km, { units:'kilometers' });
  S.bufL = L.geoJSON(S.buffer,{ style:{ color:'#1a4e8c', weight:1.5, dashArray:'6 4', fillColor:'#1a4e8c', fillOpacity:0.06 }}).addTo(map);
  if (!document.getElementById('togBuffer').checked) map.removeLayer(S.bufL);
}

function redrawBuffer() { drawBuffer(); }
function toggleBuf() {
  if (!S.bufL) return;
  document.getElementById('togBuffer').checked ? map.addLayer(S.bufL) : map.removeLayer(S.bufL);
}

// ══════════════════════════════════════════════════════════════
// LPA DETECTION
// ══════════════════════════════════════════════════════════════
async function detectLPAs(feat) {
  const bbox = turf.bbox(feat);
  const centroid = turf.centroid(feat).geometry.coordinates;

  // Sample points: centroid + 4 corners + 4 edge midpoints
  const pts = [
    centroid,
    [bbox[0],bbox[1]], [bbox[2],bbox[1]], [bbox[0],bbox[3]], [bbox[2],bbox[3]],
    [(bbox[0]+bbox[2])/2,bbox[1]], [(bbox[0]+bbox[2])/2,bbox[3]],
    [bbox[0],(bbox[1]+bbox[3])/2], [bbox[2],(bbox[1]+bbox[3])/2],
  ];

  const seen = new Map();
  await Promise.allSettled(pts.map(async([lng,lat])=>{
    try {
      const r = await fetch(`https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lng}&format=json&addressdetails=1&zoom=10`,
        { headers:{'Accept-Language':'en-GB'}, signal:AbortSignal.timeout(6000) });
      const d = await r.json();
      const a = d.address||{};
      const name = a.county||a.city||a.state_district||a.town||'Unknown LPA';
      if (!seen.has(name)) {
        const portal = findPortal(name);
        seen.set(name,{ name, region:a.state||a.country, portal, postcode:a.postcode });
      }
    } catch{}
  }));

  S.lpas = [...seen.values()];

  const sec  = document.getElementById('lpaSection');
  const list = document.getElementById('lpaList');
  sec.style.display = 'block';
  document.getElementById('stLPAs').textContent = S.lpas.length;

  if (!S.lpas.length) {
    list.innerHTML = '<div style="font-size:10px;color:var(--muted)">Could not detect LPA — check connectivity.</div>';
    return;
  }

  list.innerHTML = S.lpas.map(l=>`
    <div class="lpa-badge">
      <div class="lpa-dot"></div>
      <div style="flex:1;min-width:0;">
        <div class="lpa-name">${l.name}</div>
        <div class="lpa-url">${l.portal ? new URL(l.portal).hostname : 'Portal not indexed'}</div>
      </div>
      ${l.portal?`<a class="lpa-open" href="${l.portal}" target="_blank">Open →</a>`:''}
    </div>`).join('');
}

function findPortal(name) {
  if (PORTALS[name]) return PORTALS[name];
  const nl = name.toLowerCase();
  for (const [k,v] of Object.entries(PORTALS)) {
    if (nl.includes(k.toLowerCase()) || k.toLowerCase().includes(nl)) return v;
  }
  return null;
}

// ══════════════════════════════════════════════════════════════
// MAIN SEARCH
// ══════════════════════════════════════════════════════════════
async function runSearch() {
  if (!S.site || !S.buffer) { notify('Draw a site boundary first','warn'); return; }
  S.results = [];
  S.resL.clearLayers();
  S.markers = [];

  setBtnWorking(true);
  setStatus('working','Searching…');
  const log = document.getElementById('progLog');
  log.innerHTML = '';
  log.classList.add('on');
  stepActive(4);

  const bbox    = turf.bbox(S.buffer);
  const yrs     = parseInt(document.getElementById('yrsBack').value)||5;
  const fromDt  = new Date(); fromDt.setFullYear(fromDt.getFullYear()-yrs);
  const inclRef = document.getElementById('togRefused').checked;

  addLog('info',`Search bbox: ${bbox.map(v=>v.toFixed(3)).join(', ')}`);
  addLog('info',`Date range: ${fromDt.toLocaleDateString('en-GB')} → today`);
  addLog('info',`Threshold: >${document.getElementById('minUnits').value} units OR ≥${document.getElementById('minHa').value} ha`);

  // ── Source 1: PlanningAlerts ──
  addLog('info','Querying PlanningAlerts…');
  try {
    const pa = await srcPlanningAlerts(bbox, fromDt);
    addLog('ok',`PlanningAlerts: ${pa.length} raw results`);
    S.results.push(...pa);
  } catch(e) { addLog('err','PlanningAlerts: '+e.message); }

  // ── Source 2: GLA (if London bbox) ──
  if (bboxOverlap(bbox,[-0.55,51.28,0.35,51.72])) {
    addLog('info','Querying GLA London data…');
    try {
      const gla = await srcGLA(bbox, fromDt);
      addLog('ok',`GLA: ${gla.length} results`);
      S.results.push(...gla);
    } catch(e) { addLog('err','GLA: '+e.message); }
  }

  // ── Source 3: Idox portals for detected LPAs ──
  for (const lpa of S.lpas) {
    if (!lpa.portal) { addLog('info',`${lpa.name}: no portal indexed`); continue; }
    addLog('info',`Querying Idox: ${lpa.name}…`);
    try {
      const idox = await srcIdox(lpa, fromDt);
      addLog('ok',`${lpa.name}: ${idox.length} results`);
      S.results.push(...idox);
    } catch(e) { addLog('err',`${lpa.name}: ${e.message}`); }
  }

  addLog('info',`Total raw: ${S.results.length} — deduplicating & filtering…`);

  // ── Deduplicate ──
  const seen2 = new Set();
  S.results = S.results.filter(r=>{
    const k = r.reference||(r.lat+','+r.lng);
    if (seen2.has(k)) return false;
    seen2.add(k); return true;
  });

  // ── Geocode missing coords ──
  const noCoord = S.results.filter(r=>!r.lat||!r.lng);
  if (noCoord.length) {
    addLog('info',`Geocoding ${noCoord.length} missing addresses…`);
    await geocodeBatch(noCoord);
  }

  // ── Filter: inside buffer ──
  S.results = S.results.filter(r=>{
    if (!r.lat||!r.lng) return false;
    return turf.booleanPointInPolygon(turf.point([r.lng,r.lat]), S.buffer);
  });

  // ── Filter: date ──
  S.results = S.results.filter(r=>{
    if (!r.receivedDate) return true; // can't exclude without date
    return new Date(r.receivedDate) >= fromDt;
  });

  // ── Filter: thresholds (OR logic) ──
  const minU = parseInt(document.getElementById('minUnits').value)||10;
  const minA = parseFloat(document.getElementById('minHa').value)||0.5;
  S.results = S.results.filter(r=>{
    if (!inclRef && r.status && /refus|reject/i.test(r.status)) return false;
    if (!isResidential(r.description)) return false;
    const u = r.units||0, a = r.siteHa||0;
    // If neither quantified, include if residential keyword found — flag as unquantified
    if (!r.units && !r.siteHa) { r._unquant=true; return true; }
    return u > minU || a >= minA;
  });

  // ── Compute distances from site polygon edge ──
  S.results = S.results.map(r=>{
    const pt = turf.point([r.lng,r.lat]);
    try {
      const line = turf.polygonToLine(S.site);
      const nearest = turf.nearestPointOnLine(line, pt);
      r.distKm = +turf.distance(pt, nearest,{units:'kilometers'}).toFixed(3);
    } catch { r.distKm = +turf.distance(pt, turf.centroid(S.site),{units:'kilometers'}).toFixed(3); }
    return r;
  });

  addLog('ok',`After filtering: ${S.results.length} qualifying applications`);

  // ── Fallback: demo data if nothing returned ──
  if (!S.results.length) {
    addLog('info','No live results — injecting demo data (CORS limitation)…');
    injectDemo();
    notify('Live APIs blocked by CORS. Demo data shown — see companion Python script for live scraping.','warn');
  }

  sortResults();
  renderResults();
  renderMarkers();

  stepDone(4); stepDone(5);
  const tot = S.results.length;
  const totUnits = S.results.reduce((a,r)=>a+(r.units||0),0);
  document.getElementById('stTotal').textContent = tot;
  document.getElementById('stUnits').textContent = totUnits||'—';
  setStatus('ok',`${tot} schemes found`);
  setBtnWorking(false);
  if (tot>0) { openResults(); notify(`${tot} qualifying schemes found within buffer`,'ok'); }
}

// ══════════════════════════════════════════════════════════════
// DATA SOURCES
// ══════════════════════════════════════════════════════════════
const PROXIES = [
  u=>`https://api.allorigins.win/raw?url=${encodeURIComponent(u)}`,
  u=>`https://corsproxy.io/?${encodeURIComponent(u)}`,
];

async function proxyFetch(url, opts={}) {
  for (const px of PROXIES) {
    try {
      const r = await fetch(px(url),{ signal:AbortSignal.timeout(12000), ...opts });
      if (r.ok) return r;
    } catch {}
  }
  throw new Error('All proxies failed');
}

// ── PlanningAlerts ──
async function srcPlanningAlerts(bbox, fromDt) {
  const results=[];
  // PlanningAlerts RSS/JSON feeds per authority
  for (const lpa of S.lpas) {
    const slug = lpa.name.toLowerCase().replace(/\s+/g,'-').replace(/[^a-z0-9-]/g,'');
    const url = `https://www.planningalerts.org.au/authorities/${slug}/applications.json`;
    try {
      const r = await proxyFetch(url);
      const d = await r.json();
      if (d.applications) d.applications.forEach(a=>results.push(normPA(a)));
    } catch {}
  }
  return results.filter(r=>r.lat&&r.lng);
}

function normPA(a) {
  return {
    reference:   a.council_reference||a.id,
    description: a.description||'',
    address:     a.address||'',
    lat:         parseFloat(a.lat)||null,
    lng:         parseFloat(a.lng)||null,
    units:       getUnits(a.description||''),
    siteHa:      getHa(a.description||''),
    status:      a.status||null,
    source:      'PlanningAlerts',
    receivedDate:a.date_received||null,
    portalUrl:   a.info_url||null,
    lpa:         a.authority_name||'Unknown',
    _docs:[],
  };
}

// ── GLA ──
async function srcGLA(bbox, fromDt) {
  const [w,s,e,n]=bbox;
  const results=[];
  const url=`https://www.london.gov.uk/sites/default/files/strategic_applications.json`;
  try {
    const r = await proxyFetch(url);
    const d = await r.json();
    if (Array.isArray(d)) d.forEach(a=>{
      if (a.latitude&&a.longitude&&a.latitude>=s&&a.latitude<=n&&a.longitude>=w&&a.longitude<=e)
        results.push(normGLA(a));
    });
  } catch {}
  return results;
}

function normGLA(a) {
  return {
    reference:   a.reference||a.application_reference,
    description: a.description||a.proposal||'',
    address:     a.address||a.site_address||'',
    lat:         parseFloat(a.latitude)||null,
    lng:         parseFloat(a.longitude)||null,
    units:       parseInt(a.units_proposed||a.residential_units)||getUnits(a.description||''),
    siteHa:      parseFloat(a.site_area)||getHa(a.description||''),
    status:      a.status||a.decision||null,
    source:      'GLA London',
    receivedDate:a.date_received||null,
    portalUrl:   a.planning_portal_url||null,
    lpa:         a.lpa||a.borough||'London',
    _docs:[],
  };
}

// ── Idox scrape ──
async function srcIdox(lpa, fromDt) {
  const results=[];
  const p=lpa.portal;
  const fy=fromDt.getFullYear(), fm=fromDt.getMonth()+1, fd=fromDt.getDate();
  const ny=new Date().getFullYear(), nm=new Date().getMonth()+1, nd=new Date().getDate();

  // Build search URL — Idox advanced search endpoint
  const url=`${p}/search.do?action=advanced&description=residential+dwellings&dateType=DC_Received&dateRangeType=custom&startDateDay=${fd}&startDateMonth=${fm}&startDateYear=${fy}&endDateDay=${nd}&endDateMonth=${nm}&endDateYear=${ny}`;

  try {
    const r = await proxyFetch(url);
    const html = await r.text();
    const parsed = parseIdoxHTML(html, lpa);
    results.push(...parsed);

    // Second page if available
    if (/next/i.test(html) && parsed.length>0) {
      try {
        const r2=await proxyFetch(url+'&page=2');
        if (r2.ok) results.push(...parseIdoxHTML(await r2.text(), lpa));
      } catch{}
    }
  } catch(e) { throw e; }

  return results;
}

function parseIdoxHTML(html, lpa) {
  const doc = new DOMParser().parseFromString(html,'text/html');
  const out=[];
  doc.querySelectorAll('li.searchresult, .search-result, tr.searchresult').forEach(el=>{
    try {
      const ref  = (el.querySelector('.result-title a, h2 a, .reference')?.textContent||'').trim();
      const desc = (el.querySelector('.result-description, p.description, .description')?.textContent||'').trim();
      const addr = (el.querySelector('.address, .result-address')?.textContent||'').trim();
      const date = (el.querySelector('.received-date, .date, td.date')?.textContent||'').trim();
      const href = el.querySelector('a[href*="applicationDetails"]')?.getAttribute('href')||'';
      if (!ref&&!desc) return;
      out.push({
        reference:ref, description:desc, address:addr,
        units:getUnits(desc), siteHa:getHa(desc), status:null,
        source:lpa.name, lpa:lpa.name,
        receivedDate:date||null,
        portalUrl:href?(href.startsWith('http')?href:lpa.portal+href):null,
        lat:null, lng:null, _docs:[],
      });
    } catch{}
  });
  return out;
}

// ══════════════════════════════════════════════════════════════
// TEXT EXTRACTION HELPERS
// ══════════════════════════════════════════════════════════════
function getUnits(t) {
  if (!t) return 0;
  const pats=[
    /(\d+)\s*(?:no\.?|x|×)?\s*(?:new\s+)?(?:residential\s+)?(?:dwelling|unit|apartment|flat|house|home)/i,
    /erection\s+of\s+(\d+)/i,
    /up\s+to\s+(\d+)\s*(?:dwelling|unit|home|house|flat)/i,
    /provision\s+of\s+(\d+)/i,
    /(\d+)\s+affordable/i,
    /(\d+)\s+bed/i,
  ];
  for (const p of pats){ const m=t.match(p); if(m) return +m[1]; }
  return 0;
}

function getHa(t) {
  if (!t) return 0;
  const m=t.match(/(\d+\.?\d*)\s*(?:ha|hectare)/i);
  return m?+m[1]:0;
}

function isResidential(t) {
  if (!t) return false;
  return /dwellings?|residential|housing|apartments?|flats?|homes?|units?\s+(?:of\s+)?(?:accommodation|housing)|student\s+accommodation|build\s+to\s+rent|affordable/i.test(t);
}

function getStatusClass(s) {
  if (!s) return 'ch-status-pend';
  const sl=s.toLowerCase();
  if (/approv|grant|permit|allowed/.test(sl)) return 'ch-status-appr';
  if (/refus|reject/.test(sl)) return 'ch-status-ref';
  return 'ch-status-pend';
}

function getStripeClass(r) {
  if (r.description&&/eia|environmental impact assessment/i.test(r.description)) return 'cs-red';
  if ((r.siteHa||0)>=0.5||(r.units||0)>=50) return 'cs-green';
  return 'cs-blue';
}

function bboxOverlap(a,b){ return !(a[2]<b[0]||a[0]>b[2]||a[3]<b[1]||a[1]>b[3]); }

// ══════════════════════════════════════════════════════════════
// GEOCODING
// ══════════════════════════════════════════════════════════════
async function geocodeBatch(items) {
  for (let i=0;i<Math.min(items.length,15);i++) {
    if (!items[i].address) continue;
    try {
      const r=await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(items[i].address+', UK')}&limit=1&countrycodes=gb`,
        { headers:{'Accept-Language':'en-GB','User-Agent':'CumulativePlanningTool/1.0'}, signal:AbortSignal.timeout(5000) });
      const d=await r.json();
      if (d.length){ items[i].lat=+d[0].lat; items[i].lng=+d[0].lon; }
    } catch{}
    if (i<items.length-1) await new Promise(r=>setTimeout(r,350));
  }
}

// ══════════════════════════════════════════════════════════════
// DEMO DATA — positioned relative to drawn site centroid
// ══════════════════════════════════════════════════════════════
function injectDemo() {
  const [clng,clat] = turf.centroid(S.site).geometry.coordinates;
  const demo=[
    { ref:'DEMO/2023/0845/FULM', desc:'Erection of 92 residential dwellings with associated access, open space, drainage and landscaping infrastructure', addr:'Land North of Brixton Road', units:92, ha:2.8, status:'Approved', dlat:0.018, dlng:0.014, lpa:S.lpas[0]?.name||'Demo LPA', docs:['Location Plan.pdf','Site Layout Drawing.pdf','Design and Access Statement.pdf'], receivedDate:'2023-08-15' },
    { ref:'DEMO/2022/1112/EIA',  desc:'Environmental Impact Assessment: mixed-use development comprising 240 residential units (C3) and 1,200sqm commercial (E class) with public realm improvements', addr:'Former Gasworks, Station Road', units:240, ha:5.4, status:'Approved with conditions', dlat:-0.023, dlng:0.028, lpa:S.lpas[0]?.name||'Demo LPA', docs:['Site Location Plan.pdf','Environmental Statement - Chapter 1.pdf','Transport Assessment.pdf'], receivedDate:'2022-11-08' },
    { ref:'DEMO/2024/0076/REMM', desc:'Reserved matters — appearance, landscaping, layout and scale for 38 residential dwellings pursuant to outline permission DEMO/2021/0344/OUTM', addr:'Meadow Farm, Northside Lane', units:38, ha:1.1, status:'Pending consideration', dlat:0.033, dlng:-0.019, lpa:S.lpas[1]?.name||S.lpas[0]?.name||'Demo LPA', docs:['Site Plan.pdf','Landscape Strategy.pdf'], receivedDate:'2024-02-12' },
    { ref:'DEMO/2021/0899/OUTM', desc:'Outline application with all matters reserved except access for up to 180 dwellings, vehicular access, public open space and associated infrastructure', addr:'Greenfield Allocation, Western Edge', units:180, ha:4.2, status:'Approved', dlat:-0.037, dlng:-0.022, lpa:S.lpas[0]?.name||'Demo LPA', docs:['Location Plan.pdf','Flood Risk Assessment.pdf'], receivedDate:'2021-09-20' },
    { ref:'DEMO/2023/0567/S73',  desc:'Section 73 application to vary condition 4 (approved plans) of planning permission DEMO/2022/0123/FULM to amend house types on plots 14–22 — total 44 dwellings', addr:'Olympia Quarter Phase 2', units:44, ha:0.9, status:'Approved', dlat:0.006, dlng:0.038, lpa:S.lpas[0]?.name||'Demo LPA', docs:['Revised Site Layout.pdf','Planning Statement.pdf'], receivedDate:'2023-06-03' },
    { ref:'DEMO/2020/1234/OUTM', desc:'Outline application for residential development of up to 65 dwellings (Use Class C3) with public open space and associated infrastructure', addr:'Land South of Church Lane', units:65, ha:1.7, status:'Refused', dlat:-0.011, dlng:-0.033, lpa:S.lpas[0]?.name||'Demo LPA', docs:['Location Plan.pdf','Heritage Impact Assessment.pdf'], receivedDate:'2020-12-01' },
  ];

  S.results = demo.map(d=>{
    const lat=clat+d.dlat, lng=clng+d.dlng;
    const pt=turf.point([lng,lat]);
    let distKm=0;
    try{ const line=turf.polygonToLine(S.site); distKm=+turf.distance(pt,turf.nearestPointOnLine(line,pt),{units:'kilometers'}).toFixed(3); }
    catch{ distKm=+turf.distance(pt,turf.centroid(S.site),{units:'kilometers'}).toFixed(3); }
    return {
      reference:d.ref, description:d.desc, address:d.addr,
      units:d.units, siteHa:d.ha, status:d.status,
      source:'Demo', lpa:d.lpa, lat, lng, distKm,
      receivedDate:d.receivedDate, portalUrl:null,
      _docs:d.docs.map(n=>({ name:n, url:null, type:docType(n) })),
    };
  }).filter(r=>turf.booleanPointInPolygon(turf.point([r.lng,r.lat]),S.buffer));

  sortResults();
}

function docType(name) {
  const n=name.toLowerCase();
  if (/location|site\s+plan|red\s*line/.test(n)) return 'SITE PLAN';
  if (/transport/.test(n)) return 'TRANSPORT';
  if (/heritage|historic/.test(n)) return 'HERITAGE';
  if (/design/.test(n)) return 'D&A';
  if (/environment|eia|es/.test(n)) return 'EIA';
  if (/flood/.test(n)) return 'FRA';
  if (/landscape/.test(n)) return 'LANDSCAPE';
  return 'DOCUMENT';
}

// ══════════════════════════════════════════════════════════════
// RENDER RESULTS
// ══════════════════════════════════════════════════════════════
function sortResults() {
  const k=S.sortKey;
  S.results.sort((a,b)=>{
    if (k==='dist')  return (a.distKm||99)-(b.distKm||99);
    if (k==='units') return (b.units||0)-(a.units||0);
    if (k==='date')  return new Date(b.receivedDate||0)-new Date(a.receivedDate||0);
    return 0;
  });
}

function setSort(k) {
  S.sortKey=k;
  document.querySelectorAll('.sort-btn').forEach(b=>b.classList.toggle('on',b.dataset.k===k));
  sortResults();
  renderResults();
}

function renderResults() {
  const el=document.getElementById('resScroll');
  const n=S.results.length;

  if (!n) {
    el.innerHTML=`<div class="empty-state"><div class="es-icon">◎</div><div class="es-title">No qualifying schemes</div><div class="es-sub">Adjust threshold filters or expand the buffer radius.</div></div>`;
    document.getElementById('expBar').classList.remove('on');
    return;
  }

  el.innerHTML=`
    <div class="res-header">
      <div><div class="res-count">${n}</div><div class="res-sub">qualifying schemes within ${document.getElementById('bufKm').value}km buffer</div></div>
      <div class="res-sort">
        <button class="sort-btn on" data-k="dist" onclick="setSort('dist')">Distance</button>
        <button class="sort-btn" data-k="units" onclick="setSort('units')">Units</button>
        <button class="sort-btn" data-k="date" onclick="setSort('date')">Date</button>
      </div>
    </div>
    <div class="app-list">${S.results.map((r,i)=>cardHTML(r,i)).join('')}</div>`;

  document.getElementById('expBar').classList.add('on');
  document.querySelectorAll('.sort-btn').forEach(b=>b.classList.toggle('on',b.dataset.k===S.sortKey));
}

function cardHTML(r,i) {
  const stripe=getStripeClass(r);
  const sc=getStatusClass(r.status);
  const hasDocs=r._docs&&r._docs.length>0;
  return `
    <div class="app-card" id="ac${i}" onclick="focusApp(${i})">
      <div class="card-top">
        <div class="card-stripe ${stripe}"></div>
        <div class="card-body">
          <div class="card-ref">${r.reference||'—'} · ${r.lpa||r.source}</div>
          <div class="card-desc">${r.description||'No description'}</div>
        </div>
      </div>
      <div class="card-chips">
        ${r.units?`<span class="chip ch-units">${r.units} units</span>`:''}
        ${r.siteHa?`<span class="chip ch-ha">${r.siteHa.toFixed(1)} ha</span>`:''}
        ${r._unquant?`<span class="chip ch-unquant">unquantified</span>`:''}
        ${r.status?`<span class="chip ${sc}">${r.status}</span>`:''}
        ${r.distKm!=null?`<span class="chip ch-dist">${r.distKm}km</span>`:''}
        ${r.receivedDate?`<span class="chip ch-lpa">${r.receivedDate.slice(0,10)}</span>`:''}
        ${hasDocs?`<span class="chip" style="background:var(--blue-l);color:var(--blue);">📄 ${r._docs.length} doc${r._docs.length>1?'s':''}</span>`:''}
        ${r.portalUrl?`<a class="card-portal" href="${r.portalUrl}" target="_blank" onclick="event.stopPropagation()">Portal →</a>`:''}
        ${hasDocs?`<button class="card-portal" style="border-color:rgba(191,59,26,0.3);color:var(--red);" onclick="event.stopPropagation();openPDF(${i})">View plans →</button>`:''}
      </div>
    </div>`;
}

// ══════════════════════════════════════════════════════════════
// MAP MARKERS
// ══════════════════════════════════════════════════════════════
function renderMarkers() {
  S.resL.clearLayers();
  S.markers=[];
  S.results.forEach((r,i)=>{
    if (!r.lat||!r.lng){ S.markers.push(null); return; }
    const isMaj=r.description&&/eia|environmental impact/i.test(r.description);
    const isLg=(r.siteHa||0)>=0.5||(r.units||0)>=50;
    const c=isMaj?'#bf3b1a':isLg?'#2d6b3e':'#1a4e8c';
    const icon=L.divIcon({
      html:`<div style="width:24px;height:24px;border-radius:50%;background:${c};border:2.5px solid white;box-shadow:0 1px 8px rgba(0,0,0,0.25);display:flex;align-items:center;justify-content:center;font-family:'Courier Prime',monospace;font-size:9px;font-weight:700;color:white;">${i+1}</div>`,
      iconSize:[24,24],iconAnchor:[12,12],className:''
    });

    const hasDocs=r._docs&&r._docs.length>0;
    const pop=`
      <div class="popup-ref">${r.reference||'—'} · ${r.lpa||r.source}</div>
      <div class="popup-desc">${(r.description||'').slice(0,100)}${(r.description||'').length>100?'…':''}</div>
      <div class="popup-chips">
        ${r.units?`<span class="chip ch-units" style="font-family:'Courier Prime',monospace;font-size:8px;padding:2px 5px;border-radius:2px;background:var(--blue-l);color:var(--blue);">${r.units} units</span>`:''}
        ${r.siteHa?`<span class="chip" style="font-family:'Courier Prime',monospace;font-size:8px;padding:2px 5px;border-radius:2px;background:var(--green-l);color:var(--green);">${r.siteHa.toFixed(1)} ha</span>`:''}
        ${r.distKm!=null?`<span style="font-size:9px;color:var(--muted);">${r.distKm}km from site edge</span>`:''}
      </div>
      ${r.address?`<div class="popup-addr">${r.address}</div>`:''}
      <div class="popup-links">
        ${r.portalUrl?`<a class="popup-btn" href="${r.portalUrl}" target="_blank">Portal →</a>`:''}
        ${hasDocs?`<button class="popup-btn red" onclick="openPDF(${i})">📄 View site plans</button>`:''}
        <button class="popup-btn" onclick="document.getElementById('ac${i}').scrollIntoView({behavior:'smooth',block:'nearest'});switchTab('res')">List →</button>
      </div>`;

    const m=L.marker([r.lat,r.lng],{icon}).bindPopup(pop,{maxWidth:280}).addTo(S.resL);
    m.on('click',()=>{
      document.querySelectorAll('.app-card').forEach((c,j)=>c.classList.toggle('sel',j===i));
    });
    S.markers.push(m);
  });
}

function focusApp(i) {
  const r=S.results[i];
  document.querySelectorAll('.app-card').forEach((c,j)=>c.classList.toggle('sel',j===i));
  if (r.lat&&r.lng) { map.setView([r.lat,r.lng],15,{animate:true}); if(S.markers[i]) S.markers[i].openPopup(); }
}

// ══════════════════════════════════════════════════════════════
// PDF / SITE PLAN VIEWER
// ══════════════════════════════════════════════════════════════
function openPDF(i) {
  const r=S.results[i];
  document.getElementById('pdfTitle').textContent = r.reference||r.description?.slice(0,40)||'Application';
  document.getElementById('pdfSub').textContent   = r.address||r.lpa||'';

  const docList=document.getElementById('pdfDocList');
  if (!r._docs||!r._docs.length) {
    docList.innerHTML='<div style="font-size:10px;color:var(--muted);padding:4px 0;">No documents found for this application.</div>';
  } else {
    // Flag site plan docs first
    const sorted=[...r._docs].sort((a,b)=>{ const sp=/site.?plan|location.?plan|red.?line/i; return sp.test(b.name)?1:sp.test(a.name)?-1:0; });
    docList.innerHTML=sorted.map((d,j)=>{
      const isSitePlan=/site.?plan|location.?plan|red.?line/i.test(d.name);
      return `<button class="pdf-doc-btn ${isSitePlan?'on':''}" id="db${j}" onclick="loadDocFrame('${d.url||''}','${d.name}',${j})">
        <span class="pdf-doc-icon">${isSitePlan?'🗺':'📄'}</span>
        <span style="flex:1;min-width:0;">
          <div class="pdf-doc-name">${d.name}</div>
          <div class="pdf-doc-type">${d.type||docType(d.name)}</div>
        </span>
        ${isSitePlan?`<span style="font-family:'Courier Prime',monospace;font-size:8px;color:var(--red);border:1px solid rgba(191,59,26,0.3);padding:1px 4px;border-radius:2px;">SITE PLAN</span>`:''}
      </button>`;
    }).join('');

    // Auto-load first site plan if exists
    const first=sorted[0];
    if (first) loadDocFrame(first.url||'', first.name, 0);
  }

  // If portal URL exists, try to fetch actual documents
  if (r.portalUrl) fetchDocs(r,i);

  switchTab('pdf');
  openResults();
}

async function fetchDocs(r,i) {
  // Try to load the documents tab from the Idox portal
  if (!r.portalUrl) return;
  const docsUrl=r.portalUrl.replace('activeTab=summary','activeTab=documents');
  try {
    const resp=await proxyFetch(docsUrl);
    const html=await resp.text();
    const doc=new DOMParser().parseFromString(html,'text/html');
    const found=[];
    doc.querySelectorAll('a[href*=".pdf"],[href*="download"]').forEach(a=>{
      const href=a.getAttribute('href');
      const name=(a.textContent||a.title||href).trim().slice(0,80);
      if (href&&name&&!found.find(f=>f.url===href)) {
        found.push({ name, url:href.startsWith('http')?href:new URL(href,r.portalUrl).href, type:docType(name) });
      }
    });
    if (found.length) {
      S.results[i]._docs=found;
      openPDF(i); // re-render with real docs
    }
  } catch {}
}

function loadDocFrame(url, name, idx) {
  document.querySelectorAll('.pdf-doc-btn').forEach((b,j)=>b.classList.toggle('on',j===idx));
  const wrap=document.getElementById('pdfFrameWrap');
  const note=document.getElementById('pdfNote');
  const isSitePlan=/site.?plan|location.?plan|red.?line/i.test(name);

  if (!url) {
    wrap.innerHTML=`
      <div class="pdf-empty">
        <div class="pdf-empty-icon">🔒</div>
        <div class="pdf-empty-txt">
          <strong>${name}</strong><br><br>
          This document is hosted on the council's planning portal. CORS restrictions prevent direct embedding.<br><br>
          <a href="#" style="color:var(--blue)">Open portal to view</a>
        </div>
      </div>`;
  } else {
    wrap.innerHTML=`<iframe src="${url}" allowfullscreen title="${name}"></iframe>`;
  }

  if (isSitePlan) { note.style.display='block'; } else { note.style.display='none'; }
}

function clearPDF() {
  document.getElementById('pdfTitle').textContent='No application selected';
  document.getElementById('pdfSub').textContent='Select a result to view its planning documents';
  document.getElementById('pdfDocList').innerHTML='<div style="font-size:10px;color:var(--muted);padding:4px 0;">No documents loaded.</div>';
  document.getElementById('pdfFrameWrap').innerHTML=`<div class="pdf-empty"><div class="pdf-empty-icon">📄</div><div class="pdf-empty-txt">Select an application from the results list, then choose a document above to view it.</div></div>`;
  document.getElementById('pdfNote').style.display='none';
  switchTab('res');
}

// ══════════════════════════════════════════════════════════════
// EXPORTS
// ══════════════════════════════════════════════════════════════
function exportCSV() {
  const cols=['reference','description','address','units','siteHa','status','lpa','source','distKm','receivedDate','lat','lng','portalUrl'];
  const hdr=cols.join(',');
  const rows=S.results.map(r=>cols.map(c=>`"${String(r[c]??'').replace(/"/g,'""')}"`).join(','));
  dl([hdr,...rows].join('\n'), `cumulative_planning_${today()}.csv`, 'text/csv');
  notify('CSV exported.');
}

function exportGeoJSON() {
  const fc={ type:'FeatureCollection', features:[
    ...(S.site?[{ type:'Feature', geometry:S.site.geometry||S.site, properties:{ layer:'site_boundary', area_ha:+document.getElementById('siteHa').textContent } }]:[]),
    ...(S.buffer?[{ type:'Feature', geometry:S.buffer.geometry||S.buffer, properties:{ layer:'search_buffer', radius_km:+document.getElementById('bufKm').value } }]:[]),
    ...S.results.filter(r=>r.lat&&r.lng).map((r,i)=>({
      type:'Feature',
      geometry:{ type:'Point', coordinates:[r.lng,r.lat] },
      properties:{ n:i+1, reference:r.reference, description:r.description, address:r.address, units:r.units, site_ha:r.siteHa, status:r.status, lpa:r.lpa, source:r.source, dist_km:r.distKm, received:r.receivedDate, portal:r.portalUrl }
    }))
  ]};
  dl(JSON.stringify(fc,null,2), `cumulative_planning_${today()}.geojson`, 'application/json');
  notify('GeoJSON exported — includes site boundary, buffer and all schemes.');
}

function exportKML() {
  const marks=S.results.filter(r=>r.lat&&r.lng).map((r,i)=>{
    const c=((r.siteHa||0)>=0.5||(r.units||0)>=50)?'ff2d6b3e':'ff1a4e8c';
    return `<Placemark><name>${r.reference||'App '+(i+1)}</name>
      <description><![CDATA[<b>${r.description||''}</b><br/>${r.address||''}<br/>Units: ${r.units||'—'} | ${r.siteHa||'—'} ha<br/>Status: ${r.status||'—'}<br/>LPA: ${r.lpa||''}<br/>Dist: ${r.distKm||'—'}km]]></description>
      <Style><IconStyle><color>ff${c}</color><scale>0.8</scale></IconStyle></Style>
      <Point><coordinates>${r.lng},${r.lat},0</coordinates></Point></Placemark>`;
  }).join('\n');
  const kml=`<?xml version="1.0" encoding="UTF-8"?><kml xmlns="http://www.opengis.net/kml/2.2"><Document><name>Cumulative Planning Search ${today()}</name>\n${marks}\n</Document></kml>`;
  dl(kml, `cumulative_planning_${today()}.kml`, 'application/vnd.google-earth.kml+xml');
  notify('KML exported — open in QGIS, ArcGIS or Google Earth.');
}

function dl(content, filename, mime) {
  const a=document.createElement('a');
  a.href=URL.createObjectURL(new Blob([content],{type:mime}));
  a.download=filename; a.click(); URL.revokeObjectURL(a.href);
}
function today(){ return new Date().toISOString().split('T')[0]; }

// ══════════════════════════════════════════════════════════════
// FILE UPLOAD
// ══════════════════════════════════════════════════════════════
function loadFile(e) {
  const f=e.target.files[0]; if(!f) return;
  const rd=new FileReader();
  rd.onload=ev=>{
    try {
      let gj;
      if (f.name.endsWith('.kml')) gj=kmlParse(ev.target.result);
      else gj=JSON.parse(ev.target.result);
      const feat=gj.type==='FeatureCollection'?gj.features[0]:gj;
      drawn.clearLayers();
      const l=L.geoJSON(feat).addTo(drawn);
      map.fitBounds(l.getBounds(),{padding:[30,30]});
      processSite(feat.geometry||feat);
      notify('Boundary loaded from file.','ok');
    } catch { notify('Could not parse file — use GeoJSON or KML.','err'); }
  };
  rd.readAsText(f);
  e.target.value='';
}

function kmlParse(txt) {
  const doc=new DOMParser().parseFromString(txt,'text/xml');
  const c=doc.querySelector('coordinates')?.textContent?.trim();
  if (!c) throw new Error('No coordinates in KML');
  const ring=c.split(/\s+/).filter(Boolean).map(s=>{ const[lng,lat]=s.split(',').map(Number); return [lng,lat]; });
  return { type:'Feature', geometry:{ type:'Polygon', coordinates:[ring] } };
}

// ══════════════════════════════════════════════════════════════
// UI HELPERS
// ══════════════════════════════════════════════════════════════
function clearAll() {
  drawn.clearLayers();
  if(S.siteL){ map.removeLayer(S.siteL); S.siteL=null; }
  if(S.bufL){  map.removeLayer(S.bufL);  S.bufL=null; }
  S.resL.clearLayers();
  S.site=S.buffer=null; S.results=[]; S.lpas=[]; S.markers=[];
  document.getElementById('siteStats').style.display='none';
  document.getElementById('lpaSection').style.display='none';
  document.getElementById('searchBtn').disabled=true;
  document.getElementById('srchHint').textContent='Draw a site boundary to enable search';
  document.getElementById('stTotal').textContent='—';
  document.getElementById('stUnits').textContent='—';
  document.getElementById('stLPAs').textContent='—';
  document.getElementById('resScroll').innerHTML=`<div class="empty-state"><div class="es-icon">◎</div><div class="es-title">No results yet</div><div class="es-sub">Draw a site boundary and run a search.</div></div>`;
  document.getElementById('expBar').classList.remove('on');
  document.getElementById('progLog').classList.remove('on');
  document.getElementById('progLog').innerHTML='';
  [1,2,3,4,5].forEach(n=>{ const s=document.getElementById('stp'+n); if(s){ s.classList.remove('on','done'); if(n===1)s.classList.add('on'); }});
  setStatus('idle','Ready');
  closeResults();
}

function startDraw() {
  document.querySelectorAll('.tbtn').forEach(b=>b.classList.remove('active'));
  document.getElementById('btnDraw').classList.add('active');
  try{ drawCtrl._toolbars.draw._modes.polygon.handler.enable(); }catch{}
}

function fitSite() {
  if (!S.site) return;
  const b=turf.bbox(S.site);
  map.fitBounds([[b[1],b[0]],[b[3],b[2]]],{padding:[40,40]});
}

function fitAll() {
  const layers=[];
  if(S.bufL) layers.push(S.bufL);
  if(S.siteL) layers.push(S.siteL);
  if(!layers.length) return;
  map.fitBounds(L.featureGroup(layers).getBounds(),{padding:[40,40]});
}

function openResults() {
  document.getElementById('panelR').classList.add('open');
  S.panelOpen=true;
}
function closeResults() {
  document.getElementById('panelR').classList.remove('open');
  S.panelOpen=false;
}

function switchTab(id) {
  ['res','pdf'].forEach(t=>{
    document.getElementById('tab'+t.charAt(0).toUpperCase()+t.slice(1)).classList.toggle('on',t===id);
    document.getElementById('pane'+t.charAt(0).toUpperCase()+t.slice(1)).classList.toggle('on',t===id);
  });
}

let baseIdx=0;
function cycleBase() {
  baseIdx=(baseIdx+1)%TILES.length;
  map.removeLayer(tileLayer);
  tileLayer=L.tileLayer(TILES[baseIdx].url,{attribution:TILES[baseIdx].attr,maxZoom:19}).addTo(map);
  document.getElementById('mcBase').textContent=TILES[baseIdx].name;
}

function toggleSat() {
  S.satOn=!S.satOn;
  if(S.satOn){ satLayer.setOpacity(0.8); map.addLayer(satLayer); }
  else{ map.removeLayer(satLayer); }
  document.getElementById('mcSat').classList.toggle('on',S.satOn);
}

function stepDone(n){ const e=document.getElementById('stp'+n); if(e){ e.classList.remove('on'); e.classList.add('done'); }}
function stepActive(n){ const e=document.getElementById('stp'+n); if(e){ e.classList.add('on'); e.classList.remove('done'); }}

function setStatus(type,txt) {
  document.getElementById('stDot').className='st-dot '+type;
  document.getElementById('stTxt').textContent=txt;
}

function setBtnWorking(on) {
  const b=document.getElementById('searchBtn');
  b.disabled=on;
  b.innerHTML=on
    ?'<div class="spin"></div> Searching databases…'
    :`<svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>Search Planning Databases`;
}

function addLog(type,txt) {
  const l=document.getElementById('progLog');
  const t=new Date().toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const ic=type==='ok'?'<span class="log-ok">✓</span>':type==='err'?'<span class="log-err">✗</span>':'<span style="color:#555">·</span>';
  l.innerHTML+=`<div class="log-line"><span class="log-t">${t}</span>${ic}<span class="log-txt">${txt}</span></div>`;
  l.scrollTop=l.scrollHeight;
}

function showBanner(){ document.getElementById('drawBanner').classList.add('on'); }
function hideBanner(){ document.getElementById('drawBanner').classList.remove('on'); }

let ntTimer;
function notify(msg,type='info') {
  const n=document.getElementById('notif');
  n.textContent=msg;
  n.className='notif on'+(type!=='info'?' '+type:'');
  clearTimeout(ntTimer);
  ntTimer=setTimeout(()=>n.classList.remove('on'),4500);
}

// ─── init ───
setTimeout(()=>notify('Draw a site boundary polygon to begin. Use the polygon tool in the left panel.'),700);
</script>
</body>
</html>
