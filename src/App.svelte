<script>
	export let name;
	$: pie = name;

	let mainHeigth = window.innerHeight;

	let viewStates = {
		'home': true,
		'entries': false,
		'exercises': false
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
		selectedEntries = entries.filter(entry => entry.exercise === exercisesToView).reverse();
	}

	const addExercise = () => {
		if (newExercise !== null && newExercise.length > 0) {
			exercises = [...exercises, newExercise];
			localStorage.setItem('exercises', JSON.stringify(exercises));
			newExercise = null;
		}
	}

	const addSet = () => {
		// get last and use it for the new values
		const last = newEntries[newEntries.length - 1];

		if (last && (parseInt(last.reps) > 0 === false || parseInt(last.weight) > 0 === false)) {
			alert('Make sure reps and weight are over 0');
			return;
		}

		if (last) {
			last.open = false;
		}

		newEntries = [...newEntries, {
			reps: last ? last.reps : null,
			weight: last ? last.weight : null,
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

		newEntries = [];

		localStorage.setItem('entries', JSON.stringify(entries));
	}

	const deleteEntry = (entry) => {
		console.log(entries.indexOf(entry));
		//entries = entries.splice(entries.indexOf(entry), 1);
		//localStorage.setItem('entries', JSON.stringify(entries));
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

	@keyframes opacity {
		from {
			opacity: 0;
		}

		to {
			opacity: 1;
		}
	}

	.main {
		/*min-height: 100vh;
		max-height: 100vh;*/
		overflow: hidden;
		height: auto;
		animation: opacity 0.7s ease-in;
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
		min-height: calc(100% - 60px);
		max-height: calc(100% - 60px);
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

	.menu-selected {
		color: #212121;
	}

	.view {
		margin-bottom: 10px;
		animation: opacity 0.5s ease-in;
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

<div class="main" style="height:{mainHeigth}px">
	<div class="menu">
		{#each Object.keys(viewStates) as viewState }
			<span class="menu-option {viewStates[viewState] ? 'menu-selected' : null}" on:click={() => selectScreen(viewState)}>
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

		{#if viewStates.exercises === true}
			<div class="view">
				<h2>Add Excercise</h2>
				<input bind:value={newExercise} />
				<button class="green-button" on:click={addExercise}>Add Exercise</button>

				<h2>Exercises</h2>
				<ul>
				{#each exercises as excerise}
					<li>{excerise}</li>
				{/each}
				</ul>
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
									<strong on:click={() => deleteEntry(entry)}>Delete</strong>
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