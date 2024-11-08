<script>
	import { Input } from "$lib/components/ui/input";
	import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "$lib/components/ui/table";
	import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
	import { onMount, onDestroy } from "svelte";
	import ApexCharts from "apexcharts";

	let start = $state(2);
	let kids = $state(2);
	let birthAge = $state(20);
	let generations = $state(1);

	let populationPerGeneration = $derived.by(() => {
		const result = [
			{
				generation: 1,
				population: start,
				currentGenerationPopulation: start,
			},
		];
		if (generations === 1) return result;

		for (let i = 1; i < generations; i++) {
			const currentGenerationPopulation = result[i - 1].currentGenerationPopulation * kids;
			const population = result[i - 1].population + currentGenerationPopulation;
			result.push({ generation: i + 1, population, currentGenerationPopulation });
		}
		return result.reverse();
	});

	let lastThreePopulation = $derived.by(() => {
		if (generations <= 3) {
			return populationPerGeneration[0].population;
		} else {
			return populationPerGeneration.slice(0, 3).reduce((sum, gen) => sum + gen.currentGenerationPopulation, 0);
		}
	});

	/**
	 * @type {ApexCharts}
	 */
	let chart = $state();

	$effect(() => {
		if (chart) {
			const seriesData = populationPerGeneration
				.map((gen, index) => {
					return gen.currentGenerationPopulation + (populationPerGeneration[index + 1]?.currentGenerationPopulation || 0) + (populationPerGeneration[index + 2]?.currentGenerationPopulation || 0);
				})
				.filter((val) => val !== null)
				.reverse();

			chart.updateOptions({
				xaxis: {
					categories: populationPerGeneration
						.slice(-seriesData.length)
						.map((gen) => {
							const years = generations*birthAge;
							const curYears = gen.generation*birthAge;
							if (years >= 600) {
								if (curYears%100 === 0) {
									return gen.generation*birthAge;
								} else {
									return '';
								}
							} else if (years >= 300) {
								if (curYears%40 === 0) {
									return gen.generation*birthAge;
								} else {
									return '';
								}
							} else {
								return gen.generation*birthAge;
							}
						})
						.reverse(),
				},
				series: [
					{
						name: "Population (Last 3 Generations)",
						data: seriesData,
					},
				],
			});
		}
	});

	onMount(() => {
		const options = {
			chart: {
				type: "line",
				height: 350,
			},
			series: [
				{
					name: "Population (Last 3 Generations)",
					data: [],
				},
			],
			xaxis: {
				categories: [],
				labels: {
					formatter: function (value) {
						return 0;
					},
				},
			},
			yaxis: {
				labels: {
					formatter: function (value) {
						return value.toLocaleString("en-US");
					},
				},
			},
			tooltip: {
				y: {
					formatter: function (value) {
						return value.toLocaleString("en-US");
					},
				},
			},
		};

		chart = new ApexCharts(document.querySelector("#chart"), options);
		chart.render();
	});

	onDestroy(() => {
		if (chart) {
			chart.destroy();
		}
	});
</script>

<div class="container mx-auto p-4 space-y-6">
	<Card>
		<CardHeader>
			<CardTitle>Population Growth Calculator</CardTitle>
		</CardHeader>
		<CardContent>
			<div class="grid grid-cols-1 md:grid-cols-3 gap-4">
				<div>
					<label for="kids" class="block text-sm font-medium mb-2">KIDS (per adult)</label>
					<Input type="number" id="kids" bind:value={kids} min="0" step="1" />
				</div>
				<div>
					<label for="generations" class="block text-sm font-medium mb-2">GENERATIONS</label>
					<Input type="number" id="generations" bind:value={generations} min="1" step="1" />
				</div>
				<div>
					<label for="birthAge" class="block text-sm font-medium mb-2">AGE (when kids are born)</label>
					<Input type="number" id="birthAge" bind:value={birthAge} min="0" step="1" />
				</div>
			</div>
		</CardContent>
	</Card>

	<Card>
		<CardContent>
			<p class="text-2xl font-bold text-center">
				The total population after <span class="text-blue-600">{(generations - 1) * birthAge}</span>
				years is
				<span class="text-green-600">{lastThreePopulation.toLocaleString("en-US")}</span>
				humans.
			</p>
			<p class="text-sm text-gray-500 text-center mt-2">- last three generations count -</p>
		</CardContent>
	</Card>

	<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
		<Card>
			<CardContent>
				<div class="overflow-auto max-h-[50vh]">
					<TableHeader class="sticky top-0 bg-white z-10">
						<TableRow>
							<TableHead>Generation</TableHead>
							<TableHead>New humans per generation</TableHead>
							<TableHead>Total humans ever born</TableHead>
						</TableRow>
					</TableHeader>
					<Table>
						<TableBody>
							{#each populationPerGeneration as { generation, currentGenerationPopulation, population }}
								<TableRow>
									<TableCell>{generation}</TableCell>
									<TableCell>{currentGenerationPopulation.toLocaleString("en-US")}</TableCell>
									<TableCell>{population.toLocaleString("en-US")}</TableCell>
								</TableRow>
							{/each}
						</TableBody>
					</Table>
				</div>
			</CardContent>
		</Card>
	
		<Card>
			<CardHeader>
				<CardTitle>Population Growth Chart</CardTitle>
			</CardHeader>
			<CardContent>
				<div id="chart"></div>
			</CardContent>
		</Card>
	</div>
</div>
