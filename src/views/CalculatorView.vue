<script setup>
import { onMounted, ref, watch } from 'vue'
import VueApexCharts from 'vue3-apexcharts'

// costs
const years = ref(10),
	initialCost = ref(0.00),
	electricityCost = ref(0.00),
	internetCost = ref(0.00),
	maintenanceCost = ref(0.00),
	otherCost = ref(0.00),
	// storage node
	totalStorage = ref(20.00),
	// pricing
	storagePricing = ref(2.50),
	ingressPricing = ref(0.00),
	egressPricing = ref(0.00),
	collateralRatio = ref(2.00),
	// usage
	ingressUsage = ref(5.00),
	egressUsage = ref(0.00),

	chartSeries = ref([{ data: [] }, { data: [] }]),
	chartOpts = {
		chart: {
			type: 'line',
			animations: {
				enabled: true,
				easing: 'easeinout',
				speed: 800,
				animateGradually: {
					enabled: true,
					delay: 150
				},
				dynamicAnimation: {
					enabled: true,
					speed: 350
				}
			}
		},
		xaxis: {
			type: 'numeric',
			labels: {
				formatter: function (value) {
					return `Month ${Math.floor(value)}`;
				}
			}
		},
		yaxis: {
			labels: {
				formatter: function (value) {
					return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value);
				}
			}
		}
	};

function generateChart() {
	console.log('Generating chart...');
	let storageUsage = 0,
		totalRevenue = 0,
		totalCollateral = 0,
		totalCost = initialCost.value || 0;

	const series = [
			{
				name: 'Revenue',
				data: [],
				color: '#6bc37d'
			},
			{
				name: 'Risked Collateral',
				data: [],
				color: '#f6e05e'
			},
			{
				name: 'Cost',
				data: [],
				color: '#be5b5b'
			}
		],
		labels = [];

	for (let i = 0; i < years.value*12; i++) {
		let ingress = 0;
		if (storageUsage < totalStorage.value) {
			let ramp = 0.2 * i;
			if (ramp > 1) ramp = 1;

			ingress = ingressUsage.value * ramp;
			storageUsage += ingress;
		}

		totalRevenue += (storageUsage * storagePricing.value) + (ingress * ingressPricing.value) + (egressUsage.value * egressPricing.value),
			totalCollateral += (ingress * collateralRatio.value * storagePricing.value),
			totalCost += electricityCost.value + internetCost.value + maintenanceCost.value + otherCost.value;

		series[0].data.push(totalRevenue)
		series[1].data.push(totalCollateral);
		series[2].data.push(totalCost);
		labels.push(i+1);
	}
	chartSeries.value = series;
	chartOpts.labels = labels;
}

onMounted(async () => {
	await generateChart();
});

watch([years, initialCost, electricityCost, internetCost, maintenanceCost, otherCost, totalStorage, storagePricing, collateralRatio, ingressPricing, egressPricing, ingressUsage, egressUsage], generateChart);
</script>

<template>
	<main class="h-screen flex overflow-hidden">
		<form class="max-h-full p-4 bg-slate-300 overflow-y-auto">
			<div class="border-b border-gray-900/10 mb-4 pb-4 pt-4">
				<h2 class="text-base font-semibold leading-7 text-gray-900">Costs</h2>

				<label for="initialCost" class="block text-sm font-medium leading-6 text-gray-900">Initial</label>
				<input id="initialCost" type="text"
					class="form-input mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="initialCost">

				<label for="electricityCost" class="block text-sm font-medium leading-6 text-gray-900">Electricity
					(per month)</label>
				<input id="electricityCost" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="electricityCost">

				<label for="internetCost" class="block text-sm font-medium leading-6 text-gray-900">Internet (per
					month)</label>
				<input id="internetCost" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="internetCost">

				<label for="maintenanceCost" class="block text-sm font-medium leading-6 text-gray-900">Maintenance
					(per month)</label>
				<input id="maintenanceCost" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="maintenanceCost">

				<label for="otherCost" class="block text-sm font-medium leading-6 text-gray-900">Other (per
					month)</label>
				<input id="otherCost" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="otherCost">
			</div>
			<div class="border-b border-gray-900/10 mb-4 pb-4 pt-4">
				<h2 class="text-base font-semibold leading-7 text-gray-900">Storage Node</h2>
				<label for="totalStorage" class="block text-sm font-medium leading-6 text-gray-900">Total Storage
					(TB)</label>
				<input id="totalStorage" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="totalStorage">
			</div>
			<div class="border-b border-gray-900/10 mb-4 pb-4 pt-4">
				<h2 class="text-base font-semibold leading-7 text-gray-900">Pricing</h2>

				<label for="storagePrice" class="block text-sm font-medium leading-6 text-gray-900">Storage (TB/month)</label>
				<input id="storagePrice" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="storagePricing">
				
				<label for="collateralRatio" class="block text-sm font-medium leading-6 text-gray-900">Collateral ratio</label>
				<input id="collateralRatio" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="2.00" v-model.number="collateralRatio">

				<label for="ingressPrice" class="block text-sm font-medium leading-6 text-gray-900">Ingress (per
					TB)</label>
				<input id="ingressPrice" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="ingressPricing">

				<label for="egressPrice" class="block text-sm font-medium leading-6 text-gray-900">Egress (per
					TB)</label>
				<input id="egressPrice" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="egressPricing">
			</div>
			<div class="border-b border-gray-900/10 mb-4 pb-4 pt-4">
				<h2 class="text-base font-semibold leading-7 text-gray-900">Usage</h2>

				<label for="ingressUsage" class="block text-sm font-medium leading-6 text-gray-900">Ingress (TB/month)</label>
				<input id="ingressUsage" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="ingressUsage">

				<label for="egressUsage" class="block text-sm font-medium leading-6 text-gray-900">Egress (TB/month)</label>
				<input id="egressUsage" type="text"
					class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
					placehold="0.00" v-model.number="egressUsage">
			</div>
		</form>
		<VueApexCharts class="flex-1" width="100%" height="100%" :options="chartOpts" :series="chartSeries" />
	</main>
</template>
