<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CIPHER // Red Team Operator</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700;900&family=VT323&family=Rajdhani:wght@300;400;600;700&display=swap" rel="stylesheet">
<style>
:root {
  --green: #00ff41;
  --red: #ff003c;
  --blue: #00d4ff;
  --purple: #bf00ff;
  --orange: #ff6600;
  --bg: #000000;
  --bg2: #030a03;
  --glass: rgba(0,255,65,0.05);
  --glass-border: rgba(0,255,65,0.2);
  --text: #c8ffc8;
  --dim: #1a3a1a;
}

*{margin:0;padding:0;box-sizing:border-box}

html{scroll-behavior:smooth;overflow-x:hidden}

body{
  background:#000;
  color:var(--text);
  font-family:'Share Tech Mono',monospace;
  cursor:none;
  overflow-x:hidden;
}

/* ── CUSTOM CURSOR ── */
#cursor{
  position:fixed;width:20px;height:20px;
  border:1px solid var(--green);
  border-radius:50%;pointer-events:none;z-index:99999;
  transform:translate(-50%,-50%);
  transition:width .2s,height .2s,background .2s;
  mix-blend-mode:screen;
}
#cursor-dot{
  position:fixed;width:4px;height:4px;
  background:var(--green);border-radius:50%;
  pointer-events:none;z-index:99999;
  transform:translate(-50%,-50%);
  box-shadow:0 0 6px var(--green);
}
#cursor.active{width:40px;height:40px;background:rgba(0,255,65,0.1)}

/* ── BOOT SCREEN ── */
#boot{
  position:fixed;inset:0;background:#000;z-index:10000;
  display:flex;flex-direction:column;align-items:center;justify-content:center;
  font-family:'VT323',monospace;font-size:18px;color:var(--green);
}
#boot.hidden{opacity:0;pointer-events:none;transition:opacity 1s}
.boot-logo{
  font-family:'Orbitron',sans-serif;font-size:clamp(2rem,6vw,5rem);
  font-weight:900;letter-spacing:.3em;
  color:transparent;
  -webkit-text-stroke:1px var(--green);
  text-shadow:0 0 30px var(--green),0 0 60px var(--green);
  margin-bottom:2rem;animation:flicker 3s infinite;
}
#boot-log{width:min(600px,90vw);text-align:left}
.boot-line{margin:2px 0;opacity:0;animation:fadeIn .1s forwards}
.boot-line.ok::after{content:" [OK]";color:var(--green)}
.boot-line.err::after{content:" [ERR]";color:var(--red)}
.boot-line.warn::after{content:" [WARN]";color:var(--orange)}
#boot-bar-wrap{margin-top:2rem;width:min(400px,80vw)}
#boot-bar{height:4px;background:var(--dim);margin-top:6px;position:relative;overflow:hidden}
#boot-fill{height:100%;width:0;background:linear-gradient(90deg,var(--green),var(--blue));
  box-shadow:0 0 10px var(--green);transition:width .05s}
#boot-enter{
  margin-top:3rem;padding:12px 40px;
  border:1px solid var(--green);color:var(--green);
  background:rgba(0,255,65,0.05);
  font-family:'Orbitron',sans-serif;font-size:1rem;
  cursor:pointer;letter-spacing:.2em;display:none;
  text-transform:uppercase;
  box-shadow:0 0 20px rgba(0,255,65,0.3),inset 0 0 20px rgba(0,255,65,0.05);
  transition:all .3s;
}
#boot-enter:hover{background:rgba(0,255,65,0.15);box-shadow:0 0 40px rgba(0,255,65,0.5)}

/* ── MATRIX CANVAS ── */
#matrix-canvas{
  position:fixed;top:0;left:0;width:100%;height:100%;
  z-index:0;opacity:0.08;pointer-events:none;
}

/* ── SCAN LINES ── */
.scanlines{
  position:fixed;inset:0;z-index:9990;pointer-events:none;
  background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,.15) 2px,rgba(0,0,0,.15) 4px);
}
.vignette{
  position:fixed;inset:0;z-index:9989;pointer-events:none;
  background:radial-gradient(ellipse at center,transparent 50%,rgba(0,0,0,.7) 100%);
}

/* ── NAV ── */
nav{
  position:fixed;top:0;left:0;right:0;z-index:9000;
  display:flex;align-items:center;justify-content:space-between;
  padding:1rem 2rem;
  background:linear-gradient(180deg,rgba(0,0,0,.9) 0%,transparent 100%);
  border-bottom:1px solid rgba(0,255,65,0.1);
}
.nav-logo{
  font-family:'Orbitron',sans-serif;font-weight:900;font-size:1.2rem;
  color:var(--green);letter-spacing:.3em;
  text-shadow:0 0 15px var(--green);
}
.nav-links{display:flex;gap:2rem;list-style:none}
.nav-links a{
  color:rgba(200,255,200,.6);text-decoration:none;
  font-size:.75rem;letter-spacing:.15em;text-transform:uppercase;
  transition:all .3s;position:relative;
}
.nav-links a::after{
  content:'';position:absolute;bottom:-4px;left:0;right:0;
  height:1px;background:var(--green);transform:scaleX(0);transition:.3s;
}
.nav-links a:hover{color:var(--green);text-shadow:0 0 10px var(--green)}
.nav-links a:hover::after{transform:scaleX(1)}
.nav-status{display:flex;align-items:center;gap:.5rem;font-size:.7rem;color:var(--green)}
.status-dot{width:6px;height:6px;background:var(--green);border-radius:50%;animation:pulse 1.5s infinite}

/* ── SECTIONS ── */
section{position:relative;z-index:1;min-height:100vh;padding:6rem 2rem}

/* ── HERO ── */
#hero{
  display:flex;flex-direction:column;align-items:center;justify-content:center;
  padding:0;overflow:hidden;
}
#globe-canvas{
  position:absolute;inset:0;z-index:0;opacity:.7;
}
.hero-content{
  position:relative;z-index:2;text-align:center;
  padding:2rem;
}
.hero-terminal{
  background:rgba(0,0,0,.7);border:1px solid rgba(0,255,65,.3);
  padding:1rem 2rem;margin-bottom:2rem;text-align:left;
  box-shadow:0 0 30px rgba(0,255,65,.1);
  backdrop-filter:blur(10px);
  max-width:500px;margin-left:auto;margin-right:auto;
}
.terminal-bar{
  display:flex;gap:.5rem;margin-bottom:.8rem;padding-bottom:.5rem;
  border-bottom:1px solid var(--dim);
}
.t-dot{width:10px;height:10px;border-radius:50%}
.t-dot.r{background:#ff5f57}.t-dot.y{background:#ffbd2e}.t-dot.g{background:#28ca41}
#type-text{color:var(--green);min-height:60px;font-size:.85rem;line-height:1.8}
.cursor-blink{display:inline-block;width:8px;height:14px;background:var(--green);animation:blink .7s infinite;vertical-align:middle}

.hero-name{
  font-family:'Orbitron',sans-serif;
  font-size:clamp(2.5rem,7vw,6rem);
  font-weight:900;
  background:linear-gradient(135deg,var(--green),var(--blue),var(--purple));
  -webkit-background-clip:text;-webkit-text-fill-color:transparent;
  line-height:1;margin:1rem 0;
  filter:drop-shadow(0 0 20px rgba(0,255,65,.5));
  animation:glitch-name 8s infinite;
}
.hero-title{
  font-family:'Rajdhani',sans-serif;font-size:clamp(1rem,2.5vw,1.4rem);
  color:rgba(200,255,200,.7);letter-spacing:.3em;text-transform:uppercase;
  margin-bottom:2.5rem;
}
.hero-btns{display:flex;gap:1.5rem;justify-content:center;flex-wrap:wrap}
.btn-primary{
  padding:14px 36px;border:1px solid var(--green);
  background:rgba(0,255,65,.08);color:var(--green);
  font-family:'Orbitron',sans-serif;font-size:.85rem;letter-spacing:.2em;
  cursor:pointer;text-decoration:none;display:inline-block;
  text-transform:uppercase;position:relative;overflow:hidden;
  transition:all .3s;
  box-shadow:0 0 20px rgba(0,255,65,.2),inset 0 0 20px rgba(0,255,65,.05);
}
.btn-primary::before{
  content:'';position:absolute;top:0;left:-100%;width:100%;height:100%;
  background:linear-gradient(90deg,transparent,rgba(0,255,65,.2),transparent);
  transition:.5s;
}
.btn-primary:hover::before{left:100%}
.btn-primary:hover{box-shadow:0 0 40px rgba(0,255,65,.4),inset 0 0 30px rgba(0,255,65,.1);transform:translateY(-2px)}
.btn-secondary{
  padding:14px 36px;border:1px solid var(--red);
  background:rgba(255,0,60,.05);color:var(--red);
  font-family:'Orbitron',sans-serif;font-size:.85rem;letter-spacing:.2em;
  cursor:pointer;text-decoration:none;display:inline-block;
  text-transform:uppercase;
  box-shadow:0 0 20px rgba(255,0,60,.2);
  transition:all .3s;
}
.btn-secondary:hover{background:rgba(255,0,60,.12);box-shadow:0 0 40px rgba(255,0,60,.4);transform:translateY(-2px)}

/* ── THREAT MAP ── */
#threat-canvas{position:absolute;inset:0;z-index:1;opacity:.4}

/* scroll indicator */
.scroll-indicator{
  position:absolute;bottom:2rem;left:50%;transform:translateX(-50%);
  display:flex;flex-direction:column;align-items:center;gap:.5rem;
  color:rgba(0,255,65,.4);font-size:.65rem;letter-spacing:.2em;z-index:2;
}
.scroll-arrow{
  width:20px;height:20px;border-right:1px solid rgba(0,255,65,.4);
  border-bottom:1px solid rgba(0,255,65,.4);
  transform:rotate(45deg);animation:scroll-bounce 2s infinite;
}

/* ── SECTION HEADER ── */
.sec-header{
  display:flex;align-items:center;gap:1rem;margin-bottom:3rem;
}
.sec-num{
  font-family:'VT323',monospace;font-size:1rem;color:var(--green);
  background:rgba(0,255,65,.1);padding:2px 8px;
  border:1px solid rgba(0,255,65,.3);
}
.sec-title{
  font-family:'Orbitron',sans-serif;font-size:clamp(1.5rem,4vw,2.5rem);
  font-weight:700;color:#fff;letter-spacing:.1em;
}
.sec-line{flex:1;height:1px;background:linear-gradient(90deg,rgba(0,255,65,.5),transparent)}

/* ── ABOUT ── */
#about{background:linear-gradient(180deg,#000 0%,#020808 100%)}
.about-grid{display:grid;grid-template-columns:1fr 1fr;gap:3rem;max-width:1200px;margin:0 auto}
@media(max-width:768px){.about-grid{grid-template-columns:1fr}}

/* ID CARD */
.id-card{
  background:rgba(0,20,10,.7);
  border:1px solid rgba(0,255,65,.3);
  padding:2rem;position:relative;overflow:hidden;
  box-shadow:0 0 40px rgba(0,255,65,.08);
  backdrop-filter:blur(10px);
}
.id-card::before{
  content:'CLASSIFIED';position:absolute;top:50%;left:50%;
  transform:translate(-50%,-50%) rotate(-30deg);
  font-family:'Orbitron',sans-serif;font-size:4rem;font-weight:900;
  color:rgba(255,0,60,.05);letter-spacing:.3em;white-space:nowrap;
  pointer-events:none;
}
.id-header{
  display:flex;align-items:center;gap:1rem;margin-bottom:1.5rem;
  padding-bottom:1rem;border-bottom:1px solid rgba(0,255,65,.2);
}
.id-avatar{
  width:80px;height:80px;border:2px solid var(--green);
  display:flex;align-items:center;justify-content:center;
  font-size:2.5rem;background:rgba(0,255,65,.05);
  position:relative;flex-shrink:0;
}
.id-avatar::after{
  content:'';position:absolute;inset:-4px;border:1px solid rgba(0,255,65,.3);
  animation:rotate-border 4s linear infinite;
}
.id-name{font-family:'Orbitron',sans-serif;font-size:1.3rem;font-weight:700;color:#fff}
.id-role{color:var(--green);font-size:.75rem;letter-spacing:.15em}
.id-tag{
  display:inline-block;padding:2px 8px;font-size:.65rem;
  border:1px solid rgba(255,0,60,.4);color:var(--red);
  background:rgba(255,0,60,.08);letter-spacing:.1em;margin-top:.3rem;
}
.id-field{margin-bottom:.8rem}
.id-label{font-size:.65rem;color:rgba(200,255,200,.4);letter-spacing:.2em;text-transform:uppercase}
.id-value{font-size:.85rem;color:var(--text)}
.id-bar-wrap{margin-top:1.5rem}
.id-bar-label{display:flex;justify-content:space-between;font-size:.7rem;margin-bottom:.3rem}
.id-bar-label span:first-child{color:rgba(200,255,200,.5);letter-spacing:.1em}
.id-bar-label span:last-child{color:var(--green)}
.id-bar{height:3px;background:rgba(0,255,65,.1);position:relative;overflow:hidden}
.id-bar-fill{height:100%;width:0;transition:width 1.5s cubic-bezier(.25,.46,.45,.94)}
.id-bar-fill.green{background:linear-gradient(90deg,var(--green),#00ff88);box-shadow:0 0 8px var(--green)}
.id-bar-fill.blue{background:linear-gradient(90deg,var(--blue),#0099ff);box-shadow:0 0 8px var(--blue)}
.id-bar-fill.purple{background:linear-gradient(90deg,var(--purple),#ff00ff);box-shadow:0 0 8px var(--purple)}
.id-bar-fill.red{background:linear-gradient(90deg,var(--red),#ff6600);box-shadow:0 0 8px var(--red)}

.about-text h3{font-family:'Orbitron',sans-serif;font-size:1rem;color:var(--green);margin-bottom:1rem;letter-spacing:.1em}
.about-text p{color:rgba(200,255,200,.7);line-height:1.8;font-size:.85rem;margin-bottom:1rem}
.about-stats{
  display:grid;grid-template-columns:1fr 1fr;gap:1rem;margin-top:2rem;
}
.stat-box{
  background:rgba(0,255,65,.04);border:1px solid rgba(0,255,65,.15);
  padding:1rem;text-align:center;
}
.stat-num{font-family:'Orbitron',sans-serif;font-size:2rem;font-weight:900;color:var(--green);
  text-shadow:0 0 20px var(--green)}
.stat-label{font-size:.65rem;letter-spacing:.15em;color:rgba(200,255,200,.4);margin-top:.2rem}

/* ── SKILLS ── */
#skills{background:#000}
.skills-grid{
  display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));
  gap:1.5rem;max-width:1200px;margin:0 auto;
}
.skill-card{
  background:rgba(0,10,20,.8);
  border:1px solid rgba(0,212,255,.15);
  padding:1.8rem;position:relative;overflow:hidden;
  cursor:pointer;transition:all .4s;
  group:hover;
}
.skill-card::before{
  content:'';position:absolute;top:0;left:0;right:0;height:2px;
  background:linear-gradient(90deg,transparent,var(--blue),transparent);
  transform:translateX(-100%);transition:.6s;
}
.skill-card:hover::before{transform:translateX(100%)}
.skill-card:hover{
  border-color:rgba(0,212,255,.5);
  box-shadow:0 0 40px rgba(0,212,255,.15),inset 0 0 40px rgba(0,212,255,.03);
  transform:translateY(-4px);
}
.skill-icon{font-size:2.5rem;margin-bottom:1rem}
.skill-name{font-family:'Orbitron',sans-serif;font-size:.9rem;font-weight:700;
  color:#fff;letter-spacing:.1em;margin-bottom:.5rem}
.skill-desc{font-size:.75rem;color:rgba(200,255,200,.5);line-height:1.6;margin-bottom:1.2rem}
.skill-tags{display:flex;flex-wrap:wrap;gap:.4rem}
.skill-tag{
  padding:2px 8px;font-size:.6rem;letter-spacing:.1em;
  border:1px solid rgba(0,212,255,.25);color:rgba(0,212,255,.7);
  background:rgba(0,212,255,.05);
}
.skill-level{
  position:absolute;top:1rem;right:1rem;
  font-family:'VT323',monospace;font-size:1.2rem;
}
.skill-level.expert{color:var(--green);text-shadow:0 0 8px var(--green)}
.skill-level.advanced{color:var(--blue);text-shadow:0 0 8px var(--blue)}
.skill-level.master{color:var(--purple);text-shadow:0 0 8px var(--purple)}

/* ── PROJECTS ── */
#projects{background:linear-gradient(180deg,#000 0%,#020508 100%)}
.projects-grid{
  display:grid;grid-template-columns:repeat(auto-fit,minmax(340px,1fr));
  gap:2rem;max-width:1200px;margin:0 auto;
}
.mission-card{
  background:rgba(5,0,15,.9);
  border:1px solid rgba(191,0,255,.2);
  position:relative;overflow:hidden;
  transition:all .4s;cursor:pointer;
}
.mission-card:hover{
  border-color:rgba(191,0,255,.6);
  box-shadow:0 0 60px rgba(191,0,255,.15),0 20px 40px rgba(0,0,0,.5);
  transform:translateY(-8px);
}
.mission-header{
  padding:1.5rem;
  border-bottom:1px solid rgba(191,0,255,.15);
  display:flex;justify-content:space-between;align-items:flex-start;
}
.mission-code{font-size:.65rem;color:rgba(191,0,255,.6);letter-spacing:.2em}
.mission-status{
  padding:3px 10px;font-size:.6rem;letter-spacing:.15em;
  border:1px solid rgba(0,255,65,.4);color:var(--green);
  background:rgba(0,255,65,.08);
}
.mission-status.active{animation:pulse-glow-green 2s infinite}
.mission-status.classified{border-color:rgba(255,0,60,.4);color:var(--red);background:rgba(255,0,60,.08)}
.mission-title{
  font-family:'Orbitron',sans-serif;font-size:1.1rem;font-weight:700;
  color:#fff;padding:1.5rem;padding-bottom:.5rem;letter-spacing:.05em;
}
.mission-desc{
  padding:0 1.5rem 1.5rem;font-size:.78rem;color:rgba(200,255,200,.55);
  line-height:1.7;
}
.mission-chain{padding:0 1.5rem 1rem}
.mission-chain-label{font-size:.6rem;letter-spacing:.2em;color:rgba(191,0,255,.6);margin-bottom:.5rem}
.chain-steps{display:flex;align-items:center;gap:.3rem;flex-wrap:wrap}
.chain-step{
  padding:2px 8px;font-size:.6rem;
  background:rgba(191,0,255,.08);border:1px solid rgba(191,0,255,.2);
  color:rgba(191,0,255,.8);
}
.chain-arrow{color:rgba(191,0,255,.4);font-size:.7rem}
.mission-footer{
  padding:1rem 1.5rem;border-top:1px solid rgba(191,0,255,.1);
  display:flex;gap:1rem;
}
.mission-btn{
  flex:1;padding:8px;text-align:center;font-size:.65rem;letter-spacing:.15em;
  text-decoration:none;cursor:pointer;border:none;font-family:'Share Tech Mono',monospace;
  transition:all .3s;text-transform:uppercase;
}
.mission-btn.demo{
  background:rgba(0,255,65,.08);border:1px solid rgba(0,255,65,.3);color:var(--green);
}
.mission-btn.demo:hover{background:rgba(0,255,65,.15);box-shadow:0 0 15px rgba(0,255,65,.3)}
.mission-btn.github{
  background:rgba(191,0,255,.08);border:1px solid rgba(191,0,255,.3);color:var(--purple);
}
.mission-btn.github:hover{background:rgba(191,0,255,.15);box-shadow:0 0 15px rgba(191,0,255,.3)}

/* ── TERMINAL ── */
#terminal-section{background:#000;padding:4rem 2rem}
.terminal-wrap{
  max-width:900px;margin:0 auto;
  background:rgba(0,5,0,.95);
  border:1px solid rgba(0,255,65,.3);
  box-shadow:0 0 60px rgba(0,255,65,.1),0 0 120px rgba(0,255,65,.05);
}
.terminal-titlebar{
  background:rgba(0,20,0,.9);padding:.7rem 1rem;
  display:flex;align-items:center;gap:1rem;
  border-bottom:1px solid rgba(0,255,65,.2);
}
.terminal-titlebar span{font-size:.7rem;color:rgba(0,255,65,.6);flex:1;text-align:center;letter-spacing:.2em}
#terminal-output{
  height:400px;overflow-y:auto;padding:1.5rem;
  font-size:.85rem;line-height:1.8;
  scrollbar-width:thin;scrollbar-color:var(--green) transparent;
}
#terminal-output::-webkit-scrollbar{width:3px}
#terminal-output::-webkit-scrollbar-thumb{background:var(--green)}
.t-out{color:rgba(200,255,200,.8)}
.t-err{color:var(--red)}
.t-info{color:var(--blue)}
.t-success{color:var(--green)}
.t-warn{color:var(--orange)}
.t-prompt{color:rgba(200,255,200,.4)}
.terminal-input-row{
  display:flex;align-items:center;padding:.5rem 1.5rem 1rem;
  border-top:1px solid rgba(0,255,65,.1);
  gap:.5rem;
}
.t-prompt-label{color:var(--green);white-space:nowrap;font-size:.85rem}
#terminal-input{
  flex:1;background:transparent;border:none;outline:none;
  color:var(--green);font-family:'Share Tech Mono',monospace;font-size:.85rem;
  caret-color:var(--green);
}

/* ── CERTS ── */
#certs{background:linear-gradient(180deg,#000 0%,#050010 100%)}
.certs-grid{
  display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));
  gap:1.5rem;max-width:1200px;margin:0 auto;
}
.cert-card{
  background:rgba(10,0,30,.7);
  border:1px solid rgba(191,0,255,.2);
  padding:1.5rem;text-align:center;
  position:relative;overflow:hidden;transition:all .4s;
}
.cert-card::after{
  content:'VERIFIED';position:absolute;bottom:.5rem;right:.5rem;
  font-size:.55rem;letter-spacing:.2em;color:rgba(0,255,65,.3);
}
.cert-card:hover{
  border-color:rgba(191,0,255,.5);
  box-shadow:0 0 40px rgba(191,0,255,.15);
  transform:translateY(-4px) rotateX(5deg);
}
.cert-icon{font-size:2.5rem;margin-bottom:.8rem}
.cert-name{
  font-family:'Orbitron',sans-serif;font-size:.8rem;font-weight:700;
  color:#fff;letter-spacing:.05em;margin-bottom:.3rem;
}
.cert-org{font-size:.65rem;color:rgba(191,0,255,.7);letter-spacing:.15em;margin-bottom:.5rem}
.cert-year{
  display:inline-block;padding:2px 10px;font-size:.6rem;
  border:1px solid rgba(0,255,65,.3);color:var(--green);
  background:rgba(0,255,65,.05);
}
.cert-shield{
  width:40px;height:40px;margin:0 auto 1rem;
  background:rgba(191,0,255,.1);border:1px solid rgba(191,0,255,.3);
  display:flex;align-items:center;justify-content:center;font-size:1.2rem;
}

/* ── DASHBOARD ── */
#dashboard{background:#000}
.dash-grid{
  display:grid;grid-template-columns:1fr 1fr 1fr;
  gap:1.5rem;max-width:1400px;margin:0 auto;
}
@media(max-width:900px){.dash-grid{grid-template-columns:1fr}}
.dash-panel{
  background:rgba(0,5,10,.8);
  border:1px solid rgba(0,212,255,.15);
  padding:1.2rem;
}
.dash-panel-title{
  font-size:.65rem;letter-spacing:.25em;color:rgba(0,212,255,.6);
  text-transform:uppercase;margin-bottom:1rem;
  padding-bottom:.5rem;border-bottom:1px solid rgba(0,212,255,.1);
  display:flex;align-items:center;gap:.5rem;
}
.live-dot{width:5px;height:5px;background:var(--red);border-radius:50%;animation:pulse 1s infinite}

/* Threat feed */
.threat-item{
  display:flex;align-items:center;gap:.8rem;padding:.4rem 0;
  border-bottom:1px solid rgba(0,212,255,.05);font-size:.7rem;
  animation:slideIn .3s ease;
}
.threat-flag{font-size:1rem}
.threat-type{color:var(--red);letter-spacing:.05em}
.threat-target{color:rgba(200,255,200,.4)}
.threat-time{margin-left:auto;color:rgba(200,255,200,.3)}

/* SOC widgets */
.soc-metric{
  display:flex;justify-content:space-between;align-items:center;
  padding:.5rem 0;border-bottom:1px solid rgba(0,212,255,.05);
}
.soc-label{font-size:.7rem;color:rgba(200,255,200,.5)}
.soc-value{font-family:'Orbitron',sans-serif;font-size:.9rem}
.soc-value.green{color:var(--green);text-shadow:0 0 8px var(--green)}
.soc-value.red{color:var(--red);text-shadow:0 0 8px var(--red)}
.soc-value.blue{color:var(--blue);text-shadow:0 0 8px var(--blue)}

/* Radar */
#radar-canvas{display:block;margin:0 auto}

/* Packet flow */
.packet-row{
  height:2px;background:rgba(0,255,65,.1);margin:8px 0;
  position:relative;overflow:hidden;
}
.packet{
  position:absolute;height:100%;width:20px;
  background:linear-gradient(90deg,transparent,var(--green),transparent);
  animation:packet-flow linear infinite;
  box-shadow:0 0 4px var(--green);
}
.packet.blue{background:linear-gradient(90deg,transparent,var(--blue),transparent);box-shadow:0 0 4px var(--blue)}
.packet.red{background:linear-gradient(90deg,transparent,var(--red),transparent);box-shadow:0 0 4px var(--red)}

/* ── GITHUB / WRITEUPS ── */
#writeups{background:linear-gradient(180deg,#000 0%,#000510 100%)}
.writeups-wrap{max-width:900px;margin:0 auto}
.db-header{
  background:rgba(0,5,20,.9);border:1px solid rgba(0,212,255,.2);
  padding:.8rem 1.2rem;display:flex;align-items:center;gap:1rem;
  font-size:.7rem;color:rgba(0,212,255,.6);letter-spacing:.1em;
  margin-bottom:1px;
}
.writeup-item{
  background:rgba(0,2,10,.7);border:1px solid rgba(0,212,255,.08);
  border-top:none;padding:1rem 1.2rem;
  display:flex;align-items:center;gap:1rem;cursor:pointer;
  transition:all .3s;
}
.writeup-item:hover{background:rgba(0,212,255,.05);border-color:rgba(0,212,255,.25)}
.writeup-id{color:rgba(0,212,255,.4);font-size:.65rem;width:80px;flex-shrink:0}
.writeup-title{flex:1;color:var(--text);font-size:.8rem}
.writeup-sev{
  padding:2px 8px;font-size:.6rem;letter-spacing:.1em;
}
.writeup-sev.critical{border:1px solid rgba(255,0,60,.4);color:var(--red);background:rgba(255,0,60,.08)}
.writeup-sev.high{border:1px solid rgba(255,102,0,.4);color:var(--orange);background:rgba(255,102,0,.08)}
.writeup-sev.medium{border:1px solid rgba(255,200,0,.4);color:#ffc800;background:rgba(255,200,0,.08)}
.writeup-date{color:rgba(200,255,200,.3);font-size:.65rem;width:80px;text-align:right;flex-shrink:0}

/* ── CONTACT ── */
#contact{background:#000;padding:6rem 2rem}
.contact-wrap{max-width:700px;margin:0 auto}
.contact-terminal{
  background:rgba(0,5,0,.95);
  border:1px solid rgba(0,255,65,.3);
  box-shadow:0 0 60px rgba(0,255,65,.08);
}
.contact-header{
  background:rgba(0,20,0,.9);padding:1rem 1.5rem;
  border-bottom:1px solid rgba(0,255,65,.2);
  font-size:.7rem;color:rgba(0,255,65,.6);letter-spacing:.2em;
}
.contact-body{padding:2rem}
.contact-info{margin-bottom:2rem}
.contact-row{
  display:flex;align-items:center;gap:1rem;padding:.8rem 0;
  border-bottom:1px solid rgba(0,255,65,.08);font-size:.8rem;
}
.contact-key{color:rgba(200,255,200,.4);width:120px;flex-shrink:0;letter-spacing:.1em}
.contact-val{color:var(--green)}
.contact-val a{color:var(--green);text-decoration:none}
.contact-val a:hover{text-shadow:0 0 10px var(--green)}
.contact-form{display:flex;flex-direction:column;gap:1rem}
.cf-group{position:relative}
.cf-label{font-size:.65rem;letter-spacing:.2em;color:rgba(200,255,200,.4);margin-bottom:.4rem;display:block}
.cf-input,.cf-textarea{
  width:100%;background:rgba(0,255,65,.03);
  border:1px solid rgba(0,255,65,.2);
  color:var(--text);font-family:'Share Tech Mono',monospace;font-size:.8rem;
  padding:.8rem 1rem;outline:none;
  transition:all .3s;
}
.cf-input:focus,.cf-textarea:focus{
  border-color:rgba(0,255,65,.5);
  box-shadow:0 0 15px rgba(0,255,65,.1);
  background:rgba(0,255,65,.05);
}
.cf-textarea{height:100px;resize:none}
.cf-submit{
  padding:12px;background:rgba(0,255,65,.08);
  border:1px solid rgba(0,255,65,.4);color:var(--green);
  font-family:'Orbitron',sans-serif;font-size:.8rem;letter-spacing:.2em;
  cursor:pointer;text-transform:uppercase;transition:all .3s;
}
.cf-submit:hover{background:rgba(0,255,65,.15);box-shadow:0 0 30px rgba(0,255,65,.3)}

/* ── FOOTER ── */
footer{
  border-top:1px solid rgba(0,255,65,.1);
  padding:2rem;text-align:center;
  font-size:.7rem;color:rgba(200,255,200,.3);
  letter-spacing:.2em;
}
footer span{color:var(--green)}

/* ── ANIMATIONS ── */
@keyframes flicker{0%,100%{opacity:1}92%{opacity:.9}93%{opacity:.3}94%{opacity:.9}96%{opacity:.6}97%{opacity:1}}
@keyframes blink{0%,100%{opacity:1}50%{opacity:0}}
@keyframes pulse{0%,100%{opacity:1;transform:scale(1)}50%{opacity:.5;transform:scale(.8)}}
@keyframes pulse-glow-green{0%,100%{box-shadow:0 0 5px rgba(0,255,65,.3)}50%{box-shadow:0 0 15px rgba(0,255,65,.7)}}
@keyframes scroll-bounce{0%,100%{transform:rotate(45deg) translateY(0)}50%{transform:rotate(45deg) translateY(4px)}}
@keyframes rotate-border{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}
@keyframes glitch-name{
  0%,100%{filter:drop-shadow(0 0 20px rgba(0,255,65,.5))}
  92%{filter:drop-shadow(0 0 20px rgba(0,255,65,.5))}
  93%{filter:drop-shadow(3px 0 10px rgba(255,0,60,.8)) drop-shadow(-3px 0 10px rgba(0,212,255,.8));transform:skewX(2deg)}
  94%{filter:drop-shadow(0 0 20px rgba(0,255,65,.5));transform:skewX(0)}
}
@keyframes fadeIn{to{opacity:1}}
@keyframes slideIn{from{opacity:0;transform:translateX(-10px)}to{opacity:1;transform:translateX(0)}}
@keyframes packet-flow{from{left:-20px}to{left:100%}}

/* grid lines bg */
.cyber-grid{
  position:fixed;inset:0;z-index:0;pointer-events:none;opacity:.03;
  background-image:
    linear-gradient(rgba(0,255,65,.5) 1px,transparent 1px),
    linear-gradient(90deg,rgba(0,255,65,.5) 1px,transparent 1px);
  background-size:40px 40px;
}

/* glitch text */
.glitch{position:relative}
.glitch::before,.glitch::after{
  content:attr(data-text);position:absolute;inset:0;
  font:inherit;color:inherit;
}
.glitch::before{left:2px;text-shadow:-1px 0 var(--red);clip-path:polygon(0 20%,100% 20%,100% 40%,0 40%);
  animation:glitch1 4s infinite}
.glitch::after{left:-2px;text-shadow:1px 0 var(--blue);clip-path:polygon(0 60%,100% 60%,100% 80%,0 80%);
  animation:glitch2 4s infinite}
@keyframes glitch1{0%,90%,100%{transform:translate(0)}93%{transform:translate(-3px,1px)}96%{transform:translate(3px,-1px)}}
@keyframes glitch2{0%,90%,100%{transform:translate(0)}94%{transform:translate(3px,1px)}97%{transform:translate(-3px,-1px)}}

/* reveal on scroll */
.reveal{opacity:0;transform:translateY(30px);transition:opacity .8s,transform .8s}
.reveal.visible{opacity:1;transform:translateY(0)}

/* mobile nav */
@media(max-width:600px){
  .nav-links{display:none}
  .about-stats{grid-template-columns:1fr 1fr}
}
</style>
</head>
<body>

<!-- Custom Cursor -->
<div id="cursor"></div>
<div id="cursor-dot"></div>

<!-- Background Effects -->
<canvas id="matrix-canvas"></canvas>
<div class="cyber-grid"></div>
<div class="scanlines"></div>
<div class="vignette"></div>

<!-- BOOT SCREEN -->
<div id="boot">
  <div class="boot-logo">CIPHER</div>
  <div id="boot-log"></div>
  <div id="boot-bar-wrap">
    <div style="font-size:.65rem;color:rgba(0,255,65,.5);letter-spacing:.2em">SYSTEM INITIALIZATION</div>
    <div id="boot-bar"><div id="boot-fill"></div></div>
  </div>
  <button id="boot-enter" onclick="enterSite()">[ ENTER THE SYSTEM ]</button>
</div>

<!-- NAV -->
<nav id="main-nav" style="opacity:0">
  <div class="nav-logo">CIPHER//SYS</div>
  <ul class="nav-links">
    <li><a href="#about">Profile</a></li>
    <li><a href="#skills">Arsenal</a></li>
    <li><a href="#projects">Operations</a></li>
    <li><a href="#terminal-section">Terminal</a></li>
    <li><a href="#dashboard">Dashboard</a></li>
    <li><a href="#contact">Secure Comms</a></li>
  </ul>
  <div class="nav-status">
    <div class="status-dot"></div>
    <span>ONLINE // ENCRYPTED</span>
  </div>
</nav>

<!-- HERO -->
<section id="hero">
  <canvas id="threat-canvas"></canvas>
  <canvas id="globe-canvas"></canvas>

  <div class="hero-content">
    <div class="hero-terminal">
      <div class="terminal-bar">
        <div class="t-dot r"></div><div class="t-dot y"></div><div class="t-dot g"></div>
        <span style="font-size:.6rem;color:rgba(0,255,65,.4);letter-spacing:.1em;margin-left:.5rem">root@cipher:~#</span>
      </div>
      <div id="type-text"></div>
    </div>

    <h1 class="hero-name glitch" data-text="CIPHER">CIPHER</h1>
    <p class="hero-title">Ethical Hacker &nbsp;|&nbsp; Penetration Tester &nbsp;|&nbsp; Red Team Operator</p>

    <div class="hero-btns">
      <a href="#projects" class="btn-primary">[ Launch Mission ]</a>
      <a href="#writeups" class="btn-secondary">[ View Exploits ]</a>
    </div>
  </div>

  <div class="scroll-indicator">
    <span>SCROLL TO INITIATE</span>
    <div class="scroll-arrow"></div>
  </div>
</section>

<!-- ABOUT -->
<section id="about" class="reveal">
  <div class="sec-header" style="max-width:1200px;margin:0 auto 3rem">
    <div class="sec-num">01</div>
    <h2 class="sec-title">OPERATOR PROFILE</h2>
    <div class="sec-line"></div>
  </div>

  <div class="about-grid">
    <div class="id-card reveal">
      <div style="font-size:.55rem;letter-spacing:.3em;color:rgba(255,0,60,.5);margin-bottom:1rem">
        ██ CLASSIFIED — LEVEL 5 CLEARANCE REQUIRED ██
      </div>
      <div class="id-header">
        <div class="id-avatar">👤</div>
        <div>
          <div class="id-name">CIPHER</div>
          <div class="id-role">SENIOR RED TEAM OPERATOR</div>
          <div class="id-tag">TOP SECRET // REDACTED</div>
        </div>
      </div>
      <div class="id-field"><div class="id-label">Designation</div><div class="id-value">Ghost Protocol – Unit 0x1</div></div>
      <div class="id-field"><div class="id-label">Clearance</div><div class="id-value" style="color:var(--red)">CLASSIFIED – REDACTED</div></div>
      <div class="id-field"><div class="id-label">Specialization</div><div class="id-value">Offensive Security / APT Simulation</div></div>
      <div class="id-field"><div class="id-label">Threat Model</div><div class="id-value">Nation-State / Advanced Persistent</div></div>
      <div class="id-field"><div class="id-label">Active Since</div><div class="id-value">2018 — Present</div></div>

      <div class="id-bar-wrap">
        <div class="id-bar-label"><span>Offensive Ops</span><span>95%</span></div>
        <div class="id-bar"><div class="id-bar-fill red" data-width="95"></div></div>
        <br>
        <div class="id-bar-label"><span>Network Exploitation</span><span>90%</span></div>
        <div class="id-bar"><div class="id-bar-fill blue" data-width="90"></div></div>
        <br>
        <div class="id-bar-label"><span>Malware Analysis</span><span>85%</span></div>
        <div class="id-bar"><div class="id-bar-fill purple" data-width="85"></div></div>
        <br>
        <div class="id-bar-label"><span>OSINT & Recon</span><span>92%</span></div>
        <div class="id-bar"><div class="id-bar-fill green" data-width="92"></div></div>
      </div>
    </div>

    <div class="about-text reveal">
      <h3>// MISSION BRIEFING</h3>
      <p>I am an elite Ethical Hacker and Red Team Operator with 6+ years specializing in advanced persistent threat (APT) simulation, penetration testing, and adversarial security research.</p>
      <p>Operating in the shadows between defender and attacker — I break systems so organizations can build stronger ones. Every engagement is a mission; every vulnerability, an intelligence asset.</p>
      <p>My toolkit spans from web application exploitation and Active Directory attacks to custom malware development and evasion techniques. I think like an adversary so I can defend like a fortress.</p>
      <p style="color:var(--green);font-size:.75rem;">// Currently accepting: Bug Bounty collaborations, Red Team engagements, CTF partnerships</p>

      <div class="about-stats">
        <div class="stat-box">
          <div class="stat-num" id="ctr-bugs">0</div>
          <div class="stat-label">Bugs Reported</div>
        </div>
        <div class="stat-box" style="border-color:rgba(0,212,255,.15)">
          <div class="stat-num" style="color:var(--blue)" id="ctr-pentests">0</div>
          <div class="stat-label">Pentest Engagements</div>
        </div>
        <div class="stat-box" style="border-color:rgba(191,0,255,.15)">
          <div class="stat-num" style="color:var(--purple)" id="ctr-certs">0</div>
          <div class="stat-label">Certifications</div>
        </div>
        <div class="stat-box" style="border-color:rgba(255,0,60,.15)">
          <div class="stat-num" style="color:var(--red)" id="ctr-cves">0</div>
          <div class="stat-label">CVEs Found</div>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- SKILLS -->
<section id="skills">
  <div class="sec-header" style="max-width:1200px;margin:0 auto 3rem">
    <div class="sec-num">02</div>
    <h2 class="sec-title">SKILLS ARSENAL</h2>
    <div class="sec-line"></div>
  </div>

  <div class="skills-grid reveal">
    <div class="skill-card">
      <div class="skill-level expert">EXPERT</div>
      <div class="skill-icon">🌐</div>
      <div class="skill-name">Web Pentesting</div>
      <div class="skill-desc">OWASP Top 10, SQLi, XSS, SSRF, XXE, IDOR, authentication bypass, API exploitation, and business logic vulnerabilities.</div>
      <div class="skill-tags">
        <span class="skill-tag">Burp Suite</span><span class="skill-tag">SQLmap</span>
        <span class="skill-tag">OWASP ZAP</span><span class="skill-tag">Nuclei</span>
      </div>
    </div>
    <div class="skill-card">
      <div class="skill-level master">MASTER</div>
      <div class="skill-icon">🏛️</div>
      <div class="skill-name">Active Directory</div>
      <div class="skill-desc">Kerberoasting, Pass-the-Hash, DCSync, BloodHound enumeration, privilege escalation and domain dominance.</div>
      <div class="skill-tags">
        <span class="skill-tag">BloodHound</span><span class="skill-tag">Mimikatz</span>
        <span class="skill-tag">Impacket</span><span class="skill-tag">CrackMapExec</span>
      </div>
    </div>
    <div class="skill-card">
      <div class="skill-level master">MASTER</div>
      <div class="skill-icon">🎯</div>
      <div class="skill-name">Red Teaming</div>
      <div class="skill-desc">Full-scope adversary simulations, C2 infrastructure setup, phishing campaigns, lateral movement and OPSEC.</div>
      <div class="skill-tags">
        <span class="skill-tag">Cobalt Strike</span><span class="skill-tag">Havoc</span>
        <span class="skill-tag">Sliver</span><span class="skill-tag">Metasploit</span>
      </div>
    </div>
    <div class="skill-card">
      <div class="skill-level expert">EXPERT</div>
      <div class="skill-icon">💰</div>
      <div class="skill-name">Bug Bounty</div>
      <div class="skill-desc">Hall-of-fame entries across HackerOne and Bugcrowd. P1/Critical findings in Fortune 500 programs, RCE and auth bypass specialists.</div>
      <div class="skill-tags">
        <span class="skill-tag">HackerOne</span><span class="skill-tag">Bugcrowd</span>
        <span class="skill-tag">Recon-ng</span><span class="skill-tag">Amass</span>
      </div>
    </div>
    <div class="skill-card">
      <div class="skill-level advanced">ADVANCED</div>
      <div class="skill-icon">🔌</div>
      <div class="skill-name">Network Exploitation</div>
      <div class="skill-desc">Network discovery, MITM attacks, protocol exploits, VPN/firewall bypass, wireless pentesting and pivoting techniques.</div>
      <div class="skill-tags">
        <span class="skill-tag">Nmap</span><span class="skill-tag">Wireshark</span>
        <span class="skill-tag">Scapy</span><span class="skill-tag">Responder</span>
      </div>
    </div>
    <div class="skill-card">
      <div class="skill-level advanced">ADVANCED</div>
      <div class="skill-icon">🦠</div>
      <div class="skill-name">Malware Analysis</div>
      <div class="skill-desc">Static and dynamic analysis, sandbox evasion, reverse engineering, custom implant development and payload obfuscation.</div>
      <div class="skill-tags">
        <span class="skill-tag">Ghidra</span><span class="skill-tag">x64dbg</span>
        <span class="skill-tag">IDA Pro</span><span class="skill-tag">YARA</span>
      </div>
    </div>
  </div>
</section>

<!-- PROJECTS -->
<section id="projects">
  <div class="sec-header" style="max-width:1200px;margin:0 auto 3rem">
    <div class="sec-num">03</div>
    <h2 class="sec-title">OPERATIONS</h2>
    <div class="sec-line"></div>
  </div>

  <div class="projects-grid reveal">
    <div class="mission-card">
      <div class="mission-header">
        <div>
          <div class="mission-code">OP // 0x001 // CLASSIFIED</div>
        </div>
        <div class="mission-status active">ACTIVE</div>
      </div>
      <div class="mission-title">Operation ShadowRoot</div>
      <div class="mission-desc">Full-chain exploit targeting enterprise web infrastructure. Achieved RCE via chained SSRF → internal metadata exposure → IAM privilege escalation. Discovered in major cloud provider's internal tooling.</div>
      <div class="mission-chain">
        <div class="mission-chain-label">// EXPLOIT CHAIN</div>
        <div class="chain-steps">
          <span class="chain-step">Recon</span><span class="chain-arrow">→</span>
          <span class="chain-step">SSRF</span><span class="chain-arrow">→</span>
          <span class="chain-step">IMDS</span><span class="chain-arrow">→</span>
          <span class="chain-step">IAM Priv-Esc</span><span class="chain-arrow">→</span>
          <span class="chain-step">RCE</span>
        </div>
      </div>
      <div class="mission-footer">
        <a href="#" class="mission-btn demo">Live Demo</a>
        <a href="#" class="mission-btn github">GitHub</a>
      </div>
    </div>

    <div class="mission-card">
      <div class="mission-header">
        <div>
          <div class="mission-code">OP // 0x002 // RED TEAM</div>
        </div>
        <div class="mission-status">COMPLETE</div>
      </div>
      <div class="mission-title">Operation GhostRecon</div>
      <div class="mission-desc">Advanced red team engagement simulating APT29 TTPs. Achieved domain dominance in 48 hours via phishing → initial access → AD lateral movement → DCSync → domain admin.</div>
      <div class="mission-chain">
        <div class="mission-chain-label">// EXPLOIT CHAIN</div>
        <div class="chain-steps">
          <span class="chain-step">Phishing</span><span class="chain-arrow">→</span>
          <span class="chain-step">C2 Beacon</span><span class="chain-arrow">→</span>
          <span class="chain-step">Kerberoast</span><span class="chain-arrow">→</span>
          <span class="chain-step">DCSync</span>
        </div>
      </div>
      <div class="mission-footer">
        <a href="#" class="mission-btn demo">Write-up</a>
        <a href="#" class="mission-btn github">GitHub</a>
      </div>
    </div>

    <div class="mission-card">
      <div class="mission-header">
        <div>
          <div class="mission-code">OP // 0x003 // 0DAY</div>
        </div>
        <div class="mission-status classified">CLASSIFIED</div>
      </div>
      <div class="mission-title">Operation ZeroDay</div>
      <div class="mission-desc">Discovered a zero-day vulnerability in a widely deployed enterprise VPN solution. Responsible disclosure led to CVE assignment and critical patch deployment across 50,000+ endpoints.</div>
      <div class="mission-chain">
        <div class="mission-chain-label">// EXPLOIT CHAIN</div>
        <div class="chain-steps">
          <span class="chain-step">Fuzzing</span><span class="chain-arrow">→</span>
          <span class="chain-step">Buffer Overflow</span><span class="chain-arrow">→</span>
          <span class="chain-step">Auth Bypass</span><span class="chain-arrow">→</span>
          <span class="chain-step">CVE</span>
        </div>
      </div>
      <div class="mission-footer">
        <a href="#" class="mission-btn demo">Advisory</a>
        <a href="#" class="mission-btn github">PoC</a>
      </div>
    </div>

    <div class="mission-card">
      <div class="mission-header">
        <div><div class="mission-code">OP // 0x004 // TOOL</div></div>
        <div class="mission-status active">OPEN SOURCE</div>
      </div>
      <div class="mission-title">Operation PhantomC2</div>
      <div class="mission-desc">Custom Command & Control framework with encrypted channels, modular post-exploitation modules, and advanced AV/EDR evasion. 2k+ stars on GitHub.</div>
      <div class="mission-chain">
        <div class="mission-chain-label">// STACK</div>
        <div class="chain-steps">
          <span class="chain-step">Go</span><span class="chain-arrow">+</span>
          <span class="chain-step">Python</span><span class="chain-arrow">+</span>
          <span class="chain-step">HTTPS/DNS</span><span class="chain-arrow">+</span>
          <span class="chain-step">mTLS</span>
        </div>
      </div>
      <div class="mission-footer">
        <a href="#" class="mission-btn demo">Docs</a>
        <a href="#" class="mission-btn github">GitHub ⭐ 2.1k</a>
      </div>
    </div>

    <div class="mission-card">
      <div class="mission-header">
        <div><div class="mission-code">OP // 0x005 // BUG BOUNTY</div></div>
        <div class="mission-status">P1 CRITICAL</div>
      </div>
      <div class="mission-title">Operation NightCrawler</div>
      <div class="mission-desc">Account takeover via JWT algorithm confusion attack combined with OAuth state parameter manipulation. Affected 10M+ user accounts of a Fortune 100 company. $40,000 bounty.</div>
      <div class="mission-chain">
        <div class="mission-chain-label">// EXPLOIT CHAIN</div>
        <div class="chain-steps">
          <span class="chain-step">JWT Confusion</span><span class="chain-arrow">→</span>
          <span class="chain-step">OAuth Bypass</span><span class="chain-arrow">→</span>
          <span class="chain-step">ATO</span>
        </div>
      </div>
      <div class="mission-footer">
        <a href="#" class="mission-btn demo">Write-up</a>
        <a href="#" class="mission-btn github">PoC</a>
      </div>
    </div>

    <div class="mission-card">
      <div class="mission-header">
        <div><div class="mission-code">OP // 0x006 // RESEARCH</div></div>
        <div class="mission-status">PUBLISHED</div>
      </div>
      <div class="mission-title">Operation IronVeil</div>
      <div class="mission-desc">Research project on EDR evasion techniques using process hollowing, syscall unhooking and AMSI bypass. Published at DEF CON 31. Tools open-sourced.</div>
      <div class="mission-chain">
        <div class="mission-chain-label">// TECHNIQUES</div>
        <div class="chain-steps">
          <span class="chain-step">Syscall Hook</span><span class="chain-arrow">→</span>
          <span class="chain-step">Process Hollow</span><span class="chain-arrow">→</span>
          <span class="chain-step">AMSI Bypass</span>
        </div>
      </div>
      <div class="mission-footer">
        <a href="#" class="mission-btn demo">Paper</a>
        <a href="#" class="mission-btn github">Tools</a>
      </div>
    </div>
  </div>
</section>

<!-- TERMINAL -->
<section id="terminal-section">
  <div class="sec-header" style="max-width:900px;margin:0 auto 3rem">
    <div class="sec-num">04</div>
    <h2 class="sec-title">INTERACTIVE TERMINAL</h2>
    <div class="sec-line"></div>
  </div>

  <div class="terminal-wrap reveal">
    <div class="terminal-titlebar">
      <div class="t-dot r"></div><div class="t-dot y"></div><div class="t-dot g"></div>
      <span>cipher@redteam — bash — 80x24</span>
    </div>
    <div id="terminal-output">
      <div class="t-success">╔══════════════════════════════════════════════╗</div>
      <div class="t-success">║   CIPHER TERMINAL v2.4.1 — SECURE SHELL      ║</div>
      <div class="t-success">╚══════════════════════════════════════════════╝</div>
      <div class="t-info">System initialized. All channels encrypted.</div>
      <div class="t-out">Type <span class="t-success">help</span> for available commands.</div>
      <div><br></div>
    </div>
    <div class="terminal-input-row">
      <span class="t-prompt-label">cipher@redteam:~$</span>
      <input type="text" id="terminal-input" placeholder="type a command..." autocomplete="off" spellcheck="false">
    </div>
  </div>
</section>

<!-- CERTS -->
<section id="certs">
  <div class="sec-header" style="max-width:1200px;margin:0 auto 3rem">
    <div class="sec-num">05</div>
    <h2 class="sec-title">CREDENTIALS VAULT</h2>
    <div class="sec-line"></div>
  </div>

  <div class="certs-grid reveal">
    <div class="cert-card">
      <div class="cert-shield">🛡️</div>
      <div class="cert-icon">🔴</div>
      <div class="cert-name">OSCP</div>
      <div class="cert-org">OFFENSIVE SECURITY</div>
      <div class="cert-year">2022</div>
    </div>
    <div class="cert-card">
      <div class="cert-shield">🛡️</div>
      <div class="cert-icon">⚡</div>
      <div class="cert-name">CRTO</div>
      <div class="cert-org">ZERO-POINT SECURITY</div>
      <div class="cert-year">2023</div>
    </div>
    <div class="cert-card">
      <div class="cert-shield">🛡️</div>
      <div class="cert-icon">🌐</div>
      <div class="cert-name">BSCP</div>
      <div class="cert-org">PORTSWIGGER / BURP</div>
      <div class="cert-year">2023</div>
    </div>
    <div class="cert-card">
      <div class="cert-shield">🛡️</div>
      <div class="cert-icon">🔵</div>
      <div class="cert-name">CEH Master</div>
      <div class="cert-org">EC-COUNCIL</div>
      <div class="cert-year">2021</div>
    </div>
    <div class="cert-card">
      <div class="cert-shield">🛡️</div>
      <div class="cert-icon">☁️</div>
      <div class="cert-name">AWS Security</div>
      <div class="cert-org">AMAZON WEB SERVICES</div>
      <div class="cert-year">2024</div>
    </div>
    <div class="cert-card">
      <div class="cert-shield">🛡️</div>
      <div class="cert-icon">💜</div>
      <div class="cert-name">CRTE</div>
      <div class="cert-org">ALTERED SECURITY</div>
      <div class="cert-year">2024</div>
    </div>
  </div>
</section>

<!-- DASHBOARD -->
<section id="dashboard">
  <div class="sec-header" style="max-width:1400px;margin:0 auto 3rem">
    <div class="sec-num">06</div>
    <h2 class="sec-title">ATTACK SIMULATION DASHBOARD</h2>
    <div class="sec-line"></div>
  </div>

  <div class="dash-grid reveal">
    <!-- Threat feed -->
    <div class="dash-panel">
      <div class="dash-panel-title"><div class="live-dot"></div> LIVE THREAT FEED</div>
      <div id="threat-feed"></div>
    </div>

    <!-- SOC metrics -->
    <div class="dash-panel">
      <div class="dash-panel-title"><div class="live-dot"></div> SOC METRICS</div>
      <div class="soc-metric"><span class="soc-label">Active Threats</span><span class="soc-value red" id="metric-threats">247</span></div>
      <div class="soc-metric"><span class="soc-label">Blocked Attacks</span><span class="soc-value green" id="metric-blocked">18,423</span></div>
      <div class="soc-metric"><span class="soc-label">Packets/sec</span><span class="soc-value blue" id="metric-packets">9,841</span></div>
      <div class="soc-metric"><span class="soc-label">Open Ports Scanned</span><span class="soc-value red" id="metric-ports">65,535</span></div>
      <div class="soc-metric"><span class="soc-label">Systems Compromised</span><span class="soc-value red">3</span></div>
      <div class="soc-metric"><span class="soc-label">Encryption Status</span><span class="soc-value green">AES-256</span></div>
      <br>
      <div class="dash-panel-title" style="margin-top:1rem">PACKET FLOW</div>
      <div id="packet-flow"></div>
    </div>

    <!-- Radar -->
    <div class="dash-panel">
      <div class="dash-panel-title"><div class="live-dot"></div> THREAT RADAR</div>
      <canvas id="radar-canvas" width="280" height="280"></canvas>
    </div>
  </div>
</section>

<!-- WRITEUPS -->
<section id="writeups">
  <div class="sec-header" style="max-width:900px;margin:0 auto 3rem">
    <div class="sec-num">07</div>
    <h2 class="sec-title">EXPLOIT DATABASE</h2>
    <div class="sec-line"></div>
  </div>

  <div class="writeups-wrap reveal">
    <div class="db-header">
      <span style="width:80px">ID</span>
      <span style="flex:1">TITLE</span>
      <span style="width:80px">SEVERITY</span>
      <span style="width:80px;text-align:right">DATE</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">CVE-2024-001</span>
      <span class="writeup-title">JWT Algorithm Confusion → Account Takeover (Fortune 100)</span>
      <span class="writeup-sev critical">CRITICAL</span>
      <span class="writeup-date">2024-09</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">CVE-2023-891</span>
      <span class="writeup-title">Enterprise VPN Zero-Day: Pre-auth Buffer Overflow RCE</span>
      <span class="writeup-sev critical">CRITICAL</span>
      <span class="writeup-date">2023-11</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">WU-2024-012</span>
      <span class="writeup-title">SSRF to IMDS to IAM Privilege Escalation Chain</span>
      <span class="writeup-sev high">HIGH</span>
      <span class="writeup-date">2024-06</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">WU-2024-008</span>
      <span class="writeup-title">Active Directory DCSync via Kerberoasting Chain</span>
      <span class="writeup-sev high">HIGH</span>
      <span class="writeup-date">2024-03</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">WU-2023-044</span>
      <span class="writeup-title">EDR Evasion: Syscall Unhooking + Process Hollowing</span>
      <span class="writeup-sev high">HIGH</span>
      <span class="writeup-date">2023-08</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">WU-2023-031</span>
      <span class="writeup-title">OAuth 2.0 State Parameter Bypass → Mass ATO</span>
      <span class="writeup-sev critical">CRITICAL</span>
      <span class="writeup-date">2023-05</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">WU-2022-017</span>
      <span class="writeup-title">GraphQL Introspection → IDOR → PII Exfiltration</span>
      <span class="writeup-sev medium">MEDIUM</span>
      <span class="writeup-date">2022-12</span>
    </div>
    <div class="writeup-item">
      <span class="writeup-id">WU-2022-009</span>
      <span class="writeup-title">XXE in SAML Parser → Internal Network Mapping</span>
      <span class="writeup-sev high">HIGH</span>
      <span class="writeup-date">2022-07</span>
    </div>
  </div>
</section>

<!-- CONTACT -->
<section id="contact">
  <div class="sec-header" style="max-width:700px;margin:0 auto 3rem">
    <div class="sec-num">08</div>
    <h2 class="sec-title">SECURE COMMS</h2>
    <div class="sec-line"></div>
  </div>

  <div class="contact-wrap reveal">
    <div class="contact-terminal">
      <div class="contact-header">
        🔐 ENCRYPTED COMMUNICATION TERMINAL // PGP ENABLED // TLS 1.3
      </div>
      <div class="contact-body">
        <div class="contact-info">
          <div class="contact-row">
            <span class="contact-key">Signal</span>
            <span class="contact-val">@cipher_redteam</span>
          </div>
          <div class="contact-row">
            <span class="contact-key">Email</span>
            <span class="contact-val"><a href="mailto:cipher@protonmail.com">cipher@protonmail.com</a></span>
          </div>
          <div class="contact-row">
            <span class="contact-key">GitHub</span>
            <span class="contact-val"><a href="#">github.com/cipher-sec</a></span>
          </div>
          <div class="contact-row">
            <span class="contact-key">HackerOne</span>
            <span class="contact-val"><a href="#">hackerone.com/cipher</a></span>
          </div>
          <div class="contact-row">
            <span class="contact-key">LinkedIn</span>
            <span class="contact-val"><a href="#">linkedin.com/in/cipher-sec</a></span>
          </div>
          <div class="contact-row">
            <span class="contact-key">PGP Key</span>
            <span class="contact-val" style="color:rgba(0,255,65,.5)">0xDEADBEEF C1PH3R</span>
          </div>
        </div>

        <div class="contact-form">
          <div style="font-size:.65rem;letter-spacing:.2em;color:rgba(0,255,65,.5);margin-bottom:1rem">
            // TRANSMIT ENCRYPTED MESSAGE
          </div>
          <div class="cf-group">
            <label class="cf-label">CALLSIGN (NAME)</label>
            <input class="cf-input" type="text" placeholder="Your name...">
          </div>
          <div class="cf-group">
            <label class="cf-label">SECURE CHANNEL (EMAIL)</label>
            <input class="cf-input" type="email" placeholder="your@email.com">
          </div>
          <div class="cf-group">
            <label class="cf-label">MISSION DETAILS</label>
            <textarea class="cf-textarea" placeholder="Describe your operation..."></textarea>
          </div>
          <button class="cf-submit" onclick="submitForm()">[ TRANSMIT ENCRYPTED ]</button>
        </div>
      </div>
    </div>
  </div>
</section>

<footer>
  <div style="font-family:'Orbitron',sans-serif;font-size:1rem;color:var(--green);letter-spacing:.3em;margin-bottom:.5rem;text-shadow:0 0 15px var(--green)">
    CIPHER // RED TEAM OPERATOR
  </div>
  <div>© 2024 — <span>ALL SYSTEMS OPERATIONAL</span> — ENCRYPTED WITH AES-256</div>
  <div style="margin-top:.5rem;font-size:.6rem;color:rgba(200,255,200,.15)">
    ██████████████████████████████████████ CLASSIFIED ████████████████████████████████████████
  </div>
</footer>

<script>
// ── CURSOR ──
const cursor = document.getElementById('cursor');
const cursorDot = document.getElementById('cursor-dot');
let mx=0,my=0,cx=0,cy=0;

document.addEventListener('mousemove',e=>{
  mx=e.clientX;my=e.clientY;
  cursorDot.style.left=mx+'px';cursorDot.style.top=my+'px';
});
setInterval(()=>{
  cx+=(mx-cx)*.15;cy+=(my-cy)*.15;
  cursor.style.left=cx+'px';cursor.style.top=cy+'px';
},16);
document.querySelectorAll('a,button,.skill-card,.mission-card,.writeup-item').forEach(el=>{
  el.addEventListener('mouseenter',()=>cursor.classList.add('active'));
  el.addEventListener('mouseleave',()=>cursor.classList.remove('active'));
});

// ── MATRIX ──
const mc=document.getElementById('matrix-canvas');
const mctx=mc.getContext('2d');
mc.width=window.innerWidth;mc.height=window.innerHeight;
const cols=Math.floor(mc.width/16);
const drops=Array(cols).fill(1);
const chars='アイウエオカキクケコサシスセソタチツテトナニヌネノ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ!@#$%^&*';
function drawMatrix(){
  mctx.fillStyle='rgba(0,0,0,.05)';
  mctx.fillRect(0,0,mc.width,mc.height);
  mctx.fillStyle='#00ff41';mctx.font='14px Share Tech Mono';
  drops.forEach((y,i)=>{
    mctx.fillText(chars[Math.floor(Math.random()*chars.length)],i*16,y*16);
    if(y*16>mc.height&&Math.random()>.975)drops[i]=0;
    drops[i]++;
  });
}
setInterval(drawMatrix,40);
window.addEventListener('resize',()=>{mc.width=window.innerWidth;mc.height=window.innerHeight});

// ── BOOT ──
const bootLines=[
  {t:'CIPHER/OS v7.4.1 — Secure Shell Interface',cls:''},
  {t:'Loading kernel modules...',cls:'ok'},
  {t:'Initializing encryption subsystems...',cls:'ok'},
  {t:'Establishing secure tunnel...',cls:'ok'},
  {t:'Checking identity verification...',cls:'ok'},
  {t:'Loading exploit database [6,841 entries]...',cls:'ok'},
  {t:'Mounting encrypted volumes...',cls:'ok'},
  {t:'Red Team framework: ACTIVE',cls:'ok'},
  {t:'C2 Infrastructure: ONLINE',cls:'ok'},
  {t:'Anomaly detected in sector 7...',cls:'warn'},
  {t:'Threat neutralized',cls:'ok'},
  {t:'All systems operational.',cls:'ok'},
  {t:'Identity: CIPHER // AUTHENTICATED',cls:'ok'},
];
const bootLog=document.getElementById('boot-log');
const bootFill=document.getElementById('boot-fill');
let bi=0;
function nextBootLine(){
  if(bi>=bootLines.length){
    document.getElementById('boot-enter').style.display='block';
    return;
  }
  const d=document.createElement('div');
  d.className='boot-line '+bootLines[bi].cls;
  d.textContent=bootLines[bi].t;
  d.style.animationDelay='0ms';
  bootLog.appendChild(d);
  bootFill.style.width=((bi+1)/bootLines.length*100)+'%';
  bi++;
  setTimeout(nextBootLine,180+Math.random()*120);
}
setTimeout(nextBootLine,500);

function enterSite(){
  document.getElementById('boot').classList.add('hidden');
  document.getElementById('main-nav').style.opacity='1';
  document.getElementById('main-nav').style.transition='opacity 1s';
  startHero();
}

// ── HERO TYPING ──
const lines=[
  {t:'> Initializing Operator...',delay:100},
  {t:'> Loading Red Team Framework...',delay:60},
  {t:'> Access Granted.',delay:80,color:'#00ff41'},
  {t:'> Red Team Specialist: LOADED',delay:60,color:'#00d4ff'},
];
let typeDiv=document.getElementById('type-text');
function startHero(){
  let li=0;
  function typeLine(i){
    if(i>=lines.length)return;
    const line=lines[i];
    const div=document.createElement('div');
    if(line.color)div.style.color=line.color;
    typeDiv.appendChild(div);
    let ci=0;
    const iv=setInterval(()=>{
      div.textContent=line.t.substring(0,ci+1);
      ci++;
      if(ci>=line.t.length){
        clearInterval(iv);
        setTimeout(()=>typeLine(i+1),400);
      }
    },line.delay);
  }
  setTimeout(()=>typeLine(0),300);
}

// ── THREAT MAP CANVAS ──
const tc=document.getElementById('threat-canvas');
const tctx=tc.getContext('2d');
tc.width=window.innerWidth;tc.height=window.innerHeight;
const attacks=[];
function spawnAttack(){
  attacks.push({
    x:Math.random()*tc.width,y:Math.random()*tc.height,
    tx:Math.random()*tc.width,ty:Math.random()*tc.height,
    progress:0,speed:.005+Math.random()*.01,
    color:['#ff003c','#00d4ff','#bf00ff','#ff6600'][Math.floor(Math.random()*4)]
  });
}
for(let i=0;i<8;i++)spawnAttack();
function drawThreats(){
  tctx.clearRect(0,0,tc.width,tc.height);
  attacks.forEach((a,idx)=>{
    const cx=a.x+(a.tx-a.x)*a.progress;
    const cy=a.y+(a.ty-a.y)*a.progress;
    // Line
    tctx.beginPath();tctx.moveTo(a.x,a.y);tctx.lineTo(cx,cy);
    tctx.strokeStyle=a.color+'40';tctx.lineWidth=1;tctx.stroke();
    // Dot
    tctx.beginPath();tctx.arc(cx,cy,2,0,Math.PI*2);
    tctx.fillStyle=a.color;tctx.fill();
    tctx.shadowBlur=6;tctx.shadowColor=a.color;tctx.fill();tctx.shadowBlur=0;
    a.progress+=a.speed;
    if(a.progress>=1){attacks.splice(idx,1);spawnAttack()}
  });
  requestAnimationFrame(drawThreats);
}
drawThreats();
window.addEventListener('resize',()=>{tc.width=window.innerWidth;tc.height=window.innerHeight});

// ── GLOBE CANVAS ──
(function(){
  const c=document.getElementById('globe-canvas');
  const ctx=c.getContext('2d');
  c.width=window.innerWidth;c.height=window.innerHeight;
  const cx2=c.width/2,cy2=c.height/2;
  const R=Math.min(c.width,c.height)*.25;
  let angle=0;
  const dots=[];
  for(let lat=-80;lat<=80;lat+=20){
    for(let lng=0;lng<360;lng+=20){
      dots.push({lat:lat*Math.PI/180,lng:lng*Math.PI/180});
    }
  }
  function draw(){
    ctx.clearRect(0,0,c.width,c.height);
    dots.forEach(d=>{
      const lng2=d.lng+angle;
      const x=R*Math.cos(d.lat)*Math.sin(lng2)+cx2;
      const y=R*Math.sin(d.lat)+cy2;
      const z=Math.cos(d.lat)*Math.cos(lng2);
      if(z>0){
        ctx.beginPath();ctx.arc(x,y,1.5,0,Math.PI*2);
        ctx.fillStyle=`rgba(0,212,255,${z*.5})`;
        ctx.shadowBlur=3;ctx.shadowColor='#00d4ff';
        ctx.fill();ctx.shadowBlur=0;
      }
    });
    // Equator ring
    ctx.beginPath();
    for(let i=0;i<=360;i+=2){
      const lng2=i*Math.PI/180+angle;
      const x=R*Math.cos(lng2)+cx2;
      const y=R*.3*Math.sin(lng2)+cy2;
      const z=Math.cos(lng2);
      if(i===0)ctx.moveTo(x,y);else if(z>0)ctx.lineTo(x,y);else ctx.moveTo(x,y);
    }
    ctx.strokeStyle='rgba(0,212,255,0.15)';ctx.lineWidth=1;ctx.stroke();
    angle+=.003;
    requestAnimationFrame(draw);
  }
  draw();
  window.addEventListener('resize',()=>{c.width=window.innerWidth;c.height=window.innerHeight});
})();

// ── SCROLL REVEAL ──
const reveals=document.querySelectorAll('.reveal');
const ro=new IntersectionObserver(entries=>{
  entries.forEach(e=>{
    if(e.isIntersecting){
      e.target.classList.add('visible');
      // Animate skill bars
      e.target.querySelectorAll('.id-bar-fill').forEach(b=>{
        b.style.width=b.dataset.width+'%';
      });
      // Animate counters
      if(e.target.contains(document.getElementById('ctr-bugs')))animCounters();
    }
  });
},{threshold:.1});
reveals.forEach(r=>ro.observe(r));

function animCounters(){
  const targets={bugs:[347,document.getElementById('ctr-bugs')],
    pentests:[89,document.getElementById('ctr-pentests')],
    certs:[12,document.getElementById('ctr-certs')],
    cvs:[7,document.getElementById('ctr-cves')]};
  Object.values(targets).forEach(([target,el])=>{
    let v=0;const step=target/60;
    const iv=setInterval(()=>{v=Math.min(v+step,target);el.textContent=Math.floor(v);if(v>=target)clearInterval(iv)},30);
  });
}

// ── TERMINAL COMMANDS ──
const termInput=document.getElementById('terminal-input');
const termOut=document.getElementById('terminal-output');
const cmdHistory=[];
let histIdx=-1;

const cmds={
  help:{fn:()=>[
    {t:'╔═══ AVAILABLE COMMANDS ══════════════════╗',c:'t-success'},
    {t:'  whoami          — Operator identity',c:'t-out'},
    {t:'  show skills     — Arsenal listing',c:'t-out'},
    {t:'  open projects   — Active operations',c:'t-out'},
    {t:'  cat achievements — Trophy room',c:'t-out'},
    {t:'  ls certs        — Credentials vault',c:'t-out'},
    {t:'  ping target     — Connectivity check',c:'t-out'},
    {t:'  nmap -sV        — Port scan demo',c:'t-out'},
    {t:'  exploit         — Run demo exploit',c:'t-out'},
    {t:'  matrix          — Enable matrix mode',c:'t-out'},
    {t:'  clear           — Clear terminal',c:'t-out'},
    {t:'  secret          — ???',c:'t-warn'},
    {t:'╚═══════════════════════════════════════╝',c:'t-success'},
  ]},
  whoami:{fn:()=>[
    {t:'cipher@redteam',c:'t-success'},
    {t:'Role: Senior Red Team Operator',c:'t-out'},
    {t:'Clearance: TOP SECRET',c:'t-warn'},
    {t:'Affiliation: [REDACTED]',c:'t-err'},
    {t:'Active engagements: 3',c:'t-out'},
    {t:'Bug bounty rank: #47 (Global)',c:'t-blue'},
  ]},
  'show skills':{fn:()=>[
    {t:'SKILLS ARSENAL:',c:'t-success'},
    {t:'[██████████] Web Pentesting    95%',c:'t-out'},
    {t:'[█████████░] Active Directory  90%',c:'t-blue'},
    {t:'[██████████] Red Teaming       98%',c:'t-err'},
    {t:'[█████████░] Bug Bounty        92%',c:'t-out'},
    {t:'[████████░░] Network Exploit.  85%',c:'t-blue'},
    {t:'[████████░░] Malware Analysis  82%',c:'t-warn'},
  ]},
  'open projects':{fn:()=>[
    {t:'ACTIVE OPERATIONS:',c:'t-success'},
    {t:'[ACTIVE]     Op ShadowRoot    — Cloud RCE chain',c:'t-err'},
    {t:'[COMPLETE]   Op GhostRecon   — AD domination',c:'t-out'},
    {t:'[CLASSIFIED] Op ZeroDay      — ████████████',c:'t-warn'},
    {t:'[OPEN SRC]   PhantomC2       — Custom C2 framework',c:'t-blue'},
    {t:'Navigate to #projects section for full details.',c:'t-info'},
  ]},
  'cat achievements':{fn:()=>[
    {t:'TROPHY ROOM:',c:'t-success'},
    {t:'🏆 DEF CON 31 — Research Presentation',c:'t-out'},
    {t:'🥇 HackerOne H1-702 CTF — 1st Place',c:'t-out'},
    {t:'💰 $200K+ Total Bug Bounty Earnings',c:'t-warn'},
    {t:'🎖️ 7 CVEs Assigned',c:'t-blue'},
    {t:'🛡️ Hall of Fame: Google, Apple, Microsoft',c:'t-success'},
    {t:'📜 12 Security Certifications',c:'t-out'},
  ]},
  'ls certs':{fn:()=>[
    {t:'CREDENTIALS VAULT:',c:'t-success'},
    {t:'OSCP   — Offensive Security Certified Professional',c:'t-out'},
    {t:'CRTO   — Certified Red Team Operator',c:'t-blue'},
    {t:'BSCP   — Burp Suite Certified Practitioner',c:'t-out'},
    {t:'CEH    — Certified Ethical Hacker Master',c:'t-warn'},
    {t:'AWS-S  — AWS Certified Security Specialty',c:'t-out'},
    {t:'CRTE   — Certified Red Team Expert',c:'t-blue'},
  ]},
  'ping target':{fn:()=>[
    {t:'PING 192.168.1.1 (target.local)',c:'t-out'},
    {t:'64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.4ms',c:'t-success'},
    {t:'64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.3ms',c:'t-success'},
    {t:'64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.5ms',c:'t-success'},
    {t:'--- target.local ping statistics ---',c:'t-info'},
    {t:'3 packets transmitted, 3 received, 0% packet loss',c:'t-success'},
  ]},
  'nmap -sv':{fn:()=>[
    {t:'Starting Nmap 7.95 — https://nmap.org',c:'t-out'},
    {t:'Scanning target (10.10.10.1) [65535 ports]',c:'t-warn'},
    {t:'PORT    STATE  SERVICE      VERSION',c:'t-info'},
    {t:'22/tcp  open   ssh          OpenSSH 8.9',c:'t-success'},
    {t:'80/tcp  open   http         Apache httpd 2.4.54',c:'t-success'},
    {t:'443/tcp open   ssl/http     nginx 1.22.1',c:'t-success'},
    {t:'3306/tcp open  mysql        MySQL 8.0.31',c:'t-err'},
    {t:'Nmap done: 1 IP address scanned in 2.34s',c:'t-out'},
  ]},
  exploit:{fn:()=>[
    {t:'[*] Loading exploit module: ms17_010_eternalblue',c:'t-warn'},
    {t:'[*] Target: 192.168.1.50:445',c:'t-out'},
    {t:'[*] Sending stage (200774 bytes) to target...',c:'t-warn'},
    {t:'[+] Meterpreter session 1 opened!',c:'t-success'},
    {t:'meterpreter > getuid',c:'t-out'},
    {t:'Server username: NT AUTHORITY\\SYSTEM',c:'t-err'},
    {t:'[!] DEMO MODE — No actual systems harmed.',c:'t-info'},
  ]},
  matrix:{fn:()=>[
    {t:'Enabling matrix overdrive...',c:'t-warn'},
    {t:'[DONE] The matrix is everywhere.',c:'t-success'},
  ],post:()=>{
    document.getElementById('matrix-canvas').style.opacity='.25';
    setTimeout(()=>document.getElementById('matrix-canvas').style.opacity='.08',5000);
  }},
  secret:{fn:()=>[
    {t:'You found the easter egg. 🥚',c:'t-success'},
    {t:'The quieter you become, the more you are able to hear.',c:'t-warn'},
    {t:'— Ram Dass / BackTrack motto',c:'t-out'},
    {t:'[HIDDEN] PGP: 0xDEADBEEF C1PH3R',c:'t-err'},
    {t:'[HIDDEN] Onion: cipher[REDACTED].onion',c:'t-err'},
  ]},
  clear:{fn:()=>{termOut.innerHTML='';return[]}},
};

function printLines(lines){
  lines.forEach((l,i)=>{
    setTimeout(()=>{
      const d=document.createElement('div');
      d.className=l.c||'t-out';d.textContent=l.t;
      termOut.appendChild(d);
      termOut.scrollTop=termOut.scrollHeight;
    },i*30);
  });
}

termInput.addEventListener('keydown',e=>{
  if(e.key==='Enter'){
    const v=termInput.value.trim().toLowerCase();
    if(!v)return;
    cmdHistory.unshift(v);histIdx=-1;
    const prompt=document.createElement('div');
    prompt.className='t-prompt';
    prompt.textContent='cipher@redteam:~$ '+v;
    termOut.appendChild(prompt);
    const cmd=cmds[v];
    if(cmd){
      const lines=cmd.fn();
      if(lines)printLines(lines);
      if(cmd.post)setTimeout(cmd.post,lines?lines.length*30+100:100);
    }else{
      const d=document.createElement('div');
      d.className='t-err';
      d.textContent=`bash: ${v}: command not found. Type 'help' for commands.`;
      termOut.appendChild(d);
    }
    termInput.value='';
    setTimeout(()=>termOut.scrollTop=termOut.scrollHeight,500);
  }
  if(e.key==='ArrowUp'){
    histIdx=Math.min(histIdx+1,cmdHistory.length-1);
    termInput.value=cmdHistory[histIdx]||'';
  }
  if(e.key==='ArrowDown'){
    histIdx=Math.max(histIdx-1,-1);
    termInput.value=histIdx>=0?cmdHistory[histIdx]:'';
  }
});

// ── THREAT FEED ──
const attackTypes=['SQL Injection','XSS Attack','Brute Force','DDoS','Port Scan','MITM','Ransomware','Phishing'];
const flags=['🇺🇸','🇷🇺','🇨🇳','🇮🇷','🇰🇵','🇧🇷','🇮🇳','🇩🇪','🇫🇷','🇬🇧','🇺🇦','🇹🇷'];
const targets=['192.168.1.'+Math.floor(Math.random()*254),'10.0.0.'+Math.floor(Math.random()*254),'172.16.0.'+Math.floor(Math.random()*254)];
function genThreat(){
  const feed=document.getElementById('threat-feed');
  if(!feed)return;
  const item=document.createElement('div');
  item.className='threat-item';
  const now=new Date();
  const time=now.getHours().toString().padStart(2,'0')+':'+now.getMinutes().toString().padStart(2,'0')+':'+now.getSeconds().toString().padStart(2,'0');
  item.innerHTML=`
    <span class="threat-flag">${flags[Math.floor(Math.random()*flags.length)]}</span>
    <span class="threat-type">${attackTypes[Math.floor(Math.random()*attackTypes.length)]}</span>
    <span class="threat-target">→ ${['10.0.0','192.168.1','172.16.0'][Math.floor(Math.random()*3)]}.${Math.floor(Math.random()*254)}</span>
    <span class="threat-time">${time}</span>
  `;
  feed.insertBefore(item,feed.firstChild);
  if(feed.children.length>8)feed.removeChild(feed.lastChild);
}
setInterval(genThreat,1200);

// ── SOC METRICS LIVE ──
setInterval(()=>{
  const t=document.getElementById('metric-threats');
  const b=document.getElementById('metric-blocked');
  const p=document.getElementById('metric-packets');
  if(t)t.textContent=Math.floor(200+Math.random()*100);
  if(b){const v=parseInt(b.textContent.replace(',',''));b.textContent=(v+Math.floor(Math.random()*10)).toLocaleString();}
  if(p)p.textContent=Math.floor(8000+Math.random()*4000).toLocaleString();
},1500);

// ── PACKET FLOW ──
const pf=document.getElementById('packet-flow');
if(pf){
  const colors=['','blue','red'];
  for(let i=0;i<8;i++){
    const row=document.createElement('div');row.className='packet-row';
    const p2=document.createElement('div');
    p2.className='packet '+colors[Math.floor(Math.random()*3)];
    p2.style.animationDuration=(1+Math.random()*2)+'s';
    p2.style.animationDelay=(-Math.random()*2)+'s';
    row.appendChild(p2);pf.appendChild(row);
  }
}

// ── RADAR ──
(function(){
  const c=document.getElementById('radar-canvas');
  if(!c)return;
  const ctx=c.getContext('2d');
  const cx2=140,cy2=140,R=120;
  let sweep=0;
  const blips=[];
  for(let i=0;i<6;i++){
    const a=Math.random()*Math.PI*2,r=30+Math.random()*80;
    blips.push({x:cx2+Math.cos(a)*r,y:cy2+Math.sin(a)*r,age:0,maxAge:200+Math.random()*200});
  }
  function drawRadar(){
    ctx.clearRect(0,0,280,280);
    // Rings
    [1,.7,.4,.2].forEach(s=>{
      ctx.beginPath();ctx.arc(cx2,cy2,R*s,0,Math.PI*2);
      ctx.strokeStyle=`rgba(0,255,65,${s*.15})`;ctx.lineWidth=1;ctx.stroke();
    });
    // Cross
    ctx.strokeStyle='rgba(0,255,65,.1)';ctx.lineWidth=1;
    ctx.beginPath();ctx.moveTo(cx2,cy2-R);ctx.lineTo(cx2,cy2+R);ctx.stroke();
    ctx.beginPath();ctx.moveTo(cx2-R,cy2);ctx.lineTo(cx2+R,cy2);ctx.stroke();
    // Sweep
    const grad=ctx.createConicalGradient?null:null;
    ctx.save();ctx.beginPath();
    ctx.moveTo(cx2,cy2);
    ctx.arc(cx2,cy2,R,sweep-1,sweep);
    ctx.closePath();
    ctx.fillStyle='rgba(0,255,65,.15)';ctx.fill();
    ctx.restore();
    // Sweep line
    ctx.beginPath();ctx.moveTo(cx2,cy2);
    ctx.lineTo(cx2+Math.cos(sweep)*R,cy2+Math.sin(sweep)*R);
    ctx.strokeStyle='rgba(0,255,65,.6)';ctx.lineWidth=2;ctx.stroke();
    // Blips
    blips.forEach(b=>{
      const alpha=1-b.age/b.maxAge;
      ctx.beginPath();ctx.arc(b.x,b.y,3,0,Math.PI*2);
      ctx.fillStyle=`rgba(255,0,60,${alpha})`;
      ctx.shadowBlur=8;ctx.shadowColor='#ff003c';
      ctx.fill();ctx.shadowBlur=0;
      b.age++;
      if(b.age>=b.maxAge){
        const a=Math.random()*Math.PI*2,r=20+Math.random()*90;
        b.x=cx2+Math.cos(a)*r;b.y=cy2+Math.sin(a)*r;b.age=0;
      }
    });
    sweep+=.03;
    requestAnimationFrame(drawRadar);
  }
  drawRadar();
})();

// ── PARALLAX ──
document.addEventListener('mousemove',e=>{
  const xp=(e.clientX/window.innerWidth-.5)*.03;
  const yp=(e.clientY/window.innerHeight-.5)*.03;
  document.querySelectorAll('.mission-card,.skill-card').forEach(c=>{
    c.style.transform=`perspective(1000px) rotateY(${xp*5}deg) rotateX(${-yp*5}deg) translateY(-4px)`;
  });
});

// ── FORM ──
function submitForm(){
  const btn=document.querySelector('.cf-submit');
  btn.textContent='[ TRANSMITTING... ]';
  setTimeout(()=>{btn.textContent='[ ✓ MESSAGE ENCRYPTED & SENT ]';btn.style.borderColor='var(--green)';btn.style.color='var(--green)';},1500);
}

// ── GLITCH EFFECT ──
setInterval(()=>{
  document.querySelectorAll('.sec-title').forEach(el=>{
    if(Math.random()>.97){
      el.style.textShadow='2px 0 var(--red),-2px 0 var(--blue)';
      setTimeout(()=>el.style.textShadow='',100);
    }
  });
},200);

console.log('%cCIPHER // RED TEAM OPERATOR','color:#00ff41;font-size:20px;font-family:monospace');
console.log('%c[!] You found the developer console. Scanning your system...','color:#ff003c;font-family:monospace');
console.log('%c    Just kidding. But nice try. 👀','color:#00d4ff;font-family:monospace');
</script>
</body>
</html>
