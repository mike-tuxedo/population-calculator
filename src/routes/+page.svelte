<script>
	import { Input } from "$lib/components/ui/input";
	import { Button } from "$lib/components/ui/button";
	import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "$lib/components/ui/table";
	import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
	import { Plus, Minus } from "lucide-svelte";;
	import { onMount, onDestroy } from "svelte";
	import ApexCharts from "apexcharts";
	import { translations } from "$lib/translations.js";

	let language = navigator.language;
	let start = $state(2);
	let kids = $state(2);
	let birthAge = $state(20);
	let generations = $state(1);

	function increment(variable) {
		if (variable === "kids") kids++;
		if (variable === "generations") generations++;
		if (variable === "birthAge") birthAge++;
	}

	function decrement(variable) {
		if (variable === "kids" && kids > 0) kids--;
		if (variable === "generations" && generations > 1) generations--;
		if (variable === "birthAge" && birthAge > 0) birthAge--;
	}

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
			let values = populationPerGeneration;
			let seriesData = populationPerGeneration
				.map((gen, index) => {
					const gencount = gen.currentGenerationPopulation + (populationPerGeneration[index + 1]?.currentGenerationPopulation || 0) + (populationPerGeneration[index + 2]?.currentGenerationPopulation || 0);
					return gencount;
				})
				.filter((val) => val !== null)
				.reverse();

			if (seriesData.length > 35) {
				seriesData.splice(0, 30);
				values.reverse().splice(0, 30);
				values = values.reverse();
			} else if (seriesData.length > 25) {
				seriesData.splice(0, 20);
				values.reverse().splice(0, 20);
				values = values.reverse();
			} else if (seriesData.length > 15) {
				seriesData.splice(0, 10);
				values.reverse().splice(0, 10)
				values = values.reverse();
			}
			console.log(seriesData, values);

			chart.updateOptions({
				xaxis: {
					categories: values
						.slice(-seriesData.length)
						.map((gen) => {
							// const years = generations * birthAge;
							// const curYears = gen.generation * birthAge;
							// if (years >= 600) {
							// 	if (curYears % 100 === 0) {
							// 		return gen.generation * birthAge;
							// 	} else {
							// 		return "";
							// 	}
							// } else if (years >= 300) {
							// 	if (curYears % 40 === 0) {
							// 		return gen.generation * birthAge;
							// 	} else {
							// 		return "";
							// 	}
							// } else {
							// 	return gen.generation * birthAge;
							// }
							return gen.generation * birthAge;
						})
						.reverse(),
				},
				series: [
					{
						name: "Population",
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
			colors: ["#f97316"],
			series: [
				{
					name: "Population",
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
	<h1 class="font-bold text-2xl">{translations['Population Growth Calculator'][language]}</h1>
	<Card>
		<CardContent class="p-3">
			<div class="grid grid-cols-1 lg:grid-cols-3 gap-y-4 gap-x-8">
				<div class="flex items-center space-x-2">
					<label for="kids" class="text-sm font-medium whitespace-nowrap" style="flex-grow: 1;">{translations['KIDS (per adult)'][language]}:</label>
					<div class="flex items-center">
						<Input type="number" id="kids" bind:value={kids} min="0" step="1" class="w-20" />
						<Button onclick={() => increment("kids")} class="ml-1 p-2"><Plus size={16} /></Button>
						<Button onclick={() => decrement("kids")} class="ml-1 p-2"><Minus size={16} /></Button>
					</div>
				</div>
				<div class="flex items-center space-x-2">
					<label for="generations" class="text-sm font-medium whitespace-nowrap" style="flex-grow: 1;">{translations['GENERATIONS'][language]}:</label>
					<div class="flex items-center">
						<Input type="number" id="generations" bind:value={generations} min="1" step="1" class="w-20" />
						<Button onclick={() => increment("generations")} class="ml-1 p-2"><Plus size={16} /></Button>
						<Button onclick={() => decrement("generations")} class="ml-1 p-2"><Minus size={16} /></Button>
					</div>
				</div>
				<div class="flex items-center space-x-2">
					<label for="birthAge" class="text-sm font-medium whitespace-nowrap" style="flex-grow: 1;">{translations['AGE (when kids are born)'][language]}:</label>
					<div class="flex items-center">
						<Input type="number" id="birthAge" bind:value={birthAge} min="0" step="1" class="w-20" />
						<Button onclick={() => increment("birthAge")} class="ml-1 p-2"><Plus size={16} /></Button>
						<Button onclick={() => decrement("birthAge")} class="ml-1 p-2"><Minus size={16} /></Button>
					</div>
				</div>
			</div>
		</CardContent>
	</Card>

	<Card>
		<CardContent class="p-3">
			<p class="text-2xl font-bold text-center">
				{translations['The total population after'][language]} <span class="text-primary">{(generations - 1) * birthAge}</span>
				{translations['years is'][language]}
				<span class="text-primary">{lastThreePopulation.toLocaleString("de-DE", { maximumFractionDigits: 0 })}</span>
				{translations['humans'][language]}.
			</p>
			<p class="text-sm text-gray-500 text-center mt-2">- {translations['last three generations count'][language]} -</p>
		</CardContent>
	</Card>

	<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
		<Card>
			<CardContent class="p-3">
				<div class="overflow-auto max-h-[50vh]">
					<TableHeader class="sticky top-0 bg-white z-10" style="display: block;">
						<TableRow class="flex justify-end">
							<TableHead class="text-sm text-right">Generation</TableHead>
							<TableHead class="text-sm text-right" style="flex-grow: 1;">{translations['New humans per generation'][language]}</TableHead>
							<TableHead class="text-sm text-right" style="flex-grow: 1;">{translations['Total humans ever born'][language]}</TableHead>
						</TableRow>
					</TableHeader>
					<Table>
						<TableBody>
							{#each populationPerGeneration as { generation, currentGenerationPopulation, population }}
								<TableRow class="flex justify-end">
									<TableCell class="text-right">{generation}</TableCell>
									<TableCell class="text-right" style="flex-grow: 1;">{currentGenerationPopulation.toLocaleString("de-DE", { maximumFractionDigits: 0 })}</TableCell>
									<TableCell class="text-right"style="flex-grow: 1;">{population.toLocaleString("de-DE", { maximumFractionDigits: 0 })}</TableCell>
								</TableRow>
							{/each}
						</TableBody>
					</Table>
				</div>
			</CardContent>
		</Card>

		<Card>
			<CardHeader class="p-3">
				<CardTitle>{translations['Population Growth Chart'][language]}</CardTitle>
			</CardHeader>
			<CardContent class="p-3">
				<div id="chart"></div>
			</CardContent>
		</Card>
	</div>
</div>
