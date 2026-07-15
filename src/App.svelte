<script>
  // El mini-juego queda oculto por ahora — para mostrarlo de nuevo,
  // descomenta la importacion y usa <Gym /> en lugar del texto.
  // import Gym from './lib/Gym.svelte'
</script>

<main>
  <h1>Javi te amo</h1>
</main>

<style>
  main {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    background: #000;
  }

  h1 {
    color: #fff;
    font-family: system-ui, sans-serif;
    font-weight: 600;
    letter-spacing: 0.02em;
    font-size: clamp(2rem, 8vw, 4rem);
    text-align: center;
  }
</style>
