<script>
  import { onMount } from "svelte";
  import { resolve } from "$app/paths";

  let violent_sentences = $state([]);

  let q = $state("");
  let actualQ = $state("");
  let loaded = $state(false);
  let acts = $state([]);
  let perps = $state([]);
  let victs = $state([]);

  let act = $state(null);

  const searches = [
    {
      q: "hamas-run|\\brun by hamas|led by hamas|hamas-led|hamas run|hamas led",
      label: "Hamas-run",
    },
    { q: "\\d+ killed|killed \\d+", label: "killed" },
    { q: "civilian", label: "civilian" },
    { q: "terror", label: "terror" },
  ];

  let filtered = $derived.by(() => {
    let toReturn = [...violent_sentences];

    if (actualQ) {
      const regex = new RegExp(actualQ, "i");
      toReturn = violent_sentences.filter(
        (sentence) => sentence.sentence.match(regex),
        // sentence.sentence.toLowerCase().includes(actualQ.toLowerCase()),
      );
    }

    if (act) {
      toReturn = toReturn.filter((sentence) => sentence.act === act);
    }
    return toReturn.sort((a, b) => a.sentence.localeCompare(b.sentence));
  });

  function search() {
    actualQ = q;
  }

  onMount(async () => {
    const response = await fetch(resolve("/violent_sentences.json"));
    const data = await response.json();
    violent_sentences = data;
    victs = Array.from(new Set(data.map((s) => s.victim))).sort();
    perps = Array.from(new Set(data.map((s) => s.perpetrator))).sort();
    console.log(perps);
    console.log(victs);
    acts = Array.from(new Set(data.map((s) => s.act))).sort();
    loaded = true;
  });
</script>

<div class="wrapper">
  {#if loaded}
    <div class="left">
      <!-- {#each acts as a} -->
      <!--   <div><a href="#" onclick={() => (act = a)}>{a}</a></div> -->
      <!-- {/each} -->
    </div>
    <div class="right">
      <form onsubmit={search}>
        <input type="text" placeholder="Search" bind:value={q} />
        <button type="submit">Search</button>
        <span> Total Results: {filtered.length}</span>
      </form>

      <div class="searches">
        {#each searches as searchItem}
          <button
            class="premade-search"
            onclick={() => {
              q = searchItem.q;
              search();
            }}
          >
            {searchItem.label}
          </button>
        {/each}
      </div>

      {#each filtered as sentence}
        <p>
          {sentence.sentence} <a href={sentence.url} target="_blank">↗</a>
          <!-- {sentence.passive_voice} / {sentence.act} -->
        </p>
      {/each}
    </div>
  {:else}
    <p>Loading...</p>
  {/if}
</div>

<style>
  .wrapper {
    max-width: 800px;
    margin: 0 auto;
    padding: 1rem;
    display: flex;
  }

  form {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
    align-items: center;
  }

  input[type="text"] {
    flex: 1;
    padding: 0.5rem;
    font-size: 1rem;
  }

  button {
    padding: 0.5rem 1rem;
    font-size: 1rem;
    cursor: pointer;
  }

  .premade-search {
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
    padding: 0;
    font-size: 0.8rem;
  }

  p {
    margin: 0.5rem 0;
    line-height: 1.5;
    border-bottom: 1px solid #ccc;
  }
</style>
