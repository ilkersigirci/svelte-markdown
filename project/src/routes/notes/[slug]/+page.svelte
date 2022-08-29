<script>
	export let data;
	const { title, date, Content } = data;
</script>

<article>
	<h1>{title}</h1>
	<p>Published slug: {date}</p>
	<!-- <svelte:component this={Content} /> -->
	<Content />
</article>
