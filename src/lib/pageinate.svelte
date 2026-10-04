<script>
  let {
    page = $bindable(1),
    totalItems,
    perPage = 10,
    siblings = 1,
    onchange,
    label = "Pagination",
  } = $props();

  const totalPages = $derived(Math.max(1, Math.ceil(totalItems / perPage)));

  // Keep page in range if totalItems or perPage changes underneath us.
  $effect(() => {
    if (page > totalPages) page = totalPages;
    if (page < 1) page = 1;
  });

  const pages = $derived.by(() => {
    // first + last + current + siblings on both sides + two gaps
    const slots = siblings * 2 + 5;
    if (totalPages <= slots) {
      return Array.from({ length: totalPages }, (_, i) => i + 1);
    }

    const left = Math.max(page - siblings, 1);
    const right = Math.min(page + siblings, totalPages);
    const showLeftGap = left > 2;
    const showRightGap = right < totalPages - 1;
    const edgeCount = siblings * 2 + 3;

    if (!showLeftGap && showRightGap) {
      return [...range(1, edgeCount), "gap", totalPages];
    }
    if (showLeftGap && !showRightGap) {
      return [1, "gap", ...range(totalPages - edgeCount + 1, totalPages)];
    }
    return [1, "gap", ...range(left, right), "gap", totalPages];
  });

  function range(start, end) {
    return Array.from({ length: end - start + 1 }, (_, i) => start + i);
  }

  function goTo(target) {
    const next = Math.min(Math.max(target, 1), totalPages);
    if (next === page) return;
    page = next;
    onchange?.(next);
  }
</script>

{#if totalPages > 1}
  <nav aria-label={label} class="pagination">
    <button
      type="button"
      class="step"
      onclick={() => goTo(page - 1)}
      disabled={page === 1}
      aria-label="Previous page"
    >
      Previous
    </button>

    <ul>
      {#each pages as item, i (item === "gap" ? `gap-${i}` : item)}
        <li>
          {#if item === "gap"}
            <span class="gap" aria-hidden="true">…</span>
          {:else}
            <button
              type="button"
              class="num"
              class:active={item === page}
              aria-current={item === page ? "page" : undefined}
              aria-label="Page {item}"
              onclick={() => goTo(item)}
            >
              {item}
            </button>
          {/if}
        </li>
      {/each}
    </ul>

    <button
      type="button"
      class="step"
      onclick={() => goTo(page + 1)}
      disabled={page === totalPages}
      aria-label="Next page"
    >
      Next
    </button>
  </nav>
{/if}

<style>
  /* Override these custom properties from a parent to theme the component. */
  .pagination {
    --pg-accent: var(--pagination-accent, #2f5d8a);
    --pg-text: var(--pagination-text, currentColor);
    --pg-border: var(--pagination-border, #c9ced6);
    --pg-radius: var(--pagination-radius, 6px);

    display: flex;
    align-items: center;
    gap: 0.5rem;
    flex-wrap: wrap;
    font: inherit;
    color: var(--pg-text);
  }

  ul {
    display: flex;
    gap: 0.25rem;
    list-style: none;
    margin: 0;
    padding: 0;
  }

  button {
    font: inherit;
    color: inherit;
    background: transparent;
    border: 1px solid var(--pg-border);
    border-radius: var(--pg-radius);
    cursor: pointer;
    min-height: 2.25rem;
  }

  .num {
    min-width: 2.25rem;
    padding: 0 0.5rem;
    font-variant-numeric: tabular-nums;
  }

  .step {
    padding: 0 0.85rem;
  }

  button:hover:not(:disabled):not(.active) {
    border-color: var(--pg-accent);
  }

  button:focus-visible {
    outline: 2px solid var(--pg-accent);
    outline-offset: 2px;
  }

  button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  .active {
    background: var(--pg-accent);
    border-color: var(--pg-accent);
    color: #fff;
    cursor: default;
  }

  .gap {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 2.25rem;
    min-height: 2.25rem;
    opacity: 0.6;
  }
</style>
