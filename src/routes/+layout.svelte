<script>
	import favicon from "$lib/assets/favicon.svg";
	import { resolve } from "$app/paths";

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<nav>
	<a href={resolve("/")}>Home</a>
	<a href={resolve("/sentences")}>Sentences</a>
	<a href={resolve("/quotes")}>Quotations</a>
</nav>
{@render children()}
