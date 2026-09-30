<script>
  import { onMount } from "svelte";
  import { resolve } from "$app/paths";

  let allQuotes = $state([]);

  let q = $state("");
  let actualQ = $state("");
  let loaded = $state(false);
  let cats = $state([]);
  let cat = $state(null);

  let start = $state(0);
  let total = $state(200);
  let end = $derived(start + total);

  // const searches = [
  //   {
  //     q: "hamas-run|\\brun by hamas|led by hamas|hamas-led|hamas run|hamas led",
  //     label: "Hamas-run",
  //   },
  //   { q: "\\d+ killed|killed \\d+", label: "killed" },
  //   { q: "civilian", label: "civilian" },
  //   { q: "terror", label: "terror" },
  // ];

  let filtered = $derived.by(() => {
    let toReturn = [...allQuotes];

    if (actualQ && actualQ.trim() !== "") {
      const regex = new RegExp(actualQ, "i");
      toReturn = allQuotes.filter((sentence) => sentence.q.match(regex));
    }

    if (cat) {
      toReturn = toReturn.filter((sentence) => sentence.af_cat === cat);
    }
    // return toReturn;
    toReturn = toReturn.sort((a, b) => a.q.localeCompare(b.q));
    return toReturn.slice(start, end);
  });

  function search() {
    actualQ = q;
  }

  onMount(async () => {
    const response = await fetch(resolve("/quotes.json"));
    const data = await response.json();
    allQuotes = data;
    cats = Array.from(new Set(data.map((s) => s.af_cat))).sort();
    loaded = true;
  });

  $inspect(filtered);
</script>

{#if loaded}
  <div class="search">
    <input
      type="text"
      placeholder="Search..."
      bind:value={q}
      onkeydown={(e) => e.key === "Enter" && search()}
    />
    <button onclick={search}>Search</button>
    <p>Showing {filtered.length} of {allQuotes.length} quotes</p>
  </div>

  <div class="cats">
    {#each cats as c}
      <button onclick={() => (cat = c)}>{c}</button>
    {/each}
  </div>

  <div class="results">
    {#each filtered as quote}
      <div class="quote">
        <div class="q">
          {quote.q} <a href={quote.url} target="_blank">[link]</a>
        </div>
        <div class="q">{quote.s}, {quote.af}, {quote.af_cat}</div>
        <!-- {#if quote.s} -->
        <!--   <p class="source">Source: {quote.s}</p> -->
        <!-- {/if} -->
      </div>
    {/each}
  </div>
{:else}
  <p>Loading...</p>
{/if}

<style>
  .quote {
    margin-bottom: 1rem;
  }
</style>
