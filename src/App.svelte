<script>
	import { onMount } from 'svelte';
	import Chart from './Chart.svelte';

	let forces = [];
	let force = '';
	let crimeBreakdown;
	let loading = false;
	const FORCE_URL = 'https://data.police.uk/api/forces';
	$: CRIME_URL = `https://data.police.uk/api/crimes-no-location?category=all-crime&force=${force}`;
	
	onMount(() => {
		fetch(FORCE_URL)
			.then(res => res.json())
			.then(json => 
				forces = json
			);
	});

	const resetCrimeBreakdown = () => {
		crimeBreakdown = undefined;
	};

	const handleSubmit = () => {
		loading = true;

		fetch(CRIME_URL)
			.then(res => res.json())
			.then(crimes => {
				loading = false;
				
				crimeBreakdown = (crimes.length)
					? crimes.reduce((accumulator, crime) => {
							const crimeCategory = accumulator[crime.category];
	
							accumulator[crime.category] = crimeCategory
								? crimeCategory + 1
								: 1;
	
							return accumulator;
							},
							{}
						)
					: crimeBreakdown = null;
			});
	};
</script>

<div>
	<h1>Crimes of the UK</h1>
	<p>
		This month's location-agnostic crimes, filtered by police force. Data from <a href="https://data.police.uk">https://data.police.uk</a>
	</p>
	<form on:submit|preventDefault={handleSubmit}>
		<select bind:value={force} on:change={resetCrimeBreakdown}>
			<option selected hidden>
				Choose a police force
			</option>
			{#each forces as { id, name }}
				<option value={id}>{name}</option>
			{/each}
		</select>

		<button type="submit" disabled={!force}>
			Search
		</button>
	</form>

	{#if loading}
		<p>loading...</p>
	{/if}

	{#if crimeBreakdown === null}
		No crime data for {force}
	{:else}
		<Chart {crimeBreakdown} />
	{/if}
</div>