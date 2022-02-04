<script>
  import { goto } from '$app/navigation'
  // 9SKv4reG8488MeXs
  // WSqhn2n3E9cfa6aJ

  let mapId = 'WSqhn2n3E9cfa6aJ'

  const handleSubmit = () => {
    goto(`/maps/${mapId}`)
  }
</script>

<h1>Allmaps Here</h1>
<form on:submit|preventDefault={handleSubmit}>
  <input type="text" bind:value={mapId} />
  <button type="submit"> Show map ID </button>
</form>

<style>
  :global(html, body, #svelte) {
    box-sizing: border-box;
    margin: 0;
    height: 100%;
  }
  :global(#svelte) {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
</style>
