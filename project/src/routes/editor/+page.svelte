<script>
	import { Editor } from 'bytemd';
	import gfm from '@bytemd/plugin-gfm';
	import 'bytemd/dist/index.css';

	let value = '';
	let plugins = [gfm()];
</script>

<div class="container">
	<Editor
		{value}
		on:change={(e) => {
			value = e.detail.value;
		}}
		{plugins}
	/>
	<p>Raw input</p>
	<p>{value}</p>
</div>

<style>
	.container {
		max-width: 1200px;
		margin: 0 auto;
	}
</style>
