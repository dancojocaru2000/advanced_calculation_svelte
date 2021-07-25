<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	export let paramNo;
	export let value;

	function keyup() {
		change()
	}

	function change() {
		dispatch('change', {
			paramNo: paramNo,
			value: value,
		})
	}
</script>

<main>
	<div class="card">
		<h3>Parameter {paramNo + 1}</h3>
		<input type="number" pattern="[0-9\.]+" name={`param${paramNo}`} bind:value on:keyup={keyup} on:change={change}>
	</div>
</main>

<style>
	.card {
		margin: 8px;
		border: 1px solid black;
		padding: 8px;
	}
</style>