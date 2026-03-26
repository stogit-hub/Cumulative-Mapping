[cumulative_planning_search (2).html](https://github.com/user-attachments/files/26280814/cumulative_planning_search.2.html)
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
  --purple:   #6b2d8b;
  --purple-l: rgba(107,45,139,0.08);
  --panel-l:  300px;
  --panel-r:  370px;
  --top:      46px;
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html,body{height:100%;overflow:hidden}
body{font-family:'Barlow',sans-serif;background:var(--paper);color:var(--ink);font-size:13px;line-height:1.5}

/* TOPBAR */
.topbar{position:fixed;top:0;left:0;right:0;height:var(--top);background:var(--ink);z-index:2000;display:flex;align-items:center;padding:0 16px;gap:12px;border-bottom:1px solid rgba(255,255,255,0.07);}
.tb-logo{font-family:'Barlow Condensed',sans-serif;font-size:16px;font-weight:600;letter-spacing:0.06em;color:#f4f1e8;text-transform:uppercase;white-space:nowrap;}
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

/* LAYOUT */
.shell{display:flex;height:100vh;padding-top:var(--top);}

/* LEFT PANEL */
.panel-l{width:var(--panel-l);flex-shrink:0;background:var(--paper);border-right:1px solid var(--rule);display:flex;flex-direction:column;overflow:hidden;z-index:100;}
.panel-scroll{flex:1;overflow-y:auto;overflow-x:hidden;}
.panel-scroll::-webkit-scrollbar{width:3px;}
.panel-scroll::-webkit-scrollbar-thumb{background:var(--rule);border-radius:2px;}

/* RIGHT PANEL */
.panel-r{width:var(--panel-r);flex-shrink:0;background:var(--paper);border-left:1px solid var(--rule);display:flex;flex-direction:column;overflow:hidden;z-index:100;transform:translateX(var(--panel-r));transition:transform 0.28s cubic-bezier(0.4,0,0.2,1);}
.panel-r.open{transform:translateX(0);}

/* MAP */
.map-wrap{flex:1;position:relative;overflow:hidden;}
#map{width:100%;height:100%;}

/* SECTION HEADERS */
.sec{padding:14px 16px;border-bottom:1px solid var(--rule2);}
.sec:last-child{border-bottom:none;}
.sec-label{font-family:'Courier Prime',monospace;font-size:9px;font-weight:700;text-transform:uppercase;letter-spacing:0.12em;color:var(--muted);margin-bottom:10px;display:flex;align-items:center;gap:8px;}
.sec-label::after{content:'';flex:1;height:1px;background:var(--rule);}

/* DRAW TOOLS */
.tool-grid{display:grid;grid-template-columns:1fr 1fr;gap:5px;}
.tbtn{display:flex;flex-direction:column;align-items:center;gap:4px;padding:9px 6px;border:1px solid var(--rule);border-radius:4px;background:var(--paper2);cursor:pointer;transition:all 0.12s;font-family:'Barlow',sans-serif;color:var(--ink2);}
.tbtn:hover{border-color:var(--ink2);background:var(--paper3);}
.tbtn.active{border-color:var(--red);background:var(--red-l);color:var(--red);}
.tbtn svg{width:16px;height:16px;stroke-width:1.5;}
.tbtn-lbl{font-size:9px;font-weight:500;letter-spacing:0.03em;text-align:center;line-height:1.2;}

/* SITE STATS */
.site-stats{display:grid;grid-template-columns:1fr 1fr;gap:5px;margin-top:8px;}
.stat-box{background:var(--paper2);border:1px solid var(--rule);border-radius:4px;padding:7px 9px;}
.stat-lbl{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);letter-spacing:0.08em;text-transform:uppercase;margin-bottom:1px;}
.stat-val{font-family:'Barlow Condensed',sans-serif;font-size:20px;font-weight:300;color:var(--ink);line-height:1;}
.stat-unit{font-size:10px;color:var(--muted);}

/* LPA BADGES */
.lpa-list{display:flex;flex-direction:column;gap:4px;}
.lpa-badge{display:flex;align-items:center;gap:8px;padding:6px 9px;background:var(--paper2);border:1px solid var(--rule);border-radius:4px;}
.lpa-dot{width:6px;height:6px;border-radius:50%;background:var(--blue);flex-shrink:0;}
.lpa-name{flex:1;font-size:11px;font-weight:500;color:var(--ink);}
.lpa-url{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);}
.lpa-open{font-size:9px;color:var(--blue);text-decoration:none;padding:1px 5px;border:1px solid rgba(26,78,140,0.25);border-radius:2px;transition:background 0.12s;white-space:nowrap;}
.lpa-open:hover{background:var(--blue-l);}

/* FORM CONTROLS */
.f-grid{display:grid;grid-template-columns:1fr 1fr;gap:6px;}
.f-item{display:flex;flex-direction:column;gap:3px;}
.f-lbl{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);letter-spacing:0.06em;text-transform:uppercase;}
input[type="number"],input[type="text"],select,textarea{width:100%;padding:6px 8px;border:1px solid var(--rule);border-radius:3px;background:var(--paper2);color:var(--ink);font-family:'Courier Prime',monospace;font-size:11px;outline:none;transition:border-color 0.12s;-moz-appearance:textfield;}
input:focus,select:focus,textarea:focus{border-color:var(--red);}
input[type="number"]::-webkit-inner-spin-button{-webkit-appearance:none;}
textarea{resize:vertical;min-height:80px;line-height:1.6;}
select{cursor:pointer;}
.trow{display:flex;align-items:center;justify-content:space-between;margin-top:6px;}
.trow-lbl{font-size:11px;color:var(--ink2);}
.tog{position:relative;width:32px;height:17px;cursor:pointer;flex-shrink:0;}
.tog input{opacity:0;width:0;height:0;}
.tog-track{position:absolute;inset:0;border-radius:99px;background:var(--paper3);border:1px solid var(--rule);transition:background 0.2s;}
.tog input:checked~.tog-track{background:var(--red);border-color:var(--red);}
.tog-thumb{position:absolute;top:2px;left:2px;width:11px;height:11px;border-radius:50%;background:white;transition:transform 0.18s;pointer-events:none;}
.tog input:checked~.tog-thumb{transform:translateX(15px);}

/* SEARCH BUTTON */
.search-btn{width:100%;padding:10px;background:var(--ink);color:var(--paper);border:none;border-radius:4px;font-family:'Barlow Condensed',sans-serif;font-size:14px;font-weight:500;letter-spacing:0.08em;text-transform:uppercase;cursor:pointer;transition:all 0.15s;display:flex;align-items:center;justify-content:center;gap:8px;}
.search-btn:hover{background:var(--red);}
.search-btn:disabled{opacity:0.35;cursor:default;pointer-events:none;}
.search-btn .spin{width:12px;height:12px;border:2px solid rgba(244,241,232,0.25);border-top-color:#f4f1e8;border-radius:50%;animation:spin 0.7s linear infinite;}
@keyframes spin{to{transform:rotate(360deg)}}

/* PROGRESS LOG */
.prog-log{display:none;padding:8px 12px;background:#1e1d19;border-top:1px solid rgba(255,255,255,0.06);max-height:120px;overflow-y:auto;font-family:'Courier Prime',monospace;font-size:10px;line-height:1.9;}
.prog-log.on{display:block;}
.prog-log::-webkit-scrollbar{width:2px;}
.prog-log::-webkit-scrollbar-thumb{background:#444;}
.log-line{display:flex;gap:8px;}
.log-t{color:#e8a030;flex-shrink:0;}
.log-ok{color:#4a9e60;flex-shrink:0;}
.log-err{color:var(--red);flex-shrink:0;}
.log-warn{color:#e8a030;flex-shrink:0;}
.log-txt{color:rgba(244,241,232,0.55);}

/* PANEL TABS */
.panel-tabs{display:flex;border-bottom:1px solid var(--rule);flex-shrink:0;}
.p-tab{flex:1;padding:9px 8px;text-align:center;cursor:pointer;font-family:'Courier Prime',monospace;font-size:9px;font-weight:700;text-transform:uppercase;letter-spacing:0.1em;color:var(--muted);border-bottom:2px solid transparent;transition:all 0.15s;}
.p-tab:hover{color:var(--ink);}
.p-tab.on{color:var(--red);border-bottom-color:var(--red);}
.tab-pane{display:none;flex:1;flex-direction:column;overflow:hidden;}
.tab-pane.on{display:flex;}

/* RESULTS */
.res-header{display:flex;align-items:flex-start;justify-content:space-between;margin-bottom:10px;}
.res-count{font-family:'Barlow Condensed',sans-serif;font-size:28px;font-weight:300;color:var(--ink);line-height:1;}
.res-sub{font-size:10px;color:var(--muted);margin-top:2px;}
.res-sort{display:flex;gap:4px;flex-wrap:wrap;}
.sort-btn{font-family:'Courier Prime',monospace;font-size:8px;padding:3px 7px;border:1px solid var(--rule);border-radius:2px;background:transparent;color:var(--muted);cursor:pointer;transition:all 0.12s;letter-spacing:0.04em;text-transform:uppercase;}
.sort-btn:hover{border-color:var(--ink);color:var(--ink);}
.sort-btn.on{border-color:var(--red);color:var(--red);background:var(--red-l);}

/* SOURCE BADGES */
.src-badge{display:inline-flex;align-items:center;gap:3px;font-family:'Courier Prime',monospace;font-size:8px;padding:1px 5px;border-radius:2px;margin-left:4px;}
.src-pd{background:rgba(26,78,140,0.12);color:var(--blue);border:1px solid rgba(26,78,140,0.2);}
.src-gla{background:rgba(45,107,62,0.12);color:var(--green);border:1px solid rgba(45,107,62,0.2);}
.src-idox{background:rgba(176,107,0,0.12);color:var(--amber);border:1px solid rgba(176,107,0,0.2);}
.src-import{background:rgba(107,45,139,0.12);color:var(--purple);border:1px solid rgba(107,45,139,0.2);}

/* APP CARDS */
.app-list{display:flex;flex-direction:column;gap:4px;}
.app-card{border:1px solid var(--rule);border-radius:4px;overflow:hidden;cursor:pointer;transition:all 0.12s;background:var(--paper);}
.app-card:hover{border-color:rgba(30,29,25,0.28);box-shadow:0 2px 8px rgba(0,0,0,0.07);}
.app-card.sel{border-color:var(--red);border-width:1.5px;}
.card-top{display:flex;align-items:stretch;}
.card-stripe{width:3px;flex-shrink:0;}
.cs-blue{background:var(--blue);}
.cs-red{background:var(--red);}
.cs-green{background:var(--green);}
.cs-amber{background:var(--amber);}
.cs-purple{background:var(--purple);}
.card-body{flex:1;padding:8px 9px 6px;min-width:0;}
.card-ref{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);margin-bottom:2px;letter-spacing:0.02em;}
.card-desc{font-size:11px;color:var(--ink);line-height:1.35;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden;}
.card-chips{display:flex;gap:4px;flex-wrap:wrap;margin-top:5px;padding:0 9px 7px 12px;}
.chip{font-family:'Courier Prime',monospace;font-size:8px;padding:2px 5px;border-radius:2px;letter-spacing:0.02em;}
.ch-units{background:var(--blue-l);color:var(--blue);}
.ch-ha{background:var(--green-l);color:var(--green);}
.ch-pend{background:var(--amber-l);color:var(--amber);}
.ch-appr{background:var(--green-l);color:var(--green);}
.ch-ref{background:var(--red-l);color:var(--red);}
.ch-dist{background:var(--rule2);color:var(--muted);}
.ch-lpa{background:var(--paper3);color:var(--muted);}
.ch-unq{background:rgba(176,107,0,0.12);color:var(--amber);}
.card-portal{display:inline-block;font-size:9px;color:var(--blue);text-decoration:none;padding:1px 5px;border:1px solid rgba(26,78,140,0.25);border-radius:2px;margin-left:2px;transition:background 0.12s;}
.card-portal:hover{background:var(--blue-l);}
.card-plans{display:inline-block;font-size:9px;color:var(--red);text-decoration:none;padding:1px 5px;border:1px solid rgba(191,59,26,0.25);border-radius:2px;margin-left:2px;cursor:pointer;background:none;transition:background 0.12s;}
.card-plans:hover{background:var(--red-l);}

/* EXPORT BAR */
.export-bar{padding:8px 14px;border-top:1px solid var(--rule);flex-shrink:0;display:none;align-items:center;gap:6px;flex-wrap:wrap;}
.export-bar.on{display:flex;}
.exp-lbl{font-family:'Courier Prime',monospace;font-size:8px;text-transform:uppercase;letter-spacing:0.08em;color:var(--muted);margin-right:2px;}
.exp-btn{display:flex;align-items:center;gap:4px;padding:4px 8px;border:1px solid var(--rule);border-radius:2px;background:transparent;font-family:'Courier Prime',monospace;font-size:9px;color:var(--ink2);cursor:pointer;transition:all 0.12s;letter-spacing:0.03em;text-transform:uppercase;}
.exp-btn:hover{border-color:var(--ink);color:var(--ink);}

/* PDF VIEWER */
.pdf-header{display:flex;align-items:flex-start;justify-content:space-between;padding:10px 14px;border-bottom:1px solid var(--rule);flex-shrink:0;}
.pdf-title{font-size:11px;font-weight:500;color:var(--ink);line-height:1.3;}
.pdf-sub{font-size:9px;color:var(--muted);margin-top:1px;font-family:'Courier Prime',monospace;}
.pdf-close{background:none;border:none;cursor:pointer;color:var(--muted);font-size:18px;line-height:1;padding:0 2px;}
.pdf-close:hover{color:var(--ink);}
.pdf-docs{padding:8px 12px;border-bottom:1px solid var(--rule2);display:flex;flex-direction:column;gap:4px;max-height:140px;overflow-y:auto;}
.pdf-doc-btn{display:flex;align-items:center;gap:8px;padding:6px 8px;border:1px solid var(--rule);border-radius:3px;background:var(--paper2);cursor:pointer;transition:all 0.12s;text-align:left;width:100%;}
.pdf-doc-btn:hover{border-color:var(--blue);background:var(--blue-l);}
.pdf-doc-btn.on{border-color:var(--red);background:var(--red-l);}
.pdf-doc-name{font-size:10px;font-weight:500;color:var(--ink);flex:1;line-height:1.3;}
.pdf-doc-type{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);}
.pdf-frame-wrap{flex:1;position:relative;background:var(--paper3);}
.pdf-frame-wrap iframe{width:100%;height:100%;border:none;}
.pdf-empty{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:10px;height:100%;color:var(--muted);text-align:center;padding:20px;}
.pdf-empty-icon{font-size:28px;opacity:0.3;}
.pdf-empty-txt{font-size:11px;line-height:1.6;}
.pdf-note{padding:7px 12px;background:var(--amber-l);border-top:1px solid rgba(176,107,0,0.18);font-size:10px;color:var(--amber);line-height:1.5;flex-shrink:0;}

/* NO RESULTS / IMPORT */
.no-results-panel{padding:16px;display:flex;flex-direction:column;gap:10px;}
.nr-title{font-family:'Barlow Condensed',sans-serif;font-size:16px;font-weight:400;color:var(--ink);}
.nr-sub{font-size:11px;color:var(--muted);line-height:1.5;}
.source-status{display:flex;flex-direction:column;gap:5px;margin:8px 0;}
.ss-row{display:flex;align-items:center;gap:8px;padding:5px 8px;background:var(--paper2);border:1px solid var(--rule);border-radius:3px;font-size:10px;}
.ss-dot{width:7px;height:7px;border-radius:50%;flex-shrink:0;}
.ss-ok{background:var(--green);}
.ss-err{background:var(--red);}
.ss-skip{background:var(--muted);}
.ss-name{flex:1;font-weight:500;color:var(--ink);}
.ss-msg{font-family:'Courier Prime',monospace;font-size:9px;color:var(--muted);}

/* IMPORT SECTION */
.import-box{background:var(--paper2);border:1px solid var(--rule);border-radius:4px;padding:10px 12px;}
.import-title{font-family:'Barlow Condensed',sans-serif;font-size:13px;font-weight:500;color:var(--ink);margin-bottom:4px;display:flex;align-items:center;gap:6px;}
.import-sub{font-size:10px;color:var(--muted);line-height:1.5;margin-bottom:8px;}
.import-format{font-family:'Courier Prime',monospace;font-size:9px;color:var(--muted);background:var(--paper3);padding:4px 7px;border-radius:2px;margin-bottom:8px;line-height:1.6;}
.btn-sm{padding:5px 10px;border:1px solid var(--rule);border-radius:3px;background:transparent;font-family:'Courier Prime',monospace;font-size:9px;color:var(--ink2);cursor:pointer;transition:all 0.12s;letter-spacing:0.04em;text-transform:uppercase;display:inline-flex;align-items:center;gap:4px;}
.btn-sm:hover{border-color:var(--ink);color:var(--ink);}
.btn-sm.primary{background:var(--ink);color:var(--paper);border-color:var(--ink);}
.btn-sm.primary:hover{background:var(--red);border-color:var(--red);}

/* LPA LABEL on map */
.lpa-map-label{
  font-family:'Courier Prime',monospace;
  font-size:9px;
  font-weight:700;
  color:var(--purple);
  background:rgba(244,241,232,0.82);
  padding:1px 4px;
  border-radius:2px;
  white-space:nowrap;
  pointer-events:none;
  letter-spacing:0.05em;
  text-transform:uppercase;
  border:1px solid rgba(107,45,139,0.2);
}

/* MAP OVERLAY */
.map-legend{position:absolute;bottom:24px;left:14px;z-index:500;background:rgba(244,241,232,0.94);border:1px solid var(--rule);border-radius:4px;padding:9px 11px;min-width:155px;backdrop-filter:blur(6px);pointer-events:none;}
.leg-title{font-family:'Courier Prime',monospace;font-size:8px;font-weight:700;text-transform:uppercase;letter-spacing:0.1em;color:var(--muted);margin-bottom:7px;}
.leg-row{display:flex;align-items:center;gap:7px;margin-bottom:4px;font-size:10px;color:var(--ink);}
.leg-swatch{width:20px;height:10px;border-radius:2px;flex-shrink:0;}
.ls-site{background:var(--red-l);border:2px solid var(--red);}
.ls-buf{background:rgba(26,78,140,0.07);border:2px dashed var(--blue);}
.ls-std{background:var(--blue-l);border:1.5px solid var(--blue);}
.ls-maj{background:var(--red-l);border:1.5px solid var(--red);}
.ls-lg{background:var(--green-l);border:1.5px solid var(--green);}
.ls-lpa{background:transparent;border:1.5px dashed var(--purple);}

.draw-banner{position:absolute;top:14px;left:50%;transform:translateX(-50%);background:var(--ink);color:var(--paper);font-family:'Courier Prime',monospace;font-size:11px;letter-spacing:0.04em;padding:7px 16px;border-radius:99px;z-index:800;white-space:nowrap;pointer-events:none;opacity:0;transition:opacity 0.25s;}
.draw-banner.on{opacity:1;}

.map-ctrl{position:absolute;top:14px;right:14px;z-index:500;display:flex;flex-direction:column;gap:5px;}
.mc-btn{display:flex;align-items:center;gap:6px;padding:6px 10px;background:rgba(244,241,232,0.94);border:1px solid var(--rule);border-radius:4px;font-family:'Courier Prime',monospace;font-size:9px;color:var(--ink2);cursor:pointer;transition:all 0.12s;white-space:nowrap;letter-spacing:0.04em;text-transform:uppercase;backdrop-filter:blur(6px);}
.mc-btn:hover{border-color:var(--ink2);}
.mc-btn.on{border-color:var(--red);color:var(--red);background:rgba(191,59,26,0.06);}
.mc-btn svg{width:12px;height:12px;}

/* STEPS */
.steps{display:flex;flex-direction:column;gap:4px;}
.step{display:flex;align-items:flex-start;gap:8px;padding:5px 7px;border-radius:3px;transition:background 0.15s;}
.step.on{background:var(--rule2);}
.step.done{opacity:0.45;}
.step-n{font-family:'Courier Prime',monospace;font-size:9px;color:var(--muted);width:14px;flex-shrink:0;margin-top:1px;}
.step.on .step-n{color:var(--red);}
.step-t{font-size:11px;color:var(--ink2);line-height:1.4;}

/* NOTIFICATION */
.notif{position:fixed;bottom:20px;right:20px;z-index:9999;background:var(--ink);color:var(--paper);padding:9px 14px;border-radius:4px;font-size:11px;max-width:320px;line-height:1.5;box-shadow:0 4px 20px rgba(0,0,0,0.2);transform:translateY(60px);opacity:0;transition:all 0.25s;pointer-events:none;}
.notif.on{transform:translateY(0);opacity:1;}
.notif.warn{background:#7a5500;}
.notif.err{background:var(--red);}
.notif.ok{background:var(--green);}

/* LEAFLET */
.leaflet-container{font-family:'Barlow',sans-serif;}
.leaflet-popup-content-wrapper{background:var(--paper);border:1px solid var(--rule);border-radius:4px;box-shadow:0 4px 20px rgba(0,0,0,0.12);color:var(--ink);}
.leaflet-popup-tip{background:var(--paper);}
.leaflet-popup-content{margin:10px 12px;}
.leaflet-bar a{background:var(--paper) !important;color:var(--muted) !important;border-color:var(--rule) !important;}
.leaflet-bar a:hover{color:var(--ink) !important;}
.leaflet-draw-toolbar a{background-color:var(--paper) !important;}
.popup-ref{font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);margin-bottom:3px;}
.popup-desc{font-size:11px;font-weight:500;color:var(--ink);line-height:1.4;margin-bottom:5px;}
.popup-chips{display:flex;gap:3px;flex-wrap:wrap;margin-bottom:6px;}
.popup-addr{font-size:10px;color:var(--muted);margin-bottom:6px;}
.popup-links{display:flex;gap:5px;flex-wrap:wrap;}
.popup-btn{font-size:10px;color:var(--blue);text-decoration:none;padding:3px 7px;border:1px solid rgba(26,78,140,0.3);border-radius:2px;cursor:pointer;background:none;}
.popup-btn:hover{background:var(--blue-l);}
.popup-btn.red{color:var(--red);border-color:rgba(191,59,26,0.3);}
.popup-btn.red:hover{background:var(--red-l);}
</style>
</head>
<body>

<!-- TOPBAR -->
<header class="topbar">
  <div class="tb-logo">Cumulative <span>Planning</span> Search</div>
  <div class="tb-div"></div>
  <div class="tb-sub">UK Residential · Real Data</div>
  <div class="tb-spacer"></div>
  <div class="tb-stats">
    <div class="tb-stat"><div class="tb-stat-val" id="stTotal">—</div><div class="tb-stat-lbl">Results</div></div>
    <div class="tb-stat"><div class="tb-stat-val" id="stUnits">—</div><div class="tb-stat-lbl">Total units</div></div>
    <div class="tb-stat"><div class="tb-stat-val" id="stLPAs">—</div><div class="tb-stat-lbl">LPAs</div></div>
  </div>
  <div class="tb-div"></div>
  <div class="tb-status"><div class="st-dot idle" id="stDot"></div><span id="stTxt">Ready</span></div>
</header>

<div class="draw-banner" id="drawBanner">Click to place vertices — double-click to close polygon</div>

<div class="shell">

<!-- LEFT PANEL -->
<aside class="panel-l">
  <div class="panel-scroll">

    <div class="sec">
      <div class="sec-label">Workflow</div>
      <div class="steps">
        <div class="step on"  id="stp1"><div class="step-n">01</div><div class="step-t">Draw site boundary polygon</div></div>
        <div class="step"     id="stp2"><div class="step-n">02</div><div class="step-t">LPA(s) detected — boundaries loaded</div></div>
        <div class="step"     id="stp3"><div class="step-n">03</div><div class="step-t">5km buffer from polygon edge</div></div>
        <div class="step"     id="stp4"><div class="step-n">04</div><div class="step-t">Live databases queried</div></div>
        <div class="step"     id="stp5"><div class="step-n">05</div><div class="step-t">Site plans retrieved and listed</div></div>
      </div>
    </div>

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
          <div class="tbtn-lbl">Clear</div>
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

    <div class="sec" id="lpaSection" style="display:none">
      <div class="sec-label">Local Planning Authorities</div>
      <div class="lpa-list" id="lpaList"></div>
    </div>

    <div class="sec">
      <div class="sec-label">Search Parameters</div>
      <div class="f-grid">
        <div class="f-item"><div class="f-lbl">Buffer (km)</div><input type="number" id="bufKm" value="5" min="0.5" max="25" step="0.5" onchange="redrawBuffer()"></div>
        <div class="f-item"><div class="f-lbl">Years back</div><input type="number" id="yrsBack" value="5" min="1" max="20" step="1"></div>
        <div class="f-item"><div class="f-lbl">Min units (OR)</div><input type="number" id="minUnits" value="10" min="0" step="1"></div>
        <div class="f-item"><div class="f-lbl">Min area ha (OR)</div><input type="number" id="minHa" value="0.5" min="0" step="0.1"></div>
      </div>
      <div class="trow"><span class="trow-lbl">Include refused applications</span><label class="tog"><input type="checkbox" id="togRefused" checked><div class="tog-track"></div><div class="tog-thumb"></div></label></div>
      <div class="trow"><span class="trow-lbl">Show LPA boundaries</span><label class="tog"><input type="checkbox" id="togLPA" checked onchange="toggleLPALayer()"><div class="tog-track"></div><div class="tog-thumb"></div></label></div>
      <div class="trow"><span class="trow-lbl">Show search buffer</span><label class="tog"><input type="checkbox" id="togBuffer" checked onchange="toggleBuf()"><div class="tog-track"></div><div class="tog-thumb"></div></label></div>
    </div>

    <div class="sec">
      <button class="search-btn" id="searchBtn" onclick="runSearch()" disabled>
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        Search Planning Databases
      </button>
      <div style="margin-top:7px;font-size:10px;color:var(--muted);text-align:center;line-height:1.5" id="srchHint">Draw a site boundary to enable search</div>
    </div>

    <div class="sec">
      <div class="sec-label">Data Sources</div>
      <div style="display:flex;flex-direction:column;gap:4px;">
        <div style="display:flex;align-items:center;gap:7px;font-size:10px;">
          <div style="width:7px;height:7px;border-radius:50%;background:var(--blue);flex-shrink:0;"></div>
          <span style="flex:1;font-weight:500;">planning.data.gov.uk</span>
          <span style="font-family:'Courier Prime',monospace;font-size:8px;color:var(--green);">CORS ✓</span>
        </div>
        <div style="display:flex;align-items:center;gap:7px;font-size:10px;">
          <div style="width:7px;height:7px;border-radius:50%;background:var(--green);flex-shrink:0;"></div>
          <span style="flex:1;font-weight:500;">GLA London (if applicable)</span>
          <span style="font-family:'Courier Prime',monospace;font-size:8px;color:var(--green);">CORS ✓</span>
        </div>
        <div style="display:flex;align-items:center;gap:7px;font-size:10px;">
          <div style="width:7px;height:7px;border-radius:50%;background:var(--amber);flex-shrink:0;"></div>
          <span style="flex:1;font-weight:500;">Idox portals (proxy)</span>
          <span style="font-family:'Courier Prime',monospace;font-size:8px;color:var(--amber);">PROXY</span>
        </div>
        <div style="display:flex;align-items:center;gap:7px;font-size:10px;">
          <div style="width:7px;height:7px;border-radius:50%;background:var(--purple);flex-shrink:0;"></div>
          <span style="flex:1;font-weight:500;">Manual import (CSV)</span>
          <span style="font-family:'Courier Prime',monospace;font-size:8px;color:var(--muted);">FALLBACK</span>
        </div>
      </div>
    </div>

  </div>
  <div class="prog-log" id="progLog"></div>
</aside>

<!-- MAP -->
<div class="map-wrap">
  <div id="map"></div>
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
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><line x1="8" y1="6" x2="21" y2="6"/><line x1="8" y1="12" x2="21" y2="12"/><line x1="8" y1="18" x2="21" y2="18"/><line x1="3" y1="6" x2="3.01" y2="6"/><line x1="3" y1="12" x2="3.01" y2="12"/><line x1="3" y1="18" x2="3.01" y2="18"/></svg>
      Results panel
    </button>
    <button class="mc-btn" onclick="fitAll()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"/></svg>
      Fit all
    </button>
  </div>
  <div class="map-legend">
    <div class="leg-title">Legend</div>
    <div class="leg-row"><div class="leg-swatch ls-site"></div>Site boundary</div>
    <div class="leg-row"><div class="leg-swatch ls-buf"></div>Search buffer</div>
    <div class="leg-row"><div class="leg-swatch ls-std"></div>Residential scheme</div>
    <div class="leg-row"><div class="leg-swatch ls-maj"></div>EIA / Major</div>
    <div class="leg-row"><div class="leg-swatch ls-lg"></div>Large ≥0.5ha / 50+ units</div>
    <div class="leg-row"><div class="leg-swatch ls-lpa"></div>LPA boundary</div>
  </div>
</div>

<!-- RIGHT PANEL -->
<aside class="panel-r" id="panelR">
  <div class="panel-tabs">
    <div class="p-tab on"  id="tabRes" onclick="switchTab('res')">Results</div>
    <div class="p-tab"     id="tabPDF" onclick="switchTab('pdf')">Site Plans</div>
    <div class="p-tab"     id="tabImp" onclick="switchTab('imp')">Import</div>
  </div>

  <!-- RESULTS TAB -->
  <div class="tab-pane on" id="paneRes" style="flex-direction:column;overflow:hidden;">
    <div style="flex:1;overflow-y:auto;padding:14px;" id="resScroll">
      <div style="display:flex;flex-direction:column;align-items:center;justify-content:center;padding:30px 16px;text-align:center;gap:9px;color:var(--muted);">
        <div style="width:44px;height:44px;border:1px solid var(--rule);border-radius:50%;display:flex;align-items:center;justify-content:center;font-family:'Courier Prime',monospace;font-size:18px;opacity:0.5;">◎</div>
        <div style="font-family:'Barlow Condensed',sans-serif;font-size:15px;font-weight:400;color:var(--ink2);">No results yet</div>
        <div style="font-size:11px;line-height:1.5;">Draw a site boundary and run a search to find qualifying planning applications.</div>
      </div>
    </div>
    <div class="export-bar" id="expBar">
      <span class="exp-lbl">Export</span>
      <button class="exp-btn" onclick="exportCSV()">CSV</button>
      <button class="exp-btn" onclick="exportGeoJSON()">GeoJSON</button>
      <button class="exp-btn" onclick="exportKML()">KML</button>
    </div>
  </div>

  <!-- PDF TAB -->
  <div class="tab-pane" id="panePDF" style="flex-direction:column;overflow:hidden;">
    <div class="pdf-header">
      <div><div class="pdf-title" id="pdfTitle">No application selected</div><div class="pdf-sub" id="pdfSub">Select a result to view its planning documents</div></div>
      <button class="pdf-close" onclick="clearPDF()">×</button>
    </div>
    <div style="display:flex;flex-direction:column;flex:1;overflow:hidden;">
      <div class="pdf-docs" id="pdfDocList"><div style="font-size:10px;color:var(--muted);padding:4px 0;">No documents loaded.</div></div>
      <div class="pdf-frame-wrap" id="pdfFrameWrap">
        <div class="pdf-empty"><div class="pdf-empty-icon">📄</div><div class="pdf-empty-txt">Select an application, then choose a document above to view it alongside the map for visual comparison.</div></div>
      </div>
      <div class="pdf-note" id="pdfNote" style="display:none"><strong>Georeferencing:</strong> PDF shown alongside map for visual assessment. Compare grid references or recognisable features to orient the site plan.</div>
    </div>
  </div>

  <!-- IMPORT TAB -->
  <div class="tab-pane" id="paneImp" style="flex-direction:column;overflow:hidden;">
    <div style="flex:1;overflow-y:auto;padding:14px;display:flex;flex-direction:column;gap:12px;">

      <div>
        <div style="font-family:'Barlow Condensed',sans-serif;font-size:15px;font-weight:400;color:var(--ink);margin-bottom:4px;">Manual Import</div>
        <div style="font-size:11px;color:var(--muted);line-height:1.6;">When council portals block automated access, use this tab to import applications you've found manually. The tool will geocode and map them identically to live results.</div>
      </div>

      <div class="import-box">
        <div class="import-title">📋 Paste CSV rows</div>
        <div class="import-sub">One application per line. Works with copy-paste from a spreadsheet or portal export.</div>
        <div class="import-format">reference, description, address, units, site_ha, status, lpa, date</div>
        <textarea id="csvPaste" placeholder="ZG2025/0884/REMM, Erection of 85 dwellings..., Land off High Street Selby, 85, 2.4, Pending, North Yorkshire, 2025-03-10"></textarea>
        <div style="display:flex;gap:6px;margin-top:8px;flex-wrap:wrap;">
          <button class="btn-sm primary" onclick="importCSVPaste()">Import rows</button>
          <button class="btn-sm" onclick="document.getElementById('csvFileIn').click()">Upload CSV file</button>
          <button class="btn-sm" onclick="document.getElementById('geojsonFileIn').click()">Upload GeoJSON</button>
        </div>
        <input type="file" id="csvFileIn" accept=".csv" style="display:none" onchange="importCSVFile(event)">
        <input type="file" id="geojsonFileIn" accept=".geojson,.json" style="display:none" onchange="importGeoJSONFile(event)">
      </div>

      <div class="import-box">
        <div class="import-title">🔗 Open portal searches</div>
        <div class="import-sub">These links open the correct advanced search on each detected LPA's portal, pre-configured for your date range. Export results from there and paste above.</div>
        <div id="portalLinks" style="display:flex;flex-direction:column;gap:5px;margin-top:6px;">
          <div style="font-size:10px;color:var(--muted);">Draw a site boundary to generate portal links.</div>
        </div>
      </div>

      <div class="import-box">
        <div class="import-title">🐍 Python fetch script</div>
        <div class="import-sub">Run this once locally to scrape real Idox data server-side. Outputs a GeoJSON you upload above.</div>
        <pre style="font-family:'Courier Prime',monospace;font-size:9px;color:var(--muted);background:var(--paper3);padding:8px;border-radius:3px;overflow-x:auto;line-height:1.7;">pip install requests beautifulsoup4
python fetch_planning.py \
  --portal https://public.selby.gov.uk/online-applications \
  --bbox -1.15,53.70,-0.90,53.85 \
  --years 5 \
  --out results.geojson</pre>
        <button class="btn-sm primary" onclick="downloadFetchScript()" style="margin-top:8px;">Download fetch_planning.py</button>
      </div>

    </div>
  </div>

</aside>
</div>

<div class="notif" id="notif"></div>

<script>
'use strict';
// ══════════════════════════════════════════════════════════════
// STATE
// ══════════════════════════════════════════════════════════════
const S = {
  site:null, buffer:null, siteL:null, bufL:null,
  lpaL:null,      // LPA boundary layer
  lpaLabelsL:null,// LPA label layer
  resL:null,
  results:[], lpas:[], markers:[],
  sourceLog:[], // [{name, status, count, msg}]
  sortKey:'dist', baseCycle:0, satOn:false,
};

// ══════════════════════════════════════════════════════════════
// IDOX PORTAL DIRECTORY
// ══════════════════════════════════════════════════════════════
const PORTALS = {
  'North Yorkshire':      'https://public.selby.gov.uk/online-applications',
  'Craven':               'https://publicaccess.cravendc.gov.uk/online-applications',
  'Leeds':                'https://publicaccess.leeds.gov.uk/online-applications',
  'Bradford':             'https://planning.bradford.gov.uk/online-applications',
  'Sheffield':            'https://planningapps.sheffield.gov.uk/online-applications',
  'Wakefield':            'https://planning.wakefield.gov.uk/online-applications',
  'Kirklees':             'https://www.kirklees.gov.uk/planning',
  'Harrogate':            'https://uniformonline.harrogate.gov.uk/online-applications',
  'Hambleton':            'https://planning.hambleton.gov.uk/online-applications',
  'Scarborough':          'https://planning.scarborough.gov.uk/online-applications',
  'Richmondshire':        'https://planningregister.richmondshire.gov.uk/online-applications',
  'Ryedale':              'https://www.ryedale.gov.uk/planning',
  'Selby':                'https://public.selby.gov.uk/online-applications',
  'Manchester':           'https://www.manchester.gov.uk/planning',
  'Salford':              'https://publicaccess.salford.gov.uk/online-applications',
  'Trafford':             'https://planning.trafford.gov.uk/online-applications',
  'Stockport':            'https://planning.stockport.gov.uk/online-applications',
  'Tameside':             'https://planning.tameside.gov.uk/online-applications',
  'Oldham':               'https://planning.oldham.gov.uk/online-applications',
  'Bury':                 'https://planning.bury.gov.uk/online-applications',
  'Bolton':               'https://planning.bolton.gov.uk/online-applications',
  'Wigan':                'https://planning.wigan.gov.uk/online-applications',
  'Birmingham':           'https://eplanning.birmingham.gov.uk/Northgate/PlanningExplorer',
  'Coventry':             'https://planningapps.coventry.gov.uk/online-applications',
  'Wolverhampton':        'https://planningapps.wolverhampton.gov.uk/online-applications',
  'Bristol':              'https://pa.bristol.gov.uk/online-applications',
  'Bath and North East Somerset': 'https://www.bathnes.gov.uk/services/planning',
  'South Gloucestershire':'https://developments.southglos.gov.uk/online-applications',
  'Oxford':               'https://www.oxford.gov.uk/info/20214/search_for_planning_applications',
  'Cambridge':            'https://applications.greatercambridgeplanning.org/online-applications',
  'South Cambridgeshire': 'https://applications.greatercambridgeplanning.org/online-applications',
  'Norwich':              'https://planning.norwich.gov.uk/online-applications',
  'Westminster':          'https://idoxpa.westminster.gov.uk/online-applications',
  'Camden':               'https://planning.camden.gov.uk/online-applications',
  'Hackney':              'https://planningscrutiny.hackney.gov.uk/online-applications',
  'Tower Hamlets':        'https://development.towerhamlets.gov.uk/online-applications',
  'Southwark':            'https://planning.southwark.gov.uk/online-applications',
  'Lambeth':              'https://planning.lambeth.gov.uk/online-applications',
  'Lewisham':             'https://planning.lewisham.gov.uk/online-applications',
  'Greenwich':            'https://planning.royalgreenwich.gov.uk/online-applications',
  'Islington':            'https://www.islington.gov.uk/planning',
  'Haringey':             'https://www.planning.haringey.gov.uk/online-applications',
  'Enfield':              'https://planning.enfield.gov.uk/online-applications',
  'Barnet':               'https://barnet.gov.uk/planning',
  'Ealing':               'https://pam.ealing.gov.uk/online-applications',
  'Hounslow':             'https://planningandbuilding.hounslow.gov.uk/online-applications',
  'Wandsworth':           'https://planning.wandsworth.gov.uk/online-applications',
  'Merton':               'https://planning.merton.gov.uk/online-applications',
  'Croydon':              'https://planning.croydon.gov.uk/online-applications',
};

// ══════════════════════════════════════════════════════════════
// MAP INITIALISATION
// ══════════════════════════════════════════════════════════════
const TILES = [
  { name:'Street map', url:'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png',  attr:'© OpenStreetMap, © CARTO' },
  { name:'OS style',   url:'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',               attr:'© OpenStreetMap' },
  { name:'Dark',       url:'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',   attr:'© OpenStreetMap, © CARTO' },
];

const map = L.map('map',{ center:[52.9,-1.5], zoom:7 });
let tileLayer = L.tileLayer(TILES[0].url,{ attribution:TILES[0].attr, maxZoom:19 }).addTo(map);
const satLayer = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',{ attribution:'© Esri', maxZoom:19 });

S.resL      = L.layerGroup().addTo(map);
S.lpaL      = L.layerGroup().addTo(map);
S.lpaLabelsL= L.layerGroup().addTo(map);

const drawn = new L.FeatureGroup().addTo(map);
const drawCtrl = new L.Control.Draw({
  draw:{ polygon:{ allowIntersection:false, shapeOptions:{ color:'#bf3b1a', fillColor:'#bf3b1a', fillOpacity:0.1, weight:2.5 }}, rectangle:false,polyline:false,circle:false,circlemarker:false,marker:false },
  edit:{ featureGroup:drawn }
});
map.addControl(drawCtrl);

map.on(L.Draw.Event.DRAWSTART, ()=>document.getElementById('drawBanner').classList.add('on'));
map.on(L.Draw.Event.DRAWSTOP,  ()=>document.getElementById('drawBanner').classList.remove('on'));
map.on(L.Draw.Event.CREATED,   e=>{ drawn.clearLayers(); drawn.addLayer(e.layer); processSite(e.layer.toGeoJSON().geometry); });
map.on(L.Draw.Event.EDITED,    e=>e.layers.eachLayer(l=>processSite(l.toGeoJSON().geometry)));
map.on(L.Draw.Event.DELETED,   ()=>clearAll());

// ══════════════════════════════════════════════════════════════
// SITE PROCESSING
// ══════════════════════════════════════════════════════════════
async function processSite(geom) {
  if (S.siteL) map.removeLayer(S.siteL);
  if (S.bufL)  map.removeLayer(S.bufL);

  let feat;
  try {
    if      (geom.type==='Polygon')      feat = turf.polygon(geom.coordinates);
    else if (geom.type==='MultiPolygon') feat = turf.multiPolygon(geom.coordinates);
    else if (geom.type==='Feature')      feat = geom;
    else { notify('Unsupported geometry type','err'); return; }
  } catch(e) { notify('Geometry error: '+e.message,'err'); return; }

  S.site = feat;
  document.getElementById('siteHa').textContent = (turf.area(feat)/10000).toFixed(2);
  document.getElementById('siteStats').style.display = 'block';

  S.siteL = L.geoJSON(feat,{ style:{ color:'#bf3b1a', weight:2.5, fillColor:'#bf3b1a', fillOpacity:0.1 }}).addTo(map);
  drawBuffer();
  setStatus('working','Detecting LPAs…');
  await detectLPAs(feat);

  // Load LPA boundaries from ONS
  addLog('info','Loading LPA boundaries from ONS…');
  await loadLPABoundaries(turf.bbox(S.buffer||feat));

  stepDone(1); stepDone(2); stepDone(3);
  document.getElementById('searchBtn').disabled = false;
  document.getElementById('srchHint').textContent = 'LPAs detected — click to search';
  setStatus('idle','Site ready');
  notify('Site set. '+S.lpas.length+' LPA(s) detected.','ok');
}

function drawBuffer() {
  if (!S.site) return;
  if (S.bufL) map.removeLayer(S.bufL);
  const km = parseFloat(document.getElementById('bufKm').value)||5;
  document.getElementById('bufKmDisp').textContent = km.toFixed(1);
  S.buffer = turf.buffer(S.site, km, { units:'kilometers' });
  S.bufL = L.geoJSON(S.buffer,{ style:{ color:'#1a4e8c', weight:1.5, dashArray:'6 4', fillColor:'#1a4e8c', fillOpacity:0.05 }}).addTo(map);
  if (!document.getElementById('togBuffer').checked) map.removeLayer(S.bufL);
}

function redrawBuffer() { drawBuffer(); if(S.site) loadLPABoundaries(turf.bbox(S.buffer||S.site)); }
function toggleBuf()    { if(!S.bufL) return; document.getElementById('togBuffer').checked ? map.addLayer(S.bufL) : map.removeLayer(S.bufL); }
function toggleLPALayer(){ const on=document.getElementById('togLPA').checked; [S.lpaL,S.lpaLabelsL].forEach(l=>{ if(!l)return; on?map.addLayer(l):map.removeLayer(l); }); }

// ══════════════════════════════════════════════════════════════
// LPA BOUNDARY LAYER — ONS ArcGIS Online (CORS-safe)
// ══════════════════════════════════════════════════════════════
async function loadLPABoundaries(bbox) {
  S.lpaL.clearLayers();
  S.lpaLabelsL.clearLayers();

  const [w,s,e,n] = bbox;
  // Expand bbox slightly to ensure boundary polygons fully load
  const pad = 0.15;
  const esriGeom = JSON.stringify({ xmin:w-pad, ymin:s-pad, xmax:e+pad, ymax:n+pad, spatialReference:{ wkid:4326 }});

  const params = new URLSearchParams({
    geometry:         esriGeom,
    geometryType:     'esriGeometryEnvelope',
    inSR:             4326,
    spatialRel:       'esriSpatialRelIntersects',
    outFields:        'LPA24NM,LPA24CD',
    returnGeometry:   true,
    f:                'geojson',
    where:            '1=1',
    outSR:            4326,
    resultRecordCount:50,
  });

  // ONS BUC = boundary ultra clipped (web-optimised)
  const url = `https://services1.arcgis.com/ESMARspQHYMw9BZ9/arcgis/rest/services/Local_Planning_Authorities_April_2024_UK_BUC/FeatureServer/0/query?${params}`;

  try {
    const r = await fetch(url, { signal:AbortSignal.timeout(12000) });
    if (!r.ok) throw new Error('HTTP '+r.status);
    const gj = await r.json();

    if (!gj.features || !gj.features.length) {
      addLog('warn','LPA boundaries: no features returned for this area');
      return;
    }

    addLog('ok', `LPA boundaries: ${gj.features.length} polygon(s) loaded from ONS`);

    // Draw boundaries
    L.geoJSON(gj, {
      style: {
        color: '#6b2d8b',
        weight: 1.5,
        dashArray: '5 4',
        fillOpacity: 0,
        opacity: 0.65,
      },
      onEachFeature: (feat, layer) => {
        const name = feat.properties.LPA24NM || feat.properties.LPA23NM || '';
        if (name) layer.bindTooltip(name, { permanent:false, direction:'center', className:'lpa-map-label' });
      }
    }).addTo(S.lpaL);

    // Add permanent name labels at polygon centroids
    gj.features.forEach(feat => {
      const name = feat.properties.LPA24NM || feat.properties.LPA23NM || '';
      if (!name) return;
      try {
        // Use turf centroid; skip if outside current view
        const c = turf.centroid(feat).geometry.coordinates;
        const labelIcon = L.divIcon({
          html: `<div class="lpa-map-label">${name}</div>`,
          className: '',
          iconAnchor: [0, 0],
          iconSize: null,
        });
        L.marker([c[1], c[0]], { icon:labelIcon, interactive:false, zIndexOffset:-1000 })
          .addTo(S.lpaLabelsL);
      } catch(e) {}
    });

    if (!document.getElementById('togLPA').checked) { map.removeLayer(S.lpaL); map.removeLayer(S.lpaLabelsL); }

  } catch(e) {
    addLog('err', 'LPA boundaries: '+e.message+' — check connection');
  }
}

// ══════════════════════════════════════════════════════════════
// LPA NAME DETECTION — Nominatim reverse geocode
// ══════════════════════════════════════════════════════════════
async function detectLPAs(feat) {
  const bbox = turf.bbox(feat);
  const pts = [
    turf.centroid(feat).geometry.coordinates,
    [bbox[0],bbox[1]],[bbox[2],bbox[1]],[bbox[0],bbox[3]],[bbox[2],bbox[3]],
    [(bbox[0]+bbox[2])/2,bbox[1]],[(bbox[0]+bbox[2])/2,bbox[3]],
  ];
  const seen = new Map();
  await Promise.allSettled(pts.map(async([lng,lat])=>{
    try {
      const r = await fetch(
        `https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lng}&format=json&addressdetails=1&zoom=10`,
        { headers:{'Accept-Language':'en-GB'}, signal:AbortSignal.timeout(6000) }
      );
      const d = await r.json();
      const a = d.address||{};
      const name = a.county||a.city||a.state_district||a.town||'Unknown';
      if (!seen.has(name)) seen.set(name,{ name, region:a.state||'', portal:findPortal(name), postcode:a.postcode||'' });
    } catch{}
  }));

  S.lpas = [...seen.values()];
  document.getElementById('stLPAs').textContent = S.lpas.length;
  document.getElementById('lpaSection').style.display = 'block';

  document.getElementById('lpaList').innerHTML = S.lpas.map(l=>`
    <div class="lpa-badge">
      <div class="lpa-dot"></div>
      <div style="flex:1;min-width:0;">
        <div class="lpa-name">${l.name}</div>
        <div class="lpa-url">${l.portal ? new URL(l.portal).hostname : 'Portal not indexed'}</div>
      </div>
      ${l.portal?`<a class="lpa-open" href="${l.portal}" target="_blank">Open →</a>`:''}
    </div>`).join('');

  // Build portal links in import tab
  buildPortalLinks();
}

function findPortal(name) {
  if (PORTALS[name]) return PORTALS[name];
  const nl = name.toLowerCase();
  for (const [k,v] of Object.entries(PORTALS)) {
    if (nl.includes(k.toLowerCase())||k.toLowerCase().includes(nl)) return v;
  }
  return null;
}

function buildPortalLinks() {
  const yrs = parseInt(document.getElementById('yrsBack').value)||5;
  const fromDate = new Date(); fromDate.setFullYear(fromDate.getFullYear()-yrs);
  const fy=fromDate.getFullYear(), fm=fromDate.getMonth()+1, fd=fromDate.getDate();

  const el = document.getElementById('portalLinks');
  if (!S.lpas.length) { el.innerHTML='<div style="font-size:10px;color:var(--muted)">No LPAs detected yet.</div>'; return; }

  el.innerHTML = S.lpas.map(l=>{
    if (!l.portal) return `<div style="font-size:10px;color:var(--muted);">${l.name} — portal not indexed</div>`;
    const searchUrl = `${l.portal}/search.do?action=advanced&description=residential+dwellings&dateType=DC_Received&dateRangeType=custom&startDateDay=${fd}&startDateMonth=${fm}&startDateYear=${fy}&endDateDay=${new Date().getDate()}&endDateMonth=${new Date().getMonth()+1}&endDateYear=${new Date().getFullYear()}`;
    return `<a href="${searchUrl}" target="_blank" style="display:flex;align-items:center;gap:8px;padding:7px 9px;background:var(--paper2);border:1px solid var(--rule);border-radius:4px;text-decoration:none;font-size:11px;color:var(--ink);transition:border-color 0.12s;" onmouseover="this.style.borderColor='var(--blue)'" onmouseout="this.style.borderColor='var(--rule)'">
      <div style="width:6px;height:6px;border-radius:50%;background:var(--blue);flex-shrink:0;"></div>
      <span style="flex:1;font-weight:500;">${l.name}</span>
      <span style="font-family:'Courier Prime',monospace;font-size:9px;color:var(--blue);">Open search →</span>
    </a>`;
  }).join('');
}

// ══════════════════════════════════════════════════════════════
// MAIN SEARCH
// ══════════════════════════════════════════════════════════════
async function runSearch() {
  if (!S.site||!S.buffer) { notify('Draw a site boundary first','warn'); return; }
  S.results=[]; S.resL.clearLayers(); S.markers=[]; S.sourceLog=[];
  setBtnWorking(true); setStatus('working','Searching…');
  const log=document.getElementById('progLog'); log.innerHTML=''; log.classList.add('on');
  stepActive(4);

  const bbox   = turf.bbox(S.buffer);
  const yrs    = parseInt(document.getElementById('yrsBack').value)||5;
  const fromDt = new Date(); fromDt.setFullYear(fromDt.getFullYear()-yrs);

  addLog('info',`Buffer: ${bbox.map(v=>v.toFixed(3)).join(', ')}`);
  addLog('info',`From: ${fromDt.toLocaleDateString('en-GB')} → today`);
  addLog('info',`Filter: >${document.getElementById('minUnits').value} units OR ≥${document.getElementById('minHa').value} ha`);

  // ── Source 1: planning.data.gov.uk ──
  addLog('info','→ planning.data.gov.uk entity API…');
  try {
    const pd = await srcPlanningData(S.buffer, fromDt);
    logSource('planning.data.gov.uk', pd.length>0?'ok':'warn', pd.length, pd.length?null:'No data for this area yet');
    S.results.push(...pd);
  } catch(e) { logSource('planning.data.gov.uk','err',0,e.message); addLog('err','planning.data.gov.uk: '+e.message); }

  // ── Source 2: GLA London (if bbox overlaps London) ──
  if (bboxOverlap(bbox,[-0.56,51.28,0.36,51.72])) {
    addLog('info','→ GLA London Planning Data…');
    try {
      const gla = await srcGLA(bbox, fromDt);
      logSource('GLA London', gla.length>0?'ok':'warn', gla.length, gla.length?null:'No results for this area');
      S.results.push(...gla);
    } catch(e) { logSource('GLA London','err',0,e.message); addLog('err','GLA: '+e.message); }
  }

  // ── Source 3: Idox portals via CORS proxy ──
  for (const lpa of S.lpas) {
    if (!lpa.portal) { logSource(lpa.name,'skip',0,'No portal URL indexed'); continue; }
    addLog('info',`→ Idox: ${lpa.name}…`);
    try {
      const idox = await srcIdox(lpa, fromDt);
      logSource(lpa.name, idox.length>0?'ok':'warn', idox.length, idox.length?null:'Proxy blocked or no results');
      S.results.push(...idox);
    } catch(e) { logSource(lpa.name,'err',0,e.message+' (CORS — use Import tab)'); addLog('warn',lpa.name+' Idox blocked — try Import tab'); }
  }

  addLog('info',`Raw total: ${S.results.length} — deduplicating & filtering…`);

  // ── Deduplicate ──
  const seen=new Set();
  S.results=S.results.filter(r=>{ const k=r.reference||(r.lat+','+r.lng); if(seen.has(k))return false; seen.add(k); return true; });

  // ── Geocode missing coords ──
  const noCoord=S.results.filter(r=>!r.lat||!r.lng);
  if (noCoord.length) { addLog('info',`Geocoding ${Math.min(noCoord.length,15)} addresses…`); await geocodeBatch(noCoord); }

  // ── Filter: inside buffer ──
  S.results=S.results.filter(r=>{ if(!r.lat||!r.lng)return false; return turf.booleanPointInPolygon(turf.point([r.lng,r.lat]),S.buffer); });

  // ── Filter: date ──
  S.results=S.results.filter(r=>{ if(!r.receivedDate)return true; return new Date(r.receivedDate)>=fromDt; });

  // ── Filter: threshold (OR logic) + residential ──
  const minU=parseInt(document.getElementById('minUnits').value)||10;
  const minA=parseFloat(document.getElementById('minHa').value)||0.5;
  const inclRef=document.getElementById('togRefused').checked;
  S.results=S.results.filter(r=>{
    if (!inclRef && r.status && /refus|reject/i.test(r.status)) return false;
    if (!isResidential(r.description)) return false;
    if (!r.units&&!r.siteHa) { r._unquant=true; return true; }
    return (r.units||0)>minU || (r.siteHa||0)>=minA;
  });

  // ── Distance from site polygon edge ──
  S.results=S.results.map(r=>{
    const pt=turf.point([r.lng,r.lat]);
    try{ const line=turf.polygonToLine(S.site); r.distKm=+turf.distance(pt,turf.nearestPointOnLine(line,pt),{units:'kilometers'}).toFixed(3); }
    catch{ r.distKm=+turf.distance(pt,turf.centroid(S.site),{units:'kilometers'}).toFixed(3); }
    return r;
  });

  addLog('ok',`After filtering: ${S.results.length} qualifying applications`);

  // ── Total units ──
  const totUnits=S.results.reduce((a,r)=>a+(r.units||0),0);
  document.getElementById('stTotal').textContent=S.results.length;
  document.getElementById('stUnits').textContent=totUnits||'—';

  // ── Render ──
  sortResults();

  if (!S.results.length) {
    renderNoResults();
    setStatus('idle','0 results — see import options');
    notify('No live results. Use the Import tab to add data manually.','warn');
  } else {
    renderResults();
    renderMarkers();
    stepDone(4); stepDone(5);
    setStatus('ok',`${S.results.length} schemes found`);
    openResults();
    notify(`${S.results.length} qualifying schemes found`,'ok');
  }

  setBtnWorking(false);
}

// ══════════════════════════════════════════════════════════════
// DATA SOURCES
// ══════════════════════════════════════════════════════════════
function toWKT(feat) {
  const geom = feat.geometry||feat;
  const coords = geom.type==='MultiPolygon' ? geom.coordinates[0][0] : geom.coordinates[0];
  return `POLYGON((${coords.map(c=>c[0]+' '+c[1]).join(',')}))`;
}

// ── planning.data.gov.uk entity API (CORS-safe, real data) ──
async function srcPlanningData(bufferFeat, fromDt) {
  const results=[];
  const wkt=toWKT(bufferFeat);
  const fromStr=fromDt.toISOString().split('T')[0];

  // Try both dataset names used on the platform
  const datasets=['planning-application','planning-permission'];
  for (const ds of datasets) {
    const params=new URLSearchParams({
      dataset:ds, geometry:wkt, geometry_relation:'intersects',
      start_date_after:fromStr, limit:100,
      field:['reference','name','notes','point','geometry','organisation-entity','entry-date','start-date'].join('&field='),
    });
    try {
      const r=await fetch(`https://www.planning.data.gov.uk/entity.json?${params}`,{ signal:AbortSignal.timeout(12000) });
      if (!r.ok) continue;
      const d=await r.json();
      if (d.entities && d.entities.length) {
        addLog('ok',`planning.data.gov.uk [${ds}]: ${d.entities.length} entities`);
        d.entities.forEach(e=>results.push(normPD(e)));
      }
    } catch{}
  }

  // Also try the datasette instance which has broader coverage
  try {
    const bbox=turf.bbox(bufferFeat);
    const [w,s,e,n]=bbox;
    const sql=encodeURIComponent(`SELECT * FROM entity WHERE dataset IN ('planning-application','planning-permission') AND (point LIKE '%') LIMIT 200`);
    // Datasette SQL endpoint
    const r2=await fetch(`https://datasette.planning.data.gov.uk/digital-land.json?sql=${sql}`,{ signal:AbortSignal.timeout(10000) });
    if (r2.ok) {
      const d2=await r2.json();
      if (d2.rows) {
        // Filter by bbox manually
        d2.rows.forEach(row=>{
          const e={}; d2.columns.forEach((c,i)=>e[c]=row[i]);
          results.push(normPD(e));
        });
      }
    }
  } catch{}

  return results.filter(r=>r.lat&&r.lng);
}

function normPD(e) {
  // Parse point — format is usually "POINT(lng lat)"
  let lat=null,lng=null;
  const pt=e.point||e.geometry||'';
  const m=pt.match(/POINT\s*\(\s*(-?\d+\.?\d*)\s+(-?\d+\.?\d*)/i);
  if (m){ lng=+m[1]; lat=+m[2]; }
  else if (e.latitude)  { lat=+e.latitude;  lng=+e.longitude; }

  const notes=e.notes||e.description||e.name||'';
  return {
    reference:   e.reference||e.entity,
    description: notes,
    address:     e.address||e['street-address']||e.name||'',
    lat, lng,
    units:       getUnits(notes),
    siteHa:      getHa(notes),
    status:      e.status||null,
    source:      'planning.data.gov.uk',
    receivedDate:e['start-date']||e['entry-date']||null,
    portalUrl:   e['documentation-url']||null,
    lpa:         e['organisation-entity']||'Unknown',
    _docs:[],
  };
}

// ── GLA London Planning Data ──
async function srcGLA(bbox, fromDt) {
  const results=[];
  const [w,s,e,n]=bbox;

  // Try GLA ArcGIS REST endpoint
  const esriGeom=JSON.stringify({ xmin:w, ymin:s, xmax:e, ymax:n, spatialReference:{ wkid:4326 }});
  const params=new URLSearchParams({
    geometry:esriGeom, geometryType:'esriGeometryEnvelope', inSR:4326,
    spatialRel:'esriSpatialRelIntersects',
    where:`application_date >= '${fromDt.toISOString().split('T')[0]}'`,
    outFields:'*', returnGeometry:true, f:'geojson', resultRecordCount:200,
  });

  // GLA Strategic Applications on ArcGIS Online
  const glaUrl=`https://services1.arcgis.com/ESMARspQHYMw9BZ9/arcgis/rest/services/London_Planning_Applications/FeatureServer/0/query?${params}`;
  try {
    const r=await fetch(glaUrl,{ signal:AbortSignal.timeout(12000) });
    if (r.ok) {
      const gj=await r.json();
      if (gj.features) gj.features.forEach(f=>{
        const p=f.properties||{};
        const c=f.geometry?.coordinates||[];
        results.push({
          reference:   p.reference||p.app_ref||p.appId,
          description: p.description||p.proposal||p.name||'',
          address:     p.address||p.site_address||'',
          lat:         c[1]||null, lng:c[0]||null,
          units:       parseInt(p.units_proposed||p.residential_units)||getUnits(p.description||''),
          siteHa:      parseFloat(p.site_area)||getHa(p.description||''),
          status:      p.decision||p.status||null,
          source:      'GLA London',
          receivedDate:p.application_date||p.date_received||null,
          portalUrl:   p.url||null,
          lpa:         p.borough||p.lpa||'London',
          _docs:[],
        });
      });
    }
  } catch{}

  return results.filter(r=>r.lat&&r.lng);
}

// ── Idox scrape via CORS proxy ──
async function srcIdox(lpa, fromDt) {
  const fy=fromDt.getFullYear(), fm=fromDt.getMonth()+1, fd=fromDt.getDate();
  const ny=new Date().getFullYear(), nm=new Date().getMonth()+1, nd=new Date().getDate();
  const searchUrl=`${lpa.portal}/search.do?action=advanced&description=residential+dwellings&dateType=DC_Received&dateRangeType=custom&startDateDay=${fd}&startDateMonth=${fm}&startDateYear=${fy}&endDateDay=${nd}&endDateMonth=${nm}&endDateYear=${ny}`;

  const proxies=[
    u=>`https://api.allorigins.win/raw?url=${encodeURIComponent(u)}`,
    u=>`https://corsproxy.io/?${encodeURIComponent(u)}`,
    u=>`https://proxy.cors.sh/${u}`,
  ];

  for (const px of proxies) {
    try {
      const r=await fetch(px(searchUrl),{ signal:AbortSignal.timeout(14000) });
      if (!r.ok) continue;
      const html=await r.text();
      const parsed=parseIdoxHTML(html,lpa);
      if (parsed.length>0) {
        addLog('ok',`${lpa.name} Idox: ${parsed.length} results via proxy`);
        return parsed;
      }
    } catch{}
  }
  throw new Error('All CORS proxies blocked');
}

function parseIdoxHTML(html,lpa) {
  const doc=new DOMParser().parseFromString(html,'text/html');
  const out=[];
  doc.querySelectorAll('li.searchresult, .search-result, tr.searchresult').forEach(el=>{
    try {
      const ref  =(el.querySelector('.result-title a,h2 a,.reference')?.textContent||'').trim();
      const desc =(el.querySelector('.result-description,p.description,.description')?.textContent||'').trim();
      const addr =(el.querySelector('.address,.result-address')?.textContent||'').trim();
      const date =(el.querySelector('.received-date,.date,td.date')?.textContent||'').trim();
      const href  =el.querySelector('a[href*="applicationDetails"]')?.getAttribute('href')||'';
      if (!ref&&!desc) return;
      out.push({ reference:ref, description:desc, address:addr, units:getUnits(desc), siteHa:getHa(desc), status:null, source:lpa.name, lpa:lpa.name, receivedDate:date||null, portalUrl:href?(href.startsWith('http')?href:lpa.portal+href):null, lat:null, lng:null, _docs:[] });
    } catch{}
  });
  return out;
}

// ══════════════════════════════════════════════════════════════
// GEOCODING
// ══════════════════════════════════════════════════════════════
async function geocodeBatch(items) {
  const limit=Math.min(items.length,15);
  for (let i=0;i<limit;i++) {
    if (!items[i].address) continue;
    try {
      const r=await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(items[i].address+', UK')}&limit=1&countrycodes=gb`,
        { headers:{'Accept-Language':'en-GB','User-Agent':'CumulativePlanningTool/1.0'}, signal:AbortSignal.timeout(5000) });
      const d=await r.json();
      if (d.length){ items[i].lat=+d[0].lat; items[i].lng=+d[0].lon; }
    } catch{}
    if (i<limit-1) await new Promise(r=>setTimeout(r,350));
  }
}

// ══════════════════════════════════════════════════════════════
// TEXT EXTRACTION
// ══════════════════════════════════════════════════════════════
function getUnits(t) {
  if (!t) return 0;
  const pats=[
    /(\d+)\s*(?:no\.?|x|×)?\s*(?:new\s+)?(?:residential\s+)?(?:dwelling|unit|apartment|flat|house|home)/i,
    /erection\s+of\s+(\d+)/i,/up\s+to\s+(\d+)\s*(?:dwelling|unit|home|house|flat)/i,
    /provision\s+of\s+(\d+)/i,/(\d+)\s+affordable/i,/(\d+)\s*(?:no\.)?\s*bed/i,
  ];
  for (const p of pats){ const m=t.match(p); if(m&&m[1]) return +m[1]; }
  return 0;
}
function getHa(t) { if(!t)return 0; const m=t.match(/(\d+\.?\d*)\s*(?:ha|hectare)/i); return m?+m[1]:0; }
function isResidential(t) { if(!t)return false; return /dwellings?|residential|housing|apartments?|flats?|homes?|student\s+accommodation|build\s+to\s+rent|affordable\s+housing/i.test(t); }
function bboxOverlap(a,b){ return !(a[2]<b[0]||a[0]>b[2]||a[3]<b[1]||a[1]>b[3]); }

function getStatusClass(s) {
  if (!s) return 'ch-pend';
  const sl=s.toLowerCase();
  if (/approv|grant|permit|allowed/.test(sl)) return 'ch-appr';
  if (/refus|reject/.test(sl)) return 'ch-ref';
  return 'ch-pend';
}

function getStripeClass(r) {
  if (r.description&&/eia|environmental impact assessment/i.test(r.description)) return 'cs-red';
  if ((r.siteHa||0)>=0.5||(r.units||0)>=50) return 'cs-green';
  if (r.source==='Manual import') return 'cs-purple';
  return 'cs-blue';
}

// ══════════════════════════════════════════════════════════════
// RENDER RESULTS
// ══════════════════════════════════════════════════════════════
function sortResults() {
  const k=S.sortKey;
  S.results.sort((a,b)=>{ if(k==='dist') return (a.distKm||99)-(b.distKm||99); if(k==='units') return (b.units||0)-(a.units||0); if(k==='date') return new Date(b.receivedDate||0)-new Date(a.receivedDate||0); return 0; });
}
function setSort(k) { S.sortKey=k; document.querySelectorAll('.sort-btn').forEach(b=>b.classList.toggle('on',b.dataset.k===k)); sortResults(); renderResults(); renderMarkers(); }

function renderResults() {
  const el=document.getElementById('resScroll');
  const n=S.results.length;
  if (!n) { renderNoResults(); return; }

  const srcClasses={'planning.data.gov.uk':'src-pd','GLA London':'src-gla','Manual import':'src-import'};
  function srcBadge(src){ const cls=srcClasses[src]||'src-idox'; return `<span class="src-badge ${cls}">${src}</span>`; }

  el.innerHTML=`
    <div class="res-header">
      <div><div class="res-count">${n}</div><div class="res-sub">qualifying schemes · ${document.getElementById('bufKm').value}km buffer</div></div>
      <div class="res-sort">
        <button class="sort-btn on" data-k="dist" onclick="setSort('dist')">Distance</button>
        <button class="sort-btn" data-k="units" onclick="setSort('units')">Units</button>
        <button class="sort-btn" data-k="date" onclick="setSort('date')">Date</button>
      </div>
    </div>
    <div class="app-list">${S.results.map((r,i)=>`
      <div class="app-card" id="ac${i}" onclick="focusApp(${i})">
        <div class="card-top">
          <div class="card-stripe ${getStripeClass(r)}"></div>
          <div class="card-body">
            <div class="card-ref">${r.reference||'—'} · ${r.lpa||r.source}</div>
            <div class="card-desc">${r.description||'No description'}</div>
          </div>
        </div>
        <div class="card-chips">
          ${r.units?`<span class="chip ch-units">${r.units} units</span>`:''}
          ${r.siteHa?`<span class="chip ch-ha">${r.siteHa.toFixed(1)} ha</span>`:''}
          ${r._unquant?`<span class="chip ch-unq">unquantified</span>`:''}
          ${r.status?`<span class="chip ${getStatusClass(r.status)}">${r.status}</span>`:''}
          ${r.distKm!=null?`<span class="chip ch-dist">${r.distKm}km</span>`:''}
          ${r.receivedDate?`<span class="chip ch-lpa">${r.receivedDate.slice(0,10)}</span>`:''}
          ${srcBadge(r.source)}
          ${r.portalUrl?`<a class="card-portal" href="${r.portalUrl}" target="_blank" onclick="event.stopPropagation()">Portal →</a>`:''}
          ${r._docs&&r._docs.length?`<button class="card-plans" onclick="event.stopPropagation();openPDF(${i})">📄 Plans</button>`:''}
        </div>
      </div>`).join('')}
    </div>`;

  document.getElementById('expBar').classList.add('on');
  document.querySelectorAll('.sort-btn').forEach(b=>b.classList.toggle('on',b.dataset.k===S.sortKey));
}

function renderNoResults() {
  const log=S.sourceLog;
  const el=document.getElementById('resScroll');
  el.innerHTML=`
    <div class="no-results-panel">
      <div class="nr-title">No live results returned</div>
      <div class="nr-sub">The live APIs were queried but returned no qualifying applications for this location. This is usually a CORS restriction on council portals or the area not yet covered by planning.data.gov.uk.</div>
      <div class="source-status">
        ${log.map(s=>`
          <div class="ss-row">
            <div class="ss-dot ${s.status==='ok'?'ss-ok':s.status==='err'?'ss-err':'ss-skip'}"></div>
            <div class="ss-name">${s.name}</div>
            <div class="ss-msg">${s.status==='ok'?s.count+' results':s.msg||s.status}</div>
          </div>`).join('')}
      </div>
      <div class="nr-sub">Use the <strong>Import tab</strong> to add applications manually — portal links for each detected LPA are pre-configured there with the correct date range.</div>
      <button class="btn-sm primary" onclick="switchTab('imp');openResults()">Go to Import tab →</button>
    </div>`;
  document.getElementById('expBar').classList.remove('on');
}

function focusApp(i) {
  const r=S.results[i];
  document.querySelectorAll('.app-card').forEach((c,j)=>c.classList.toggle('sel',j===i));
  if (r.lat&&r.lng){ map.setView([r.lat,r.lng],15,{animate:true}); if(S.markers[i]) S.markers[i].openPopup(); }
}

// ══════════════════════════════════════════════════════════════
// MAP MARKERS
// ══════════════════════════════════════════════════════════════
function renderMarkers() {
  S.resL.clearLayers(); S.markers=[];
  S.results.forEach((r,i)=>{
    if (!r.lat||!r.lng){ S.markers.push(null); return; }
    const isMaj=r.description&&/eia|environmental impact/i.test(r.description);
    const isLg=(r.siteHa||0)>=0.5||(r.units||0)>=50;
    const c=isMaj?'#bf3b1a':isLg?'#2d6b3e':'#1a4e8c';
    const icon=L.divIcon({
      html:`<div style="width:24px;height:24px;border-radius:50%;background:${c};border:2.5px solid white;box-shadow:0 1px 8px rgba(0,0,0,0.25);display:flex;align-items:center;justify-content:center;font-family:'Courier Prime',monospace;font-size:9px;font-weight:700;color:white;">${i+1}</div>`,
      iconSize:[24,24],iconAnchor:[12,12],className:''
    });
    const hasDocs=r._docs&&r._docs.length;
    const pop=`<div class="popup-ref">${r.reference||'—'} · ${r.lpa||r.source}</div>
      <div class="popup-desc">${(r.description||'').slice(0,100)}${(r.description||'').length>100?'…':''}</div>
      <div class="popup-chips">
        ${r.units?`<span class="chip ch-units" style="font-family:'Courier Prime',monospace;font-size:8px;padding:2px 5px;border-radius:2px;background:var(--blue-l);color:var(--blue);">${r.units} units</span>`:''}
        ${r.siteHa?`<span class="chip" style="font-family:'Courier Prime',monospace;font-size:8px;padding:2px 5px;border-radius:2px;background:var(--green-l);color:var(--green);">${r.siteHa.toFixed(1)} ha</span>`:''}
        ${r.distKm!=null?`<span style="font-size:9px;color:var(--muted);">${r.distKm}km from site</span>`:''}
      </div>
      ${r.address?`<div class="popup-addr">${r.address}</div>`:''}
      <div class="popup-links">
        ${r.portalUrl?`<a class="popup-btn" href="${r.portalUrl}" target="_blank">Portal →</a>`:''}
        ${hasDocs?`<button class="popup-btn red" onclick="openPDF(${i})">📄 Site plans</button>`:''}
        <button class="popup-btn" onclick="switchTab('res');setTimeout(()=>{const c=document.getElementById('ac${i}');if(c)c.scrollIntoView({behavior:'smooth',block:'nearest'})},100)">List →</button>
      </div>`;
    const m=L.marker([r.lat,r.lng],{icon}).bindPopup(pop,{maxWidth:290}).addTo(S.resL);
    m.on('click',()=>document.querySelectorAll('.app-card').forEach((c,j)=>c.classList.toggle('sel',j===i)));
    S.markers.push(m);
  });
}

// ══════════════════════════════════════════════════════════════
// PDF / SITE PLAN VIEWER
// ══════════════════════════════════════════════════════════════
function openPDF(i) {
  const r=S.results[i];
  document.getElementById('pdfTitle').textContent=r.reference||r.description?.slice(0,40)||'Application';
  document.getElementById('pdfSub').textContent=r.address||r.lpa||'';

  const docList=document.getElementById('pdfDocList');
  if (!r._docs||!r._docs.length) {
    docList.innerHTML='<div style="font-size:10px;color:var(--muted);padding:4px 0;">No documents indexed. Try opening the portal link directly.</div>';
  } else {
    const sorted=[...r._docs].sort((a,b)=>(/site.?plan|location.?plan|red.?line/i.test(b.name)?1:-1));
    docList.innerHTML=sorted.map((d,j)=>{
      const isSP=/site.?plan|location.?plan|red.?line/i.test(d.name);
      return `<button class="pdf-doc-btn${isSP?' on':''}" onclick="loadDocFrame('${d.url||''}','${d.name}',${j})">
        <span style="font-size:13px;flex-shrink:0;">${isSP?'🗺':'📄'}</span>
        <span style="flex:1;min-width:0;"><div class="pdf-doc-name">${d.name}</div><div class="pdf-doc-type">${d.type||docType(d.name)}</div></span>
        ${isSP?`<span style="font-family:'Courier Prime',monospace;font-size:8px;color:var(--red);border:1px solid rgba(191,59,26,0.3);padding:1px 4px;border-radius:2px;">SITE PLAN</span>`:''}
      </button>`;
    }).join('');
    if (sorted[0]) loadDocFrame(sorted[0].url||'',sorted[0].name,0);
  }
  if (r.portalUrl) fetchDocs(r,i);
  switchTab('pdf'); openResults();
}

async function fetchDocs(r,i) {
  if (!r.portalUrl) return;
  const docsUrl=r.portalUrl.replace('activeTab=summary','activeTab=documents');
  const proxies=[u=>`https://api.allorigins.win/raw?url=${encodeURIComponent(u)}`,u=>`https://corsproxy.io/?${encodeURIComponent(u)}`];
  for (const px of proxies) {
    try {
      const resp=await fetch(px(docsUrl),{signal:AbortSignal.timeout(10000)});
      if (!resp.ok) continue;
      const html=await resp.text();
      const doc=new DOMParser().parseFromString(html,'text/html');
      const found=[];
      doc.querySelectorAll('a[href*=".pdf"],a[href*="download"]').forEach(a=>{
        const href=a.getAttribute('href');
        const name=(a.textContent||a.title||href).trim().slice(0,80);
        if (href&&name&&!found.find(f=>f.url===href))
          found.push({ name, url:href.startsWith('http')?href:new URL(href,r.portalUrl).href, type:docType(name) });
      });
      if (found.length) { S.results[i]._docs=found; openPDF(i); break; }
    } catch{}
  }
}

function loadDocFrame(url,name,idx) {
  document.querySelectorAll('.pdf-doc-btn').forEach((b,j)=>b.classList.toggle('on',j===idx));
  const wrap=document.getElementById('pdfFrameWrap');
  const note=document.getElementById('pdfNote');
  const isSP=/site.?plan|location.?plan|red.?line/i.test(name);
  if (!url) {
    wrap.innerHTML=`<div class="pdf-empty"><div class="pdf-empty-icon">🔒</div><div class="pdf-empty-txt"><strong>${name}</strong><br><br>Hosted on council portal — CORS prevents direct embedding.<br><br>Use the portal link below to view.</div></div>`;
  } else {
    wrap.innerHTML=`<iframe src="${url}" allowfullscreen title="${name}"></iframe>`;
  }
  note.style.display=isSP?'block':'none';
}

function clearPDF() {
  document.getElementById('pdfTitle').textContent='No application selected';
  document.getElementById('pdfSub').textContent='Select a result to view its planning documents';
  document.getElementById('pdfDocList').innerHTML='<div style="font-size:10px;color:var(--muted);padding:4px 0;">No documents loaded.</div>';
  document.getElementById('pdfFrameWrap').innerHTML=`<div class="pdf-empty"><div class="pdf-empty-icon">📄</div><div class="pdf-empty-txt">Select an application from the results list, then choose a document to view it.</div></div>`;
  document.getElementById('pdfNote').style.display='none';
  switchTab('res');
}

function docType(name) {
  const n=name.toLowerCase();
  if (/location|site\s*plan|red.?line/.test(n)) return 'SITE PLAN';
  if (/transport/.test(n)) return 'TRANSPORT';
  if (/heritage|historic/.test(n)) return 'HERITAGE';
  if (/design/.test(n)) return 'D&A';
  if (/environment|eia|es\b/.test(n)) return 'EIA';
  if (/flood/.test(n)) return 'FRA';
  if (/landscape/.test(n)) return 'LANDSCAPE';
  return 'DOCUMENT';
}

// ══════════════════════════════════════════════════════════════
// MANUAL IMPORT
// ══════════════════════════════════════════════════════════════
function importCSVPaste() {
  const raw=document.getElementById('csvPaste').value.trim();
  if (!raw) { notify('Paste some rows first','warn'); return; }
  const lines=raw.split('\n').filter(l=>l.trim()&&!l.startsWith('reference'));
  const imported=lines.map(line=>{
    const cols=parseCSVLine(line);
    return buildImportedApp(cols);
  }).filter(Boolean);
  if (!imported.length) { notify('Could not parse rows. Check format.','err'); return; }
  finaliseImport(imported);
}

function importCSVFile(e) {
  const f=e.target.files[0]; if(!f) return;
  const rd=new FileReader();
  rd.onload=ev=>{
    const lines=ev.target.result.split('\n').filter(l=>l.trim());
    const header=lines[0].toLowerCase();
    const dataLines=header.includes('reference')?lines.slice(1):lines;
    const imported=dataLines.map(l=>buildImportedApp(parseCSVLine(l))).filter(Boolean);
    finaliseImport(imported);
  };
  rd.readAsText(f); e.target.value='';
}

function importGeoJSONFile(e) {
  const f=e.target.files[0]; if(!f) return;
  const rd=new FileReader();
  rd.onload=ev=>{
    try {
      const gj=JSON.parse(ev.target.result);
      const feats=(gj.type==='FeatureCollection'?gj.features:[gj]).filter(f=>f.geometry?.type==='Point');
      const imported=feats.map(f=>{
        const p=f.properties||{};
        const [lng,lat]=f.geometry.coordinates;
        return { reference:p.reference||p.ref, description:p.description||p.desc||'', address:p.address||'', units:+p.units||getUnits(p.description||''), siteHa:+p.site_ha||+p.siteHa||getHa(p.description||''), status:p.status||null, source:'Manual import', lpa:p.lpa||'Imported', receivedDate:p.received||p.date||null, portalUrl:p.portal||p.portalUrl||null, lat, lng, distKm:null, _docs:[] };
      }).filter(r=>r.lat&&r.lng);
      finaliseImport(imported);
    } catch(err) { notify('Could not parse GeoJSON: '+err.message,'err'); }
  };
  rd.readAsText(f); e.target.value='';
}

function parseCSVLine(line) {
  const result=[]; let cur='', inQ=false;
  for (const ch of line) {
    if (ch==='"') { inQ=!inQ; }
    else if (ch===','&&!inQ) { result.push(cur.trim()); cur=''; }
    else cur+=ch;
  }
  result.push(cur.trim()); return result;
}

function buildImportedApp(cols) {
  if (!cols||cols.length<2) return null;
  return { reference:cols[0]||null, description:cols[1]||'', address:cols[2]||'', units:+cols[3]||getUnits(cols[1]||''), siteHa:+cols[4]||getHa(cols[1]||''), status:cols[5]||null, source:'Manual import', lpa:cols[6]||'Imported', receivedDate:cols[7]||null, portalUrl:null, lat:+cols[8]||null, lng:+cols[9]||null, distKm:null, _docs:[] };
}

async function finaliseImport(apps) {
  notify(`Processing ${apps.length} imported application${apps.length>1?'s':''} — geocoding missing coordinates…`);

  if (!S.buffer) { notify('Draw a site boundary first, then import.','warn'); return; }

  // Geocode those without coords
  const noCoord=apps.filter(r=>!r.lat||!r.lng);
  if (noCoord.length) await geocodeBatch(noCoord);

  // Filter: in buffer
  const inBuf=apps.filter(r=>{ if(!r.lat||!r.lng)return false; return turf.booleanPointInPolygon(turf.point([r.lng,r.lat]),S.buffer); });
  if (!inBuf.length) { notify('No imported apps fell within the 5km buffer.','warn'); return; }

  // Compute distances
  inBuf.forEach(r=>{
    const pt=turf.point([r.lng,r.lat]);
    try{ const line=turf.polygonToLine(S.site); r.distKm=+turf.distance(pt,turf.nearestPointOnLine(line,pt),{units:'kilometers'}).toFixed(3); }
    catch{ r.distKm=+turf.distance(pt,turf.centroid(S.site),{units:'kilometers'}).toFixed(3); }
  });

  // Merge with existing results (deduplicate by reference)
  const existingRefs=new Set(S.results.map(r=>r.reference).filter(Boolean));
  const newApps=inBuf.filter(r=>!r.reference||!existingRefs.has(r.reference));
  S.results.push(...newApps);

  const totUnits=S.results.reduce((a,r)=>a+(r.units||0),0);
  document.getElementById('stTotal').textContent=S.results.length;
  document.getElementById('stUnits').textContent=totUnits||'—';

  sortResults(); renderResults(); renderMarkers();
  openResults(); switchTab('res');
  notify(`${newApps.length} application${newApps.length>1?'s':''} imported and mapped.`,'ok');
  document.getElementById('csvPaste').value='';
}

// ══════════════════════════════════════════════════════════════
// PYTHON FETCH SCRIPT DOWNLOAD
// ══════════════════════════════════════════════════════════════
function downloadFetchScript() {
  const lpaList=S.lpas.map(l=>l.portal||'').filter(Boolean);
  const script=`#!/usr/bin/env python3
"""
fetch_planning.py — Companion script for Cumulative Planning Search tool
Scrapes Idox planning portals and outputs GeoJSON for import into the tool.

Usage:
  pip install requests beautifulsoup4
  python fetch_planning.py --bbox -1.15,53.70,-0.90,53.85 --years 5 --out results.geojson

Auto-detected portals: ${lpaList.join(', ')||'(draw a site first)'}
"""
import argparse, json, re, time, sys
from datetime import datetime, timedelta
from urllib.parse import urljoin
try:
    import requests
    from bs4 import BeautifulSoup
except ImportError:
    print("Install deps: pip install requests beautifulsoup4"); sys.exit(1)

PORTALS = ${JSON.stringify(Object.fromEntries(Object.entries(PORTALS).slice(0,20)),null,2)}

HEADERS = {'User-Agent':'CumulativePlanningTool/1.0'}

def geocode(address):
    r = requests.get('https://nominatim.openstreetmap.org/search',
        params={'format':'json','q':address+', UK','limit':1,'countrycodes':'gb'},
        headers={**HEADERS,'Accept-Language':'en-GB'}, timeout=8)
    d = r.json()
    if d: return float(d[0]['lat']), float(d[0]['lon'])
    return None, None

def extract_units(text):
    if not text: return 0
    for pat in [r'(\\d+)\\s*(?:no\\.?)?\\s*(?:new\\s+)?(?:residential\\s+)?(?:dwelling|unit|apartment|flat|house)', r'erection\\s+of\\s+(\\d+)', r'up\\s+to\\s+(\\d+)\\s*(?:dwelling|unit)']:
        m = re.search(pat, text, re.I)
        if m: return int(m.group(1))
    return 0

def search_idox(portal_url, from_date, to_date):
    results = []
    params = {
        'action': 'advanced', 'description': 'residential dwellings',
        'dateType': 'DC_Received', 'dateRangeType': 'custom',
        'startDateDay': from_date.day, 'startDateMonth': from_date.month, 'startDateYear': from_date.year,
        'endDateDay': to_date.day, 'endDateMonth': to_date.month, 'endDateYear': to_date.year,
    }
    try:
        r = requests.get(portal_url+'/search.do', params=params, headers=HEADERS, timeout=20)
        r.raise_for_status()
        soup = BeautifulSoup(r.text, 'html.parser')
        for el in soup.select('li.searchresult, .search-result'):
            ref  = (el.select_one('.result-title a, h2 a, .reference') or {}).get_text('').strip()
            desc = (el.select_one('.result-description, p.description') or {}).get_text('').strip()
            addr = (el.select_one('.address, .result-address') or {}).get_text('').strip()
            href = ''
            a = el.select_one('a[href*="applicationDetails"]')
            if a: href = urljoin(portal_url, a['href'])
            if not ref and not desc: continue
            lat, lng = geocode(addr) if addr else (None, None)
            time.sleep(0.35)
            results.append({'reference':ref,'description':desc,'address':addr,
                'units':extract_units(desc),'status':None,'lat':lat,'lng':lng,
                'portal_url':href,'lpa':portal_url.split('/')[2]})
    except Exception as e:
        print(f"  Error scraping {portal_url}: {e}")
    return results

def main():
    ap = argparse.ArgumentParser()
    ap.add_argument('--portal', nargs='*', help='Portal URL(s)')
    ap.add_argument('--bbox', help='minLng,minLat,maxLng,maxLat')
    ap.add_argument('--years', type=int, default=5)
    ap.add_argument('--out', default='results.geojson')
    args = ap.parse_args()

    from_date = datetime.now() - timedelta(days=365*args.years)
    to_date   = datetime.now()
    portals   = args.portal or list(PORTALS.values())[:3]

    all_results = []
    for portal in portals:
        print(f"Scraping {portal}…")
        results = search_idox(portal, from_date, to_date)
        print(f"  {len(results)} results")
        all_results.extend(results)

    # Filter by bbox if provided
    if args.bbox:
        w,s,e,n = [float(x) for x in args.bbox.split(',')]
        all_results = [r for r in all_results if r['lat'] and s<=r['lat']<=n and w<=r['lng']<=e]

    fc = {'type':'FeatureCollection','features':[
        {'type':'Feature','geometry':{'type':'Point','coordinates':[r['lng'],r['lat']]},
         'properties':{k:v for k,v in r.items() if k not in ('lat','lng')}}
        for r in all_results if r['lat'] and r['lng']
    ]}
    with open(args.out,'w') as f: json.dump(fc, f, indent=2)
    print(f"\\nWrote {len(fc['features'])} features to {args.out}")
    print("Now upload this file in the Import tab of the planning tool.")

if __name__ == '__main__': main()
`;
  const blob=new Blob([script],{type:'text/plain'});
  const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='fetch_planning.py'; a.click();
  notify('Python script downloaded.','ok');
}

// ══════════════════════════════════════════════════════════════
// EXPORTS
// ══════════════════════════════════════════════════════════════
function exportCSV() {
  const cols=['reference','description','address','units','siteHa','status','lpa','source','distKm','receivedDate','lat','lng','portalUrl'];
  const hdr=cols.join(',');
  const rows=S.results.map(r=>cols.map(c=>`"${String(r[c]??'').replace(/"/g,'""')}"`).join(','));
  dl([hdr,...rows].join('\n'),`cumulative_planning_${today()}.csv`,'text/csv');
  notify('CSV exported.');
}
function exportGeoJSON() {
  const fc={type:'FeatureCollection',features:[
    ...(S.site?[{type:'Feature',geometry:S.site.geometry||S.site,properties:{layer:'site_boundary',area_ha:+document.getElementById('siteHa').textContent}}]:[]),
    ...(S.buffer?[{type:'Feature',geometry:S.buffer.geometry||S.buffer,properties:{layer:'search_buffer',radius_km:+document.getElementById('bufKm').value}}]:[]),
    ...S.results.filter(r=>r.lat&&r.lng).map((r,i)=>({type:'Feature',geometry:{type:'Point',coordinates:[r.lng,r.lat]},properties:{n:i+1,reference:r.reference,description:r.description,address:r.address,units:r.units,site_ha:r.siteHa,status:r.status,lpa:r.lpa,source:r.source,dist_km:r.distKm,received:r.receivedDate,portal:r.portalUrl}}))
  ]};
  dl(JSON.stringify(fc,null,2),`cumulative_planning_${today()}.geojson`,'application/json');
  notify('GeoJSON exported — site boundary, buffer and all scheme points.');
}
function exportKML() {
  const marks=S.results.filter(r=>r.lat&&r.lng).map((r,i)=>{
    const c=((r.siteHa||0)>=0.5||(r.units||0)>=50)?'ff2d6b3e':'ff1a4e8c';
    return `<Placemark><name>${(r.reference||'App '+(i+1)).replace(/&/g,'&amp;')}</name><description><![CDATA[<b>${r.description||''}</b><br/>${r.address||''}<br/>Units: ${r.units||'—'} | ${r.siteHa||'—'} ha<br/>Status: ${r.status||'—'}<br/>Dist: ${r.distKm||'—'}km<br/>LPA: ${r.lpa||''}]]></description><Style><IconStyle><color>ff${c}</color><scale>0.8</scale></IconStyle></Style><Point><coordinates>${r.lng},${r.lat},0</coordinates></Point></Placemark>`;
  }).join('\n');
  dl(`<?xml version="1.0" encoding="UTF-8"?><kml xmlns="http://www.opengis.net/kml/2.2"><Document><name>Cumulative Planning Search ${today()}</name>\n${marks}\n</Document></kml>`,`cumulative_planning_${today()}.kml`,'application/vnd.google-earth.kml+xml');
  notify('KML exported.');
}
function dl(c,fn,mime){ const a=document.createElement('a'); a.href=URL.createObjectURL(new Blob([c],{type:mime})); a.download=fn; a.click(); URL.revokeObjectURL(a.href); }
function today(){ return new Date().toISOString().split('T')[0]; }

// ══════════════════════════════════════════════════════════════
// FILE UPLOAD
// ══════════════════════════════════════════════════════════════
function loadFile(e) {
  const f=e.target.files[0]; if(!f) return;
  const rd=new FileReader();
  rd.onload=ev=>{
    try {
      let gj; if(f.name.endsWith('.kml')) gj=kmlParse(ev.target.result); else gj=JSON.parse(ev.target.result);
      const feat=gj.type==='FeatureCollection'?gj.features[0]:gj;
      drawn.clearLayers(); const l=L.geoJSON(feat).addTo(drawn); map.fitBounds(l.getBounds(),{padding:[30,30]});
      processSite(feat.geometry||feat); notify('Boundary loaded.','ok');
    } catch{ notify('Could not parse file — use GeoJSON or KML.','err'); }
  };
  rd.readAsText(f); e.target.value='';
}
function kmlParse(txt) {
  const doc=new DOMParser().parseFromString(txt,'text/xml');
  const c=doc.querySelector('coordinates')?.textContent?.trim();
  if (!c) throw new Error('No coordinates');
  const ring=c.split(/\s+/).filter(Boolean).map(s=>{ const[lng,lat]=s.split(',').map(Number); return [lng,lat]; });
  return {type:'Feature',geometry:{type:'Polygon',coordinates:[ring]}};
}

// ══════════════════════════════════════════════════════════════
// UI HELPERS
// ══════════════════════════════════════════════════════════════
function logSource(name, status, count, msg) { S.sourceLog.push({name,status,count,msg}); }

function clearAll() {
  drawn.clearLayers();
  [S.siteL,S.bufL].forEach(l=>{ if(l) map.removeLayer(l); });
  S.lpaL.clearLayers(); S.lpaLabelsL.clearLayers(); S.resL.clearLayers();
  S.site=S.buffer=null; S.results=[]; S.lpas=[]; S.markers=[]; S.sourceLog=[];
  document.getElementById('siteStats').style.display='none';
  document.getElementById('lpaSection').style.display='none';
  document.getElementById('searchBtn').disabled=true;
  document.getElementById('srchHint').textContent='Draw a site boundary to enable search';
  ['stTotal','stUnits','stLPAs'].forEach(id=>document.getElementById(id).textContent='—');
  document.getElementById('resScroll').innerHTML=`<div style="display:flex;flex-direction:column;align-items:center;justify-content:center;padding:30px 16px;text-align:center;gap:9px;color:var(--muted);"><div style="width:44px;height:44px;border:1px solid var(--rule);border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:18px;opacity:0.5;">◎</div><div style="font-family:'Barlow Condensed',sans-serif;font-size:15px;color:var(--ink2);">No results yet</div><div style="font-size:11px;line-height:1.5;">Draw a site boundary and run a search.</div></div>`;
  document.getElementById('expBar').classList.remove('on');
  document.getElementById('progLog').classList.remove('on'); document.getElementById('progLog').innerHTML='';
  [1,2,3,4,5].forEach(n=>{ const s=document.getElementById('stp'+n); if(s){s.classList.remove('on','done'); if(n===1)s.classList.add('on');}});
  setStatus('idle','Ready'); closeResults();
}

function startDraw() {
  document.querySelectorAll('.tbtn').forEach(b=>b.classList.remove('active'));
  document.getElementById('btnDraw').classList.add('active');
  try{ drawCtrl._toolbars.draw._modes.polygon.handler.enable(); }catch{}
}
function fitSite() { if(!S.site){return;} const b=turf.bbox(S.site); map.fitBounds([[b[1],b[0]],[b[3],b[2]]],{padding:[40,40]}); }
function fitAll() { const layers=[S.bufL,S.siteL].filter(Boolean); if(!layers.length)return; map.fitBounds(L.featureGroup(layers).getBounds(),{padding:[40,40]}); }
function openResults()  { document.getElementById('panelR').classList.add('open'); }
function closeResults() { document.getElementById('panelR').classList.remove('open'); }
function switchTab(id) {
  ['res','pdf','imp'].forEach(t=>{
    const tid=t.charAt(0).toUpperCase()+t.slice(1);
    document.getElementById('tab'+tid).classList.toggle('on',t===id);
    document.getElementById('pane'+tid).classList.toggle('on',t===id);
  });
}
let baseIdx=0;
function cycleBase() { baseIdx=(baseIdx+1)%TILES.length; map.removeLayer(tileLayer); tileLayer=L.tileLayer(TILES[baseIdx].url,{attribution:TILES[baseIdx].attr,maxZoom:19}).addTo(map); document.getElementById('mcBase').childNodes[2].textContent=' '+TILES[baseIdx].name; }
function toggleSat() { S.satOn=!S.satOn; S.satOn?map.addLayer(satLayer):map.removeLayer(satLayer); document.getElementById('mcSat').classList.toggle('on',S.satOn); }
function stepDone(n){ const e=document.getElementById('stp'+n); if(e){e.classList.remove('on');e.classList.add('done');} }
function stepActive(n){ const e=document.getElementById('stp'+n); if(e){e.classList.add('on');e.classList.remove('done');} }
function setStatus(type,txt){ document.getElementById('stDot').className='st-dot '+type; document.getElementById('stTxt').textContent=txt; }
function setBtnWorking(on){ const b=document.getElementById('searchBtn'); b.disabled=on; b.innerHTML=on?'<div class="spin"></div> Searching databases…':'<svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg> Search Planning Databases'; }
function addLog(type,txt){ const l=document.getElementById('progLog'); const t=new Date().toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit',second:'2-digit'}); const ic=type==='ok'?'<span class="log-ok">✓</span>':type==='err'||type==='warn'?`<span class="log-${type==='warn'?'warn':'err'}">${type==='warn'?'⚠':'✗'}</span>`:'<span style="color:#555">·</span>'; l.innerHTML+=`<div class="log-line"><span class="log-t">${t}</span>${ic}<span class="log-txt">${txt}</span></div>`; l.scrollTop=l.scrollHeight; }
let ntTimer;
function notify(msg,type='info'){ const n=document.getElementById('notif'); n.textContent=msg; n.className='notif on'+(type!=='info'?' '+type:''); clearTimeout(ntTimer); ntTimer=setTimeout(()=>n.classList.remove('on'),5000); }

// Init
setTimeout(()=>notify('Draw a site boundary polygon to begin — use the polygon tool in the left panel.'),700);
</script>
</body>
</html>
