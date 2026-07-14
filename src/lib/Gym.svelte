<script>
  // --- Estado del juego ---
  let power = $state(0)      // 0 a 100
  let clicks = $state(0)     // repeticiones totales
  let down = $state(false)   // pump de la barra (arriba/abajo)
  let maxed = $state(false)  // ya llegó al máximo
  let canReset = $state(false) // el boton de reinicio recien aparece tras un momento

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
    '¡BESTIA MODE! 🦾',
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

  // Cara / cuerpo
  const abs = $derived(Math.max(0, (m - 0.5) / 0.5))
  const strain = $derived(Math.max(0, (m - 0.68) / 0.32))
  const sadAmt = $derived(Math.max(0, Math.min(1, 1 - m * 1.7))) // triste al principio

  // --- Expresiones reales del avatar ---
  const faces = ['cansado', 'normal', 'esfuerzo', 'feliz']
  const faceImg = $derived(
    maxed ? 'feliz' : stage === 0 ? 'cansado' : stage <= 2 ? 'normal' : 'esfuerzo'
  )
  const faceFilter = $derived(
    `saturate(${(0.7 + m * 0.5).toFixed(2)}) brightness(${(0.92 + m * 0.12).toFixed(2)})`
  )
  const faceTransform = $derived(
    maxed
      ? 'rotate(-6deg)' // cabeza ladeada como en la foto
      : `translate(0px, ${(sadAmt * 2).toFixed(1)}px) rotate(${(sadAmt * 5).toFixed(1)}deg)`
  )
  const moodEmoji = $derived(['😔', '😐', '🙂', '😤', '🥵', '🦾'][stage])

  // Polera rota al máximo
  const shirtL = $derived(maxed ? 'translate(-34px,20px) rotate(-34deg)' : 'none')
  const shirtR = $derived(maxed ? 'translate(34px,20px) rotate(34deg)' : 'none')

  function pump() {
    if (maxed) return
    clicks++
    down = !down
    power = Math.min(100, power + 3)
    if (power >= 100) {
      maxed = true
      canReset = false
      // el boton de reinicio recien se habilita tras 2.5s (para ver el musculo)
      setTimeout(() => { canReset = true }, 2500)
    }
  }

  function reset() {
    power = 0; clicks = 0; maxed = false; down = false; canReset = false
  }

  function onKey(e) {
    // la barra espaciadora / enter solo sirven para levantar, nunca para reiniciar
    if (!maxed && (e.code === 'Space' || e.code === 'Enter')) { e.preventDefault(); pump() }
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

        <!-- cabeza: expresiones reales del avatar (cambian segun el estado) -->
        {#if maxed}
          <circle cx={cx} cy="58" r="42" fill="none" stroke="#ffd24a" stroke-width="3" class="glow" />
        {/if}
        {#each faces as f}
          <image
            href="/faces/{f}.png"
            x="82" y="12" width="76" height="82"
            preserveAspectRatio="xMidYMax meet"
            opacity={faceImg === f ? 1 : 0}
            style="filter:{faceFilter}; transform:{faceTransform}; transform-box:fill-box; transform-origin:center bottom; transition: opacity .18s ease, transform .3s ease, filter .3s ease" />
        {/each}

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
    <div class="win-panel">
      <p class="win-text">🦾 ¡BESTIA MODE! Disfruta tus músculos 💪</p>
      {#if canReset}
        <button class="reset-mini" onclick={reset}>↺ Reiniciar</button>
      {:else}
        <p class="hint">Admirando el físico…</p>
      {/if}
    </div>
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
  .hint { color: #6f8296; font-size: 0.78rem; margin: 8px 0 0; }

  /* panel al ganar: el reinicio queda lejos del boton de levantar y llega tarde */
  .win-panel { text-align: center; }
  .win-text {
    margin: 4px 0 10px;
    font-size: 1.05rem;
    font-weight: 800;
    color: #ffd24a;
    text-shadow: 0 2px 6px rgba(0, 0, 0, 0.5);
  }
  .reset-mini {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 8px 18px;
    font-size: 0.9rem;
    font-weight: 600;
    color: #9fb2c4;
    background: transparent;
    border: 1.5px solid #3a4a5a;
    border-radius: 999px;
    cursor: pointer;
    transition: background 0.15s, color 0.15s, border-color 0.15s;
    animation: fadeUp 0.4s ease-out both;
  }
  .reset-mini:hover { color: #eef2f7; border-color: #6f8296; background: rgba(255, 255, 255, 0.05); }
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(8px); }
    to { opacity: 1; transform: translateY(0); }
  }

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
