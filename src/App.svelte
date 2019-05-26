<script>
	export let name;
	$: pie = name;

	let viewStates = {
		home: true,
		entries: false
	}

	let defaultExercises = [
		'Deadlift',
		'Bent over row',
		'Squat'
	];

	const storedExercises = localStorage.getItem('exercises');
	let exercises = defaultExercises;

	if (storedExercises) {
		const parsedStoredExercises = JSON.parse(storedExercises);
		exercises = parsedStoredExercises;
	}

	let option = exercises[0];
	
	let newExercise = null;

	let newEntries = [];

	let entries = [];

	const stored = localStorage.getItem('entries');

	if (stored) {
		entries = JSON.parse(stored);
	}

	let exercisesToView = exercises[0];
	let selectedEntries = [];
	$: {
		selectedEntries = entries.filter(entry => entry.exercise === exercisesToView);
	}

	const addExercise = () => {
		if (newExercise !== null && newExercise.length > 0) {
			exercises = [...exercises, newExercise];
			localStorage.setItem('exercises', JSON.stringify(exercises));
			newExercise = null;
		}
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

	const selectScreen = (viewState) => {
		let newStates = viewStates;
		Object.keys(newStates).forEach(state => newStates[state] = state === viewState);
		viewStates = newStates;
	}
</script>

<style>
	* {
		font-family: -apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Oxygen,Ubuntu,Fira Sans,Droid Sans,Helvetica Neue,sans-serif;
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
		font-weight: 300;
	}

	strong {
		font-weight: 600;
	}

	.app {
		max-width: 680px;
		width: 100%;
		min-height: calc(100vh - 60px);
		max-height: calc(100vh - 60px);
		overflow: scroll;
		height: auto;
		margin: 0 auto;
		background: white;
		padding: 10px;
		border-radius: 2px;
	}

	.menu {
		height: 60px;
    	background: #4DB6AC;
	}

	.menu-option {
	    padding: 20px;
    	display: inline-block;
		color: white;
		letter-spacing: 1px;
	}

	.view {
		margin-bottom: 10px;
	}

	label {
		font-weight: 600;
		font-size: 16px;
	}

	select {
		width: 100%;
		padding: 0;
		margin: 0 0 10px 0;
		height: 40px;
		border-radius: 0px;
	}

	input {
		width: 100%;
	}

	button {
		height: 40px;
    	padding: 10px;
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
	<div class="menu">
		{#each Object.keys(viewStates) as viewState }
			<span class="menu-option" on:click={() => selectScreen(viewState)}>
				{viewState[0].toUpperCase() + viewState.substr(1, viewState.length)}
			</span>
		{/each}
	</div>
	<div class="app">
		<h1>Fit Track</h1>

		{#if viewStates.home === true}
			<div class="view">
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
					<button class="green-button" on:click={addEntry}>Complete Entry</button>
				</div>

				<h2>Add Excercise</h2>
				<input bind:value={newExercise} />
				<button class="green-button" on:click={addExercise}>Add Exercise</button>

				{#if entries.length > 0}
					<div>
						<h2>Recent Entries</h2>
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
				{/if}

			</div>
		{/if}

		{#if viewStates.entries === true}
			<div class="view">
				<h2>View Entries</h2>

				<select bind:value={exercisesToView}>
					{#each exercises as excerise}
						<option value={excerise}>{excerise}</option>
					{/each}
				</select>

				{#if entries.length > 0}
					<div>
						<!--<h2>{exercisesToView} Entries</h2>-->
						{#if selectedEntries.length > 0}
						<ul>
							{#each selectedEntries as entry, i}
								<li>
									<strong>Exercise:</strong> {entry.exercise} <br />
									<strong>Avg Sets:</strong> {entry.sets.length} 
									<strong>Reps:</strong> {createAverage(entry.sets.map(set => set.reps))} 
									<strong>Avg Weight:</strong> {createAverage(entry.sets.map(set => set.weight))} <br />
									<strong>Date:</strong> {entry.date}
								</li>
							{/each}
						</ul>
						{:else}
							No entries
						{/if}
					</div>
				{/if}
			</div>
		{/if}

	</div>
</div>