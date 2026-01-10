<!-- GitHub Copilot Chat Assistant -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Key System — Generator & Verify</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    :root{
      --bg:#0d1117;
      --card:#161b22;
      --accent:#00e676;
      --accent-hover:#00b84a;
      --danger:#ff1744;
      --text:#ffffff;
      --muted:#cbd5e1;
    }
    html,body{height:100%;margin:0;background:var(--bg);color:var(--text);font-family:Poppins,system-ui,-apple-system,"Segoe UI",Roboto,Helvetica,Arial;}
    .wrap{min-height:100vh;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:24px;box-sizing:border-box;}
    .card{background:var(--card);padding:28px;border-radius:12px;box-shadow:0 6px 24px rgba(0,0,0,0.5);width:100%;max-width:520px;}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:18px;}
    h1{font-size:20px;margin:0;display:flex;align-items:center;gap:10px;color:var(--accent);}
    .tabs{display:flex;gap:8px;}
    .tab{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);padding:8px 12px;border-radius:8px;cursor:pointer;font-weight:600;}
    .tab.active{background:linear-gradient(90deg, rgba(0,230,118,0.10), rgba(0,184,74,0.06));color:var(--accent);border-color:transparent;box-shadow:inset 0 -1px 0 rgba(0,0,0,0.2);}
    .content{display:grid;gap:18px;}
    .key-box{background:transparent;padding:18px;border-radius:10px;margin:0;font-size:22px;letter-spacing:2px;border:1px dashed rgba(255,255,255,0.04);text-align:center;}
    .controls{display:flex;gap:10px;flex-wrap:wrap;}
    button{padding:10px 14px;border:none;border-radius:8px;cursor:pointer;background:var(--accent);color:#000;font-weight:700;}
    button.secondary{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);font-weight:600;}
    button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.04);color:var(--muted);}
    button:hover{background:var(--accent-hover);}
    input[type="text"]{padding:10px 14px;border-radius:8px;border:none;width:100%;box-sizing:border-box;background:transparent;color:var(--text);text-align:center;font-size:16px;border:1px solid rgba(255,255,255,0.04);}
    #result{font-size:18px;margin-top:4px}
    .muted{color:var(--muted);font-size:13px}
    footer{margin-top:12px;text-align:center;color:var(--muted);font-size:13px}
    @media (max-width:520px){
      .card{padding:18px;}
      .key-box{font-size:18px;padding:16px;}
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card" role="main">
      <header>
        <h1>🔑 Key System</h1>
        <div class="tabs" role="tablist" aria-label="Views">
          <button id="tab-gen" class="tab active" role="tab" aria-selected="true">Generate</button>
          <button id="tab-verify" class="tab" role="tab" aria-selected="false">Verify</button>
        </div>
      </header>

      <div class="content">
        <!-- Generator view -->
        <section id="view-gen" aria-hidden="false">
          <div class="muted" style="margin-bottom:8px">Click Generate to create a new key. Copy to clipboard.</div>
          <div class="key-box" id="keyDisplay">Click "Generate Key"</div>
          <div class="controls" style="margin-top:12px">
            <button id="genBtn">Generate Key</button>
            <button id="copyBtn" class="secondary">Copy Key</button>
            <button id="useVerify" class="ghost">Go to Verify</button>
          </div>
        </section>

        <!-- Verify view -->
        <section id="view-verify" style="display:none" aria-hidden="true">
          <div class="muted" style="margin-bottom:8px">Enter your key to check if it's valid.</div>
          <input type="text" id="keyInput" placeholder="e.g. 4869-1834-5638" inputmode="numeric" pattern="[0-9\-]*" />
          <div class="controls" style="margin-top:8px">
            <button id="verifyBtn">Verify</button>
            <button id="goGen" class="secondary">Go to Generator</button>
          </div>
          <div id="result" aria-live="polite"></div>
        </section>
      </div>

      <footer>
        Embedded keys: <span id="count"></span> • Local-only verification (no network)
      </footer>
    </div>
  </div>

  <script>
    // Embedded list of valid keys (was keys.json)
    const validKeys = [
      "4869-1834-5638",
      "7420-1629-8314",
      "1992-4817-5620"
    ];

    // State
    let currentKey = '';

    // Elements
    const keyDisplay = document.getElementById('keyDisplay');
    const genBtn = document.getElementById('genBtn');
    const copyBtn = document.getElementById('copyBtn');
    const verifyBtn = document.getElementById('verifyBtn');
    const keyInput = document.getElementById('keyInput');
    const result = document.getElementById('result');
    const countEl = document.getElementById('count');

    const tabGen = document.getElementById('tab-gen');
    const tabVerify = document.getElementById('tab-verify');
    const viewGen = document.getElementById('view-gen');
    const viewVerify = document.getElementById('view-verify');
    const useVerify = document.getElementById('useVerify');
    const goGen = document.getElementById('goGen');

    // Initialize
    countEl.textContent = validKeys.length + ' keys';

    // Helpers
    function setView(view){
      if(view === 'gen'){
        viewGen.style.display = '';
        viewVerify.style.display = 'none';
        viewGen.setAttribute('aria-hidden','false');
        viewVerify.setAttribute('aria-hidden','true');
        tabGen.classList.add('active');
        tabGen.setAttribute('aria-selected','true');
        tabVerify.classList.remove('active');
        tabVerify.setAttribute('aria-selected','false');
      } else {
        viewGen.style.display = 'none';
        viewVerify.style.display = '';
        viewGen.setAttribute('aria-hidden','true');
        viewVerify.setAttribute('aria-hidden','false');
        tabVerify.classList.add('active');
        tabVerify.setAttribute('aria-selected','true');
        tabGen.classList.remove('active');
        tabGen.setAttribute('aria-selected','false');
        keyInput.focus();
      }
      result.textContent = '';
      result.style.color = '';
    }

    // Key generation (3 parts of 4 digits)
    function generateKey(){
      const parts = [];
      for(let i=0;i<3;i++){
        parts.push(Math.floor(1000 + Math.random()*9000).toString());
      }
      currentKey = parts.join('-');
      keyDisplay.innerText = currentKey;
      // Pre-fill input in verify for convenience
      keyInput.value = currentKey;
    }

    async function copyKey(){
      if(!currentKey){
        alert('No key to copy. Generate one first.');
        return;
      }
      try{
        await navigator.clipboard.writeText(currentKey);
        alert('Copied: ' + currentKey);
      }catch(e){
        // fallback
        const ta = document.createElement('textarea');
        ta.value = currentKey;
        document.body.appendChild(ta);
        ta.select();
        try{ document.execCommand('copy'); alert('Copied: ' + currentKey); }catch(err){ prompt('Copy the key:', currentKey); }
        ta.remove();
      }
    }

    function verifyKey(){
      const userKey = (keyInput.value || '').trim();
      if(!userKey){
        result.textContent = '❗ Please enter a key to verify.';
        result.style.color = 'var(--muted)';
        return;
      }
      // Normalise: allow users to paste lowercase/spaces (though keys are numeric)
      const normalized = userKey.replace(/\s+/g,'');
      if(validKeys.includes(normalized)){
        result.innerHTML = '✅ Valid Key';
        result.style.color = 'var(--accent)';
      } else {
        result.innerHTML = '❌ Invalid Key';
        result.style.color = 'var(--danger)';
      }
    }

    // Events
    genBtn.addEventListener('click', generateKey);
    copyBtn.addEventListener('click', copyKey);
    verifyBtn.addEventListener('click', verifyKey);

    tabGen.addEventListener('click', ()=>setView('gen'));
    tabVerify.addEventListener('click', ()=>setView('verify'));
    useVerify.addEventListener('click', ()=>setView('verify'));
    goGen.addEventListener('click', ()=>setView('gen'));

    // Support Enter key in input
    keyInput.addEventListener('keydown', (e)=>{
      if(e.key === 'Enter'){ verifyKey(); }
    });

    // If URL hash indicates view, set initial
    if(location.hash === '#verify'){ setView('verify'); } else { setView('gen'); }

    // Preload: set currentKey to first valid key for demo convenience (optional)
    // currentKey = validKeys[0];
    // keyDisplay.innerText = currentKey;
  </script>
</body>
</html>
