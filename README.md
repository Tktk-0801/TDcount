# TDcount
<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>六つの電卓</title>
  <style>
    :root{
      --bg:#0f172a; --card:#0b1220; --accent:#2563eb; --glass: rgba(255,255,255,0.03);
      --gap:18px; --pad:14px; --radius:12px; --muted:rgba(255,255,255,0.65);
    }
    *{box-sizing:border-box}
    html,body{height:100%;}
    body{
      margin:0; font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Hiragino Kaku Gothic ProN", "Noto Sans JP", "メイリオ", sans-serif;
      background: linear-gradient(180deg,var(--bg), #071124 60%);
      color:#fff; padding:32px;
    }
    h1{font-size:20px;margin:0 0 16px 0;color:var(--muted)}

    .grid{
      display:grid;
      grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
      gap:var(--gap);
    }

    .card{
      background:linear-gradient(180deg,var(--card), rgba(255,255,255,0.02));
      border-radius:var(--radius); padding:var(--pad);
      box-shadow: 0 6px 18px rgba(2,6,23,0.6);
      min-height: 360px; display:flex; flex-direction:column; gap:12px; position:relative;
    }

    .title{font-size:13px; color:var(--muted); display:flex; justify-content:space-between; align-items:center}
    .display{
      background:var(--glass); border-radius:10px; padding:12px; min-height:64px; display:flex; flex-direction:column; justify-content:center; align-items:flex-end;
      font-weight:600; font-size:22px; color:#fff; letter-spacing:0.6px;
    }
    .sub{font-size:12px; color:rgba(255,255,255,0.5)}

    .pad{
      display:grid; grid-template-columns: repeat(4, 1fr); gap:8px; margin-top:8px;
    }
    button.key{
      padding:12px; border-radius:10px; border:0; background:rgba(255,255,255,0.03); color:#fff; font-size:16px; cursor:pointer; user-select:none;
      box-shadow: inset 0 -3px 0 rgba(0,0,0,0.2);
    }
    button.key.operator{background:linear-gradient(180deg, rgba(255,255,255,0.04), rgba(0,0,0,0.08));}
    button.key.equal{grid-column: span 2; background:linear-gradient(180deg,var(--accent), #1e40af); font-weight:700}
    button.key.clear{background:linear-gradient(180deg,#7f1d1d,#3b0a0a)}

    .hint{font-size:12px;color:var(--muted); margin-top:8px}

    footer{margin-top:20px; font-size:13px; color:rgba(255,255,255,0.6)}

    /* responsive tweaks */
    @media (max-width:420px){
      body{padding:12px}
      .card{min-height:320px}
    }
  </style>
</head>
<body>
  <h1>六つの電卓 — 同時に使えるシンプルな電卓が6個並んでいます</h1>

  <div class="grid" id="grid">
    <!-- 6 calculators inserted by JS for brevity -->
  </div>

  <footer>キーボードでも操作できます。数字と + - * / Enter (=) Backspace, Esc (C)。</footer>

  <script>
    // Calculator component factory
    class MiniCalc {
      constructor(container, id){
        this.container = container;
        this.id = id;
        this.value = '';
        this._build();
      }
      _build(){
        const card = document.createElement('section'); card.className='card';
        card.innerHTML = `
          <div class="title"><span>Calculator ${this.id}</span><span class="sub">独立</span></div>
          <div class="display" aria-live="polite"><div class="sub" style="opacity:.7; font-size:12px" data-expression></div><div data-result>0</div></div>
          <div class="pad" role="group">
            <button class="key clear" data-key="C">C</button>
            <button class="key" data-key="(">(</button>
            <button class="key" data-key=")">)</button>
            <button class="key operator" data-key="/">÷</button>

            <button class="key" data-key="7">7</button>
            <button class="key" data-key="8">8</button>
            <button class="key" data-key="9">9</button>
            <button class="key operator" data-key="*">×</button>

            <button class="key" data-key="4">4</button>
            <button class="key" data-key="5">5</button>
            <button class="key" data-key="6">6</button>
            <button class="key operator" data-key="-">−</button>

            <button class="key" data-key="1">1</button>
            <button class="key" data-key="2">2</button>
            <button class="key" data-key="3">3</button>
            <button class="key operator" data-key="+">+</button>

            <button class="key" data-key="0">0</button>
            <button class="key" data-key=".">.</button>
            <button class="key equal" data-key="=">=</button>
            <button class="key" data-key="%">%</button>
          </div>
          <div class="hint">例: (1+2)*3 のように式を入力できます</div>
        `;

        this.container.appendChild(card);
        this.card = card;
        this.exprEl = card.querySelector('[data-expression]');
        this.resultEl = card.querySelector('[data-result]');
        card.addEventListener('click', (e)=>{
          const btn = e.target.closest('button[data-key]');
          if(!btn) return;
          this._press(btn.dataset.key);
        });
      }
      _press(key){
        if(key === 'C'){
          this.value = '';
          this._render();
          return;
        }
        if(key === '='){
          this._calculate();
          return;
        }
        if(key === '%'){
          // percentage: convert current value to percentage
          try{
            const v = this.value === '' ? 0 : eval(this.value);
            this.value = String(v/100);
            this._render();
          }catch(e){ this._error(); }
          return;
        }
        this.value += key;
        this._render();
      }
      _render(){
        this.exprEl.textContent = this.value || '';
        try{
          if(this.value.trim() === ''){ this.resultEl.textContent = '0'; return; }
          // show a preview result without changing state
          const preview = eval(this._sanitize(this.value));
          this.resultEl.textContent = String(preview);
        }catch(e){ this.resultEl.textContent = '…'; }
      }
      _calculate(){
        try{
          const res = eval(this._sanitize(this.value || '0'));
          this.value = String(res);
          this._render();
        }catch(e){ this._error(); }
      }
      _error(){
        this.resultEl.textContent = 'Error';
        setTimeout(()=>{ this.value=''; this._render(); }, 900);
      }
      _sanitize(s){
        // Basic sanitation: allow only digits, operators and parentheses and dot
        // This is not a security sandbox for untrusted input but ok for local page.
        return s.replace(/[^0-9+\-*/().%]/g, '');
      }
    }

    // create 6 calculators
    const grid = document.getElementById('grid');
    for(let i=1;i<=6;i++){
      const wrapper = document.createElement('div'); wrapper.className='calc-wrap';
      grid.appendChild(wrapper);
      new MiniCalc(wrapper, i);
    }

    // Global keyboard routing: focus each calculator independently by clicking; if none focused, route to last clicked
    let lastCalc = null;
    document.addEventListener('click', (e)=>{
      const card = e.target.closest('.card');
      if(card) lastCalc = card;
    });

    document.addEventListener('keydown', (e)=>{
      if(!lastCalc) return; // ignore if user didn't pick one
      // find its MiniCalc instance by title text
      const label = lastCalc.querySelector('.title span')?.textContent || '';
      const id = label.replace(/[^0-9]/g,'') || '1';
      const idx = parseInt(id,10)-1;
      const calcWrap = grid.children[idx];
      if(!calcWrap) return;
      const mc = calcWrap.__miniCalc;
      // we didn't store instance on element; let's find via child elements (quick hack)
      // Instead, store instance reference when creating
    });

    // store references on wrappers for keyboard handling
    // re-create to attach references properly
    grid.innerHTML='';
    const calcs = [];
    for(let i=1;i<=6;i++){
      const wrapper = document.createElement('div'); wrapper.className='calc-wrap';
      grid.appendChild(wrapper);
      const inst = new MiniCalc(wrapper, i);
      wrapper.__miniCalc = inst;
      calcs.push(inst);
    }

    // keyboard: route keystrokes to the most recently clicked calculator (or first if none)
    let active = calcs[0];
    document.addEventListener('click', (e)=>{
      const w = e.target.closest('.calc-wrap');
      if(w && w.__miniCalc) active = w.__miniCalc;
    });

    document.addEventListener('keydown', (e)=>{
      // allow numbers, operators, Enter, Backspace, Esc
      const k = e.key;
      if(!active) return;
      if((/^[0-9.+\-*/()%]$/).test(k)){
        e.preventDefault(); active._press(k);
      }else if(k === 'Enter'){
        e.preventDefault(); active._press('=');
      }else if(k === 'Backspace'){
        e.preventDefault(); active.value = active.value.slice(0,-1); active._render();
      }else if(k === 'Escape'){
        e.preventDefault(); active._press('C');
      }
    });

    // small improvement: make sure display updates on load
    calcs.forEach(c=>c._render());

  </script>
</body>
</html>
