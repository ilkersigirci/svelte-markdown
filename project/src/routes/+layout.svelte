<script>
	import Header from '$lib/components/Header.svelte';
	import '$lib/styles/style.scss';
	import { fade } from 'svelte/transition';

	export let data;
</script>

<Header />

<!-- <main>
	<slot />
</main> -->

{#key data.currentRoute}
	<main in:fade={{ duration: 150, delay: 150 }} out:fade={{ duration: 150 }}>
		<slot />
	</main>
{/key}

<footer>Hello, I'm the footer.</footer>
