<script>
	let start = $state(2);
	let kids = $state(2);
	let birthAge = $state(20);
	let generations = $state(1);
	let populationPerGeneration = $derived.by(() => {
		const result = [
			{
				population: start,
				currentGenerationPopulation: start
			}
		];
		if (generations === 1) return result;

		for (let i = 1; i < generations; i++) {
			const currentGenerationPopulation = result[i - 1].currentGenerationPopulation * kids;
			const population = result[i - 1].population + currentGenerationPopulation;
			result.push({ population, currentGenerationPopulation });
		}
		return result;
	});
	let lastThreePopulation = $derived.by(() => {
		if (generations === 1) {
			return start;
		} else if (generations === 2) {
			return start + populationPerGeneration[generations - 1].currentGenerationPopulation;
		} else {
			console.log(populationPerGeneration[generations - 1].currentGenerationPopulation, populationPerGeneration[generations - 2].currentGenerationPopulation, populationPerGeneration[generations - 3].currentGenerationPopulation);
			return populationPerGeneration[generations - 1].currentGenerationPopulation + populationPerGeneration[generations - 2].currentGenerationPopulation + populationPerGeneration[generations - 3].currentGenerationPopulation;
		}

	});


	$effect(() => {
		console.log('Generations', populationPerGeneration.length);
		console.log('Total Population', lastThreePopulation);
	});
</script>

<div class="max-w-3xl mx-auto">
	<div class="mt-4 grid grid-cols-1 gap-3 sm:grid-cols-3 px-6">
		<div>
			<label for="first-name" class="block text-sm/6 font-medium">KIDS (per adult)</label>
			<div class="mt-2">
				<input
					type="number"
					bind:value={kids}
					class="block w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm/6"
				/>
			</div>
		</div>
		<div>
			<label for="last-name" class="block text-sm/6 font-medium">GENERATIONS</label>
			<div class="mt-2">
				<input
					type="number"
					bind:value={generations}
					class="block w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm/6"
				/>
			</div>
		</div>
		<div>
			<label for="last-name" class="block text-sm/6 font-medium">AGE (when kids are born)</label>
			<div class="mt-2">
				<input
					type="number"
					bind:value={birthAge}
					class="block w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm/6"
				/>
			</div>
		</div>
	</div>

	<div
		class="tablewrapper text-center w-full px-6 mt-4"
		style="max-height: calc(100vh - 460px); overflow: auto;"
	>
		<table class="w-full">
			<tbody>
				<tr class="sticky top-0" style="background: #fff">
					<th class="p-2">Generation</th>
					<th class="p-2">New humans per generation</th>
					<th class="p-2">Total humans ever born</th>
				</tr>
				{#each populationPerGeneration.reverse() as population, i}
					<tr>
						<td class="p-2">{generations - i}</td>
						<td class="p-2"
							>{population.currentGenerationPopulation.toLocaleString('de-DE', {
								maximumFractionDigits: 0
							})}</td
						>
						<td class="p-2"
							>{population.population.toLocaleString('de-DE', { maximumFractionDigits: 0 })}</td
						>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>

	<p class="w-full text-3xl text-center font-bold p-6" style="overflow-wrap: break-word;">
		The total population after {(generations - 1)*birthAge} years is {lastThreePopulation.toLocaleString('de-DE', { maximumFractionDigits: 0 })} humans.
		<span class="block text-sm font-normal text-gray-500 mt-2">- last three generations count -</span>
	</p>
</div>

<style>
	:global(body) {
		background: #fff;
	}
</style>
