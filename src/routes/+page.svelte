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

	let chart = $state();

	$effect(() => {
		if (chart) {
			const seriesData = populationPerGeneration
				.map((gen, index) => {
					const gencount = gen.currentGenerationPopulation + (populationPerGeneration[index + 1]?.currentGenerationPopulation || 0) + (populationPerGeneration[index + 2]?.currentGenerationPopulation || 0);
					return gencount;
				})
				.filter((val) => val !== null)
				.reverse();

			chart.updateOptions({
				xaxis: {
					categories: populationPerGeneration
						.slice(-seriesData.length)
						.map((gen) => {
							const years = generations * birthAge;
							const curYears = gen.generation * birthAge;
							if (years >= 600) {
								if (curYears % 100 === 0) {
									return gen.generation * birthAge;
								} else {
									return "";
								}
							} else if (years >= 300) {
								if (curYears % 40 === 0) {
									return gen.generation * birthAge;
								} else {
									return "";
								}
							} else {
								return gen.generation * birthAge;
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
			},
			yaxis: {
				labels: {
					formatter: function (value) {
						return value.toLocaleString("de-DE", { maximumFractionDigits: 0 });
					},
				},
			},
			tooltip: {
				y: {
					formatter: function (value) {
						return value.toLocaleString("de-DE", { maximumFractionDigits: 0 });
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
		<CardHeader class="p-3">
			<CardTitle>Population Growth Calculator</CardTitle>
		</CardHeader>
		<CardContent class="p-3">
			<div class="grid grid-cols-1 md:grid-cols-3 gap-4">
				<div class="grid grid-cols-2 items-center space-x-2">
					<label for="kids" class="text-sm font-medium whitespace-nowrap">KIDS (per adult):</label>
					<Input type="number" id="kids" bind:value={kids} min="0" step="1" class="w-full" />
				</div>
				<div class="grid grid-cols-2 items-center space-x-2">
					<label for="generations" class="text-sm font-medium whitespace-nowrap">GENERATIONS:</label>
					<Input type="number" id="generations" bind:value={generations} min="1" step="1" class="w-full" />
				</div>
				<div class="grid grid-cols-2 items-center space-x-2">
					<label for="birthAge" class="text-sm font-medium whitespace-nowrap">AGE (when kids are born):</label>
					<Input type="number" id="birthAge" bind:value={birthAge} min="0" step="1" class="w-full" />
				</div>
			</div>
		</CardContent>
	</Card>

	<Card>
		<CardContent class="p-3">
			<p class="text-2xl font-bold text-center">
				The total population after <span class="text-blue-600">{(generations - 1) * birthAge}</span>
				years is
				<span class="text-green-600">{lastThreePopulation.toLocaleString("de-DE", { maximumFractionDigits: 0 })}</span>
				humans.
			</p>
			<p class="text-sm text-gray-500 text-center mt-2">- last three generations count -</p>
		</CardContent>
	</Card>

	<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
		<Card>
			<CardContent class="p-3">
				<div class="overflow-auto max-h-[50vh]">
					<Table>
						<TableHeader class="sticky top-0 bg-white z-10">
							<TableRow>
								<TableHead class="text-sm text-center">Generation</TableHead>
								<TableHead class="text-sm text-center">New humans per generation</TableHead>
								<TableHead class="text-sm text-center">Total humans ever born</TableHead>
							</TableRow>
						</TableHeader>
						<TableBody>
							{#each populationPerGeneration as { generation, currentGenerationPopulation, population }}
								<TableRow>
									<TableCell class="text-right">{generation}</TableCell>
									<TableCell class="text-right">{currentGenerationPopulation.toLocaleString("de-DE", { maximumFractionDigits: 0 })}</TableCell>
									<TableCell class="text-right">{population.toLocaleString("de-DE", { maximumFractionDigits: 0 })}</TableCell>
								</TableRow>
							{/each}
						</TableBody>
					</Table>
				</div>
			</CardContent>
		</Card>

		<Card>
			<CardHeader class="p-3">
				<CardTitle>Population Growth Chart (Logarithmic Scale)</CardTitle>
			</CardHeader>
			<CardContent class="p-3">
				<div id="chart"></div>
			</CardContent>
		</Card>
	</div>
</div>
