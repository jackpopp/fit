<script>
	export let name;
	$: pie = name;

	let exercises = [
		'Deadlift',
		'Bent over row',
		'Squat'
	];

	let option = exercises[0];

	let newEntries = [];

	let entries = [];

	const stored = localStorage.getItem('entries');

	if (stored) {
		entries = JSON.parse(stored);
	}

	const addSet = () => {

		// get prev and use it for the new values
		const prev = newEntries[newEntries.length - 1];

		if (prev) {
			newEntries[newEntries.length - 1].open = false;
		}

		newEntries = [...newEntries, {
			reps: prev ? prev.reps : null,
			weight: prev ? prev.weight : null,
			open: true
		}];
	}

	const addEntry = () => {
		if (newEntries.length === 0) return;

		const entry = {
			exercise: option,
			sets: newEntries,
			date: new Date()
		}
		entries = [...entries, entry];

		newEntries = [{
			reps: null,
			weight: null,
			open: false
		}];

		localStorage.setItem('entries', JSON.stringify(entries));
	}

	const createAverage = (amounts = []) => {
		const total = amounts.reduce((prev, curr) =>  parseInt(prev) + parseInt(curr) , 0);
		return (total/amounts.length).toFixed(2);
	}

	const focus = (node) => node.focus();
</script>

<style>
	* {
		font-family: 'Mandali', sans-serif;
		box-sizing: border-box;
		line-height: 1.25;
	}

	.main {
		background: #009e8f;
		min-height: 100vh;
		height: auto;
	}

	h1,h2 {
		margin: 0 0 10px 0;
	}

	.app {
		max-width: 400px;
		width: 100%;
		min-height: 100vh;
		height: auto;
		margin: 0 auto;
		border: 1px solid #005f56;
		background: white;
		padding: 10px;
		border-radius: 2px;
	}

	label {
		font-weight: 600;
		font-size: 16px;
	}

	select {
		padding: 0;
		margin: 10px 0;
		height: 40px;
		border-radius: 0px;
	}

	.blue-button {
		background: rgb(0,100,200);
		border: none;
		color: white;
	}

	.green-button {
		background: #009688;
		border: none;
		color: white;
	}

	ul {
		padding: 0;
		margin-bottom: 0;
	}

	li {
		list-style-type: none;
		border-bottom: 1px solid black;
		margin: 0 0 8px 0;
		padding: 0 0 8px 0;
	}

	li:last-child {
		border-bottom: 0;
		margin: 0;
		padding: 0;
	}
</style>

<div class="main">
	<div class="app">
		<h1>Fit Track</h1>
		<div>
			<h2>Add Entry</h2>

			<select bind:value={option}>
				{#each exercises as excerise}
					<option value={excerise}>{excerise}</option>
				{/each}
			</select>

			{#each newEntries as entry, i}
				<h3>Set {i + 1}</h3>
				{#if entry.open}
					<label>Reps</label>
					<input type=number bind:value={entry.reps} use:focus />

					<label>Weight</label>
					<input type=number bind:value={entry.weight}/>
				{/if}
			{/each}

			<div>
				<button class="blue-button" on:click={addSet}>Add Set</button>
				<button class="green-button" on:click={addEntry}>Add Entry</button>
			</div>

		</div>

		<div>
			<h2>Entries</h2>
			<ul>
				{#each entries as entry, i}
					<li>
						<strong>Exercise:</strong> {entry.exercise} <br />
						<strong>Avg Sets:</strong> {entry.sets.length} 
						<strong>Reps:</strong> {createAverage(entry.sets.map(set => set.reps))} 
						<strong>Avg Weight:</strong> {createAverage(entry.sets.map(set => set.weight))} 
					</li>
				{/each}
			</ul>
		</div>
	</div>
</div>