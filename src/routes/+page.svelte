<script lang="ts">
	import SimpleSelect from '$lib/SimpleSelect.svelte';

	let options = $state([{
		label: 'Option 1',
		value: 'option1'
	},
	{
		label: 'Option 2',
		value: 'option2'
	},
	{
		label: 'Option 3',
		value: 'option3'
	}]);
</script>
<div class="p-12 w-96">
<SimpleSelect label="Select something" {options} onchange={(value) => {
	console.debug('select ', value);
}} />
</div>