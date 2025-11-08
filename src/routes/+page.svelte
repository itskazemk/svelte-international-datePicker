<script>
	import DatePicker from '$lib/DatePicker.svelte';

	let date = $state();

	$inspect(111, date);
</script>

<DatePicker bind:date />
