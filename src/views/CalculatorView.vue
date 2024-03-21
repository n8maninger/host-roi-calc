<script setup>
import { onMounted, ref, watch } from 'vue'
import VueApexCharts from 'vue3-apexcharts'
import CurrencyInput from '@/components/CurrencyInput.vue'

const years = ref(10),
	currency=ref('usd'),
	// costs
	initialCost = ref(400.00),
	maintenanceCost = ref(10.00),
	// storage node
	totalStorage = ref(20.00),
	// pricing
	storagePricing = ref(2.50),
	ingressPricing = ref(0.00),
	egressPricing = ref(0.00),
	collateralRatio = ref(2.00),
	// usage
	ingressUsage = ref(2.00),
	egressUsage = ref(0.00),

	chartSeries = ref([{ data: [] }, { data: [] }]),
	chartOpts = {
		chart: {
			height: '100%',
			width: '100%',
			type: 'line',
			toolbar: {
				show: false
			},
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
					return new Intl.NumberFormat([], { style: 'currency', currency: currency.value }).format(value);
				}
			}
		}
	};

function generateChart() {
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
			totalCost += maintenanceCost.value;

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

watch([years, currency, initialCost, maintenanceCost, totalStorage, storagePricing, collateralRatio, ingressPricing, egressPricing, ingressUsage, egressUsage], generateChart);
</script>

<template>
	<main class="md:flex md:flex-col h-screen">
		<div class="max-w-screen-md h-screen md:h-auto flex flex-col m-auto">
			<div class="flex-1 md:flex-none md:h-96">
				<VueApexCharts :options="chartOpts" height="100%" width="100%" :series="chartSeries" />
			</div>
			<form class="bg-zinc-300 p-2 md:rounded-md md:shadow-md">
				<label for="currencySelect" class="block text-sm font-medium leading-6 text-gray-900">Currency</label>
				<select id="currencySelect" v-model="currency" class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50">
					<option value="usd">USD</option>
					<option value="eur">EUR</option>
					<option value="chf">CHF</option>
					<option value="gbp">GBP</option>
					<option value="kwd">KWD</option>
					<option value="jpy">JPY</option>
				</select>
				<div class="border-b border-gray-900/10 mb-4 pb-4 gap-2">
					<h2 class="text-base font-semibold leading-7 text-gray-900">Costs</h2>
					<div class="flex gap-2">
						<div class="flex-1">
							<label for="initialCost" class="block text-sm font-medium leading-6 text-gray-900">Initial</label>
							<CurrencyInput :currency="currency" id="initialCost" type="text" class="form-input mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50" placehold="0.00" v-model.number="initialCost"/>
						</div>
						<div class="flex-1">
							<label for="maintenanceCost" class="block text-sm font-medium leading-6 text-gray-900">Maintenance (per month)</label>
							<CurrencyInput :currency="currency" id="maintenanceCost" type="text"
								class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
								placehold="0.00" v-model.number="maintenanceCost"/>
						</div>
					</div>
				</div>
				<div class="border-b border-gray-900/10 mb-4 pb-4 gap-2">
					<h2 class="text-base font-semibold leading-7 text-gray-900">Usage</h2>
					<div class="flex gap-2">
						<div class="flex-1">
							<label for="totalStorage" class="block text-sm font-medium leading-6 text-gray-900">Total Storage (TB)</label>
							<CurrencyInput :currency="currency" id="totalStorage" type="text"
							class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
							placehold="0.00" v-model.number="totalStorage"/>
						</div>
						<div class="flex-1">
								<label for="ingressUsage" class="block text-sm font-medium leading-6 text-gray-900">Ingress (TB/month)</label>
							<CurrencyInput :currency="currency" id="ingressUsage" type="text"
								class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
								placehold="0.00" v-model.number="ingressUsage"/>
						</div>
						<div class="flex-1">
							<label for="egressUsage" class="block text-sm font-medium leading-6 text-gray-900">Egress (TB/month)</label>
							<CurrencyInput :currency="currency" id="egressUsage" type="text"
								class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
								placehold="0.00" v-model.number="egressUsage"/>
						</div>
					</div>
				</div>
				<h2 class="text-base font-semibold leading-7 text-gray-900">Pricing</h2>
				<div class="flex gap-2">
					<div class="flex-1">
						<label for="storagePrice" class="block text-sm font-medium leading-6 text-gray-900">Storage (TB/month)</label>
						<CurrencyInput id="storagePrice" type="text"
							class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
							placehold="0.00" v-model.number="storagePricing"/>
					</div>
					<div class="flex-1">
						<label for="collateralRatio" class="block text-sm font-medium leading-6 text-gray-900">Collateral ratio</label>
						<CurrencyInput id="collateralRatio" type="text"
							class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
							placehold="2.00" v-model.number="collateralRatio"/>
					</div>
					<div class="flex-1">
						<label for="ingressPrice" class="block text-sm font-medium leading-6 text-gray-900">Ingress (per TB)</label>
						<CurrencyInput id="ingressPrice" type="text"
							class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
							placehold="0.00" v-model.number="ingressPricing"/>
					</div>
					<div class="flex-1">
						<label for="egressPrice" class="block text-sm font-medium leading-6 text-gray-900">Egress (per TB)</label>
						<CurrencyInput id="egressPrice" type="text"
							class="form-input mt-1 mb-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
							placehold="0.00" v-model.number="egressPricing"/>
					</div>
				</div>
			</form>
		</div>
	</main>
</template>
