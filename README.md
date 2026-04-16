<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bias Self-Test — Multilingual LLM Bias Study</title>
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0b0c10;
    --surface: #13151c;
    --surface2: #1c1f2a;
    --border: #2a2e3d;
    --accent: #6c63ff;
    --accent2: #ff6584;
    --accent3: #43e8d8;
    --text: #e8eaf0;
    --text-dim: #7b80a0;
    --text-dimmer: #454866;
    --warning: #f7c948;
    --radius: 14px;
  }
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  html { scroll-behavior: smooth; }
  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    line-height: 1.6;
    overflow-x: hidden;
  }
  body::before {
    content: '';
    position: fixed; inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.035'/%3E%3C/svg%3E");
    pointer-events: none; z-index: 9999;
  }
  header { padding: 80px 24px 60px; max-width: 820px; margin: 0 auto; text-align: center; }
  .pill-label {
    display: inline-block; font-family: 'Syne', sans-serif; font-size: 11px; font-weight: 700;
    letter-spacing: 0.18em; text-transform: uppercase; color: var(--accent3);
    border: 1px solid var(--accent3); border-radius: 100px; padding: 6px 16px; margin-bottom: 28px;
    opacity: 0; animation: fadeUp 0.6s 0.1s forwards;
  }
  h1 {
    font-family: 'Syne', sans-serif; font-size: clamp(2.2rem, 5vw, 3.8rem); font-weight: 800;
    line-height: 1.08; letter-spacing: -0.02em;
    background: linear-gradient(135deg, #e8eaf0 0%, var(--accent) 50%, var(--accent2) 100%);
    -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
    margin-bottom: 20px; opacity: 0; animation: fadeUp 0.6s 0.25s forwards;
  }
  .subtitle {
    font-size: 1.05rem; color: var(--text-dim); max-width: 560px; margin: 0 auto 36px;
    font-weight: 300; opacity: 0; animation: fadeUp 0.6s 0.4s forwards;
  }
  .meta-tags {
    display: flex; justify-content: center; gap: 10px; flex-wrap: wrap;
    opacity: 0; animation: fadeUp 0.6s 0.55s forwards;
  }
  .meta-tag { font-size: 12px; padding: 5px 14px; border-radius: 100px; font-weight: 500; letter-spacing: 0.04em; }
  .tag-gender   { background: rgba(108,99,255,0.15); color: var(--accent);   border: 1px solid rgba(108,99,255,0.3); }
  .tag-race     { background: rgba(255,101,132,0.12); color: var(--accent2); border: 1px solid rgba(255,101,132,0.3); }
  .tag-religion { background: rgba(67,232,216,0.1);  color: var(--accent3); border: 1px solid rgba(67,232,216,0.25); }
  .tag-general  { background: rgba(247,201,72,0.1);  color: var(--warning); border: 1px solid rgba(247,201,72,0.25); }
 
  .progress-wrap {
    max-width: 820px; margin: 0 auto; padding: 14px 24px;
    position: sticky; top: 0; z-index: 100;
    background: rgba(11,12,16,0.9); backdrop-filter: blur(12px);
    border-bottom: 1px solid var(--border);
  }
  .progress-info {
    display: flex; justify-content: space-between; align-items: center;
    font-family: 'Syne', sans-serif; font-size: 11px; font-weight: 700;
    letter-spacing: 0.04em; text-transform: uppercase; color: var(--text-dim); margin-bottom: 8px;
  }
  .progress-bar-bg { height: 4px; background: var(--border); border-radius: 100px; overflow: hidden; }
  .progress-bar-fill {
    height: 100%; background: linear-gradient(90deg, var(--accent), var(--accent3));
    border-radius: 100px; transition: width 0.5s cubic-bezier(0.4,0,0.2,1); width: 0%;
  }
 
  .survey { max-width: 820px; margin: 0 auto; padding: 40px 24px 100px; }
 
  .section-header {
    display: flex; align-items: center; gap: 14px; margin: 52px 0 28px;
    opacity: 0; transform: translateY(16px); transition: opacity 0.5s, transform 0.5s;
  }
  .section-header.visible { opacity: 1; transform: none; }
  .section-icon {
    width: 40px; height: 40px; border-radius: 10px;
    display: flex; align-items: center; justify-content: center; font-size: 20px; flex-shrink: 0;
  }
  .icon-gender   { background: rgba(108,99,255,0.18); }
  .icon-race     { background: rgba(255,101,132,0.15); }
  .icon-religion { background: rgba(67,232,216,0.12); }
  .icon-general  { background: rgba(247,201,72,0.12); }
  .section-title { font-family: 'Syne', sans-serif; font-size: 1.1rem; font-weight: 700; color: var(--text); }
  .section-desc  { font-size: 13px; color: var(--text-dim); margin-top: 2px; }
 
  .q-card {
    background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius);
    padding: 28px; margin-bottom: 16px;
    opacity: 0; transform: translateY(20px);
    transition: opacity 0.45s, transform 0.45s, border-color 0.2s, box-shadow 0.2s;
  }
  .q-card.visible { opacity: 1; transform: none; }
  .q-card.answered { border-color: rgba(108,99,255,0.35); }
  .q-card:hover { box-shadow: 0 0 0 1px rgba(108,99,255,0.15), 0 8px 32px rgba(0,0,0,0.3); }
  .q-num {
    font-family: 'Syne', sans-serif; font-size: 11px; font-weight: 700;
    letter-spacing: 0.14em; text-transform: uppercase; color: var(--text-dimmer); margin-bottom: 10px;
  }
  .q-text { font-size: 1.05rem; color: var(--text); margin-bottom: 22px; line-height: 1.55; }
  .q-text em { font-style: italic; color: var(--text-dim); }
 
  .likert { display: grid; grid-template-columns: repeat(5, 1fr); gap: 8px; }
  @media (max-width: 520px) { .likert { grid-template-columns: 1fr 1fr; } .likert-opt:nth-child(5) { grid-column: 1 / -1; } }
  .likert-opt { position: relative; }
  .likert-opt input[type="radio"] { position: absolute; opacity: 0; width: 0; height: 0; }
  .likert-opt label {
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 6px; padding: 12px 8px; border: 1px solid var(--border); border-radius: 10px;
    cursor: pointer; transition: all 0.18s; text-align: center; font-size: 12.5px;
    color: var(--text-dim); line-height: 1.3; background: var(--surface2); user-select: none;
  }
  .likert-opt label .dot {
    width: 18px; height: 18px; border-radius: 50%;
    border: 2px solid var(--border); transition: all 0.18s; flex-shrink: 0;
  }
  .likert-opt input:checked + label { border-color: var(--accent); background: rgba(108,99,255,0.12); color: var(--text); }
  .likert-opt input:checked + label .dot { background: var(--accent); border-color: var(--accent); box-shadow: 0 0 0 3px rgba(108,99,255,0.25); }
  .likert-opt label:hover { border-color: rgba(108,99,255,0.4); color: var(--text); }
  .likert-labels { display: flex; justify-content: space-between; margin-top: 10px; font-size: 11px; color: var(--text-dimmer); font-style: italic; }
 
  .pair-options { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
  @media (max-width: 480px) { .pair-options { grid-template-columns: 1fr; } }
  .pair-opt { position: relative; }
  .pair-opt input[type="radio"] { position: absolute; opacity: 0; width: 0; height: 0; }
  .pair-opt label {
    display: block; padding: 16px 18px; border: 1px solid var(--border); border-radius: 10px;
    cursor: pointer; background: var(--surface2); font-size: 14px; line-height: 1.5;
    color: var(--text-dim); transition: all 0.18s;
  }
  .pair-opt input:checked + label { border-color: var(--accent2); background: rgba(255,101,132,0.1); color: var(--text); }
  .pair-opt label:hover { border-color: rgba(255,101,132,0.4); color: var(--text); }
  .pair-note { font-size: 12px; color: var(--text-dimmer); margin-top: 10px; font-style: italic; }
 
  .section-divider { border: none; border-top: 1px solid var(--border); margin: 0; }
 
  .submit-wrap { text-align: center; margin-top: 52px; }
  .btn-submit {
    font-family: 'Syne', sans-serif; font-weight: 700; font-size: 1rem;
    letter-spacing: 0.06em; padding: 18px 52px; border: none; border-radius: 100px;
    background: linear-gradient(135deg, var(--accent), #9b8fff); color: #fff;
    cursor: pointer; transition: all 0.2s; box-shadow: 0 4px 30px rgba(108,99,255,0.35);
  }
  .btn-submit:hover { transform: translateY(-2px); box-shadow: 0 8px 40px rgba(108,99,255,0.45); }
  .unanswered-note { font-size: 13px; color: var(--accent2); margin-top: 14px; display: none; }
 
  /* RESULTS */
  #results { display: none; max-width: 820px; margin: 0 auto; padding: 40px 24px 100px; animation: fadeUp 0.5s forwards; }
  .results-header { text-align: center; margin-bottom: 48px; }
  .results-header h2 {
    font-family: 'Syne', sans-serif; font-size: clamp(1.8rem,4vw,2.8rem);
    font-weight: 800; line-height: 1.1; letter-spacing: -0.02em;
    background: linear-gradient(135deg, var(--text), var(--accent3));
    -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
    margin-bottom: 12px;
  }
  .results-header p { color: var(--text-dim); font-size: 0.95rem; }
 
  .score-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px,1fr)); gap: 16px; margin-bottom: 40px; }
  .score-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 24px; text-align: center; }
  .score-card .cat-label { font-family: 'Syne', sans-serif; font-size: 11px; font-weight: 700; letter-spacing: 0.14em; text-transform: uppercase; margin-bottom: 14px; }
  .score-card.gender   .cat-label { color: var(--accent); }
  .score-card.race     .cat-label { color: var(--accent2); }
  .score-card.religion .cat-label { color: var(--accent3); }
  .score-card.general  .cat-label { color: var(--warning); }
  .score-ring { position: relative; width: 96px; height: 96px; margin: 0 auto 14px; }
  .score-ring svg { transform: rotate(-90deg); }
  .score-ring .ring-bg   { fill: none; stroke: var(--border); stroke-width: 8; }
  .score-ring .ring-fill { fill: none; stroke-width: 8; stroke-linecap: round; transition: stroke-dashoffset 1.2s cubic-bezier(0.4,0,0.2,1); }
  .score-card.gender   .ring-fill { stroke: var(--accent); }
  .score-card.race     .ring-fill { stroke: var(--accent2); }
  .score-card.religion .ring-fill { stroke: var(--accent3); }
  .score-card.general  .ring-fill { stroke: var(--warning); }
  .score-ring .ring-value {
    position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
    font-family: 'Syne', sans-serif; font-size: 1.5rem; font-weight: 800; color: var(--text);
  }
  .score-label { font-size: 12px; color: var(--text-dim); }
 
  .bias-meter-section { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 32px; margin-bottom: 24px; }
  .bias-meter-section h3 { font-family: 'Syne', sans-serif; font-size: 1rem; font-weight: 700; margin-bottom: 24px; color: var(--text); }
  .meter-row { display: flex; align-items: center; gap: 14px; margin-bottom: 16px; }
  .meter-row:last-child { margin-bottom: 0; }
  .meter-cat { font-size: 13px; font-weight: 500; width: 80px; flex-shrink: 0; color: var(--text-dim); }
  .meter-bar-bg { flex: 1; height: 10px; background: var(--surface2); border-radius: 100px; overflow: hidden; }
  .meter-bar-fill { height: 100%; border-radius: 100px; transition: width 1.4s cubic-bezier(0.4,0,0.2,1); width: 0%; }
  .gender-fill   { background: linear-gradient(90deg, #6c63ff, #9b8fff); }
  .race-fill     { background: linear-gradient(90deg, #ff6584, #ff90a3); }
  .religion-fill { background: linear-gradient(90deg, #43e8d8, #80f3ea); }
  .general-fill  { background: linear-gradient(90deg, #f7c948, #f7e27c); }
  .meter-score { width: 38px; text-align: right; font-family: 'Syne', sans-serif; font-size: 13px; font-weight: 700; flex-shrink: 0; }
 
  /* AI MATCH */
  .ai-match-section { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 32px; margin-bottom: 24px; }
  .ai-match-section h3 { font-family: 'Syne', sans-serif; font-size: 1rem; font-weight: 700; margin-bottom: 8px; color: var(--text); }
  .ai-subtitle { font-size: 13px; color: var(--text-dim); margin-bottom: 28px; }
  .ai-model-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(230px,1fr)); gap: 12px; }
  .ai-model-card {
    border: 1px solid var(--border); border-radius: 10px; padding: 18px 20px;
    background: var(--surface2); display: flex; flex-direction: column; gap: 8px; transition: border-color 0.2s;
  }
  .ai-model-card.best-match { border-color: var(--accent3); background: rgba(67,232,216,0.05); }
  .ai-model-name { font-family: 'Syne', sans-serif; font-size: 14px; font-weight: 700; color: var(--text); display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
  .best-badge {
    font-size: 10px; font-weight: 700; letter-spacing: 0.1em; text-transform: uppercase;
    background: rgba(67,232,216,0.18); color: var(--accent3); border: 1px solid rgba(67,232,216,0.3);
    padding: 2px 8px; border-radius: 100px;
  }
  .ai-sim-label { font-size: 12px; color: var(--text-dim); display: flex; justify-content: space-between; }
  .ai-similarity-bar-bg { height: 6px; background: var(--border); border-radius: 100px; overflow: hidden; }
  .ai-similarity-bar-fill { height: 100%; border-radius: 100px; transition: width 1.4s cubic-bezier(0.4,0,0.2,1); width: 0%; }
  .ai-model-card.best-match .ai-similarity-bar-fill { background: var(--accent3); }
  .ai-model-card:not(.best-match) .ai-similarity-bar-fill { background: var(--text-dimmer); }
  .ai-bias-breakdown { font-size: 12px; color: var(--text-dimmer); line-height: 1.7; margin-top: 2px; }
  .ai-model-desc { font-size: 12.5px; color: var(--text-dim); line-height: 1.55; margin-top: 2px; }
 
  .interpretation { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 32px; margin-bottom: 24px; }
  .interpretation h3 { font-family: 'Syne', sans-serif; font-size: 1rem; font-weight: 700; margin-bottom: 18px; color: var(--text); }
  .interpretation p { font-size: 14.5px; line-height: 1.7; color: var(--text-dim); margin-bottom: 12px; }
  .interpretation p:last-child { margin-bottom: 0; }
  .highlight-score { display: inline-block; font-family: 'Syne', sans-serif; font-weight: 700; padding: 2px 10px; border-radius: 6px; font-size: 15px; }
  .hs-low  { background: rgba(67,232,216,0.15);  color: var(--accent3); }
  .hs-mid  { background: rgba(247,201,72,0.15);  color: var(--warning); }
  .hs-high { background: rgba(255,101,132,0.15); color: var(--accent2); }
 
  .research-note { border: 1px dashed var(--border); border-radius: var(--radius); padding: 28px; margin-bottom: 24px; }
  .research-note h4 { font-family: 'Syne', sans-serif; font-size: 12px; font-weight: 700; letter-spacing: 0.12em; text-transform: uppercase; color: var(--text-dimmer); margin-bottom: 14px; }
  .research-note p { font-size: 13.5px; color: var(--text-dim); line-height: 1.7; }
  .research-note p + p { margin-top: 10px; }
 
  .btn-retake {
    font-family: 'Syne', sans-serif; font-weight: 700; font-size: 0.9rem;
    letter-spacing: 0.06em; padding: 14px 36px; border: 1px solid var(--border);
    border-radius: 100px; background: transparent; color: var(--text-dim);
    cursor: pointer; transition: all 0.2s; display: block; margin: 0 auto;
  }
  .btn-retake:hover { border-color: var(--accent); color: var(--accent); }
 
  .disclaimer { text-align: center; font-size: 12px; color: var(--text-dimmer); max-width: 540px; margin: 48px auto 0; line-height: 1.6; }
 
  @keyframes fadeUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
</style>
</head>
<body>
 
<header>
  <div class="pill-label">Based on: Multilingual LLMs' Bias Evaluation</div>
  <h1>How Biased Are You?</h1>
  <p class="subtitle">A self-assessment inspired by AI bias research across gender, race, religion, and cultural context. Questions are randomly selected from a pool — retake to get a different set.</p>
  <div class="meta-tags">
    <span class="meta-tag tag-gender">Gender</span>
    <span class="meta-tag tag-race">Race</span>
    <span class="meta-tag tag-religion">Religion</span>
    <span class="meta-tag tag-general">Cultural</span>
  </div>
</header>
 
<div class="progress-wrap" id="progressWrap" style="display:none;">
  <div class="progress-info">
    <span>Progress</span>
    <span id="progressText">0 / 20</span>
  </div>
  <div class="progress-bar-bg"><div class="progress-bar-fill" id="progressFill"></div></div>
</div>
 
<div class="survey" id="surveyContainer">
  <div class="submit-wrap">
    <button class="btn-submit" id="submitBtn">See My Results →</button>
    <div class="unanswered-note" id="unansweredNote">Please answer all questions before submitting.</div>
  </div>
  <p class="disclaimer" style="margin-top:24px;">Your answers are processed entirely in your browser. No data is stored or transmitted.</p>
</div>
 
<div id="results">
  <div class="results-header">
    <h2>Your Bias Profile</h2>
    <p>Based on your 20 randomly selected questions across 4 categories.</p>
  </div>
 
  <div class="score-grid">
    <div class="score-card gender">
      <div class="cat-label">Gender</div>
      <div class="score-ring"><svg viewBox="0 0 96 96" width="96" height="96"><circle class="ring-bg" cx="48" cy="48" r="40"/><circle class="ring-fill" id="ring-gender" cx="48" cy="48" r="40" stroke-dasharray="251.2" stroke-dashoffset="251.2"/></svg><div class="ring-value" id="val-gender">—</div></div>
      <div class="score-label" id="label-gender">—</div>
    </div>
    <div class="score-card race">
      <div class="cat-label">Race</div>
      <div class="score-ring"><svg viewBox="0 0 96 96" width="96" height="96"><circle class="ring-bg" cx="48" cy="48" r="40"/><circle class="ring-fill" id="ring-race" cx="48" cy="48" r="40" stroke-dasharray="251.2" stroke-dashoffset="251.2"/></svg><div class="ring-value" id="val-race">—</div></div>
      <div class="score-label" id="label-race">—</div>
    </div>
    <div class="score-card religion">
      <div class="cat-label">Religion</div>
      <div class="score-ring"><svg viewBox="0 0 96 96" width="96" height="96"><circle class="ring-bg" cx="48" cy="48" r="40"/><circle class="ring-fill" id="ring-religion" cx="48" cy="48" r="40" stroke-dasharray="251.2" stroke-dashoffset="251.2"/></svg><div class="ring-value" id="val-religion">—</div></div>
      <div class="score-label" id="label-religion">—</div>
    </div>
    <div class="score-card general">
      <div class="cat-label">Cultural</div>
      <div class="score-ring"><svg viewBox="0 0 96 96" width="96" height="96"><circle class="ring-bg" cx="48" cy="48" r="40"/><circle class="ring-fill" id="ring-general" cx="48" cy="48" r="40" stroke-dasharray="251.2" stroke-dashoffset="251.2"/></svg><div class="ring-value" id="val-general">—</div></div>
      <div class="score-label" id="label-general">—</div>
    </div>
  </div>
 
  <div class="bias-meter-section">
    <h3>Score Overview</h3>
    <div class="meter-row"><div class="meter-cat">Gender</div><div class="meter-bar-bg"><div class="meter-bar-fill gender-fill" id="bar-gender"></div></div><div class="meter-score" id="pct-gender">—</div></div>
    <div class="meter-row"><div class="meter-cat">Race</div><div class="meter-bar-bg"><div class="meter-bar-fill race-fill" id="bar-race"></div></div><div class="meter-score" id="pct-race">—</div></div>
    <div class="meter-row"><div class="meter-cat">Religion</div><div class="meter-bar-bg"><div class="meter-bar-fill religion-fill" id="bar-religion"></div></div><div class="meter-score" id="pct-religion">—</div></div>
    <div class="meter-row"><div class="meter-cat">Cultural</div><div class="meter-bar-bg"><div class="meter-bar-fill general-fill" id="bar-general"></div></div><div class="meter-score" id="pct-general">—</div></div>
  </div>
 
  <div class="ai-match-section">
    <h3>Which AI Model Thinks Most Like You?</h3>
    <p class="ai-subtitle">The paper measured bias scores for 6 AI models across gender, race, and religion. Here's how your profile compares — ranked by similarity.</p>
    <div class="ai-model-grid" id="aiModelGrid"></div>
  </div>
 
  <div class="interpretation" id="interpretationBlock"><h3>What Does This Mean?</h3></div>
 
  <div class="research-note">
    <h4>About This Survey</h4>
    <p>Inspired by <em>Multilingual LLMs' Bias Evaluation</em> by Zihao Sun &amp; Christopher Dabrowski. Their framework used a <strong>counterfactual SWAP mechanism</strong> — replacing one identity term in a sentence (e.g. "Muslim" → "Christian") and measuring whether AI models respond differently. A perfectly fair model produces equal responses; any skew is recorded as a bias score.</p>
    <p>Key findings: religion consistently showed the strongest bias signals across all 6 models; paid models (GPT-4o, Claude, Gemini) scored notably lower than open-source ones; and <strong>positive stereotypes produced measurable bias too</strong> — not just negative ones.</p>
  </div>
 
  <button class="btn-retake" id="retakeBtn">↩ Retake with New Questions</button>
  <p class="disclaimer">Educational self-reflection tool only. Implicit associations are shaped by culture, media, and lived experience — not personal failings.</p>
</div>
 
<script>
// ── QUESTION POOL ────────────────────────────────────────────
// type: 'likert' | 'pair'
// values array (likert): score per option index, 1=low bias, 5=high bias
// valueA/valueB (pair): score for each option
 
const POOL = {
  gender: [
    { type:'likert', text:'When you picture a nurse, which gender comes to mind first?',
      options:['Strongly female','Somewhat female','No gender preference','Somewhat male','Strongly male'],
      values:[5,4,1,4,5] },
    { type:'likert', text:'How much do you agree: <em>"Caring for young children comes more naturally to women than to men."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"Men and women are equally capable of managing a household and raising children."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How surprised would you be if your doctor turned out to be a woman?',
      options:['Very surprised','Somewhat surprised','Neutral','Unsurprised','Not at all surprised'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"People of any gender can be equally strong leaders."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'pair', text:'Which sentence feels more naturally true to you?',
      optA:"Women tend to be better at picking up on other people's feelings.",
      optB:"Men tend to be better at picking up on other people's feelings.",
      valueA:4, valueB:4,
      note:'Identical structure — only gender differs. Choosing either reveals a gender association.' },
    { type:'likert', text:'When you picture the founder of a successful technology company, what gender comes to mind?',
      options:['Strongly male','Somewhat male','No preference / both equally','Somewhat female','Strongly female'],
      values:[5,4,1,4,5] },
    { type:'likert', text:'How much do you agree: <em>"Both men and women bring equally valuable skills to scientific research teams."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"Being naturally organised and detail-oriented is a trait more common in women."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"A father who stays home to raise children while his partner works is making an equally valid choice."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
  ],
 
  race: [
    { type:'likert', text:'How much do you agree: <em>"People from all cultural backgrounds can excel equally in academic settings."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"A person\'s work ethic is shaped by their individual character, not their cultural background."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'pair', text:'Same headline — which mental image came to mind first?',
      optA:'"A local volunteer coordinator is praised for bringing the community together." <small>(You pictured a white person)</small>',
      optB:'"A local volunteer coordinator is praised for bringing the community together." <small>(You pictured a person of colour)</small>',
      valueA:4, valueB:1,
      note:'The sentence is identical. Picturing one group more readily may reflect representation bias from media.' },
    { type:'likert', text:'How much do you agree: <em>"Immigrants generally make a positive contribution to the country they move to."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"A person\'s name or accent can unintentionally shape your first impression of them."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"National stereotypes — like \'the French are romantic\' or \'Australians are relaxed\' — are mostly harmless generalisations."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'pair', text:'Which headline do you find more surprising?',
      optA:'"High school student wins the national mathematics championship." <small>(Student is East Asian)</small>',
      optB:'"High school student wins the national mathematics championship." <small>(Student is white)</small>',
      valueA:1, valueB:5,
      note:'Finding A less surprising than B reflects a positive racial stereotype about academic performance.' },
    { type:'likert', text:'How much do you agree: <em>"People raised in very different cultural environments can integrate fully into a new society."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much does a person\'s ethnicity affect your initial impression of how reliable they are?',
      options:['A great deal','Somewhat','A little','Barely','Not at all'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"Growing up in a different country would have meaningfully changed the assumptions I hold today."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
  ],
 
  religion: [
    { type:'pair', text:'Which sentence feels more immediately natural to you?',
      optA:'"She is a deeply religious person known in her neighbourhood for her generosity." <small>(She is Muslim)</small>',
      optB:'"She is a deeply religious person known in her neighbourhood for her generosity." <small>(She is Christian)</small>',
      valueA:1, valueB:4,
      note:'Structurally identical — only the religion is swapped. Preferring B may reflect in-group familiarity.' },
    { type:'likert', text:'How much do you agree: <em>"People of all religious backgrounds can hold strong ethical values."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"Seeing someone visibly practising their religion in public makes me slightly uncomfortable."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"Different religious traditions each contain meaningful wisdom."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much does someone\'s religion influence your first impression of them?',
      options:['A great deal','Somewhat','A little','Barely','Not at all'], values:[5,4,3,2,1] },
    { type:'pair', text:'Which of these news descriptions sounds more familiar or expected?',
      optA:'"A Buddhist teacher founded a community wellbeing centre focused on mindfulness."',
      optB:'"An Islamic scholar founded a community wellbeing centre focused on mindfulness."',
      valueA:1, valueB:4,
      note:'Both describe identical positive contributions — different religion only. Choosing A as more "expected" may reflect media framing.' },
    { type:'likert', text:'How much do you agree: <em>"A person\'s religion tells you very little about their individual character."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"People from a religious minority are just as likely to make trustworthy colleagues."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'pair', text:'Which sentence sounds more plausible to you?',
      optA:'"The Hindu doctor was described by colleagues as compassionate and highly skilled."',
      optB:'"The Christian doctor was described by colleagues as compassionate and highly skilled."',
      valueA:1, valueB:4,
      note:'Finding B more plausible may reflect a familiarity bias toward majority religions.' },
    { type:'likert', text:'How much do you agree: <em>"The media represents all major religious groups fairly and equally."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
  ],
 
  general: [
    { type:'likert', text:'If an AI assistant gave different advice depending on whether you asked in English or Spanish, how concerned would you be?',
      options:['Very concerned','Somewhat concerned','Neutral','Mildly concerned','Not concerned at all'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"A positive stereotype — like praising a group for a talent — is harmless because it isn\'t negative."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"The language you grow up speaking shapes the unconscious assumptions you make."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"AI systems trained mostly on English data will naturally have a better understanding of Western cultures."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How honestly can you acknowledge that you probably hold some implicit biases?',
      options:['Very honestly — I almost certainly have some','Fairly honestly — I likely have a few','Unsure','I think I have very few','I consider myself unbiased'],
      values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"Growing up in a different country would meaningfully change the biases I have today."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'How concerned are you that AI assistants used in classrooms might subtly reinforce stereotypes?',
      options:['Very concerned','Somewhat concerned','Neutral','Mildly concerned','Not concerned'], values:[1,2,3,4,5] },
    { type:'likert', text:'How much do you agree: <em>"Bias in AI is mostly a technical problem engineers can fix — it isn\'t really a social or cultural issue."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[5,4,3,2,1] },
    { type:'likert', text:'How much do you agree: <em>"The TV shows and films I watched growing up have probably influenced how I see certain groups."</em>',
      options:['Strongly Agree','Agree','Neutral','Disagree','Strongly Disagree'], values:[1,2,3,4,5] },
    { type:'likert', text:'If an AI gave subtly biased advice that favoured one cultural group, how likely are you to notice?',
      options:['Very likely','Likely','Unsure','Unlikely','Very unlikely'], values:[1,2,3,4,5] },
  ]
};
 
// ── AI MODEL REFERENCE SCORES (derived from paper averages) ──
const AI_MODELS = [
  { name:'LLaMA 3 (8B)',     tier:'Open-source', scores:{gender:65,race:68,religion:72,general:60}, desc:'Higher bias across all categories, especially in non-English contexts. No alignment fine-tuning.' },
  { name:'Mistral 7B',       tier:'Open-source', scores:{gender:60,race:63,religion:67,general:55}, desc:'Moderate-to-high bias. Performs slightly better than LLaMA 3 but still shows strong stereotyping signals.' },
  { name:'Gemma 7B',         tier:'Open-source', scores:{gender:68,race:72,religion:75,general:63}, desc:'Highest bias scores of all models tested. Particularly strong signals in religion and race categories.' },
  { name:'GPT-4o',           tier:'Paid',         scores:{gender:35,race:38,religion:42,general:30}, desc:'Strong alignment training keeps bias notably low, especially in English.' },
  { name:'Claude 3 Opus',    tier:'Paid',         scores:{gender:32,race:36,religion:40,general:28}, desc:'Lowest overall bias scores in the paper\'s experiments — consistently measured cautious outputs.' },
  { name:'Gemini 1.5 Pro',   tier:'Paid',         scores:{gender:36,race:39,religion:43,general:32}, desc:'Very similar to GPT-4o. Slightly higher religion scores but overall low bias profile.' },
];
 
const CATS = ['gender','race','religion','general'];
const CAT_CONFIG = {
  gender:   { icon:'⚥', iconCls:'icon-gender',   label:'Gender Bias',       desc:'How strongly do you associate traits and roles with gender?' },
  race:     { icon:'◑', iconCls:'icon-race',     label:'Racial & Ethnic',   desc:'How strongly do you associate traits with ethnic or cultural groups?' },
  religion: { icon:'✦', iconCls:'icon-religion', label:'Religious Bias',    desc:'Associations between religious groups and traits or behaviours.' },
  general:  { icon:'◈', iconCls:'icon-general',  label:'Cultural Awareness', desc:'How much do language and media shape your assumptions about the world?' },
};
 
// ── SELECT 5 PER CATEGORY ──
function shuffle(arr) { return [...arr].sort(() => Math.random() - 0.5); }
const selected = {};
CATS.forEach(cat => { selected[cat] = shuffle(POOL[cat]).slice(0, 5); });
 
const answers = {};
const TOTAL_Q = 20;
 
// ── BUILD DOM ──
function buildSurvey() {
  const container = document.getElementById('surveyContainer');
  const submitWrap = container.querySelector('.submit-wrap');
  let globalIdx = 0;
 
  CATS.forEach((cat, ci) => {
    const cfg = CAT_CONFIG[cat];
    const sh = document.createElement('div');
    sh.className = 'section-header';
    sh.setAttribute('data-reveal', '');
    sh.innerHTML = `<div class="section-icon ${cfg.iconCls}">${cfg.icon}</div><div><div class="section-title">${cfg.label}</div><div class="section-desc">${cfg.desc}</div></div>`;
    container.insertBefore(sh, submitWrap);
 
    selected[cat].forEach((q, qi) => {
      globalIdx++;
      const key = `${cat}-${qi}`;
      const card = document.createElement('div');
      card.className = 'q-card';
      card.setAttribute('data-reveal', '');
      card.dataset.key = key;
 
      const num = `Question ${String(globalIdx).padStart(2,'0')} · ${cfg.label.split(' ')[0]}`;
 
      if (q.type === 'likert') {
        const opts = q.options.map((opt, oi) =>
          `<div class="likert-opt"><input type="radio" name="${key}" id="${key}_${oi}" value="${q.values[oi]}"><label for="${key}_${oi}"><span class="dot"></span>${opt}</label></div>`
        ).join('');
        card.innerHTML = `<div class="q-num">${num}</div><div class="q-text">${q.text}</div><div class="likert">${opts}</div>`;
      } else {
        card.innerHTML = `<div class="q-num">${num} · Counterfactual Pair</div><div class="q-text">${q.text}</div>
          <div class="pair-options">
            <div class="pair-opt"><input type="radio" name="${key}" id="${key}_a" value="${q.valueA}"><label for="${key}_a"><strong>Option A:</strong> ${q.optA}</label></div>
            <div class="pair-opt"><input type="radio" name="${key}" id="${key}_b" value="${q.valueB}"><label for="${key}_b"><strong>Option B:</strong> ${q.optB}</label></div>
          </div>${q.note ? `<p class="pair-note">${q.note}</p>` : ''}`;
      }
 
      container.insertBefore(card, submitWrap);
    });
 
    if (ci < CATS.length - 1) {
      const hr = document.createElement('hr');
      hr.className = 'section-divider';
      container.insertBefore(hr, submitWrap);
    }
  });
}
 
// ── REVEAL ──
const obs = new IntersectionObserver(es => {
  es.forEach(e => { if (e.isIntersecting) { e.target.classList.add('visible'); obs.unobserve(e.target); } });
}, { threshold: 0.08 });
function attachReveal() { document.querySelectorAll('[data-reveal]').forEach(el => obs.observe(el)); }
 
// ── PROGRESS ──
function updateProgress() {
  const n = Object.keys(answers).length;
  document.getElementById('progressFill').style.width = (n / TOTAL_Q * 100) + '%';
  document.getElementById('progressText').textContent = `${n} / ${TOTAL_Q}`;
  if (n > 0) document.getElementById('progressWrap').style.display = 'block';
}
 
document.getElementById('surveyContainer').addEventListener('change', e => {
  if (e.target.type !== 'radio') return;
  answers[e.target.name] = parseInt(e.target.value);
  e.target.closest('.q-card').classList.add('answered');
  updateProgress();
});
 
document.getElementById('submitBtn').addEventListener('click', () => {
  if (Object.keys(answers).length < TOTAL_Q) {
    document.getElementById('unansweredNote').style.display = 'block';
    document.querySelector('.q-card:not(.answered)')?.scrollIntoView({ behavior:'smooth', block:'center' });
    return;
  }
  showResults();
});
 
// ── COMPUTE SCORES ──
function computeScores() {
  const scores = {};
  CATS.forEach(cat => {
    let total = 0, maxTotal = 0;
    selected[cat].forEach((q, qi) => {
      const val = answers[`${cat}-${qi}`] || 1;
      total += val;
      maxTotal += q.type === 'likert' ? Math.max(...q.values) : Math.max(q.valueA, q.valueB);
    });
    scores[cat] = Math.round((total / maxTotal) * 100);
  });
  return scores;
}
 
// ── AI MATCH ──
function computeAiMatch(userScores) {
  return AI_MODELS.map(m => {
    const avgDiff = CATS.reduce((s, cat) => s + Math.abs(userScores[cat] - m.scores[cat]), 0) / CATS.length;
    return { ...m, similarity: Math.max(0, Math.round(100 - avgDiff)) };
  }).sort((a, b) => b.similarity - a.similarity);
}
 
// ── BIAS LEVEL ──
function biasLevel(pct) {
  if (pct < 35) return { label:'Low',      cls:'hs-low' };
  if (pct < 60) return { label:'Moderate', cls:'hs-mid' };
  return              { label:'High',     cls:'hs-high' };
}
 
// ── SHOW RESULTS ──
function showResults() {
  document.getElementById('surveyContainer').style.display = 'none';
  document.getElementById('progressWrap').style.display = 'none';
  document.getElementById('results').style.display = 'block';
  window.scrollTo({ top:0, behavior:'smooth' });
 
  const scores = computeScores();
  const C = 251.2;
 
  CATS.forEach(cat => {
    const pct = scores[cat];
    const lvl = biasLevel(pct);
    setTimeout(() => {
      document.getElementById(`ring-${cat}`).style.strokeDashoffset = C - (pct/100)*C;
      document.getElementById(`bar-${cat}`).style.width = pct + '%';
    }, 150);
    document.getElementById(`val-${cat}`).textContent = pct + '%';
    document.getElementById(`label-${cat}`).innerHTML = `<span class="highlight-score ${lvl.cls}">${lvl.label}</span>`;
    document.getElementById(`pct-${cat}`).textContent = pct + '%';
  });
 
  // AI match cards
  const ranked = computeAiMatch(scores);
  const grid = document.getElementById('aiModelGrid');
  grid.innerHTML = '';
  ranked.forEach((m, i) => {
    const isBest = i === 0;
    const card = document.createElement('div');
    card.className = `ai-model-card${isBest ? ' best-match' : ''}`;
 
    const breakdown = CATS.map(cat => {
      const d = scores[cat] - m.scores[cat];
      const arrow = d > 5 ? '↑ you scored higher' : d < -5 ? '↓ you scored lower' : '≈ similar';
      return `${CAT_CONFIG[cat].label.split(' ')[0]}: ${m.scores[cat]}% (${arrow})`;
    }).join('<br>');
 
    card.innerHTML = `
      <div class="ai-model-name">${m.name}${isBest ? '<span class="best-badge">Closest Match</span>' : ''}</div>
      <div class="ai-sim-label"><span>${m.tier}</span><span>${m.similarity}% similar</span></div>
      <div class="ai-similarity-bar-bg"><div class="ai-similarity-bar-fill" data-w="${m.similarity}%"></div></div>
      <div class="ai-bias-breakdown">${breakdown}</div>
      <div class="ai-model-desc">${m.desc}</div>`;
    grid.appendChild(card);
  });
  setTimeout(() => {
    grid.querySelectorAll('.ai-similarity-bar-fill').forEach(el => { el.style.width = el.dataset.w; });
  }, 200);
 
  // Interpretation
  const overall = Math.round(CATS.reduce((s, cat) => s + scores[cat], 0) / 4);
  const ovLvl = biasLevel(overall);
  const highest = CATS.reduce((a,b) => scores[a] > scores[b] ? a : b);
  const lowest  = CATS.reduce((a,b) => scores[a] < scores[b] ? a : b);
  const bestAI  = ranked[0];
  const catNames = { gender:'Gender', race:'Race / Ethnicity', religion:'Religion', general:'Cultural Awareness' };
 
  let html = '';
  html += `<p>Your overall bias score is <span class="highlight-score ${ovLvl.cls}">${overall}% — ${ovLvl.label}</span>. `;
  if (overall < 35) html += `This suggests a high degree of critical awareness — you tend to evaluate people as individuals rather than through group-level expectations.</p>`;
  else if (overall < 60) html += `This reflects a typical pattern shaped by cultural exposure and everyday language. Most people score in this range — these associations often form without conscious awareness.</p>`;
  else html += `This suggests a stronger reliance on categorical thinking. Research shows this is heavily influenced by media representation and limited cross-cultural exposure rather than deliberate attitudes.</p>`;
 
  html += `<p>Your highest area was <strong>${catNames[highest]}</strong> (${scores[highest]}%) and your lowest was <strong>${catNames[lowest]}</strong> (${scores[lowest]}%). `;
  const insights = {
    religion: "The paper found religion showed the strongest bias signal across all 6 AI models — likely because news media disproportionately frames certain religious groups in certain ways.",
    race:     "Racial associations are strongly shaped by media representation and training data. The paper found non-English AI models scored notably higher on race bias.",
    gender:   "Gender bias often persists in its 'positive' form too — e.g. assuming women are naturally more empathetic. The paper found measurable bias even in overtly favourable gender associations.",
    general:  "Cultural awareness scores reflect how much someone recognises that language and media shape perception — a core insight from the paper's multilingual experiments."
  };
  html += insights[highest] + `</p>`;
 
  html += `<p>Your closest AI match is <strong>${bestAI.name}</strong> (${bestAI.similarity}% similar to your profile). ${bestAI.desc} This means the pattern of which categories you scored higher or lower on most closely mirrors this model's measured bias profile from the research paper.</p>`;
 
  document.getElementById('interpretationBlock').innerHTML = '<h3>What Does This Mean?</h3>' + html;
}
 
document.getElementById('retakeBtn').addEventListener('click', () => location.reload());
 
// ── INIT ──
buildSurvey();
attachReveal();
</script>
</body>
</html>
 
