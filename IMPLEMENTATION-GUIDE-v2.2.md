# Optometry Market Analyzer v2.2 — Implementation Guide

## Merging Market Classification Engine into Production

**Decision:** Replace current "Market Overview" tab with interactive Market Assessment

---

## HTML Structure — New Market Section

### Step 1: Insert at Line ~850 (inside `<div id="market" class="tab-content active">`)

Replace the existing Market Overview content with this structure:

```html
<!-- MARKET ASSESSMENT v2.2 - Interactive Classifier -->
<div id="market" class="tab-content active">
  
  <!-- NEW: Interactive Market Classifier Form -->
  <div class="card" style="margin-bottom: 2rem;">
    <h2>📊 Market Assessment</h2>
    <p style="color: #94a3b8; margin-bottom: 1.5rem;">Input your market context to receive a structured viability assessment with regulatory guidance.</p>
    
    <label>Geography</label>
    <input type="text" id="market-geo" placeholder="e.g. Malaysia, Klang Valley" value="Malaysia">
    
    <label>SAM — Serviceable Addressable Market (RM / Year)</label>
    <input type="number" id="market-sam" placeholder="e.g. 2100000" value="2100000">
    
    <label>SOM — Serviceable Obtainable Market (RM / Year)</label>
    <input type="number" id="market-som" placeholder="e.g. 350000" value="350000">
    
    <div class="grid-2">
      <div>
        <label>Growth Rate (%)</label>
        <input type="number" id="market-growth" step="0.1" value="4.5">
      </div>
      <div>
        <label>Competition Density (clinics/10k pop)</label>
        <input type="number" id="market-comp" step="0.1" value="1.2">
      </div>
    </div>
    
    <label>Regulatory Friction (0-100)</label>
    <input type="number" id="market-reg" min="0" max="100" value="45">
    
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
    
    <!-- HIGH-STAKES BUTTON: Banana Yellow -->
    <button onclick="classifyMarket()" style="background: #F5C518; color: #262626; width: 100%; padding: 12px; margin-top: 1.5rem; font-weight: 700; border: none; border-radius: 8px; cursor: pointer;">
      Classify Market
    </button>
  </div>
  
  <!-- Result Card (hidden until classification) -->
  <div id="market-result" class="card" style="display: none; margin-bottom: 2rem;"></div>
  
  <!-- MDA Compliance Flag (conditional) -->
  <div id="market-mda-flag" class="card" style="display: none; background: #FFF8D6; border: 2px solid #F5C518; margin-bottom: 2rem;">
    <strong style="color: #856404; display: block; margin-bottom: 0.5rem;">⚠️ MDA COMPLIANCE ALERT</strong>
    <p style="color: #856404; font-size: 0.9rem; margin: 0;">Act 737 restrictions apply to online sales of optical devices and contact lenses in Malaysia. Consult MDA guidelines before launching digital channels.</p>
  </div>
  
  <!-- EXISTING CONTENT: Global Context (now collapsible) -->
  <details style="margin-bottom: 2rem;">
    <summary style="cursor: pointer; font-weight: 600; color: #60a5fa; padding: 1rem; background: #1e293b; border-radius: 8px;">🌍 Global Market Context (Optional Reference)</summary>
    <div style="padding: 1rem 0;">
      
      <!-- KEEP ALL EXISTING MARKET OVERVIEW CONTENT HERE -->
      <!-- Copy lines 850-950 from current index.html: -->
      <!-- - Global Optometry Market card -->
      <!-- - Regional Distribution card -->
      <!-- - Condition Prevalence list -->
      <!-- - Data & Methods card -->
      
      <div class="card">
        <h2>📊 Global Optometry Market</h2>
        <!-- existing chart content -->
      </div>
      
      <div class="grid-2">
        <div class="card">
          <!-- existing charts -->
        </div>
      </div>
      
      <!-- etc. -->
      
    </div>
  </details>
  
</div>
```

---

## JavaScript — Classification Engine

### Step 2: Add to `<script>` section (before closing `</script>` around line 1280)

```javascript
// ============================================================
// MARKET CLASSIFICATION ENGINE v2.2
// ============================================================

const MARKET_THRESHOLDS = {
  SAM_min: 2000000,
  SOM_min: 300000
};

function classifyMarket() {
  // Get inputs
  const inputs = {
    geo: document.getElementById('market-geo').value,
    SAM: parseFloat(document.getElementById('market-sam').value),
    SOM: parseFloat(document.getElementById('market-som').value),
    growth_rate: parseFloat(document.getElementById('market-growth').value),
    competition_density: parseFloat(document.getElementById('market-comp').value),
    regulatory_friction_score: parseFloat(document.getElementById('market-reg').value),
    channel: document.getElementById('market-channel').value,
    modality: document.getElementById('market-modality').value
  };
  
  // Derive bands
  const g_band = inputs.growth_rate < 0 ? 'shrink' : 
                 inputs.growth_rate < 2 ? 'flat' :
                 inputs.growth_rate < 6 ? 'mod' : 'high';
  
  const c_band = inputs.competition_density < 0.8 ? 'low' :
                 inputs.competition_density <= 1.5 ? 'med' : 'high';
  
  const r_band = inputs.regulatory_friction_score < 30 ? 'low' :
                 inputs.regulatory_friction_score < 60 ? 'med' :
                 inputs.regulatory_friction_score < 80 ? 'high' : 'v_high';
  
  let status = null;
  let reasons = [];
  
  // Classification logic
  if (r_band === 'v_high') {
    status = 'fail';
    reasons.push('regulatory_block');
  }
  
  if (!status && (inputs.SAM < MARKET_THRESHOLDS.SAM_min || inputs.SOM < MARKET_THRESHOLDS.SOM_min)) {
    status = 'fail';
    reasons.push('market_too_small');
  }
  
  if (!status && g_band === 'shrink' && c_band === 'high') {
    status = 'fail';
    reasons.push('shrinking_and_crowded');
  }
  
  if (!status && (g_band === 'flat' || c_band === 'high' || r_band === 'high')) {
    status = 'soft_pass';
    reasons.push('market_headwinds');
  }
  
  if (!status) {
    status = 'pass';
    reasons = ['structurally_attractive'];
  }
  
  // Display result
  const resultDiv = document.getElementById('market-result');
  const statusColors = {
    pass: { bg: '#E8F5E9', border: '#2E7D32', text: '#2E7D32' },
    soft_pass: { bg: '#FFF3E0', border: '#E65100', text: '#E65100' },
    fail: { bg: '#FFEBEE', border: '#B71C1C', text: '#B71C1C' }
  };
  
  const color = statusColors[status];
  resultDiv.style.display = 'block';
  resultDiv.style.background = color.bg;
  resultDiv.style.borderLeft = `4px solid ${color.border}`;
  resultDiv.innerHTML = `
    <strong style="color: ${color.text}; font-size: 1.1rem;">Status: ${status.toUpperCase().replace('_', '-')}</strong>
    <p style="margin: 0.5rem 0 0 0; color: #64748b;">Reason: ${reasons.join(', ').replace(/_/g, ' ')}</p>
  `;
  
  // Handle MDA flag
  const mdaFlag = document.getElementById('market-mda-flag');
  if (inputs.channel === 'online_first' && ['optical_retail', 'contact_lens'].includes(inputs.modality)) {
    mdaFlag.style.display = 'block';
  } else {
    mdaFlag.style.display = 'none';
  }
  
  // Store result for downstream tabs
  window.marketAssessment = { status, inputs, reasons };
}
```

---

## CSS Updates

### Step 3: Add to `<style>` section (around line 420)

```css
/* Market Assessment v2.2 Enhancements */
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
}

details summary::-webkit-details-marker {
  display: none;
}

details[open] summary {
  margin-bottom: 1rem;
}
```

---

## Migration Checklist

- [ ] Back up current `index.html` as `index-v2.1-backup.html`
- [ ] Insert new Market Assessment form at top of `#market` div
- [ ] Wrap existing charts/data in `<details>` element
- [ ] Add `classifyMarket()` function to JavaScript section
- [ ] Add Market Assessment CSS rules
- [ ] Test locally before pushing
- [ ] Update "How to Use" guidance to mention interactive classifier
- [ ] Update version number to v2.2 in header

---

## Expected User Flow

1. User lands on Market Overview (now Market Assessment)
2. Sees form pre-filled with Malaysia defaults
3. Adjusts values or keeps defaults
4. Clicks yellow "Classify Market" button
5. Gets immediate PASS/SOFT-PASS/FAIL with reasons
6. MDA warning appears if online + optical/contact lenses
7. Can expand "Global Market Context" for reference data
8. Proceeds to Top Niches with clear market status

---

**Questions? Check `/market-classifier.html` for working prototype reference.**
