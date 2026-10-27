<!DOCTYPE html>
<html lang="id">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Pricing Shopee Analytic</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&family=Tektur:wght@600;800&family=Inter:wght@500;600;700;800&display=swap" rel="stylesheet" />
    <script src="https://cdn.tailwindcss.com/3.4.17"></script>
    <script>
      tailwind.config = {
        theme: {
          extend: {
            boxShadow: {
              soft: '0 10px 30px rgba(15, 23, 42, 0.12)'
            }
          }
        }
      };
    </script>
    <script src="https://cdn.jsdelivr.net/npm/lucide@0.469.0/dist/umd/lucide.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.45.4/dist/umd/supabase.js"></script>
    <style>
      html {
        scroll-behavior: smooth;
      }

      body {
        font-family: 'Inter', ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
        background:
          radial-gradient(circle at top left, rgba(191, 164, 110, 0.14), transparent 28%),
          radial-gradient(circle at top right, rgba(148, 163, 184, 0.12), transparent 30%),
          linear-gradient(180deg, #fffdf9 0%, #f7f3ec 100%);
      }

      table {
        border-collapse: collapse;
        table-layout: auto;
      }

      #spreadsheetBody td,
      #spreadsheetBody th,
      #tableHeader th,
      #tableHeader td {
        font-size: 12.5px !important;
        line-height: 1.45;
      }

      #spreadsheetBody td,
      #tableHeader th {
        padding: 0.72rem 0.65rem !important;
        min-width: 120px;
      }

      #spreadsheetBody input,
      #tableHeader input {
        font-size: 12.5px !important;
      }

      #spreadsheetBody tr:hover {
        background: rgba(191, 164, 110, 0.04);
      }

      .luxury-card {
        background: rgba(255, 255, 255, 0.84);
        border: 1px solid rgba(148, 163, 184, 0.2);
        box-shadow: 0 10px 30px rgba(15, 23, 42, 0.05);
      }

      .sticky-col-1 {
        position: sticky;
        left: 0;
        z-index: 20;
      }

      .modal-backdrop {
        background: rgba(15, 23, 42, 0.58);
        backdrop-filter: blur(2px);
      }

      /* ===== Tema Cyberpunk HUD: navy, biru elektrik, lime ===== */
      :root { --lime:#C8FF1A; --blue:#2B3BFF; --panel:#070C3A; --line:rgba(120,135,255,.30); --txt:#F2F4FF; --mut:#8B95D9; --c:#C8FF1A; }
      * { scrollbar-width: thin; scrollbar-color: #C8FF1A #060A33; }
      ::selection { background: #C8FF1A; color: #03051C; }
      ::placeholder { color: #5C67B5 !important; }
      .modal-backdrop:not(.hidden) { display: flex; }
      @keyframes spin { to { transform: rotate(360deg); } }
      @keyframes blink { 50% { opacity: 0; } }
      @keyframes pulse { 50% { transform: scale(1.45); opacity: .65; } }
      @keyframes ping { 70% { box-shadow: 0 0 0 9px rgba(200,255,26,0); } 100% { box-shadow: 0 0 0 0 rgba(200,255,26,0); } }
      @keyframes floaty { 50% { transform: translateY(-4px); } }
      @keyframes slashes { 50% { transform: translateX(6px); opacity: .45; } }
      @keyframes sweep { to { left: 110%; } }
      @keyframes bracket { 50% { opacity: .45; } }
      @keyframes drift { to { transform: translate(-6vmax, 5vmax) scale(1.12); } }
      @keyframes stripes { to { background-position: 16px 0; } }
      @keyframes marquee { to { transform: translateX(-50%); } }

      body.bg-slate-100 {
        font-family: 'JetBrains Mono', ui-monospace, SFMono-Regular, Menlo, monospace !important;
        color: var(--txt);
        background:
          linear-gradient(rgba(120,135,255,.05) 1px, transparent 1px) 0 0/36px 36px,
          linear-gradient(90deg, rgba(120,135,255,.05) 1px, transparent 1px) 0 0/36px 36px,
          #03051C !important;
        background-attachment: fixed;
      }
      body.bg-slate-100::before {
        content: ""; position: fixed; z-index: 0; pointer-events: none;
        width: 62vmax; height: 62vmax; right: -22vmax; top: -28vmax; border-radius: 50%;
        background: radial-gradient(circle, rgba(43,59,255,.6), rgba(43,59,255,0) 66%);
        animation: drift 14s ease-in-out infinite alternate;
      }
      body.bg-slate-100 > .mx-auto, .ticker { position: relative; z-index: 1; }

      /* Bar HUD */
      #hudbar { display: flex; align-items: center; gap: .75rem; margin-bottom: .6rem; font-size: 10px; letter-spacing: .24em; color: var(--mut); }
      #hudbar .hud-x { width: 16px; height: 16px; color: var(--lime); animation: spin 10s linear infinite; }
      #hudbar .hud-line { flex: 1; height: 1px; background: linear-gradient(90deg, rgba(200,255,26,.6), rgba(120,135,255,.12)); }
      #hudbar .live { display: inline-flex; align-items: center; gap: .5rem; color: var(--txt); }
      #hudbar .live b { width: 8px; height: 8px; border-radius: 50%; background: var(--lime); animation: ping 1.8s ease-out infinite; }

      /* Bingkai sudut */
      header.shadow-soft, .luxury-card, #statsStrip { position: relative; }
      header.shadow-soft::after, .luxury-card::after, #statsStrip::after {
        content: ""; position: absolute; inset: -1px; pointer-events: none; z-index: 3;
        background:
          linear-gradient(#C8FF1A,#C8FF1A) 0 0/16px 2px no-repeat, linear-gradient(#C8FF1A,#C8FF1A) 0 0/2px 16px no-repeat,
          linear-gradient(#C8FF1A,#C8FF1A) 100% 0/16px 2px no-repeat, linear-gradient(#C8FF1A,#C8FF1A) 100% 0/2px 16px no-repeat,
          linear-gradient(#C8FF1A,#C8FF1A) 0 100%/16px 2px no-repeat, linear-gradient(#C8FF1A,#C8FF1A) 0 100%/2px 16px no-repeat,
          linear-gradient(#C8FF1A,#C8FF1A) 100% 100%/16px 2px no-repeat, linear-gradient(#C8FF1A,#C8FF1A) 100% 100%/2px 16px no-repeat;
        animation: bracket 3.4s ease-in-out infinite;
      }

      /* Header */
      header.shadow-soft {
        background: linear-gradient(135deg, rgba(7,12,58,.96), rgba(14,24,120,.92)) !important;
        border: 1px solid var(--line) !important; border-radius: 4px !important;
        box-shadow: 0 0 40px rgba(43,59,255,.28) !important;
      }
      @media (min-width: 768px) { header.shadow-soft { padding-right: 7.8rem !important; } }
      header h1 {
        font-family: 'Tektur', sans-serif; text-transform: uppercase; font-weight: 800;
        font-size: clamp(1.4rem, 3.2vw, 2.4rem) !important; letter-spacing: .02em; line-height: 1;
        color: #F2F4FF !important; text-shadow: 0 0 22px rgba(108,123,255,.85);
      }
      header h1::after { content: "_"; color: var(--lime); animation: blink 1.1s steps(1) infinite; }
      header p { color: var(--mut) !important; letter-spacing: .28em !important; font-weight: 500 !important; }
      header p::before { content: "///"; color: var(--lime); margin-right: .6rem; display: inline-block; animation: slashes 2.2s ease-in-out infinite; }
      #renameTitleBtn { color: var(--lime); background: transparent; border: 1px solid rgba(200,255,26,.6); border-radius: 3px; }
      #renameTitleBtn:hover { background: rgba(200,255,26,.12); box-shadow: 0 0 14px rgba(200,255,26,.4); }

      .orbit-badge { position: absolute; right: 16px; top: 50%; width: 84px; height: 84px; margin-top: -42px; display: none; pointer-events: none; }
      @media (min-width: 768px) { .orbit-badge { display: block; } }
      .orbit-badge text { font-family: 'JetBrains Mono', monospace; font-size: 8.4px; font-weight: 700; fill: #C8FF1A; }
      .ob-spin { transform-origin: 60px 60px; animation: spin 18s linear infinite; }
      .ob-rev { transform-origin: 60px 60px; animation: spin 9s linear infinite reverse; }
      .ob-core { transform-origin: 60px 60px; animation: pulse 2.4s ease-in-out infinite; }

      /* Strip statistik */
      #statsStrip { display: grid; grid-template-columns: repeat(2, 1fr); margin-bottom: 1rem; border: 1px solid var(--line); border-radius: 4px; background: linear-gradient(135deg, #0A1250, #0E1A8C 60%, #0A1250); }
      @media (min-width: 768px) { #statsStrip { grid-template-columns: repeat(4, 1fr) 1.35fr; } }
      .stat { display: flex; flex-direction: column; align-items: center; gap: .3rem; padding: 1rem .5rem; border-right: 1px solid var(--line); text-align: center; }
      .stat svg { width: 22px; height: 22px; color: #C9CFF5; animation: floaty 3.2s ease-in-out infinite; }
      .stat:nth-child(2) svg { animation-delay: -.8s; } .stat:nth-child(3) svg { animation-delay: -1.6s; } .stat:nth-child(4) svg { animation-delay: -2.4s; }
      .stat-num { font-family: 'Inter', sans-serif; font-weight: 700; letter-spacing: -.02em; font-variant-numeric: tabular-nums slashed-zero; font-feature-settings: 'tnum' 1, 'zero' 1; font-size: clamp(1.6rem, 3vw, 2.3rem); line-height: 1; color: var(--lime); text-shadow: 0 0 18px rgba(200,255,26,.4); }
      .stat-num small { font-size: .5em; margin-left: 2px; }
      .stat-label { font-size: 10px; letter-spacing: .2em; color: var(--mut); }
      .lime-card { grid-column: 1 / -1; position: relative; overflow: hidden; background: var(--lime); padding: 1rem 1.1rem; display: flex; flex-direction: column; justify-content: center; gap: .7rem; }
      @media (min-width: 768px) { .lime-card { grid-column: auto; } }
      .lime-card p { font-family: 'Tektur', sans-serif; font-weight: 800; font-size: .95rem; line-height: 1.2; color: #03051C !important; padding-right: 1.6rem; }
      .lime-card::after { content: "✱"; position: absolute; right: .7rem; top: .45rem; font-size: 1.3rem; color: #03051C; animation: spin 6s linear infinite; }
      .stripes { height: 8px; background: linear-gradient(135deg, #03051C 25%, transparent 25% 50%, #03051C 50% 75%, transparent 75%) 0 0/16px 16px; animation: stripes 1s linear infinite; }

      /* Tombol */
      .from-blue-600 { --c: #C8FF1A; } .from-emerald-600 { --c: #8E9BFF; } .from-amber-500 { --c: #F2F4FF; } .from-teal-700 { --c: #C8FF1A; }
      button.bg-gradient-to-r {
        background: rgba(3,5,28,.7) !important; background-image: none !important;
        color: var(--c) !important; border: 1px solid var(--c) !important; border-radius: 3px !important;
        font-weight: 700 !important; letter-spacing: .04em;
        box-shadow: 0 0 12px color-mix(in srgb, var(--c) 22%, transparent) !important;
      }
      button.bg-gradient-to-r:hover { background: var(--c) !important; color: #03051C !important; box-shadow: 0 0 22px color-mix(in srgb, var(--c) 60%, transparent) !important; }
      button.from-teal-700, #productTabsContainer button.bg-gradient-to-r { background: var(--c) !important; color: #03051C !important; }
      button.bg-white, button.bg-slate-50 { color: #C9CFF5 !important; border-color: var(--line) !important; border-radius: 3px !important; }
      button.bg-white:hover, button.bg-slate-50:hover { border-color: var(--lime) !important; box-shadow: 0 0 12px rgba(200,255,26,.28); }
      button:focus-visible { outline: 2px solid var(--lime); outline-offset: 2px; }

      /* Panel dan tabel */
      .bg-white, .bg-white\/90, .bg-white\/80 { background-color: var(--panel) !important; }
      .bg-slate-50 { background-color: #0A1048 !important; }
      .bg-slate-100:not(body), .hover\:bg-slate-50:hover, .hover\:bg-slate-100:hover { background-color: #101A66 !important; }
      #productTabsContainer > div { border-color: var(--line) !important; border-radius: 4px !important; }
      .luxury-card { background: var(--panel) !important; border: 1px solid var(--line) !important; border-radius: 4px !important; box-shadow: 0 0 36px rgba(43,59,255,.22) !important; }
      .from-stone-900 { position: relative; overflow: hidden; background-image: linear-gradient(90deg, #0B1250, #1A2599 55%, #0B1250) !important; border-bottom: 1px solid var(--lime); }
      .from-stone-900::after { content: ""; position: absolute; top: 0; bottom: 0; left: -120px; width: 100px; background: linear-gradient(90deg, transparent, rgba(200,255,26,.3), transparent); animation: sweep 4.5s ease-in-out infinite; }
      .luxury-card .font-black.text-sm { font-family: 'Tektur', sans-serif; text-transform: uppercase; color: var(--lime) !important; font-size: 1rem !important; letter-spacing: .06em; }
      #totalRowsCount { color: var(--lime) !important; border-color: rgba(200,255,26,.5) !important; }
      .from-stone-50 { background-image: none !important; background-color: #080E42 !important; }
      #spreadsheetBody tr:hover { background: rgba(200,255,26,.05); }
      .border-slate-300, .border-slate-200, .border-stone-200, .border-stone-300, .border-slate-400 { border-color: rgba(120,135,255,.22) !important; }
      .border-slate-900, .border-slate-800, .border-slate-700 { border-color: rgba(120,135,255,.5) !important; }

      .text-slate-900, .text-slate-800 { color: #F2F4FF !important; }
      .text-slate-700, .text-slate-600, .text-stone-700 { color: #C9CFF5 !important; }
      .text-slate-500, .text-stone-500 { color: #8B95D9 !important; }

      /* Kelompok warna kolom: tetap beda hue */
      .bg-rose-50, .bg-rose-100, .hover\:bg-rose-50:hover { background-color: rgba(255,59,92,.14) !important; }
      .hover\:bg-rose-100:hover { background-color: rgba(255,59,92,.24) !important; }
      .bg-indigo-100, .bg-indigo-200 { background-color: rgba(130,120,255,.2) !important; }
      .bg-teal-100, .bg-teal-200 { background-color: rgba(0,255,200,.15) !important; }
      .bg-amber-50, .bg-amber-100 { background-color: rgba(255,176,0,.14) !important; }
      .bg-emerald-50, .bg-emerald-100 { background-color: rgba(57,255,136,.12) !important; }
      .bg-cyan-50, .bg-cyan-100 { background-color: rgba(0,229,255,.12) !important; }
      [class*="text-rose-"] { color: #FF8FA3 !important; } [class*="text-indigo-"] { color: #B3AEFF !important; }
      [class*="text-teal-"] { color: #5FFFD8 !important; } [class*="text-amber-"] { color: #FFD166 !important; }
      [class*="text-emerald-"] { color: #7CFF9E !important; } [class*="text-cyan-"] { color: #6FF3FF !important; }
      [class*="text-blue-"] { color: #9DB4FF !important; }
      [class*="border-rose-"] { border-color: rgba(255,59,92,.6) !important; } [class*="border-indigo-"] { border-color: rgba(130,120,255,.6) !important; }
      [class*="border-teal-"] { border-color: rgba(0,255,200,.5) !important; } [class*="border-amber-"] { border-color: rgba(255,176,0,.6) !important; }
      [class*="border-emerald-"] { border-color: rgba(57,255,136,.55) !important; } [class*="border-cyan-"] { border-color: rgba(0,229,255,.55) !important; }
      [class*="border-blue-"] { border-color: rgba(125,160,255,.5) !important; }
      .bg-rose-600, .bg-rose-700, .bg-cyan-600, .bg-cyan-700 { color: #03051C !important; }

      input, select, textarea { background-color: #050830; color: #F2F4FF; caret-color: #C8FF1A; font-family: inherit; }
      input, select, textarea { border-color: var(--line) !important; border-radius: 3px !important; }
      input:focus, select:focus, textarea:focus { outline: none !important; border-color: var(--lime) !important; box-shadow: 0 0 0 2px rgba(200,255,26,.18), 0 0 14px rgba(200,255,26,.3) !important; }

      .modal-backdrop { background: rgba(2,3,18,.78); }
      .modal-backdrop > div { background: var(--panel) !important; border: 1px solid rgba(200,255,26,.55) !important; border-radius: 6px !important; box-shadow: 0 0 40px rgba(43,59,255,.4), 0 24px 60px rgba(0,0,0,.6) !important; }

      /* Ticker */
      .ticker { overflow: hidden; margin-top: 1.5rem; background: #2B3BFF; border-top: 1px solid rgba(200,255,26,.6); }
      .ticker-track { display: flex; width: max-content; animation: marquee 30s linear infinite; }
      .ticker:hover .ticker-track { animation-play-state: paused; }
      .ticker-track span { white-space: nowrap; padding: .7rem 1.4rem; font-size: 11px; letter-spacing: .22em; color: #F2F4FF; }
      .ticker-track span::before { content: "✱"; color: var(--lime); margin-right: 1.4rem; }

      /* ===== Warna kelompok kolom selaras tema + angka jelas ===== */
      :is(#tableHeader,#spreadsheetBody) .bg-amber-50, :is(#tableHeader,#spreadsheetBody) .bg-amber-50\/80 { background-color: rgba(200,255,26,0.08) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-amber-100, :is(#tableHeader,#spreadsheetBody) .bg-amber-100\/80 { background-color: rgba(200,255,26,0.12) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-amber-200, :is(#tableHeader,#spreadsheetBody) .bg-amber-200\/80 { background-color: rgba(200,255,26,0.2) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-amber-300, :is(#tableHeader,#spreadsheetBody) .bg-amber-300\/80 { background-color: rgba(200,255,26,0.3) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-amber-400, :is(#tableHeader,#spreadsheetBody) .bg-amber-400\/80 { background-color: rgba(200,255,26,0.38) !important; }
      :is(#tableHeader,#spreadsheetBody) [class*="text-amber-"] { color: #DDFF5C !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-emerald-50, :is(#tableHeader,#spreadsheetBody) .bg-emerald-50\/80 { background-color: rgba(0,194,255,0.08) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-emerald-100, :is(#tableHeader,#spreadsheetBody) .bg-emerald-100\/80 { background-color: rgba(0,194,255,0.12) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-emerald-200, :is(#tableHeader,#spreadsheetBody) .bg-emerald-200\/80 { background-color: rgba(0,194,255,0.2) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-emerald-300, :is(#tableHeader,#spreadsheetBody) .bg-emerald-300\/80 { background-color: rgba(0,194,255,0.3) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-emerald-400, :is(#tableHeader,#spreadsheetBody) .bg-emerald-400\/80 { background-color: rgba(0,194,255,0.38) !important; }
      :is(#tableHeader,#spreadsheetBody) [class*="text-emerald-"] { color: #8FE3FF !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-teal-50, :is(#tableHeader,#spreadsheetBody) .bg-teal-50\/80 { background-color: rgba(0,194,255,0.08) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-teal-100, :is(#tableHeader,#spreadsheetBody) .bg-teal-100\/80 { background-color: rgba(0,194,255,0.12) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-teal-200, :is(#tableHeader,#spreadsheetBody) .bg-teal-200\/80 { background-color: rgba(0,194,255,0.2) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-teal-300, :is(#tableHeader,#spreadsheetBody) .bg-teal-300\/80 { background-color: rgba(0,194,255,0.3) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-teal-400, :is(#tableHeader,#spreadsheetBody) .bg-teal-400\/80 { background-color: rgba(0,194,255,0.38) !important; }
      :is(#tableHeader,#spreadsheetBody) [class*="text-teal-"] { color: #8FE3FF !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-indigo-50, :is(#tableHeader,#spreadsheetBody) .bg-indigo-50\/80 { background-color: rgba(43,59,255,0.08) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-indigo-100, :is(#tableHeader,#spreadsheetBody) .bg-indigo-100\/80 { background-color: rgba(43,59,255,0.12) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-indigo-200, :is(#tableHeader,#spreadsheetBody) .bg-indigo-200\/80 { background-color: rgba(43,59,255,0.2) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-indigo-300, :is(#tableHeader,#spreadsheetBody) .bg-indigo-300\/80 { background-color: rgba(43,59,255,0.3) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-indigo-400, :is(#tableHeader,#spreadsheetBody) .bg-indigo-400\/80 { background-color: rgba(43,59,255,0.38) !important; }
      :is(#tableHeader,#spreadsheetBody) [class*="text-indigo-"] { color: #C5CCFF !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-cyan-50, :is(#tableHeader,#spreadsheetBody) .bg-cyan-50\/80 { background-color: rgba(108,123,255,0.08) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-cyan-100, :is(#tableHeader,#spreadsheetBody) .bg-cyan-100\/80 { background-color: rgba(108,123,255,0.12) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-cyan-200, :is(#tableHeader,#spreadsheetBody) .bg-cyan-200\/80 { background-color: rgba(108,123,255,0.2) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-cyan-300, :is(#tableHeader,#spreadsheetBody) .bg-cyan-300\/80 { background-color: rgba(108,123,255,0.3) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-cyan-400, :is(#tableHeader,#spreadsheetBody) .bg-cyan-400\/80 { background-color: rgba(108,123,255,0.38) !important; }
      :is(#tableHeader,#spreadsheetBody) [class*="text-cyan-"] { color: #C5CCFF !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-rose-50, :is(#tableHeader,#spreadsheetBody) .bg-rose-50\/80 { background-color: rgba(242,244,255,0.08) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-rose-100, :is(#tableHeader,#spreadsheetBody) .bg-rose-100\/80 { background-color: rgba(242,244,255,0.12) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-rose-200, :is(#tableHeader,#spreadsheetBody) .bg-rose-200\/80 { background-color: rgba(242,244,255,0.2) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-rose-300, :is(#tableHeader,#spreadsheetBody) .bg-rose-300\/80 { background-color: rgba(242,244,255,0.3) !important; }
      :is(#tableHeader,#spreadsheetBody) .bg-rose-400, :is(#tableHeader,#spreadsheetBody) .bg-rose-400\/80 { background-color: rgba(242,244,255,0.38) !important; }
      :is(#tableHeader,#spreadsheetBody) [class*="text-rose-8"], :is(#tableHeader,#spreadsheetBody) [class*="text-rose-9"] { color: #F2F4FF !important; }
      .bg-slate-200 { background-color: #0E1654 !important; }
      .bg-slate-300 { background-color: #131D70 !important; }

      /* Badge kelompok kolom (baris di bawah bar Spreadsheet Analisis) */
      #tableHeader tr:first-child th { border-color: rgba(3,5,28,.5) !important; letter-spacing: .1em; }
      #tableHeader th.bg-slate-900 { background: #101A66 !important; color: #F2F4FF !important; box-shadow: inset 0 -2px 0 #C8FF1A; }
      #tableHeader th.bg-amber-600 { background: #C8FF1A !important; color: #03051C !important; }
      #tableHeader th.bg-indigo-700 { background: #2B3BFF !important; color: #FFFFFF !important; }
      #tableHeader th.bg-rose-700 { background: #F2F4FF !important; color: #03051C !important; }
      #tableHeader th.bg-teal-700 { background: #00B8F0 !important; color: #03051C !important; }
      #tableHeader th.bg-slate-800 { background: #0A1048 !important; color: #8B95D9 !important; }
      .bg-rose-600:not(th), .bg-rose-700:not(th) { background-color: #2B3BFF !important; color: #F2F4FF !important; }

      /* Angka: Inter, lebar tetap, nol bergaris agar beda dari 8 */
      #spreadsheetBody, #globalParamsGrid, #totalRowsCount, #stat_products, #stat_rows, #stat_target, #stat_nett {
        font-family: 'Inter', 'JetBrains Mono', sans-serif !important;
        font-variant-numeric: tabular-nums slashed-zero;
        font-feature-settings: 'tnum' 1, 'zero' 1;
      }
      #spreadsheetBody input, #globalParamsGrid input { font-family: inherit !important; font-weight: 600; letter-spacing: .01em; }

      /* ===== Tabel futuristik: baris melayang, badge bersudut potong ===== */
      .luxury-card table { border-collapse: separate !important; border-spacing: 0 7px; }
      .luxury-card .overflow-auto { background: #070C3A; padding: 0 12px 12px; }

      /* Badge kelompok: sudut dipotong */
      #tableHeader tr:first-child th {
        border: 0 !important; padding-top: .85rem !important; padding-bottom: .85rem !important;
        clip-path: polygon(0 0, calc(100% - 14px) 0, 100% 14px, 100% 100%, 14px 100%, 0 calc(100% - 14px));
      }
      /* Sub-judul: chip dengan garis warna kelompok */
      #tableHeader tr:nth-child(2) th {
        border: 0 !important; border-right: 1px solid rgba(3,5,28,.55) !important;
        border-bottom: 2px solid rgba(120,135,255,.45) !important;
        box-shadow: 0 7px 0 0 #070C3A;
      }
      #tableHeader tr:nth-child(2) th:first-child { border-radius: 10px 0 0 0; }
      #tableHeader tr:nth-child(2) th[class*="bg-amber-"] { border-bottom-color: #C8FF1A !important; }
      #tableHeader tr:nth-child(2) th[class*="bg-indigo-"] { border-bottom-color: #6C7BFF !important; }
      #tableHeader tr:nth-child(2) th[class*="bg-rose-"] { border-bottom-color: #F2F4FF !important; }
      #tableHeader tr:nth-child(2) th[class*="bg-teal-"] { border-bottom-color: #00B8F0 !important; }

      /* Baris: pita melayang dengan aksen lime di kiri */
      #spreadsheetBody td {
        border: 0 !important;
        border-top: 1px solid rgba(120,135,255,.2) !important;
        border-bottom: 1px solid rgba(120,135,255,.2) !important;
        border-right: 1px solid rgba(3,5,28,.55) !important;
        background-image: linear-gradient(180deg, rgba(255,255,255,.05), rgba(255,255,255,0));
        transition: border-color .2s, background-image .2s;
      }
      #spreadsheetBody td:first-child {
        border-radius: 12px 0 0 12px;
        border-left: 1px solid rgba(120,135,255,.2) !important;
        box-shadow: inset 3px 0 0 #C8FF1A, 10px 0 16px -10px rgba(0,0,0,.7);
      }
      #spreadsheetBody td:nth-last-child(2) { border-radius: 0 12px 12px 0; border-right-color: rgba(120,135,255,.2) !important; }
      #spreadsheetBody tr:hover td, #spreadsheetBody tr:focus-within td {
        border-top-color: #C8FF1A !important; border-bottom-color: #C8FF1A !important;
        background-image: linear-gradient(0deg, rgba(200,255,26,.12), rgba(200,255,26,.12));
      }
      #spreadsheetBody tr:hover td:first-child, #spreadsheetBody tr:focus-within td:first-child { border-left-color: #C8FF1A !important; }
      #spreadsheetBody tr:hover td:nth-last-child(2), #spreadsheetBody tr:focus-within td:nth-last-child(2) { border-right-color: #C8FF1A !important; }
      #spreadsheetBody td:last-child { background: transparent !important; background-image: none !important; }
      #spreadsheetBody td:last-child button { border-radius: 999px !important; }
      #spreadsheetBody td input[type] {
        background-color: rgba(3,5,28,.55) !important;
        border: 1px solid rgba(120,135,255,.35) !important; border-radius: 10px !important;
      }

      @media (prefers-reduced-motion: reduce) {
        *, *::before, *::after { animation: none !important; transition: none !important; }
      }
    </style>
  </head>
  <body class="bg-slate-100 min-h-screen text-slate-800">
    <div class="max-w-[1800px] mx-auto px-3 md:px-6 py-5">
      <div id="hudbar" aria-hidden="true">
        <svg class="hud-x" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.2"><circle cx="8" cy="8" r="5"/><path d="M8 0v4M8 12v4M0 8h4M12 8h4"/></svg>
        <span>HPP CONSOLE</span><i class="hud-line"></i><span class="live"><b></b>LIVE CALC</span>
      </div>
      <header class="mb-4 bg-white/90 backdrop-blur-md border border-slate-200 rounded-2xl shadow-soft p-3 md:p-4">
        <svg class="orbit-badge" viewBox="0 0 120 120" aria-hidden="true">
          <defs><path id="obCirc" d="M60,60 m-44,0 a44,44 0 1,1 88,0 a44,44 0 1,1 -88,0"/></defs>
          <g class="ob-spin"><text><textPath href="#obCirc" textLength="272" lengthAdjust="spacing">PRICING · HPP · SHOPEE · ANALYTIC ·</textPath></text></g>
          <g class="ob-rev" fill="none" stroke="#6C7BFF" stroke-width="1.2">
            <ellipse cx="60" cy="60" rx="27" ry="10"/><ellipse cx="60" cy="60" rx="27" ry="10" transform="rotate(60 60 60)"/><ellipse cx="60" cy="60" rx="27" ry="10" transform="rotate(120 60 60)"/>
          </g>
          <circle class="ob-core" cx="60" cy="60" r="7" fill="#C8FF1A"/>
        </svg>
        <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-3">
          <div>
            <p class="text-[10px] uppercase tracking-[0.28em] text-slate-500 font-black">Kalkulator HPP</p>
            <div class="flex flex-wrap items-center gap-3">
              <h1 id="appTitle" class="text-xl md:text-2xl font-black text-slate-900 tracking-tight">Pricing Shopee Analytic</h1>
              <button onclick="openTitleModal()" id="renameTitleBtn" class="inline-flex items-center gap-1 px-2.5 py-1 text-[11px] font-bold transition-all duration-150" title="Ubah nama kalkulator" aria-label="Ubah nama kalkulator">
                <i data-lucide="pencil-line" class="w-3.5 h-3.5"></i>
                <span>Ubah Nama</span>
              </button>
            </div>
          </div>

          <div class="flex flex-wrap items-center gap-2 justify-end">
            <button onclick="openNewProductModal()" class="inline-flex items-center gap-2 bg-gradient-to-r from-blue-600 to-blue-500 text-white px-3 py-2 rounded-xl font-bold text-xs shadow-sm shadow-blue-500/20 hover:shadow-md hover:translate-y-[-1px] transition-all duration-150">
              <i data-lucide="plus" class="w-4 h-4"></i>
              Produk Baru
            </button>
            <button onclick="copyCurrentProduct()" class="inline-flex items-center gap-2 bg-white border border-slate-300 text-slate-700 px-3 py-2 rounded-xl font-bold text-xs hover:bg-slate-50 transition-all duration-150" title="Salin produk aktif beserta semua varian dan pengaturannya">
              <i data-lucide="copy" class="w-4 h-4"></i>
              Salin
            </button>
            <button onclick="renameCurrentProduct()" class="inline-flex items-center gap-2 bg-white border border-slate-300 text-slate-700 px-3 py-2 rounded-xl font-bold text-xs hover:bg-slate-50 transition-all duration-150">
              <i data-lucide="pencil" class="w-4 h-4"></i>
              Rename
            </button>
            <button onclick="deleteCurrentProduct()" class="inline-flex items-center gap-2 bg-white border border-rose-300 text-rose-700 px-3 py-2 rounded-xl font-bold text-xs hover:bg-rose-50 transition-all duration-150">
              <i data-lucide="trash-2" class="w-4 h-4"></i>
              Hapus
            </button>
            <button onclick="saveDataManually()" class="inline-flex items-center gap-2 bg-gradient-to-r from-teal-700 to-teal-600 text-white px-3 py-2 rounded-xl font-bold text-xs shadow-sm hover:shadow-md transition-all duration-150" title="Simpan data di browser ini">
              <i data-lucide="save" class="w-4 h-4"></i>
              Simpan
            </button>
            <button id="logoutBtn" hidden onclick="logout()" class="inline-flex items-center gap-2 bg-white border border-slate-300 text-slate-700 px-3 py-2 rounded-xl font-bold text-xs hover:bg-slate-50 transition-all duration-150" title="Keluar dari akun">
              <i data-lucide="log-out" class="w-4 h-4"></i>
              Keluar <span id="userEmail" class="font-medium text-slate-500"></span>
            </button>
            <span id="dataSaveStatus" class="text-[10px] font-bold text-teal-700" aria-live="polite"></span>
            <button onclick="exportToCSV()" class="inline-flex items-center gap-2 bg-gradient-to-r from-emerald-600 to-emerald-500 text-white px-3 py-2 rounded-xl font-bold text-xs shadow-sm shadow-emerald-500/20 hover:shadow-md hover:translate-y-[-1px] transition-all duration-150">
              <i data-lucide="download" class="w-4 h-4"></i>
              Export CSV
            </button>
          </div>
        </div>
      </header>

      <section id="statsStrip" aria-label="Ringkasan produk aktif">
        <div class="stat"><i data-lucide="layers"></i><span class="stat-num"><span id="stat_products" data-v="0">0</span></span><span class="stat-label">PRODUK</span></div>
        <div class="stat"><i data-lucide="rows-3"></i><span class="stat-num"><span id="stat_rows" data-v="0">0</span></span><span class="stat-label">VARIAN AKTIF</span></div>
        <div class="stat"><i data-lucide="target"></i><span class="stat-num"><span id="stat_target" data-v="0">0</span><small>%</small></span><span class="stat-label">TARGET MARGIN</span></div>
        <div class="stat"><i data-lucide="trending-up"></i><span class="stat-num"><span id="stat_nett" data-v="0">0</span><small>%</small></span><span class="stat-label">RATA² NETT MARGIN</span></div>
        <div class="lime-card"><p>Harga yang tepat bukan tebakan.</p><div class="stripes"></div></div>
      </section>

      <div class="mb-4 flex flex-col xl:flex-row xl:items-center xl:justify-between gap-3">
        <div id="productTabsContainer" class="flex flex-wrap gap-2"></div>
        <div class="flex items-center gap-2 bg-white border border-slate-200 rounded-xl px-3 py-2 shadow-sm shadow-slate-200/60">
          <label class="text-[10px] uppercase tracking-[0.2em] text-slate-500 font-black">Cari</label>
          <input id="searchInput" type="text" placeholder="Cari variasi / nama produk..." class="w-full xl:w-72 bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-xs font-bold text-slate-800 outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-300" />
        </div>
      </div>

      <div class="mb-4 flex items-center justify-between gap-3">
        <div id="globalParamsGrid" class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-5 gap-3 w-full"></div>
      </div>

      <div class="rounded-2xl border border-stone-200 bg-white/90 shadow-soft overflow-hidden luxury-card">
        <div class="flex items-center justify-between bg-gradient-to-r from-stone-900 via-stone-800 to-stone-900 text-white px-4 py-3">
          <div class="flex items-center gap-2">
            <div class="rounded-lg bg-white/10 p-1.5 border border-white/10">
              <i data-lucide="table-2" class="w-4 h-4"></i>
            </div>
            <span class="font-black text-sm tracking-wide">Spreadsheet Analisis</span>
          </div>
          <div class="flex items-center gap-2 text-[10px] uppercase tracking-widest text-stone-200">
            <span>Total Data</span>
            <span id="totalRowsCount" class="rounded-full bg-white/10 px-2 py-1 font-black text-white border border-white/10">0</span>
          </div>
        </div>

        <div class="overflow-auto max-h-[72vh]">
          <table class="min-w-full border-collapse">
            <thead id="tableHeader"></thead>
            <tbody id="spreadsheetBody"></tbody>
          </table>
        </div>

        <div class="flex flex-wrap items-center justify-between gap-3 border-t border-stone-200 bg-gradient-to-r from-stone-50 to-white px-4 py-3">
          <div class="text-[10px] uppercase tracking-[0.22em] text-stone-500 font-black">Aksi Tabel</div>
          <div class="flex flex-wrap gap-2">
            <button onclick="addTableColumn()" class="inline-flex items-center gap-2 bg-gradient-to-r from-amber-500 to-yellow-500 text-white px-3 py-2 rounded-xl font-bold text-[11px] shadow-sm shadow-amber-500/20 hover:shadow-md transition-all duration-150">
              <i data-lucide="plus" class="w-4 h-4"></i>
              Tambah Kolom
            </button>
            <button onclick="deleteTableColumn()" class="inline-flex items-center gap-2 border border-rose-200 bg-rose-50 text-rose-700 px-3 py-2 rounded-xl font-bold text-[11px] hover:bg-rose-100 transition-all duration-150">
              <i data-lucide="trash-2" class="w-4 h-4"></i>
              Hapus Kolom
            </button>
            <button onclick="addNewRow()" class="inline-flex items-center gap-2 bg-white border border-stone-300 text-stone-700 px-3 py-2 rounded-xl font-bold text-[11px] hover:bg-stone-50 transition-all duration-150">
              <i data-lucide="rows-3" class="w-4 h-4"></i>
              Tambah Baris
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="ticker" aria-hidden="true"><div class="ticker-track" id="tickerTrack"></div></div>

    <div id="loginModal" class="fixed inset-0 modal-backdrop hidden items-center justify-center z-[60] p-4" style="background:#03051C">
      <div class="bg-white w-full max-w-md rounded-3xl border border-slate-200 shadow-soft overflow-hidden">
        <div class="px-5 py-4 border-b border-slate-200">
          <p class="text-[10px] uppercase tracking-[0.25em] text-slate-500 font-black">Akun</p>
          <h2 class="text-lg font-black text-slate-900">Masuk untuk membuka data</h2>
        </div>
        <div class="p-5">
          <label for="loginEmail" class="block text-[10px] uppercase tracking-[0.2em] text-slate-500 font-black mb-1">Email</label>
          <input id="loginEmail" type="email" autocomplete="email" onkeydown="if(event.key==='Enter'){sendLoginLink();}" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-sm font-bold text-slate-800 outline-none" placeholder="nama@email.com" />
          <p id="loginMsg" class="mt-3 text-xs font-bold text-slate-600" aria-live="polite">Link login akan dikirim ke email ini. Tidak perlu password.</p>
          <div class="mt-4 flex justify-end">
            <button id="loginBtn" onclick="sendLoginLink()" class="inline-flex items-center bg-gradient-to-r from-blue-600 to-blue-500 text-white px-3 py-2 rounded-xl font-bold text-xs">Kirim Link Login</button>
          </div>
        </div>
      </div>
    </div>

    <div id="titleModal" class="fixed inset-0 modal-backdrop hidden items-center justify-center z-50 p-4">
      <div class="bg-white w-full max-w-md rounded-3xl border border-slate-200 shadow-soft overflow-hidden">
        <div class="px-5 py-4 border-b border-slate-200">
          <p class="text-[10px] uppercase tracking-[0.25em] text-slate-500 font-black">Nama Kalkulator</p>
          <h2 class="text-lg font-black text-slate-900">Ubah Nama</h2>
        </div>
        <div class="p-5">
          <input id="appTitleInput" type="text" maxlength="60" onkeydown="if(event.key==='Enter'){saveAppTitle();}" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-sm font-bold text-slate-800 outline-none" />
          <div class="mt-4 flex justify-end gap-2">
            <button onclick="closeModal('titleModal')" class="inline-flex items-center bg-white border border-slate-300 text-slate-700 px-3 py-2 rounded-xl font-bold text-xs">Batal</button>
            <button onclick="saveAppTitle()" class="inline-flex items-center bg-gradient-to-r from-blue-600 to-blue-500 text-white px-3 py-2 rounded-xl font-bold text-xs">Simpan Nama</button>
          </div>
        </div>
      </div>
    </div>

    <div id="productModal" class="fixed inset-0 modal-backdrop hidden items-center justify-center z-50 p-4">
      <div class="bg-white w-full max-w-md rounded-3xl border border-slate-200 shadow-soft overflow-hidden">
        <div class="px-5 py-4 border-b border-slate-200">
          <p id="productModalKicker" class="text-[10px] uppercase tracking-[0.25em] text-slate-500 font-black">Produk</p>
          <h2 id="productModalTitle" class="text-lg font-black text-slate-900"></h2>
        </div>
        <div class="p-5">
          <div id="productModalInputWrap">
            <label for="productModalInput" class="block text-[10px] uppercase tracking-[0.2em] text-slate-500 font-black mb-1">Nama Produk</label>
            <input id="productModalInput" type="text" maxlength="80" onkeydown="if(event.key==='Enter'){confirmProductModal();}" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-sm font-bold text-slate-800 outline-none" />
          </div>
          <p id="productModalText" hidden class="text-sm font-bold text-slate-700"></p>
          <div class="mt-4 flex justify-end gap-2">
            <button onclick="closeModal('productModal')" class="inline-flex items-center bg-white border border-slate-300 text-slate-700 px-3 py-2 rounded-xl font-bold text-xs">Batal</button>
            <button id="productModalOk" onclick="confirmProductModal()" class="inline-flex items-center bg-gradient-to-r from-blue-600 to-blue-500 text-white px-3 py-2 rounded-xl font-bold text-xs">OK</button>
          </div>
        </div>
      </div>
    </div>

    <div id="feeModal" class="fixed inset-0 modal-backdrop hidden items-center justify-center z-50 p-4">
      <div class="bg-white w-full max-w-4xl rounded-3xl border border-slate-200 shadow-soft overflow-hidden">
        <div class="flex items-center justify-between px-5 py-4 border-b border-slate-200 bg-rose-50">
          <div>
            <p class="text-[10px] uppercase tracking-[0.25em] text-rose-600 font-black">Biaya Shopee</p>
            <h2 class="text-lg font-black text-slate-900">Kelola Aturan Fee</h2>
          </div>
          <button onclick="closeModal('feeModal')" class="text-slate-500 hover:text-slate-700">
            <i data-lucide="x" class="w-5 h-5"></i>
          </button>
        </div>

        <div class="p-4">
          <div class="mb-3 flex justify-end">
            <button onclick="addNewFeeRule()" class="inline-flex items-center gap-2 bg-rose-600 text-white px-3 py-2 rounded-xl font-bold text-xs hover:bg-rose-700 transition">
              <i data-lucide="plus" class="w-4 h-4"></i>
              Tambah Fee
            </button>
          </div>
          <div id="shopeeFeeListModal" class="space-y-3"></div>
        </div>

        <div class="flex justify-end gap-2 border-t border-slate-200 px-5 py-4 bg-slate-50">
          <button onclick="closeModal('feeModal')" class="px-4 py-2 rounded-xl border border-slate-300 bg-white text-slate-700 font-bold text-xs hover:bg-slate-100 transition">Batal</button>
          <button onclick="saveFeeRulesAndClose()" class="px-4 py-2 rounded-xl bg-rose-600 text-white font-bold text-xs hover:bg-rose-700 transition">Simpan & Tutup</button>
        </div>
      </div>
    </div>

    <div id="hppColsModal" class="fixed inset-0 modal-backdrop hidden items-center justify-center z-50 p-4">
      <div class="bg-white w-full max-w-4xl rounded-3xl border border-slate-200 shadow-soft overflow-hidden">
        <div class="flex items-center justify-between px-5 py-4 border-b border-slate-200 bg-cyan-50">
          <div>
            <p class="text-[10px] uppercase tracking-[0.25em] text-cyan-600 font-black">Kolom HPP</p>
            <h2 class="text-lg font-black text-slate-900">Kelola Kolom HPP</h2>
          </div>
          <button onclick="closeModal('hppColsModal')" class="text-slate-500 hover:text-slate-700">
            <i data-lucide="x" class="w-5 h-5"></i>
          </button>
        </div>

        <div class="p-4">
          <div class="mb-3 flex justify-end">
            <button onclick="addNewHppColumn()" class="inline-flex items-center gap-2 bg-cyan-600 text-white px-3 py-2 rounded-xl font-bold text-xs hover:bg-cyan-700 transition">
              <i data-lucide="plus" class="w-4 h-4"></i>
              Tambah Kolom
            </button>
          </div>
          <div id="hppColsListModal" class="space-y-3"></div>
        </div>

        <div class="flex justify-end gap-2 border-t border-slate-200 px-5 py-4 bg-slate-50">
          <button onclick="closeModal('hppColsModal')" class="px-4 py-2 rounded-xl border border-slate-300 bg-white text-slate-700 font-bold text-xs hover:bg-slate-100 transition">Batal</button>
          <button onclick="saveHppColumnsAndClose()" class="px-4 py-2 rounded-xl bg-cyan-600 text-white font-bold text-xs hover:bg-cyan-700 transition">Simpan & Tutup</button>
        </div>
      </div>
    </div>

    <div id="newProductModal" class="fixed inset-0 modal-backdrop hidden items-center justify-center z-50 p-4">
      <div class="bg-white w-full max-w-lg rounded-3xl border border-slate-200 shadow-soft overflow-hidden">
        <div class="flex items-center justify-between px-5 py-4 border-b border-slate-200 bg-blue-50">
          <div>
            <p class="text-[10px] uppercase tracking-[0.25em] text-blue-600 font-black">Produk Baru</p>
            <h2 class="text-lg font-black text-slate-900">Buat Produk Baru</h2>
          </div>
          <button onclick="closeModal('newProductModal')" class="text-slate-500 hover:text-slate-700">
            <i data-lucide="x" class="w-5 h-5"></i>
          </button>
        </div>

        <div class="p-5 space-y-4">
          <div>
            <label class="block text-[10px] uppercase tracking-[0.2em] text-slate-500 font-black mb-1">Nama Produk</label>
            <input id="newProductName" type="text" class="w-full bg-slate-50 border border-slate-300 rounded-xl px-3 py-2 text-sm font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-blue-500" placeholder="Contoh: LEMARI BESI BARU" />
          </div>

          <div>
            <label class="block text-[10px] uppercase tracking-[0.2em] text-slate-500 font-black mb-1">Template</label>
            <select id="newProductTemplate" class="w-full bg-slate-50 border border-slate-300 rounded-xl px-3 py-2 text-sm font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-blue-500">
              <option value="lemari">Lemari</option>
              <option value="busa">Busa</option>
              <option value="springbed">Springbed</option>
              <option value="custom">Custom</option>
            </select>
          </div>
        </div>

        <div class="flex justify-end gap-2 border-t border-slate-200 px-5 py-4 bg-slate-50">
          <button onclick="closeModal('newProductModal')" class="px-4 py-2 rounded-xl border border-slate-300 bg-white text-slate-700 font-bold text-xs hover:bg-slate-100 transition">Batal</button>
          <button onclick="createNewProduct()" class="px-4 py-2 rounded-xl bg-blue-600 text-white font-bold text-xs hover:bg-blue-700 transition">Buat Produk</button>
        </div>
      </div>
    </div>

    <script>
      let appData = {
        activeProductId: 'prod_1',
        products: []
      };

      const STORAGE_KEY = 'super_app_hpp_shopee_v3';

      document.addEventListener('DOMContentLoaded', () => {
        loadStoredData();
        if (typeof lucide !== 'undefined' && lucide.createIcons) {
          lucide.createIcons();
        }
        renderApp();
        initCloudData();
        buildTicker();
      });

      function fmtStat(v) { return (Math.round(v * 10) / 10).toLocaleString('id-ID'); }

      function animateStat(el, to) {
        const from = parseFloat(el.dataset.v);
        el.dataset.v = to;
        const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
        if (reduce || isNaN(from) || from === to) { el.textContent = fmtStat(to); return; }
        const t0 = performance.now(), dur = 700;
        (function step(t) {
          const k = Math.min(1, (t - t0) / dur), e = 1 - Math.pow(1 - k, 3);
          el.textContent = fmtStat(from + (to - from) * e);
          if (k < 1 && el.dataset.v == to) requestAnimationFrame(step);
        })(t0);
      }

      function updateStats() {
        try {
          const p = getActiveProduct();
          if (!p || !appData.products) return;
          let sum = 0, n = 0;
          p.rows.forEach(r => {
            const a = getRowAnalysis(p, r);
            if (a && isFinite(a.nettMargin)) { sum += a.nettMargin; n++; }
          });
          const vals = { products: appData.products.length, rows: p.rows.length, target: p.targetMarginPct || 0, nett: n ? sum / n : 0 };
          Object.keys(vals).forEach(k => {
            const el = document.getElementById('stat_' + k);
            if (el) animateStat(el, vals[k]);
          });
        } catch (e) { console.error(e); }
      }

      function buildTicker() {
        const t = document.getElementById('tickerTrack');
        if (!t) return;
        const words = ['HITUNG HPP', 'TENTUKAN HARGA', 'JAGA MARGIN', 'UNTUNG BERSIH'];
        let half = '';
        for (let i = 0; i < 8; i++) half += '<span>' + words[i % words.length] + '</span>';
        t.innerHTML = half + half;
      }

      const DEFAULT_APP_TITLE = 'Pricing Shopee Analytic';

      function applyAppTitle() {
        const t = (appData && appData.appTitle) || DEFAULT_APP_TITLE;
        const el = document.getElementById('appTitle');
        if (el) el.textContent = t;
        document.title = t;
      }

      function openTitleModal() {
        const input = document.getElementById('appTitleInput');
        input.value = (appData && appData.appTitle) || DEFAULT_APP_TITLE;
        openModal('titleModal');
        input.focus();
        input.select();
      }

      function saveAppTitle() {
        const next = document.getElementById('appTitleInput').value.trim();
        if (next) {
          appData.appTitle = next.slice(0, 60);
          saveState();
          applyAppTitle();
        }
        closeModal('titleModal');
      }

      function formatNumber(num) {
        return new Intl.NumberFormat('id-ID').format(Math.round(num || 0));
      }

      function closeModal(id) {
        const el = document.getElementById(id);
        if (el) el.classList.add('hidden');
      }

      function openModal(id) {
        const el = document.getElementById(id);
        if (el) el.classList.remove('hidden');
      }

      function loadStoredData() {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (raw) {
          try {
            appData = JSON.parse(raw);
          } catch (e) {
            console.error(e);
            initDefaultData();
          }
        } else {
          initDefaultData();
        }
      }

      // ===== Penyimpanan server: Supabase (tabel hpp_docs, dilindungi RLS per akun) =====
      const SUPABASE_URL = 'https://vupgwtkejouxeixjkhqh.supabase.co';
      const SUPABASE_KEY = 'sb_publishable_BjjGHZVYAMfvl11k53T4RQ_d-7Trnop';
      const OWNER_KEY = STORAGE_KEY + '_owner';

      let sb = null;
      let currentUid = null;
      let autoSaveTimer = null;
      let pushing = false;
      let pushAgain = false;
      let lastPushed = {};   // doc_id -> json terakhir yang sudah tersimpan di server
      let cloudReady = false;

      function setStatus(msg) {
        const el = document.getElementById('dataSaveStatus');
        if (el) el.textContent = msg;
      }

      function saveState() {
        try {
          localStorage.setItem(STORAGE_KEY, JSON.stringify(appData));
          if (currentUid) localStorage.setItem(OWNER_KEY, currentUid);
        } catch (e) {
          console.error(e);
        }
        if (sb && cloudReady) {
          setStatus('Menyimpan ke server...');
          clearTimeout(autoSaveTimer);
          autoSaveTimer = setTimeout(pushToServer, 1000);
        }
      }

      function buildServerDocs() {
        const docs = {};
        docs['hpp_meta'] = {
          activeProductId: appData.activeProductId,
          appTitle: appData.appTitle || '',
          order: appData.products.map(p => p.id)
        };
        appData.products.forEach(p => { docs['hpp_p_' + p.id] = p; });
        return docs;
      }

      async function pushToServer() {
        if (!sb || !cloudReady || !currentUid) return false;
        if (pushing) { pushAgain = true; return true; }
        pushing = true;
        try {
          const docs = buildServerDocs();
          const changed = [];
          Object.keys(docs).forEach(docId => {
            const json = JSON.stringify(docs[docId]);
            if (lastPushed[docId] !== json) changed.push({ docId, json, row: { user_id: currentUid, doc_id: docId, body: docs[docId] } });
          });
          if (changed.length) {
            const { error } = await sb.from('hpp_docs').upsert(changed.map(c => c.row), { onConflict: 'user_id,doc_id' });
            if (error) throw error;
            changed.forEach(c => { lastPushed[c.docId] = c.json; });
          }
          const removed = Object.keys(lastPushed).filter(id => id.startsWith('hpp_p_') && !docs[id]);
          if (removed.length) {
            const { error } = await sb.from('hpp_docs').delete().in('doc_id', removed);
            if (error) throw error;
            removed.forEach(id => { delete lastPushed[id]; });
          }
          setStatus('Data tersimpan di server');
          return true;
        } catch (e) {
          console.error(e);
          setStatus('GAGAL simpan ke server: ' + ((e && e.message) || 'periksa koneksi'));
          return false;
        } finally {
          pushing = false;
          if (pushAgain) { pushAgain = false; clearTimeout(autoSaveTimer); autoSaveTimer = setTimeout(pushToServer, 300); }
        }
      }

      async function syncFromServer() {
        setStatus('Memuat data dari server...');
        try {
          // Browser ini sebelumnya dipakai akun lain: jangan campur data lokalnya.
          let owner = null;
          try { owner = localStorage.getItem(OWNER_KEY); } catch (e) {}
          if (owner && owner !== currentUid) { initDefaultData(); }

          const { data, error } = await sb.from('hpp_docs').select('doc_id, body');
          if (error) throw error;
          const meta = (data.find(r => r.doc_id === 'hpp_meta') || {}).body;
          const byId = {};
          lastPushed = {};
          data.forEach(r => {
            if (r.doc_id.startsWith('hpp_p_') && r.body && r.body.id) {
              byId[r.body.id] = r.body;
              lastPushed[r.doc_id] = JSON.stringify(r.body);
            }
          });
          const order = (meta && meta.order) || [];
          const ordered = order.filter(id => byId[id]).map(id => byId[id]);
          Object.keys(byId).forEach(id => { if (!order.includes(id)) ordered.push(byId[id]); });

          if (ordered.length) {
            appData = {
              activeProductId: ordered.some(p => p.id === meta.activeProductId) ? meta.activeProductId : ordered[0].id,
              appTitle: (meta && meta.appTitle) || undefined,
              products: ordered
            };
            lastPushed['hpp_meta'] = JSON.stringify({ activeProductId: appData.activeProductId, appTitle: (meta && meta.appTitle) || '', order: ordered.map(p => p.id) });
            cloudReady = true;
            try { localStorage.setItem(STORAGE_KEY, JSON.stringify(appData)); localStorage.setItem(OWNER_KEY, currentUid); } catch (e) {}
            renderApp();
            setStatus('Data server dimuat');
          } else {
            // Akun baru dan server masih kosong: kirim data yang ada di browser.
            cloudReady = true;
            renderApp();
            await pushToServer();
          }
        } catch (e) {
          console.error(e);
          cloudReady = false;
          setStatus('GAGAL memuat dari server: ' + ((e && e.message) || 'periksa koneksi') + '. Data lokal belum disinkronkan.');
        }
      }

      function showLogin(show) {
        const m = document.getElementById('loginModal');
        if (m) m.classList.toggle('hidden', !show);
        const out = document.getElementById('logoutBtn');
        if (out) out.hidden = show;
      }

      function handleSession(session) {
        if (session && session.user) {
          if (session.user.id === currentUid) return;
          currentUid = session.user.id;
          showLogin(false);
          const em = document.getElementById('userEmail');
          if (em) em.textContent = session.user.email || '';
          setTimeout(syncFromServer, 0);
        } else {
          currentUid = null;
          cloudReady = false;
          showLogin(true);
        }
      }

      function initCloudData() {
        const notConfigured = !SUPABASE_URL || !SUPABASE_KEY || SUPABASE_KEY.startsWith('ISI_');
        if (notConfigured || !window.supabase) {
          showLogin(false);
          setStatus(notConfigured ? 'Mode lokal: kunci Supabase belum diisi' : 'Mode lokal: library Supabase gagal dimuat');
          return;
        }
        sb = window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
        sb.auth.onAuthStateChange((event, session) => handleSession(session));
      }

      async function sendLoginLink() {
        const input = document.getElementById('loginEmail');
        const msg = document.getElementById('loginMsg');
        const btn = document.getElementById('loginBtn');
        const email = input.value.trim();
        if (!email) { msg.textContent = 'Isi alamat email dulu.'; return; }
        btn.disabled = true;
        msg.textContent = 'Mengirim link login...';
        const { error } = await sb.auth.signInWithOtp({
          email,
          options: { emailRedirectTo: location.origin + location.pathname }
        });
        btn.disabled = false;
        msg.textContent = error
          ? 'Gagal mengirim: ' + error.message
          : 'Link login sudah dikirim ke ' + email + '. Buka email itu di browser ini lalu klik link-nya.';
      }

      async function logout() {
        clearTimeout(autoSaveTimer);
        if (cloudReady) await pushToServer();
        if (sb) await sb.auth.signOut();
      }

      async function saveDataManually() {
        saveState();
        if (sb && cloudReady) {
          clearTimeout(autoSaveTimer);
          const ok = await pushToServer();
          if (!ok) setStatus('GAGAL simpan ke server. Coba lagi.');
        } else {
          setStatus(sb ? 'Belum login atau data server belum termuat' : 'Mode lokal: hanya tersimpan di browser ini');
        }
      }

      document.addEventListener('visibilitychange', () => {
        if (document.visibilityState === 'hidden' && sb && cloudReady) {
          clearTimeout(autoSaveTimer);
          pushToServer();
        }
      });

      function initDefaultData() {
        appData = {
          activeProductId: 'prod_1',
          products: [
            {
              id: 'prod_1',
              name: 'LEMARI BESI',
              type: 'lemari',
              discPct: 25,
              targetMarginPct: 20,
              shopeeFees: [
                { id: 'admin', name: 'Admin Shopee', type: 'percent', value: 10.0, cap: 0, active: true },
                { id: 'fixed', name: 'Fee Flat / Order', type: 'flat', value: 1250, cap: 0, active: true },
                { id: 'prox', name: 'PROX+ Extra', type: 'percent', value: 4.5, cap: 80000, active: true },
                { id: 'grox', name: 'GROX Extra', type: 'percent', value: 6.5, cap: 60000, active: true },
                { id: 'pph', name: 'PPh 0.5% UMKM', type: 'percent', value: 0.5, cap: 0, active: true }
              ],
              hppCols: [
                { id: 'c_variant', name: 'Spesifikasi / Varian', type: 'text' },
                { id: 'c_modal1', name: 'Modal Utama (Rp)', type: 'currency' },
                { id: 'c_modal2', name: 'Aksesoris & Packing (Rp)', type: 'currency' }
              ],
              rows: [
                { id: 1, c_variant: 'Lemari Besi 2 Pintu Standard', c_modal1: 450000, c_modal2: 35000 },
                { id: 2, c_variant: 'Lemari Besi 3 Pintu Jumbo', c_modal1: 720000, c_modal2: 50000 },
                { id: 3, c_variant: 'Lemari Sliding Premium Mirror', c_modal1: 950000, c_modal2: 65000 }
              ]
            },
            {
              id: 'prod_2',
              name: 'ROYAL D16 (BUSA)',
              type: 'busa',
              discPct: 25,
              targetMarginPct: 20,
              masterPrice: 1400000,
              shopeeFees: [
                { id: 'admin', name: 'Admin Shopee', type: 'percent', value: 10.0, cap: 0, active: true },
                { id: 'fixed', name: 'Fee Flat / Order', type: 'flat', value: 1250, cap: 0, active: true },
                { id: 'prox', name: 'PROX+ Extra', type: 'percent', value: 4.5, cap: 80000, active: true },
                { id: 'grox', name: 'GROX Extra', type: 'percent', value: 6.5, cap: 60000, active: true },
                { id: 'pph', name: 'PPh 0.5% UMKM', type: 'percent', value: 0.5, cap: 0, active: true }
              ],
              hppCols: [
                { id: 'l', name: 'L (cm)', type: 'number' },
                { id: 'p', name: 'P (cm)', type: 'number' },
                { id: 't', name: 'T (cm)', type: 'number' },
                { id: 'busa_cost', name: 'Biaya Busa (Rp)', type: 'computed_busa' },
                { id: 'cover', name: 'Cover (Rp)', type: 'currency' }
              ],
              rows: generateBusa48Rows()
            }
          ]
        };
        saveState();
      }

      function generateBusa48Rows() {
        let rows = [];
        let id = 1;
        const standardWidths = [90, 100, 120, 140, 150, 160, 180, 200];
        const standardHeights = [10, 14, 17, 20, 25, 30];

        standardHeights.forEach(t => {
          standardWidths.forEach(l => {
            let coverPrice = 75000;
            if (l >= 140) coverPrice = 100000;
            if (l >= 180) coverPrice = 120000;
            if (t >= 20) coverPrice += 15000;

            rows.push({
              id: id++,
              l: l,
              p: 200,
              t: t,
              cover: coverPrice
            });
          });
        });
        return rows;
      }

      function getActiveProduct() {
        return appData.products.find(p => p.id === appData.activeProductId) || appData.products[0];
      }

      function renderApp() {
        applyAppTitle();
        renderTabs();
        renderGlobalParams();
        renderTable();
        if (typeof lucide !== 'undefined' && lucide.createIcons) {
          lucide.createIcons();
        }
      }

      function renderTabs() {
        const container = document.getElementById('productTabsContainer');
        if (!container) return;
        container.innerHTML = '';

        appData.products.forEach((p, index) => {
          const isActive = p.id === appData.activeProductId;
          const tabWrap = document.createElement('div');
          tabWrap.className = 'flex items-center gap-1 rounded-xl border border-slate-200 bg-white p-1 shadow-sm shadow-slate-200/60';

          const btn = document.createElement('button');
          btn.type = 'button';
          btn.onclick = () => {
            appData.activeProductId = p.id;
            saveState();
            renderApp();
          };
          btn.className = `px-4 py-2 rounded-lg text-xs font-black transition flex items-center space-x-2 whitespace-nowrap ${
            isActive
              ? 'bg-gradient-to-r from-blue-600 to-blue-500 text-white shadow-md shadow-blue-500/30'
              : 'bg-slate-50 text-slate-700 hover:bg-slate-100 border border-slate-200'
          }`;
          btn.innerHTML = `
                <i data-lucide="${p.type === 'busa' ? 'box' : 'package'}" class="w-4 h-4"></i>
                <span>${p.name}</span>
            `;

          const actions = document.createElement('div');
          actions.className = 'flex items-center gap-1';

          const moveLeftBtn = document.createElement('button');
          moveLeftBtn.type = 'button';
          moveLeftBtn.title = 'Geser ke kiri';
          moveLeftBtn.className = 'w-7 h-7 rounded-md border border-slate-200 bg-slate-50 text-slate-600 hover:bg-slate-100 flex items-center justify-center disabled:opacity-30 transition';
          moveLeftBtn.disabled = index === 0;
          moveLeftBtn.innerHTML = '<i data-lucide="chevron-left" class="w-3.5 h-3.5"></i>';
          moveLeftBtn.onclick = (event) => {
            event.stopPropagation();
            reorderProduct(p.id, -1);
          };

          const moveRightBtn = document.createElement('button');
          moveRightBtn.type = 'button';
          moveRightBtn.title = 'Geser ke kanan';
          moveRightBtn.className = 'w-7 h-7 rounded-md border border-slate-200 bg-slate-50 text-slate-600 hover:bg-slate-100 flex items-center justify-center disabled:opacity-30 transition';
          moveRightBtn.disabled = index === appData.products.length - 1;
          moveRightBtn.innerHTML = '<i data-lucide="chevron-right" class="w-3.5 h-3.5"></i>';
          moveRightBtn.onclick = (event) => {
            event.stopPropagation();
            reorderProduct(p.id, 1);
          };

          actions.appendChild(moveLeftBtn);
          actions.appendChild(moveRightBtn);
          tabWrap.appendChild(btn);
          tabWrap.appendChild(actions);
          container.appendChild(tabWrap);
        });
      }

      function reorderProduct(productId, direction) {
        const index = appData.products.findIndex(p => p.id === productId);
        if (index === -1) return;

        const targetIndex = index + direction;
        if (targetIndex < 0 || targetIndex >= appData.products.length) return;

        const reordered = [...appData.products];
        const [item] = reordered.splice(index, 1);
        reordered.splice(targetIndex, 0, item);

        appData.products = reordered;
        saveState();
        renderApp();
      }

      function renderGlobalParams() {
        const p = getActiveProduct();
        const grid = document.getElementById('globalParamsGrid');
        if (!grid || !p) return;
        grid.innerHTML = '';

        grid.appendChild(createParamCard('Diskon Toko (%)', 'number', p.discPct, (v) => {
          p.discPct = parseFloat(v) || 0;
          saveState();
          renderTable();
        }));

        grid.appendChild(createParamCard('Target Margin (%)', 'number', p.targetMarginPct, (v) => {
          p.targetMarginPct = parseFloat(v) || 0;
          saveState();
          renderTable();
        }, 'bg-amber-50 border-amber-300 text-amber-900'));

        if (p.type === 'busa') {
          grid.appendChild(createParamCard('Harga Busa / m³', 'number', p.masterPrice || 1400000, (v) => {
            p.masterPrice = parseFloat(v) || 0;
            saveState();
            renderTable();
          }, 'bg-emerald-50 border-emerald-300 text-emerald-900'));
        }

        if (p && p.shopeeFees) {
          p.shopeeFees.forEach(fee => {
            if (fee.active) {
              const card = document.createElement('div');
              card.className = 'bg-rose-50 border-2 border-rose-300 rounded-xl p-2';
              card.innerHTML = `
                    <label class="text-[10px] font-black text-rose-900 uppercase tracking-tight block mb-0.5 truncate">${fee.name}</label>
                    <div class="flex gap-1">
                        <input type="number" value="${fee.value}" step="0.1" onchange="updateFeeVal('${fee.id}', this.value)" class="w-full bg-white border border-rose-300 rounded-lg px-2 py-1 text-xs font-black text-rose-900 text-center">
                        ${fee.cap > 0 ? `<span class="text-[9px] font-extrabold text-rose-600 flex items-center">Cap:${formatNumber(fee.cap)}</span>` : ''}
                    </div>
                `;
              grid.appendChild(card);
            }
          });
        }

        const feeManagerCard = document.createElement('div');
        feeManagerCard.className = 'bg-slate-100 border-2 border-dashed border-slate-300 rounded-xl p-2 flex items-center justify-center';
        feeManagerCard.innerHTML = `
          <button onclick="openManageFeeModal()" class="w-full py-3 rounded-lg border border-slate-300 bg-white hover:bg-slate-50 text-slate-700 font-black text-[10px] uppercase tracking-[0.2em] flex items-center justify-center gap-2">
            <i data-lucide="settings-2" class="w-4 h-4"></i>
            Manage Fee
          </button>
        `;
        grid.appendChild(feeManagerCard);
      }

      function createParamCard(label, type, val, onChangeFn, customBg = 'bg-slate-50 border-slate-300 text-slate-900') {
        const div = document.createElement('div');
        div.className = `${customBg} border-2 rounded-xl p-2`;
        div.innerHTML = `
            <label class="text-[10px] font-black uppercase tracking-tight block mb-0.5 truncate">${label}</label>
            <input type="${type}" value="${val}" class="w-full bg-white border border-slate-300 rounded-lg px-2 py-1 text-xs font-black focus:ring-2 focus:ring-blue-500 focus:outline-none">
        `;
        div.querySelector('input').onchange = (e) => onChangeFn(e.target.value);
        return div;
      }

      function updateFeeVal(feeId, val) {
        const p = getActiveProduct();
        const fee = p.shopeeFees.find(f => f.id === feeId);
        if (fee) {
          fee.value = parseFloat(val) || 0;
          saveState();
          renderTable();
        }
      }

      function calculateRowPrices(hpp, targetMarginPct, discPct, shopeeFees) {
        const activeFees = (shopeeFees || []).filter(f => f.active);

        let targetRate = targetMarginPct / 100;
        let flatSum = 0;

        activeFees.forEach(f => {
          if (f.type === 'flat') flatSum += Number(f.value) || 0;
        });

        let finalPrice = (hpp + flatSum) / (1 - targetRate);

        const percentFees = activeFees.filter(f => f.type === 'percent');

        for (let i = 0; i < 50; i++) {
          let effectiveTargetRate = targetRate;
          let cappedFlatSum = flatSum;

          percentFees.forEach(f => {
            const feeAmount = finalPrice * ((Number(f.value) || 0) / 100);

            if ((Number(f.cap) || 0) > 0 && feeAmount > (Number(f.cap) || 0)) {
              cappedFlatSum += Number(f.cap) || 0;
            } else {
              effectiveTargetRate += (Number(f.value) || 0) / 100;
            }
          });

          const nextFinalPrice = (hpp + cappedFlatSum) / (1 - effectiveTargetRate);

          if (Math.abs(nextFinalPrice - finalPrice) < 0.01) {
            finalPrice = nextFinalPrice;
            break;
          }

          finalPrice = nextFinalPrice;
        }

        const discFactor = 1 - (discPct / 100);
        const listingPrice = discFactor > 0 ? (finalPrice / discFactor) : finalPrice;

        return {
          finalPrice: Math.round(finalPrice),
          listingPrice: Math.round(listingPrice)
        };
      }

      function getRowAnalysis(product, row) {
        let hppTotal = 0;

        product.hppCols.forEach(c => {
          if (c.type === 'currency') {
            hppTotal += Number(row[c.id]) || 0;
          } else if (c.type === 'computed_busa') {
            const l = Number(row.l) || 0;
            const p = Number(row.p) || 200;
            const t = Number(row.t) || 0;
            const busaCost = ((l * p * t) / 1000000) * (product.masterPrice || 1400000);
            hppTotal += busaCost;
          }
        });

        const activeFees = product.shopeeFees.filter(f => f.active);
        const priceCalc = calculateRowPrices(hppTotal, product.targetMarginPct, product.discPct, activeFees);
        const finalPrice = priceCalc.finalPrice;
        const listingPrice = priceCalc.listingPrice;

        let feeBreakdownMap = {};
        let subtotalAdmin = 0;

        activeFees.forEach(f => {
          let feeAmt = 0;

          if (f.type === 'percent') {
            feeAmt = finalPrice * ((Number(f.value) || 0) / 100);
            if ((Number(f.cap) || 0) > 0 && feeAmt > (Number(f.cap) || 0)) {
              feeAmt = Number(f.cap) || 0;
            }
          } else if (f.type === 'flat') {
            feeAmt = Number(f.value) || 0;
          }

          feeBreakdownMap[f.id] = feeAmt;
          subtotalAdmin += feeAmt;
        });

        const pencairan = finalPrice - subtotalAdmin;
        const nettProfit = pencairan - hppTotal;
        const nettMargin = finalPrice > 0 ? ((nettProfit / finalPrice) * 100) : 0;

        return {
          hppTotal,
          finalPrice,
          listingPrice,
          feeBreakdownMap,
          subtotalAdmin,
          pencairan,
          nettProfit,
          nettMargin
        };
      }

      function renderTable() {
        updateStats();
        const p = getActiveProduct();
        const searchKeyword = (document.getElementById('searchInput').value || '').toLowerCase();
        const thead = document.getElementById('tableHeader');
        const tbody = document.getElementById('spreadsheetBody');

        if (!p || !thead || !tbody) return;

        tbody.innerHTML = '';
        thead.innerHTML = '';

        const activeFees = p.shopeeFees.filter(f => f.active);
        const hppColCount = p.hppCols.length;
        const feeColCount = activeFees.length + 1;

        const trCat = document.createElement('tr');
        trCat.className = 'text-[11px] font-black uppercase tracking-wider text-center text-white select-none';
        trCat.innerHTML = `
            <th colspan="${hppColCount}" class="bg-slate-900 border-b-2 border-r border-slate-700 py-2.5 px-2">
                ${p.name} - SPESIFIKASI & BIAYA BAHAN (MODAL)
            </th>
            <th colspan="2" class="bg-amber-600 border-b-2 border-r border-amber-700 py-2.5 px-2">
                TOTAL HPP & TARGET
            </th>
            <th colspan="3" class="bg-indigo-700 border-b-2 border-r border-indigo-800 py-2.5 px-2">
                STRATEGI HARGA JUAL (AUTO TARGET ${p.targetMarginPct}%)
            </th>
            <th colspan="${feeColCount}" class="bg-rose-700 border-b-2 border-r border-rose-800 py-2.5 px-2">
                RINCIAN BIAYA & POTONGAN SHOPEE
            </th>
            <th colspan="3" class="bg-teal-700 border-b-2 border-slate-800 py-2.5 px-2">
                PENCAIRAN BERSIH & NETT PROFIT
            </th>
            <th class="bg-slate-800 border-b-2 border-slate-900 py-2.5 px-2 no-print">AKSI</th>
        `;
        thead.appendChild(trCat);

        const trSub = document.createElement('tr');
        trSub.className = 'text-[10px] font-black uppercase tracking-tight text-slate-800 bg-slate-200 border-b-2 border-slate-400 text-center sticky top-0 z-30 shadow-sm';

        let subHtml = '';

        p.hppCols.forEach((c, idx) => {
          const stickyClass = idx === 0 ? 'sticky-col-1 bg-slate-300 text-slate-900' : 'text-slate-900';
          subHtml += `<th class="py-2.5 px-2 border-r border-slate-300 ${stickyClass}">
            <div class="flex items-center justify-between gap-2">
              <span>${c.name}</span>
              <button onclick="deleteTableColumn('${c.id}')" class="no-print rounded p-1 text-rose-600 hover:bg-rose-100 hover:text-rose-800" title="Hapus kolom ${c.name}" aria-label="Hapus kolom ${c.name}">
                <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
              </button>
            </div>
          </th>`;
        });

        subHtml += `
            <th class="py-2.5 px-2 border-r border-slate-300 text-amber-950 bg-amber-200 font-black">TOTAL HPP</th>
            <th class="py-2.5 px-2 border-r border-slate-300 text-amber-950 bg-amber-100">TARGET %</th>
        `;

        subHtml += `
            <th class="py-2.5 px-2 border-r border-slate-300 text-indigo-950 bg-indigo-200 font-black">LISTING PRICE</th>
            <th class="py-2.5 px-2 border-r border-slate-300 text-indigo-950 bg-indigo-100">DISC %</th>
            <th class="py-2.5 px-2 border-r border-slate-300 text-indigo-950 bg-indigo-300/80 font-black">FINAL PRICE</th>
        `;

        activeFees.forEach(f => {
          subHtml += `<th class="py-2.5 px-2 border-r border-slate-300 text-rose-950 bg-rose-100 truncate">${f.name}</th>`;
        });

        subHtml += `<th class="py-2.5 px-2 border-r border-slate-300 text-rose-950 bg-rose-200 font-black">SUB. ADMIN</th>`;

        subHtml += `
            <th class="py-2.5 px-2 border-r border-slate-300 text-teal-950 bg-teal-200 font-black">PENCAIRAN</th>
            <th class="py-2.5 px-2 border-r border-slate-300 text-teal-950 bg-teal-300 font-black">NETT PROFIT</th>
            <th class="py-2.5 px-2 border-r border-slate-300 text-teal-950 bg-teal-400/80 font-black">MARGIN %</th>
            <th class="py-2.5 px-2 w-10 no-print"></th>
        `;

        trSub.innerHTML = subHtml;
        thead.appendChild(trSub);

        const filteredRows = p.rows.filter(r => {
          const searchStr = Object.values(r).join(' ').toLowerCase();
          return searchStr.includes(searchKeyword);
        });

        filteredRows.forEach(r => {
          const analysis = getRowAnalysis(p, r);
          const finalPrice = analysis.finalPrice;
          const listingPrice = analysis.listingPrice;
          const hppTotal = analysis.hppTotal;
          const feeBreakdownMap = analysis.feeBreakdownMap;
          const subtotalAdmin = analysis.subtotalAdmin;
          const pencairan = analysis.pencairan;
          const nettProfit = analysis.nettProfit;
          const nettMargin = analysis.nettMargin;

          const tr = document.createElement('tr');
          tr.className = 'hover:bg-cyan-100/70 transition text-slate-900 border-b border-slate-300';

          let rowHtml = '';

          p.hppCols.forEach((c, idx) => {
            const stickyClass = idx === 0 ? 'sticky-col-1 bg-white' : '';

            if (c.type === 'text') {
              rowHtml += `
                        <td class="py-1.5 px-2 border-r border-slate-300 ${stickyClass}">
                            <input type="text" value="${r[c.id] || ''}" onchange="updateCellVal('${r.id}', '${c.id}', this.value)" class="w-full bg-slate-50 border border-slate-300 rounded px-1.5 py-0.5 font-bold text-slate-800">
                        </td>
                    `;
            } else if (c.type === 'number') {
              rowHtml += `
                        <td class="py-1.5 px-1 border-r border-slate-300 ${stickyClass}">
                            <input type="number" value="${r[c.id] || 0}" onchange="updateCellVal('${r.id}', '${c.id}', this.value)" class="w-16 bg-cyan-50 border border-cyan-300 text-center rounded py-0.5 font-black text-cyan-900">
                        </td>
                    `;
            } else if (c.type === 'currency') {
              rowHtml += `
                        <td class="py-1.5 px-1 border-r border-slate-300">
                            <input type="number" value="${r[c.id] || 0}" step="5000" onchange="updateCellVal('${r.id}', '${c.id}', this.value)" class="w-full bg-emerald-50 border border-emerald-300 text-right rounded px-1.5 py-0.5 font-black text-emerald-950">
                        </td>
                    `;
            } else if (c.type === 'computed_busa') {
              const busaCost = ((Number(r.l) || 0) * (Number(r.p) || 200) * (Number(r.t) || 0) / 1000000) * (p.masterPrice || 1400000);
              rowHtml += `
                        <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-emerald-950 bg-emerald-50">
                            ${formatNumber(busaCost)}
                        </td>
                    `;
            }
          });

          rowHtml += `
                <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-amber-950 bg-amber-100">
                    ${formatNumber(hppTotal)}
                </td>
                <td class="py-2 px-1 border-r border-slate-300 text-center font-black text-amber-900 bg-amber-50">
                    ${p.targetMarginPct}%
                </td>
            `;

          rowHtml += `
                <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-indigo-950 bg-indigo-100">
                    ${formatNumber(listingPrice)}
                </td>
                <td class="py-2 px-1 border-r border-slate-300 text-center font-bold text-indigo-900 bg-indigo-50">
                    ${p.discPct}%
                </td>
                <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-indigo-950 bg-indigo-200">
                    ${formatNumber(finalPrice)}
                </td>
            `;

          activeFees.forEach(f => {
            rowHtml += `
                    <td class="py-2 px-2 border-r border-slate-300 text-right font-bold text-rose-950 bg-rose-50">
                        ${formatNumber(feeBreakdownMap[f.id] || 0)}
                    </td>
                `;
          });

          rowHtml += `
                <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-rose-950 bg-rose-100">
                    ${formatNumber(subtotalAdmin)}
                </td>
                <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-teal-950 bg-teal-100">
                    ${formatNumber(pencairan)}
                </td>
                <td class="py-2 px-2 border-r border-slate-300 text-right font-black text-teal-950 bg-teal-200">
                    ${formatNumber(nettProfit)}
                </td>
                <td class="py-2 px-2 border-r border-slate-300 text-center bg-teal-300/80 font-black text-teal-950">
                    ${nettMargin.toFixed(0)}%
                </td>
                <td class="py-1 px-1 text-center no-print">
                    <button onclick="deleteRow(${r.id})" class="inline-flex items-center justify-center gap-1 rounded-lg border border-rose-200 bg-rose-50 px-2 py-1.5 text-[10px] font-black text-rose-700 hover:bg-rose-100 transition-all duration-150 shadow-sm" title="Hapus Baris">
                        <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
                        <span>Hapus</span>
                    </button>
                </td>
            `;

          tr.innerHTML = rowHtml;
          tbody.appendChild(tr);
        });

        document.getElementById('totalRowsCount').innerText = filteredRows.length;
        if (typeof lucide !== 'undefined' && lucide.createIcons) {
          lucide.createIcons();
        }
      }

      function updateCellVal(rowId, colId, val) {
        const p = getActiveProduct();
        const r = p.rows.find(row => row.id == rowId);
        if (r) {
          const col = p.hppCols.find(c => c.id === colId);
          if (col && (col.type === 'number' || col.type === 'currency')) {
            r[colId] = parseFloat(val) || 0;
          } else {
            r[colId] = val;
          }
          saveState();
          renderTable();
        }
      }

      function addNewRow() {
        const p = getActiveProduct();
        const newId = Date.now();
        let newRow = { id: newId };
        p.hppCols.forEach(c => {
          if (c.type === 'text') newRow[c.id] = 'Varian Baru';
          else if (c.type === 'number') newRow[c.id] = 100;
          else if (c.type === 'currency') newRow[c.id] = 50000;
        });
        p.rows.push(newRow);
        saveState();
        renderTable();
      }

      function deleteRow(id) {
        const p = getActiveProduct();
        p.rows = p.rows.filter(r => r.id != id);
        saveState();
        renderTable();
      }

      function openManageFeeModal() {
        const p = getActiveProduct();
        const container = document.getElementById('shopeeFeeListModal');
        if (!container || !p) return;
        container.innerHTML = '';

        p.shopeeFees.forEach((f, idx) => {
          const div = document.createElement('div');
          div.className = 'flex items-center space-x-2 bg-slate-50 border border-slate-300 p-2.5 rounded-xl';
          div.innerHTML = `
                <input type="checkbox" ${f.active ? 'checked' : ''} onchange="window.p_shopeeFees[${idx}].active = this.checked" class="w-4 h-4 text-rose-600 rounded">
                <input type="text" value="${f.name}" onchange="window.p_shopeeFees[${idx}].name = this.value" class="flex-1 bg-white border border-slate-300 rounded px-2 py-1 text-xs font-bold">
                <select onchange="window.p_shopeeFees[${idx}].type = this.value" class="bg-white border border-slate-300 rounded px-1.5 py-1 text-xs font-bold">
                    <option value="percent" ${f.type === 'percent' ? 'selected' : ''}>% Persen</option>
                    <option value="flat" ${f.type === 'flat' ? 'selected' : ''}>Rp Flat</option>
                </select>
                <input type="number" value="${f.value}" step="0.1" onchange="window.p_shopeeFees[${idx}].value = parseFloat(this.value)||0" class="w-16 bg-white border border-slate-300 rounded px-1 py-1 text-xs font-black text-center" title="Nilai Biaya">
                <input type="number" value="${f.cap || 0}" step="5000" onchange="window.p_shopeeFees[${idx}].cap = parseFloat(this.value)||0" class="w-20 bg-white border border-slate-300 rounded px-1 py-1 text-xs font-black text-right" title="Maksimal Cap (0 jika tanpa cap)" placeholder="Max Cap">
                <button onclick="deleteFeeRule(${idx})" class="text-rose-500 hover:text-rose-700 p-1"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
            `;
          container.appendChild(div);
        });

        window.p_shopeeFees = JSON.parse(JSON.stringify(p.shopeeFees));
        openModal('feeModal');
        if (typeof lucide !== 'undefined' && lucide.createIcons) {
          lucide.createIcons();
        }
      }

      function addNewFeeRule() {
        if (!window.p_shopeeFees) {
          window.p_shopeeFees = getActiveProduct().shopeeFees;
        }
        window.p_shopeeFees.push({
          id: 'fee_' + Date.now(),
          name: 'Biaya Program Baru',
          type: 'percent',
          value: 2.0,
          cap: 0,
          active: true
        });
        refreshFeeModalList();
      }

      function deleteFeeRule(idx) {
        if (!window.p_shopeeFees) return;
        window.p_shopeeFees.splice(idx, 1);
        refreshFeeModalList();
      }

      function refreshFeeModalList() {
        const container = document.getElementById('shopeeFeeListModal');
        if (!container || !window.p_shopeeFees) return;
        container.innerHTML = '';
        window.p_shopeeFees.forEach((f, idx) => {
          const div = document.createElement('div');
          div.className = 'flex items-center space-x-2 bg-slate-50 border border-slate-300 p-2.5 rounded-xl';
          div.innerHTML = `
                <input type="checkbox" ${f.active ? 'checked' : ''} onchange="window.p_shopeeFees[${idx}].active = this.checked" class="w-4 h-4 text-rose-600 rounded">
                <input type="text" value="${f.name}" onchange="window.p_shopeeFees[${idx}].name = this.value" class="flex-1 bg-white border border-slate-300 rounded px-2 py-1 text-xs font-bold">
                <select onchange="window.p_shopeeFees[${idx}].type = this.value" class="bg-white border border-slate-300 rounded px-1.5 py-1 text-xs font-bold">
                    <option value="percent" ${f.type === 'percent' ? 'selected' : ''}>% Persen</option>
                    <option value="flat" ${f.type === 'flat' ? 'selected' : ''}>Rp Flat</option>
                </select>
                <input type="number" value="${f.value}" step="0.1" onchange="window.p_shopeeFees[${idx}].value = parseFloat(this.value)||0" class="w-16 bg-white border border-slate-300 rounded px-1 py-1 text-xs font-black text-center">
                <input type="number" value="${f.cap || 0}" step="5000" onchange="window.p_shopeeFees[${idx}].cap = parseFloat(this.value)||0" class="w-20 bg-white border border-slate-300 rounded px-1 py-1 text-xs font-black text-right" placeholder="Max Cap">
                <button onclick="deleteFeeRule(${idx})" class="text-rose-500 hover:text-rose-700 p-1"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
            `;
          container.appendChild(div);
        });
        if (typeof lucide !== 'undefined' && lucide.createIcons) {
          lucide.createIcons();
        }
      }

      function saveFeeRulesAndClose() {
        const p = getActiveProduct();
        p.shopeeFees = window.p_shopeeFees || p.shopeeFees;
        saveState();
        closeModal('feeModal');
        renderApp();
      }

      function openManageHppColumnsModal() {
        const p = getActiveProduct();
        if (!p) return;
        window.p_hppCols = JSON.parse(JSON.stringify(p.hppCols));
        refreshHppColsModalList();
        openModal('hppColsModal');
      }

      function refreshHppColsModalList() {
        const container = document.getElementById('hppColsListModal');
        if (!container || !window.p_hppCols) return;
        container.innerHTML = '';
        window.p_hppCols.forEach((c, idx) => {
          const div = document.createElement('div');
          div.className = 'flex items-center space-x-2 bg-slate-50 border border-slate-300 p-2.5 rounded-xl';
          div.innerHTML = `
                <input type="text" value="${c.name}" onchange="window.p_hppCols[${idx}].name = this.value" class="flex-1 bg-white border border-slate-300 rounded px-2 py-1 text-xs font-bold">
                <select onchange="window.p_hppCols[${idx}].type = this.value" class="bg-white border border-slate-300 rounded px-1.5 py-1 text-xs font-bold">
                    <option value="currency" ${c.type === 'currency' ? 'selected' : ''}>Input Rupiah (Tambah ke HPP)</option>
                    <option value="text" ${c.type === 'text' ? 'selected' : ''}>Input Teks / Nama Varian</option>
                    <option value="number" ${c.type === 'number' ? 'selected' : ''}>Input Angka Ukuran</option>
                </select>
                <button onclick="deleteHppColumn(${idx})" class="text-rose-500 hover:text-rose-700 p-1"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
            `;
          container.appendChild(div);
        });
        if (typeof lucide !== 'undefined' && lucide.createIcons) {
          lucide.createIcons();
        }
      }

      function addNewHppColumn() {
        const p = getActiveProduct();
        const baseCols = window.p_hppCols ? JSON.parse(JSON.stringify(window.p_hppCols)) : JSON.parse(JSON.stringify(p.hppCols));
        const numericIndex = baseCols.filter(c => c.type === 'number').length + 1;
        const newCol = {
          id: 'c_' + Date.now(),
          name: `Ukuran Baru ${numericIndex}`,
          type: 'number'
        };

        baseCols.push(newCol);
        window.p_hppCols = baseCols;

        p.hppCols = JSON.parse(JSON.stringify(baseCols));
        p.rows.forEach(row => {
          row[newCol.id] = 0;
        });
        saveState();
        renderApp();
        refreshHppColsModalList();
      }

      function addTableColumn() {
        const p = getActiveProduct();
        if (!p) return;

        const newCol = {
          id: 'c_' + Date.now(),
          name: `Ukuran Baru ${p.hppCols.filter(c => c.type === 'number').length + 1}`,
          type: 'number'
        };

        p.hppCols.push(newCol);
        p.rows.forEach(row => {
          row[newCol.id] = 0;
        });
        saveState();
        renderApp();
      }

      function deleteTableColumn(colId) {
        const p = getActiveProduct();
        if (!p || p.hppCols.length === 0) return;

        const colIndex = colId
          ? p.hppCols.findIndex(col => col.id === colId)
          : p.hppCols.length - 1;
        if (colIndex < 0) return;

        const [removedCol] = p.hppCols.splice(colIndex, 1);
        p.rows.forEach(row => {
          delete row[removedCol.id];
        });

        saveState();
        renderApp();
      }

      function deleteHppColumn(idx) {
        if (!window.p_hppCols) return;
        window.p_hppCols.splice(idx, 1);
        refreshHppColsModalList();
      }

      function saveHppColumnsAndClose() {
        const p = getActiveProduct();
        p.hppCols = window.p_hppCols || p.hppCols;
        saveState();
        closeModal('hppColsModal');
        renderApp();
      }

      function openNewProductModal() {
        openModal('newProductModal');
      }

      function createNewProduct() {
        const name = document.getElementById('newProductName').value.trim() || 'PRODUK BARU';
        const template = document.getElementById('newProductTemplate').value;

        const newId = 'prod_' + Date.now();
        let newProd = {
          id: newId,
          name: name.toUpperCase(),
          type: template,
          discPct: 25,
          targetMarginPct: 20,
          shopeeFees: [
            { id: 'admin', name: 'Admin Shopee', type: 'percent', value: 10.0, cap: 0, active: true },
            { id: 'fixed', name: 'Fee Flat / Order', type: 'flat', value: 1250, cap: 0, active: true },
            { id: 'prox', name: 'PROX+ Extra', type: 'percent', value: 4.5, cap: 80000, active: true },
            { id: 'grox', name: 'GROX Extra', type: 'percent', value: 6.5, cap: 60000, active: true },
            { id: 'pph', name: 'PPh 0.5% UMKM', type: 'percent', value: 0.5, cap: 0, active: true }
          ],
          hppCols: [],
          rows: []
        };

        if (template === 'lemari') {
          newProd.hppCols = [
            { id: 'c_variant', name: 'Spesifikasi Varian', type: 'text' },
            { id: 'c_modal1', name: 'Modal Pabrik (Rp)', type: 'currency' },
            { id: 'c_modal2', name: 'Aksesoris & Cargo (Rp)', type: 'currency' }
          ];
          newProd.rows = [
            { id: 1, c_variant: 'Varian Standard 1', c_modal1: 500000, c_modal2: 50000 }
          ];
        } else if (template === 'springbed') {
          newProd.hppCols = [
            { id: 'c_variant', name: 'Ukuran / Varian', type: 'text' },
            { id: 'c_per', name: 'Biaya Per / Busa (Rp)', type: 'currency' },
            { id: 'c_kain', name: 'Biaya Kain Cover (Rp)', type: 'currency' },
            { id: 'c_kayu', name: 'Kayu Rangka & Rakit (Rp)', type: 'currency' }
          ];
          newProd.rows = [
            { id: 1, c_variant: 'Zeus 160x200 Standard', c_per: 450000, c_kain: 150000, c_kayu: 100000 }
          ];
        } else if (template === 'busa') {
          newProd.masterPrice = 1400000;
          newProd.hppCols = [
            { id: 'l', name: 'L (cm)', type: 'number' },
            { id: 'p', name: 'P (cm)', type: 'number' },
            { id: 't', name: 'T (cm)', type: 'number' },
            { id: 'busa_cost', name: 'Biaya Busa (Rp)', type: 'computed_busa' },
            { id: 'cover', name: 'Cover (Rp)', type: 'currency' }
          ];
          newProd.rows = generateBusa48Rows();
        } else {
          newProd.hppCols = [
            { id: 'c_variant', name: 'Nama Varian', type: 'text' },
            { id: 'c_cost', name: 'HPP Modal (Rp)', type: 'currency' }
          ];
          newProd.rows = [
            { id: 1, c_variant: 'Contoh Varian 1', c_cost: 100000 }
          ];
        }

        appData.products.push(newProd);
        appData.activeProductId = newId;
        saveState();
        closeModal('newProductModal');
        document.getElementById('newProductName').value = '';
        renderApp();
      }

      let productModalMode = null;

      function openProductModal(mode) {
        const p = getActiveProduct();
        if (!p) return;
        productModalMode = mode;
        const title = document.getElementById('productModalTitle');
        const kicker = document.getElementById('productModalKicker');
        const input = document.getElementById('productModalInput');
        const inputWrap = document.getElementById('productModalInputWrap');
        const text = document.getElementById('productModalText');
        const ok = document.getElementById('productModalOk');
        inputWrap.hidden = mode === 'delete';
        text.hidden = mode !== 'delete';
        if (mode === 'rename') {
          kicker.textContent = 'Produk';
          title.textContent = 'Ubah Nama Produk';
          input.value = p.name;
          ok.textContent = 'Simpan Nama';
        } else if (mode === 'copy') {
          kicker.textContent = 'Salin Produk';
          title.textContent = 'Salin "' + p.name + '"';
          input.value = p.name + ' (SALINAN)';
          ok.textContent = 'Buat Salinan';
        } else {
          kicker.textContent = 'Hapus Produk';
          title.textContent = 'Hapus "' + p.name + '"?';
          text.textContent = 'Produk beserta ' + p.rows.length + ' varian di dalamnya akan dihapus dan tidak bisa dikembalikan.';
          ok.textContent = 'Ya, Hapus';
        }
        openModal('productModal');
        if (mode !== 'delete') { input.focus(); input.select(); }
      }

      function confirmProductModal() {
        const p = getActiveProduct();
        const input = document.getElementById('productModalInput');
        const name = input.value.trim();
        if (productModalMode === 'rename') {
          if (name) { p.name = name.toUpperCase(); saveState(); renderApp(); }
        } else if (productModalMode === 'copy') {
          const copy = JSON.parse(JSON.stringify(p));
          copy.id = 'prod_' + Date.now();
          copy.name = (name || (p.name + ' (SALINAN)')).toUpperCase();
          const index = appData.products.findIndex(x => x.id === p.id);
          appData.products.splice(index + 1, 0, copy);
          appData.activeProductId = copy.id;
          saveState();
          renderApp();
        } else if (productModalMode === 'delete') {
          if (appData.products.length > 1) {
            appData.products = appData.products.filter(prod => prod.id !== p.id);
            appData.activeProductId = appData.products[0].id;
            saveState();
            renderApp();
          }
        }
        closeModal('productModal');
      }

      function renameCurrentProduct() { openProductModal('rename'); }
      function copyCurrentProduct() { openProductModal('copy'); }

      function deleteCurrentProduct() {
        if (appData.products.length <= 1) {
          setStatus('Tidak bisa menghapus satu-satunya produk yang tersisa.');
          return;
        }
        openProductModal('delete');
      }

      function exportToCSV() {
        const p = getActiveProduct();
        let csvContent = 'data:text/csv;charset=utf-8,';

        let headers = [];
        p.hppCols.forEach(c => headers.push(c.name));
        headers.push('Total HPP', 'Target Margin %', 'Listing Price', 'Discount %', 'Final Price');

        const activeFees = p.shopeeFees.filter(f => f.active);
        activeFees.forEach(f => headers.push(f.name));
        headers.push('Subtotal Admin', 'Pencairan', 'Nett Profit', 'Nett Margin %');

        csvContent += headers.join(',') + '\n';

        p.rows.forEach(r => {
          const analysis = getRowAnalysis(p, r);

          let rowArr = [];

          p.hppCols.forEach(c => {
            if (c.type === 'currency' || c.type === 'number') {
              let val = parseFloat(r[c.id]) || 0;
              rowArr.push(val);
            } else if (c.type === 'text') {
              rowArr.push(`"${(r[c.id] || '').replace(/"/g, '""')}"`);
            } else if (c.type === 'computed_busa') {
              const busaCost = ((Number(r.l) || 0) * (Number(r.p) || 200) * (Number(r.t) || 0) / 1000000) * (p.masterPrice || 1400000);
              rowArr.push(Math.round(busaCost));
            }
          });

          rowArr.push(Math.round(analysis.hppTotal), `${p.targetMarginPct}%`, analysis.listingPrice, `${p.discPct}%`, analysis.finalPrice);

          activeFees.forEach(f => {
            rowArr.push(Math.round(analysis.feeBreakdownMap[f.id] || 0));
          });

          rowArr.push(Math.round(analysis.subtotalAdmin), Math.round(analysis.pencairan), Math.round(analysis.nettProfit), `${analysis.nettMargin.toFixed(1)}%`);

          csvContent += rowArr.join(',') + '\n';
        });

        const csvText = csvContent.replace('data:text/csv;charset=utf-8,', '');
        const filename = `Kalkulator_HPP_${p.name.replace(/\s+/g, '_')}_Shopee.csv`;
        try {
          const blob = new Blob(['\uFEFF' + csvText], { type: 'text/csv;charset=utf-8' });
          const url = URL.createObjectURL(blob);
          const a = document.createElement('a');
          a.href = url;
          a.download = filename;
          document.body.appendChild(a);
          a.click();
          a.remove();
          setTimeout(() => URL.revokeObjectURL(url), 1000);
        } catch (err) {
          console.error(err);
          setStatus('Export CSV gagal. Coba lagi.');
        }
      }

      document.addEventListener('keydown', (e) => {
        if (e.key === 'Escape') {
          ['feeModal', 'hppColsModal', 'newProductModal', 'titleModal', 'productModal'].forEach(closeModal);
        }
      });
    </script>
  </body>
</html>
