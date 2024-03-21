<template>
	<input type="text" ref="input" @input="updateValue" @blur="formatValue" />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps(['modelValue', 'currency']),
	emit = defineEmits(['update:modelValue']),
	input = ref(null), // input element
	value = ref(props.modelValue);

const decimalSep = (1.1).toLocaleString().substring(1, 2),
	replaceRegex = new RegExp(`[^0-9${decimalSep}]`, 'g');

onMounted(formatValue);

function updateValue(event) {
	const str = event.target.value,
		num = parseFloat(str.replace(replaceRegex, '').replace(decimalSep, '.'));

	if (isNaN(num) || !isFinite(num))
		return

	value.value = num;	
	emit('update:modelValue', value.value);
}

function formatValue() {
	input.value.value = new Intl.NumberFormat([], { style: 'currency', currency: props.currency ? props.currency : 'usd' }).format(value.value);
}

watch(()=> props.currency, formatValue);
</script>