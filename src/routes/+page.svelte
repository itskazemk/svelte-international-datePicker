<script>
	import DatePicker from '$lib/DatePicker.svelte';

	let date = $state();

	$inspect(111, date);
</script>

<div>
	<DatePicker bind:date />

	<input />
</div>

<style>
	div {
		height: 500px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-around;
		align-content: space-between;
	}
</style>
