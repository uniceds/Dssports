<script>
  // --- Estado del juego ---
  let power = $state(0)      // 0 a 100
  let clicks = $state(0)     // repeticiones totales
  let down = $state(false)   // pump de la barra (arriba/abajo)
  let maxed = $state(false)  // ya llegó al máximo

  const m = $derived(Math.min(power, 100) / 100) // musculo normalizado 0..1

  const stage = $derived(
    maxed ? 5 :
    m < 0.15 ? 0 :
    m < 0.40 ? 1 :
    m < 0.65 ? 2 :
    m < 0.85 ? 3 : 4
  )
  const messages = [
    'Flaco y triste 😔',
    'Calentando... 🔥',
    'Sintiendo el pump 💪',
    '¡Casi lo logras! 😤',
    '¡La polera no aguanta! 🫣',
    '¡¡BESTIA MODE!! 🦾',
  ]

  // --- Geometría del personaje (viewBox 0 0 240 260) ---
  const cx = 120
  const shoulderW = $derived(40 + m * 50)
  const waistW = $derived(24 + m * 16)
  const armThick = $derived(9 + m * 22)
  const bicep = $derived(5 + m * 15)
  const plateR = $derived(11 + m * 13)
  const barHalf = $derived(shoulderW + 10)
  const barY = $derived(down ? 30 : 50)
  const neckW = $derived(12 + m * 10)
  const grow = $derived(1 + m * 0.14)

  // Cara
  const mouthCy = $derived(88 + (0.5 - m) * 26) // frunce -> sonríe
  const tear = $derived(Math.max(0, 1 - m * 2.2))
  const abs = $derived(Math.max(0, (m - 0.5) / 0.5))
  const strain = $derived(Math.max(0, (m - 0.68) / 0.32))

  // --- Animo reflejado sobre la foto ---
  const sadAmt = $derived(Math.max(0, Math.min(1, 1 - m * 1.7)))   // triste al principio
  const rageAmt = $derived(maxed ? 0 : Math.max(0, (m - 0.55) / 0.45)) // esfuerzo rojo
  const faceFilter = $derived(
    `saturate(${(0.25 + m * 1.3).toFixed(2)}) brightness(${(0.82 + m * 0.33).toFixed(2)}) contrast(${(0.95 + m * 0.2).toFixed(2)})`
  )
  const faceTransform = $derived(
    maxed
      ? 'rotate(-8deg)' // cabeza ladeada como en la foto
      : `translate(0px, ${(sadAmt * 3).toFixed(1)}px) rotate(${(sadAmt * 7).toFixed(1)}deg)`
  )
  const moodEmoji = $derived(['😔', '😐', '🙂', '😤', '🥵', '🦾'][stage])
  // expresion facial del personaje segun etapa
  const expr = $derived(
    maxed ? 'joy' : stage === 0 ? 'sad' : stage === 1 ? 'meh' : stage <= 3 ? 'focus' : 'strain'
  )

  // Polera rota al máximo
  const shirtL = $derived(maxed ? 'translate(-34px,20px) rotate(-34deg)' : 'none')
  const shirtR = $derived(maxed ? 'translate(34px,20px) rotate(34deg)' : 'none')

  function pump() {
    if (maxed) return
    clicks++
    down = !down
    power = Math.min(100, power + 3)
    if (power >= 100) maxed = true
  }

  function reset() {
    power = 0; clicks = 0; maxed = false; down = false
  }

  function onKey(e) {
    if (e.code === 'Space' || e.code === 'Enter') { e.preventDefault(); pump() }
  }

  // Decaimiento: si no aprietas, se desinfla (¡hay que ir rápido!)
  $effect(() => {
    const id = setInterval(() => {
      if (!maxed && power > 0) power = Math.max(0, power - 0.9)
    }, 120)
    return () => clearInterval(id)
  })

  // Confeti al ganar
  const confetti = Array.from({ length: 18 }, (_, i) => ({
    left: Math.random() * 100,
    delay: Math.random() * 0.5,
    hue: Math.floor(Math.random() * 360),
    dur: 1.6 + Math.random() * 1.2,
  }))
</script>

<svelte:window on:keydown={onKey} />

<div class="game" class:shake={down && m > 0.6} class:win={maxed}>
  <h2>DSSPORT · Gym Clicker</h2>
  <p class="msg" style="opacity:{0.6 + m * 0.4}">{messages[stage]}</p>

  <div class="stage">
    <div class="mood-badge" class:pop={maxed}>{moodEmoji}</div>
    <svg viewBox="-30 0 300 260" role="img" aria-label="Personaje levantando pesas">
      <defs>
        <radialGradient id="floor" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="rgba(0,0,0,.35)" />
          <stop offset="100%" stop-color="rgba(0,0,0,0)" />
        </radialGradient>
      </defs>

      <!-- sombra -->
      <ellipse cx={cx} cy="238" rx={shoulderW + 20} ry="12" fill="url(#floor)" />

      <g style="transform: scale({grow}); transform-origin: {cx}px 236px; transition: transform .18s ease-out">

        <!-- torso musculo (piel, debajo de la polera) -->
        <path
          d="M {cx - shoulderW} 116
             C {cx - shoulderW} 150, {cx - waistW} 168, {cx - waistW} 196
             L {cx + waistW} 196
             C {cx + waistW} 168, {cx + shoulderW} 150, {cx + shoulderW} 116 Z"
          fill="#e6a97c" stroke="#cf8c5f" stroke-width="2" />

        <!-- pectorales -->
        <g stroke="#cf8c5f" fill="none" stroke-linecap="round" opacity={0.3 + m * 0.7}>
          <path d="M {cx} 128 Q {cx - shoulderW * 0.55} 150 {cx - waistW * 0.5} 154" stroke-width={1.5 + m * 2} />
          <path d="M {cx} 128 Q {cx + shoulderW * 0.55} 150 {cx + waistW * 0.5} 154" stroke-width={1.5 + m * 2} />
        </g>

        <!-- abdominales (aparecen fuertes) -->
        <g stroke="#cf8c5f" stroke-linecap="round" opacity={abs}>
          <line x1={cx} y1="158" x2={cx} y2="192" stroke-width="2" />
          <line x1={cx - 14} y1="168" x2={cx + 14} y2="168" stroke-width="2" />
          <line x1={cx - 13} y1="180" x2={cx + 13} y2="180" stroke-width="2" />
        </g>

        <!-- polera (dos mitades que se rompen) -->
        <g class="shirt">
          <polygon
            style="transform:{shirtL}; transform-box:fill-box; transform-origin:100% 100%;"
            points="{cx},112 {cx - shoulderW - 2},116 {cx - waistW - 2},198 {cx},198"
            fill="#2f6fb0" stroke="#234f7f" stroke-width="2" />
          <polygon
            style="transform:{shirtR}; transform-box:fill-box; transform-origin:0% 100%;"
            points="{cx},112 {cx + shoulderW + 2},116 {cx + waistW + 2},198 {cx},198"
            fill="#3378bd" stroke="#234f7f" stroke-width="2" />
          <!-- lineas de tension antes de romperse -->
          {#if !maxed}
            <g stroke="#bcd7f0" stroke-width="1.5" stroke-linecap="round" opacity={strain}>
              <path d="M {cx - 10} 140 l 6 5 l -6 5 l 6 5" fill="none" />
              <path d="M {cx + 10} 150 l -6 5 l 6 5 l -6 5" fill="none" />
            </g>
          {/if}
        </g>

        <!-- cuello -->
        <rect x={cx - neckW / 2} y="92" width={neckW} height="22" rx="4" fill="#e6a97c" stroke="#cf8c5f" stroke-width="1.5" />

        <!-- cabeza: personaje animado con expresiones (estilo de la foto) -->
        {#if maxed}
          <circle cx={cx} cy="64" r="35" fill="none" stroke="#ffd24a" stroke-width="3" class="glow" />
        {/if}
        <g style="filter:{faceFilter}; transform:{faceTransform}; transform-box:fill-box; transform-origin:center; transition: filter .3s ease, transform .3s ease">
          <!-- orejas -->
          <ellipse cx="94" cy="70" rx="5" ry="6.5" fill="#e6a97c" stroke="#cf8c5f" stroke-width="1.5" />
          <ellipse cx="146" cy="70" rx="5" ry="6.5" fill="#e6a97c" stroke="#cf8c5f" stroke-width="1.5" />
          <!-- cara con mandibula marcada (como la foto) -->
          <path d="M 95 62 C 95 47 105 40 120 40 C 135 40 145 47 145 62 C 145 75 140 85 132 90.5 C 127 94 113 94 108 90.5 C 100 85 95 75 95 62 Z" fill="#e6a97c" stroke="#cf8c5f" stroke-width="1.5" />
          <!-- rubor por esfuerzo -->
          <ellipse cx="103" cy="77" rx="5.5" ry="3" fill="#e2574d" opacity={rageAmt * 0.55} />
          <ellipse cx="137" cy="77" rx="5.5" ry="3" fill="#e2574d" opacity={rageAmt * 0.55} />
          <!-- pelo puntudo cafe muy oscuro con flequillo sobre la frente (como la foto) -->
          <path d="M 94 64 C 92 50 93 42 98 36 L 102 42 L 105 31 L 109 39 L 113 28 L 117 37 L 121 27 L 125 36 L 129 27 L 133 37 L 137 30 L 141 40 L 144 37 C 147 44 147 54 145 63 C 143 54 139 50 135 52 L 131 47 L 125 52 L 119 46 L 112 52 L 106 48 L 102 53 C 98 55 96 59 94 64 Z" fill="#241910" />
          <!-- mechones con brillo -->
          <path d="M 109 34 l 2.5 7 M 118 31 l 2 8 M 128 32 l 1 8" stroke="#43301c" stroke-width="1.6" stroke-linecap="round" fill="none" />
          <!-- patillas -->
          <path d="M 94 58 q -1 10 2 16 l 4.5 -3 q -4 -6 -6.5 -13 Z" fill="#241910" />
          <path d="M 146 58 q 1 10 -2 16 l -4.5 -3 q 4 -6 6.5 -13 Z" fill="#241910" />
          <!-- nariz definida (como la foto) -->
          <path d="M 119.5 66 q 3.2 7.5 0.5 12 q 2 2.2 4.8 1.4" fill="none" stroke="#c9885a" stroke-width="2.2" stroke-linecap="round" />

          {#if expr === 'sad'}
            <!-- 😔 cejas preocupadas, ojos caidos, puchero -->
            <path d="M 103 60 Q 109 58 114 55.5" fill="none" stroke="#1e1207" stroke-width="4.4" stroke-linecap="round" />
            <path d="M 137 60 Q 131 58 126 55.5" fill="none" stroke="#1e1207" stroke-width="4.4" stroke-linecap="round" />
            <ellipse cx="110" cy="67.5" rx="4.5" ry="4" fill="#fff" />
            <ellipse cx="130" cy="67.5" rx="4.5" ry="4" fill="#fff" />
            <circle cx="110" cy="69" r="2.6" fill="#4a2c15" />
            <circle cx="130" cy="69" r="2.6" fill="#4a2c15" />
            <path d="M 105.5 65 h 9" stroke="#cf8c5f" stroke-width="2.5" stroke-linecap="round" />
            <path d="M 125.5 65 h 9" stroke="#cf8c5f" stroke-width="2.5" stroke-linecap="round" />
            <path d="M 112 89 Q 120 83.5 128 89" fill="none" stroke="#8a4a2a" stroke-width="3" stroke-linecap="round" />
          {:else if expr === 'meh'}
            <!-- 😐 neutro -->
            <path d="M 103 58 L 115 58" stroke="#1e1207" stroke-width="4.4" stroke-linecap="round" />
            <path d="M 125 58 L 137 58" stroke="#1e1207" stroke-width="4.4" stroke-linecap="round" />
            <ellipse cx="110" cy="67" rx="4.5" ry="4.8" fill="#fff" />
            <ellipse cx="130" cy="67" rx="4.5" ry="4.8" fill="#fff" />
            <circle cx="110" cy="67.5" r="2.8" fill="#4a2c15" />
            <circle cx="130" cy="67.5" r="2.8" fill="#4a2c15" />
            <circle cx="111" cy="66" r="0.9" fill="#fff" />
            <circle cx="131" cy="66" r="0.9" fill="#fff" />
            <path d="M 111 88 L 129 88" stroke="#8a4a2a" stroke-width="3" stroke-linecap="round" />
          {:else if expr === 'focus'}
            <!-- 😤 decidido -->
            <path d="M 103 55.5 Q 109 57.5 114 59.5" fill="none" stroke="#1e1207" stroke-width="4.6" stroke-linecap="round" />
            <path d="M 137 55.5 Q 131 57.5 126 59.5" fill="none" stroke="#1e1207" stroke-width="4.6" stroke-linecap="round" />
            <ellipse cx="110" cy="67" rx="4.5" ry="4.6" fill="#fff" />
            <ellipse cx="130" cy="67" rx="4.5" ry="4.6" fill="#fff" />
            <circle cx="110" cy="67.5" r="2.8" fill="#4a2c15" />
            <circle cx="130" cy="67.5" r="2.8" fill="#4a2c15" />
            <circle cx="111" cy="66" r="0.9" fill="#fff" />
            <circle cx="131" cy="66" r="0.9" fill="#fff" />
            <path d="M 110 87.5 Q 120 91 130 87.5" fill="none" stroke="#8a4a2a" stroke-width="3" stroke-linecap="round" />
          {:else if expr === 'strain'}
            <!-- 🥵 apretando los dientes, ojos cerrados con fuerza -->
            <path d="M 103 54 Q 110 57 115 61" fill="none" stroke="#1e1207" stroke-width="4.6" stroke-linecap="round" />
            <path d="M 137 54 Q 130 57 125 61" fill="none" stroke="#1e1207" stroke-width="4.6" stroke-linecap="round" />
            <path d="M 105 64.5 l 9 3 l -9 3" fill="none" stroke="#3a2415" stroke-width="2.8" stroke-linecap="round" stroke-linejoin="round" />
            <path d="M 135 64.5 l -9 3 l 9 3" fill="none" stroke="#3a2415" stroke-width="2.8" stroke-linecap="round" stroke-linejoin="round" />
            <rect x="108" y="83" width="24" height="8" rx="3" fill="#fff" stroke="#8a4a2a" stroke-width="2" />
            <line x1="114" y1="83" x2="114" y2="91" stroke="#d9b8a3" stroke-width="1.5" />
            <line x1="120" y1="83" x2="120" y2="91" stroke="#d9b8a3" stroke-width="1.5" />
            <line x1="126" y1="83" x2="126" y2="91" stroke="#d9b8a3" stroke-width="1.5" />
          {:else}
            <!-- 😁 sonrisa gigante con dientes (como la foto) -->
            <path d="M 102 56 Q 108 51.5 115 54" fill="none" stroke="#1e1207" stroke-width="4.6" stroke-linecap="round" />
            <path d="M 138 56 Q 132 51.5 125 54" fill="none" stroke="#1e1207" stroke-width="4.6" stroke-linecap="round" />
            <ellipse cx="110" cy="66" rx="4.6" ry="5.2" fill="#fff" />
            <ellipse cx="130" cy="66" rx="4.6" ry="5.2" fill="#fff" />
            <circle cx="110" cy="66.5" r="3" fill="#4a2c15" />
            <circle cx="130" cy="66.5" r="3" fill="#4a2c15" />
            <circle cx="110.6" cy="65.4" r="1.3" fill="#17100a" />
            <circle cx="130.6" cy="65.4" r="1.3" fill="#17100a" />
            <circle cx="111.4" cy="64.4" r="1" fill="#fff" />
            <circle cx="131.4" cy="64.4" r="1" fill="#fff" />
            <path d="M 104 81 Q 120 103 136 81 Q 120 87 104 81 Z" fill="#7a3524" stroke="#5d2718" stroke-width="1.5" stroke-linejoin="round" />
            <path d="M 107 82.5 Q 120 95 133 82.5 Q 120 88 107 82.5 Z" fill="#fff" />
            <path d="M 101 79 q -3 2 -2 5" fill="none" stroke="#cf8c5f" stroke-width="2" stroke-linecap="round" />
            <path d="M 139 79 q 3 2 2 5" fill="none" stroke="#cf8c5f" stroke-width="2" stroke-linecap="round" />
            <!-- pliegues de sonrisa como la foto -->
            <path d="M 112 74 q -6 3 -8 8" fill="none" stroke="#c9885a" stroke-width="1.8" stroke-linecap="round" />
            <path d="M 128 74 q 6 3 8 8" fill="none" stroke="#c9885a" stroke-width="1.8" stroke-linecap="round" />
          {/if}

          <!-- lagrima cuando esta flaco y triste -->
          <path d="M 133 76 q -3 6 0 9 q 3 -3 0 -9 Z" fill="#7ec7f5" opacity={tear} class="sweat-tear" />
        </g>

        <!-- brazos (frente) + barra -->
        <g style="transition: all .09s linear">
          <!-- brazo izq -->
          <line x1={cx - shoulderW + 6} y1="120" x2={cx - barHalf} y2={barY} stroke="#e6a97c" stroke-width={armThick} stroke-linecap="round" />
          <circle cx={(cx - shoulderW + 6 + cx - barHalf) / 2} cy={(120 + barY) / 2} r={bicep} fill="#e6a97c" stroke="#cf8c5f" stroke-width="1.5" />
          <!-- brazo der -->
          <line x1={cx + shoulderW - 6} y1="120" x2={cx + barHalf} y2={barY} stroke="#e6a97c" stroke-width={armThick} stroke-linecap="round" />
          <circle cx={(cx + shoulderW - 6 + cx + barHalf) / 2} cy={(120 + barY) / 2} r={bicep} fill="#e6a97c" stroke="#cf8c5f" stroke-width="1.5" />

          <!-- barra -->
          <line x1={cx - barHalf - plateR} y1={barY} x2={cx + barHalf + plateR} y2={barY} stroke="#9aa3ad" stroke-width="6" stroke-linecap="round" />
          <!-- discos -->
          <circle cx={cx - barHalf - 4} cy={barY} r={plateR} fill="#2b2f36" stroke="#11141a" stroke-width="2" />
          <circle cx={cx - barHalf - 4} cy={barY} r={plateR * 0.4} fill="#4a5058" />
          <circle cx={cx + barHalf + 4} cy={barY} r={plateR} fill="#2b2f36" stroke="#11141a" stroke-width="2" />
          <circle cx={cx + barHalf + 4} cy={barY} r={plateR * 0.4} fill="#4a5058" />
        </g>

        <!-- gotas de sudor al esforzarse -->
        {#if down && m > 0.35}
          <circle cx="150" cy="60" r="3" fill="#8fd0f5" class="sweat" />
          <circle cx="92" cy="64" r="2.5" fill="#8fd0f5" class="sweat" />
        {/if}
      </g>
    </svg>

    {#if maxed}
      <div class="confetti">
        {#each confetti as c}
          <span style="left:{c.left}%; animation-delay:{c.delay}s; animation-duration:{c.dur}s; background:hsl({c.hue} 90% 60%)"></span>
        {/each}
      </div>
    {/if}
  </div>

  <!-- barra de progreso -->
  <div class="bar"><div class="fill" style="width:{m * 100}%"></div></div>

  <div class="stats">
    <span>💪 Fuerza: {Math.round(power)}%</span>
    <span>🔁 Reps: {clicks}</span>
  </div>

  {#if maxed}
    <button class="cta reset" onclick={reset}>😎 Volver a empezar</button>
  {:else}
    <button class="cta" onclick={pump}>💥 ¡LEVANTAR!</button>
    <p class="hint">Aprieta rápido (o barra espaciadora). Si te detienes, se desinfla…</p>
  {/if}
</div>

<style>
  .game {
    width: min(420px, 92vw);
    text-align: center;
    color: #eef2f7;
    font-family: system-ui, sans-serif;
    user-select: none;
  }
  h2 {
    margin: 0 0 2px;
    font-size: 1.1rem;
    letter-spacing: 0.14em;
    color: #7ec7f5;
    font-weight: 700;
  }
  .msg {
    margin: 0 0 6px;
    font-size: 1rem;
    font-weight: 600;
    min-height: 1.3em;
    transition: opacity 0.2s;
  }
  .stage {
    position: relative;
    background: radial-gradient(120% 90% at 50% 15%, #22303f 0%, #131a22 70%);
    border: 1px solid #2b3a49;
    border-radius: 18px;
    padding: 8px;
    overflow: hidden;
  }
  svg { width: 100%; height: auto; display: block; }
  .shirt polygon { transition: transform 0.55s cubic-bezier(.4, 1.6, .6, 1); }
  .sweat { animation: drip 0.4s ease-in forwards; }
  @keyframes drip { to { transform: translateY(14px); opacity: 0; } }
  .glow { animation: pulse 0.8s ease-in-out infinite; }
  @keyframes pulse { 0%, 100% { opacity: 0.4; } 50% { opacity: 1; } }

  .mood-badge {
    position: absolute;
    top: 10px;
    right: 12px;
    font-size: 1.8rem;
    z-index: 2;
    filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.5));
    transition: transform 0.2s;
  }
  .mood-badge.pop { animation: pop 0.5s ease-out; }
  @keyframes pop {
    0% { transform: scale(0.4) rotate(-20deg); }
    60% { transform: scale(1.4) rotate(10deg); }
    100% { transform: scale(1) rotate(0); }
  }

  .game.shake .stage { animation: shk 0.09s linear infinite; }
  @keyframes shk {
    0%, 100% { transform: translate(0, 0); }
    25% { transform: translate(-2px, 1px); }
    75% { transform: translate(2px, -1px); }
  }
  .game.win h2 { color: #ffd24a; }

  .bar {
    height: 14px;
    background: #1b2530;
    border-radius: 10px;
    overflow: hidden;
    margin: 12px 0 8px;
    border: 1px solid #2b3a49;
  }
  .fill {
    height: 100%;
    background: linear-gradient(90deg, #3ba0ff, #ff5d5d, #ffd24a);
    background-size: 300% 100%;
    border-radius: 10px;
    transition: width 0.12s linear;
  }
  .stats {
    display: flex;
    justify-content: space-between;
    font-size: 0.9rem;
    color: #9fb2c4;
    margin-bottom: 12px;
    font-variant-numeric: tabular-nums;
  }
  .cta {
    width: 100%;
    padding: 16px;
    font-size: 1.15rem;
    font-weight: 800;
    letter-spacing: 0.05em;
    color: #fff;
    background: linear-gradient(180deg, #ff6b3d, #e23b3b);
    border: none;
    border-radius: 14px;
    cursor: pointer;
    box-shadow: 0 6px 0 #a52020, 0 10px 18px rgba(0, 0, 0, 0.4);
    transition: transform 0.05s, box-shadow 0.05s;
  }
  .cta:active {
    transform: translateY(5px);
    box-shadow: 0 1px 0 #a52020, 0 4px 10px rgba(0, 0, 0, 0.4);
  }
  .cta.reset { background: linear-gradient(180deg, #3ba0ff, #2b6cb0); box-shadow: 0 6px 0 #1c4a7d, 0 10px 18px rgba(0, 0, 0, 0.4); }
  .hint { color: #6f8296; font-size: 0.78rem; margin: 8px 0 0; }

  .confetti { position: absolute; inset: 0; pointer-events: none; }
  .confetti span {
    position: absolute;
    top: -10px;
    width: 8px;
    height: 12px;
    border-radius: 2px;
    animation-name: fall;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
  }
  @keyframes fall {
    0% { transform: translateY(-10px) rotate(0); opacity: 1; }
    100% { transform: translateY(260px) rotate(540deg); opacity: 0; }
  }
</style>
