<script>
	import {onMount} from 'svelte';
	import Chart from 'chart.js/auto'
	import * as graphs from 'chartjs-chart-graph';
	import AutoComplete from "simple-svelte-autocomplete";
	import ChartDataLabels from 'chartjs-plugin-datalabels';

	let canvas;
	function createChart(nodes, edges, id, type, orientation) {
		if (canvas)
			canvas.destroy();
		canvas = new Chart(document.getElementById(id).getContext("2d"), {
			type,
			data: {
				labels: nodes.map((d) => d.name),
				datasets: [{
					pointBackgroundColor: 'red',
					pointRadius: 20,
					pointHoverRadius: 25,
					data: nodes.map((d) => Object.assign({}, d)),
					edges: edges
				}]
			},
			plugins: [ChartDataLabels],
			options: {
				tree: { orientation},
				layout: { padding: { left: 5, top: 5, bottom: 5, right: 5 } },
				plugins: {
					legend: { display: false },
					tooltip: { enabled: false },
					datalabels: {
						color: 'blue',
						formatter: function(value, context) { return value.name; },
						align: 'bottom',
						offset: 25,
						font: {size: 15}
					}
				}
			}
		});
	}

	// function randRange(min, max)
	// {
	// 	return (Math.random() * (max - min) + min)
	// }

	let requisites = [];
	let courses = [];
	let selectedCourse;
	$: selectedCourse, updateMap();

	function updateMap() {
		if (requisites.length < 1)
			return
		let nodes = [], edges = [];
		const prerequisites = requisites[selectedCourse].prerequisites;
		nodes.push({ "name":selectedCourse, "x":0, "y":0 });
		prerequisites.forEach(prereq_dict => {
			prereq_dict[Object.keys(prereq_dict)[0]].forEach(prereq => {
				const angle = Math.random()*Math.PI*2;
				edges.push({ "source": 0, "target": nodes.length });
				nodes.push({ "name":prereq, "x":Math.cos(angle)*0.7, "y":Math.sin(angle)*0.7 });
				// nodes.push({ "name":prereq, "x":randRange(-0.8, 0.8), "y":randRange(-0.8, 0.8) });
			})
		})
		console.log(nodes);
		console.log(edges);
		// createChart(requisites.nodes, requisites.edges, 'dh', 'dendogram', 'horizontal');
		createChart(nodes, edges, 'dh', 'dendogram', 'vertical');
	}

	onMount(async () => {
		fetch('requisite.json')
		.then(response => response.json())
		.then(jsondata => {
			requisites = jsondata;
			courses = Object.keys(jsondata);
			selectedCourse = courses[Math.floor(Math.random()*courses.length)];
			console.log(requisites)
			updateMap();
		});
	});
</script>

<main>
	<AutoComplete items={courses} bind:selectedItem={selectedCourse} maxItemsToShowInList={3}/>
	<canvas id="dh"></canvas>
</main>

<style>
	main {
		text-align: center;
		padding: 0;
		width: 100%;
		height: 100%;
		margin: 0 0;
		overflow: hidden;
	}

	canvas {
		max-height: 100%;
		max-width: 100%;
	}
</style>