<script>
	import { onMount } from 'svelte';

	let forces = [];
	let force = '';
	let crimeBreakdown = {};
	const FORCE_URL = 'https://data.police.uk/api/forces';
	$: CRIME_URL = `https://data.police.uk/api/crimes-no-location?category=all-crime&force=${force}`;
	
	onMount(() => {
		fetch(FORCE_URL)
			.then(res => res.json())
			.then(json => 
				forces = json
			);
	});

	const handleSubmit = () => {
		fetch(CRIME_URL)
			.then(res => res.json())
			.then(crimes => {
				crimeBreakdown = crimes.reduce(
					(accumulator, crime) => {
						const crimeCategory = accumulator[crime.category];

						accumulator[crime.category] = crimeCategory
							? crimeCategory + 1
							: 1;

						return accumulator;
					},
					{}
				);
			});
	};
</script>

<div>
	<h1>Crimes of the UK</h1>
	<form on:submit|preventDefault={handleSubmit}>
		<select bind:value={force}>
			<option selected hidden>
				Choose a police force
			</option>
			{#each forces as force}
				<option value={force.id}>{force.name}</option>
			{/each}
		</select>

		<button type="submit" disabled={!force}>
			Submit
		</button>
	</form>
</div>