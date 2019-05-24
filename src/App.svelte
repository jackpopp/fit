<script>
	export let name;
	$: pie = name;

	let exercises = [
		'deadlift'
	];

	let option = exercises[0];

	let newEntries = [{
		reps: 0,
		amount: 0
	}];

	let entries = [];

	const addSet = () => {

		newEntries = [...newEntries, {
			reps: 0,
			amount: 0
		}];
	}

	const addEntry = () => {
		const entry = {
			exercise: option,
			sets: newEntries
		}
		entries = [...entries, entry];
	}
</script>

<style>
	h1 {
		color: purple;
	}
</style>

<h1>Fit Track {pie}!</h1>
<input bind:value={pie}>

<h2>Back</h2>
<h2>Add</h2>
<h2>View</h2>

<div>
	<h2>Add Excerise</h2>

	<label>Exercise</label>
	<select bind:value={option}>
		{#each exercises as excerise}
			<option value={excerise}>{excerise}</option>
		{/each}
	</select>

	{#each newEntries as entry, i}
		<h3>Set {i += 1}</h3>
		<label>Reps</label>
		<input bind:value={entry.reps}/>

		<label>Amount</label>
		<input bind:value={entry.amount}/>
	{/each}

	<div>
		<button on:click={addSet}>Add Set</button>
	</div>
	<div>
		<button on:click={newEntries}>Add Entry</button>
	</div>

</div>

<div>
	<h2>Add Exercise</h2>
</div>

<div>
	<h2>View Entries</h2>
	{#each entries as entry, i}
		Exercise: {entry.exercise} <br />
		Sets: {entry.sets.length}
	{/each}
</div>