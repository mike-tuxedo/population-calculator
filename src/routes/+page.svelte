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
			console.log(result[i - 1].population);
			const population = result[i - 1].population + currentGenerationPopulation;
			result.push({ population, currentGenerationPopulation });
		}
		return result;
	});

	$effect(() => {
		console.log('populationPerGeneration changed', populationPerGeneration.length);
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

	<div class="tablewrapper text-center w-full px-6 mt-4" style="max-height: calc(100vh - 420px); overflow: auto;">
		<table class="w-full">
			<tbody>
				<tr class="sticky top-0" style="background: #fff">
					<th class="p-2">Generation</th>
					<th class="p-2">Actual population</th>
					<th class="p-2">Total population</th>
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
		{populationPerGeneration[populationPerGeneration.length - 1].population.toLocaleString(
			'de-DE',
			{ maximumFractionDigits: 0 }
		)} humans in {(generations - 1) * birthAge} years
	</p>
</div>

<style>
	:global(body) {
		background: #fff;
	}
</style>
