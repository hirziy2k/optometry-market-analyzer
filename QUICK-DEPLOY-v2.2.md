# Quick Deploy Guide: v2.2 Market Assessment

## 10-Minute Local Deployment

Since the file is 1,303 lines, the fastest way is to apply these changes locally using Find & Replace.

---

## Prerequisites

1. Download `index.html` from GitHub
2. Open in **any text editor** (VS Code, Notepad++, Sublime, or even plain Notepad)
3. Follow the 4 simple Find/Replace operations below

---

## Change #1: Update Version Number

**Find:**
```html
<title>Optometry Market Analyzer v2.1</title>
```

**Replace with:**
```html
<title>Optometry Market Analyzer v2.2</title>
```

---

## Change #2: Add Market Assessment CSS (insert after line 821)

**Find this exact line:**
```css
footer a { color: #3b82f6; text-decoration: none }
```

**Immediately AFTER that line, add:**
```css

/* Market Assessment v2.2 */
#market input[type="number"],
#market input[type="text"],
#market select {
  width: 100%;
  padding: 10px;
  margin-top: 0.25rem;
  margin-bottom: 1rem;
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 8px;
  color: #e2e8f0;
  font-size: 0.95rem;
}

#market label {
  display: block;
  font-weight: 600;
  color: #94a3b8;
  margin-top: 1rem;
  font-size: 0.9rem;
}

details summary {
  list-style: none;
  cursor: pointer;
}

details summary::-webkit-details-marker {
  display: none;
}
```

---

## Change #3: Replace Market Section Content

**Find this EXACT block** (starts around line 850):
```html
<!-- ========== MARKET OVERVIEW ========== -->
    <div id="market" class="tab-content active">
```

**Replace the ENTIRE `<div id="market">...</div>` section with:**

[See full replacement in IMPLEMENTATION-GUIDE-v2.2.md lines 15-90]

OR use this simplified version:

```html
<!-- ========== MARKET ASSESSMENT v2.2 ========== -->
    <div id="market" class="tab-content active">
      
      <!-- Interactive Classifier Form -->
      <div class="card" style="margin-bottom: 2rem;">
        <h2>📊 Market Assessment</h2>
        <p style="color: #94a3b8; margin-bottom: 1.5rem;">Input your market context to receive a structured viability assessment.</p>
        
        <label>Geography</label>
        <input type="text" id="market-geo" value="Malaysia">
        
        <label>SAM (RM / Year)</label>
        <input type="number" id="market-sam" value="2100000">
        
        <label>SOM (RM / Year)</label>
        <input type="number" id="market-som" value="350000">
        
        <div class="grid-2">
          <div>
            <label>Growth Rate (%)</label>
            <input type="number" id="market-growth" step="0.1" value="4.5">
          </div>
          <div>
            <label>Competition Density</label>
            <input type="number" id="market-comp" step="0.1" value="1.2">
          </div>
        </div>
        
        <label>Regulatory Friction (0-100)</label>
        <input type="number" id="market-reg" value="45">
        
        <div class="grid-2">
          <div>
            <label>Channel</label>
            <select id="market-channel">
              <option value="brick_mortar">Brick & Mortar</option>
              <option value="hybrid">Hybrid</option>
              <option value="online_first">Online First</option>
            </select>
          </div>
          <div>
            <label>Modality</label>
            <select id="market-modality">
              <option value="full_scope_clinic">Full Scope Clinic</option>
              <option value="optical_retail">Optical Retail</option>
              <option value="contact_lens">Contact Lenses</option>
              <option value="teleoptometry">Teleoptometry</option>
            </select>
          </div>
        </div>
        
        <button onclick="classifyMarket()" style="background: #F5C518; color: #262626; width: 100%; padding: 12px; margin-top: 1.5rem; font-weight: 700; border: none; border-radius: 8px; cursor: pointer;">Classify Market</button>
      </div>
      
      <!-- Result Cards -->
      <div id="market-result" class="card" style="display: none; margin-bottom: 2rem;"></div>
      <div id="market-mda-flag" class="card" style="display: none; background: #FFF8D6; border: 2px solid #F5C518; margin-bottom: 2rem;">
        <strong style="color: #856404;">⚠️ MDA COMPLIANCE ALERT</strong>
        <p style="color: #856404; font-size: 0.9rem; margin: 0.5rem 0 0 0;">Act 737 restrictions apply to online sales of optical devices and contact lenses in Malaysia.</p>
      </div>
      
      <!-- Existing Global Context (collapsible) -->
      <details>
        <summary style="padding: 1rem; background: #1e293b; border-radius: 8px; color: #60a5fa; font-weight: 600;">🌍 Global Market Context</summary>
        <div style="padding-top: 1rem;">
```

**Then keep ALL the existing Market Overview content (charts, condition prevalence, etc.) and close with:**

```html
        </div>
      </details>
    </div>
```

---

## Change #4: Add Classification JavaScript

**Find this line** (around line 1275):
```javascript
// Filter System with Smooth Transitions
```

**Immediately BEFORE that line, add:**

```javascript
// ============================================================
// MARKET CLASSIFICATION ENGINE v2.2
// ============================================================

const MARKET_THRESHOLDS = { SAM_min: 2000000, SOM_min: 300000 };

function classifyMarket() {
  const inputs = {
    SAM: parseFloat(document.getElementById('market-sam').value),
    SOM: parseFloat(document.getElementById('market-som').value),
    growth_rate: parseFloat(document.getElementById('market-growth').value),
    competition_density: parseFloat(document.getElementById('market-comp').value),
    regulatory_friction_score: parseFloat(document.getElementById('market-reg').value),
    channel: document.getElementById('market-channel').value,
    modality: document.getElementById('market-modality').value
  };
  
  const g_band = inputs.growth_rate < 0 ? 'shrink' : inputs.growth_rate < 2 ? 'flat' : inputs.growth_rate < 6 ? 'mod' : 'high';
  const c_band = inputs.competition_density < 0.8 ? 'low' : inputs.competition_density <= 1.5 ? 'med' : 'high';
  const r_band = inputs.regulatory_friction_score < 30 ? 'low' : inputs.regulatory_friction_score < 60 ? 'med' : inputs.regulatory_friction_score < 80 ? 'high' : 'v_high';
  
  let status = null, reasons = [];
  
  if (r_band === 'v_high') { status = 'fail'; reasons.push('regulatory_block'); }
  if (!status && (inputs.SAM < MARKET_THRESHOLDS.SAM_min || inputs.SOM < MARKET_THRESHOLDS.SOM_min)) { status = 'fail'; reasons.push('market_too_small'); }
  if (!status && g_band === 'shrink' && c_band === 'high') { status = 'fail'; reasons.push('shrinking_and_crowded'); }
  if (!status && (g_band === 'flat' || c_band === 'high' || r_band === 'high')) { status = 'soft_pass'; reasons.push('market_headwinds'); }
  if (!status) { status = 'pass'; reasons = ['structurally_attractive']; }
  
  const colors = {
    pass: { bg: '#E8F5E9', border: '#2E7D32', text: '#2E7D32' },
    soft_pass: { bg: '#FFF3E0', border: '#E65100', text: '#E65100' },
    fail: { bg: '#FFEBEE', border: '#B71C1C', text: '#B71C1C' }
  };
  
  const c = colors[status];
  const resultDiv = document.getElementById('market-result');
  resultDiv.style.display = 'block';
  resultDiv.style.background = c.bg;
  resultDiv.style.borderLeft = '4px solid ' + c.border;
  resultDiv.innerHTML = '<strong style="color: ' + c.text + ';">Status: ' + status.toUpperCase().replace('_', '-') + '</strong><p style="margin: 0.5rem 0 0 0; color: #64748b;">Reason: ' + reasons.join(', ').replace(/_/g, ' ') + '</p>';
  
  const mdaFlag = document.getElementById('market-mda-flag');
  if (inputs.channel === 'online_first' && ['optical_retail', 'contact_lens'].includes(inputs.modality)) {
    mdaFlag.style.display = 'block';
  } else {
    mdaFlag.style.display = 'none';
  }
  
  window.marketAssessment = { status, inputs, reasons };
}

```

---

## Done!

Save the file, open in browser to test, then commit to GitHub.

**Verification:**
- Market Overview tab should now show the interactive form
- Banana-yellow "Classify Market" button visible
- Classification works correctly
- MDA flag appears for online + optical/contact lens combinations
- Global charts are still accessible in collapsible section

---

**Need the complete modified file?** Check `/market-classifier.html` for working reference of all logic.
