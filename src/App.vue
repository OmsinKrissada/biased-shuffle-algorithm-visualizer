<script setup lang="ts">
import { computed, ref } from "vue";

// onMounted(() => {
const expandBar = ref(true);
const hideZero = ref(true);
const useIdealLogic = ref(false);

let length = 10;
const array = ref(new Array(length).fill(0).map((e, i) => (e = i)));
const chosens = ref<number[]>([]);

const chances = ref<number[][]>(
	new Array<number[]>(length)
		.fill(new Array<number>(length).fill(0))
		.map((c, i) => c.map((cc, j) => i === j ? 1 : cc))
);
const maxChance = computed(() => Math.max(...chances.value.map(c => Math.max(...c))));

const switchingPosition = ref<number>();
const primePosition = ref<number>(0);

function reset() {
	array.value = new Array(length).fill(0).map((e, i) => (e = i));
	chosens.value = [];

	// chances.value.forEach(c => c.fill(0));
	// chances.value.forEach((c, i) => c.map((cc, j) => i === j ? 1 : cc));

	chances.value = new Array<number[]>(length)
		.fill(new Array<number>(length).fill(0))
		.map((c, i) => c.map((cc, j) => i === j ? 1 : cc));

	switchingPosition.value = undefined;
	primePosition.value = 0;
}
// });

function addMatrices(a: number[], b: number[]) {
	if (a.length !== b.length) throw 'matrix length not match';
	const result = new Array<number>(a.length);
	for (let i = 0; i < a.length; i++) {
		result[i] = a[i] + b[i];
	}
	return result;
}

function multiplyMatrixWithConstant(a: number[], constant: number) {
	// if (a.length !== b.length) throw 'matrix length not match';
	const result = new Array<number>(a.length);
	for (let i = 0; i < a.length; i++) {
		result[i] = a[i] * constant;
	}
	return result;
}

// const min = 1, max = 100;
function linearToLogarithmic(linearValue: number, min: number, max: number) {
	var value = Math.round(Math.pow((max - min) + 1, linearValue) + min - 1);

	if (value < min) {
		value = min;
	} else if (value > max) {
		value = max;
	}

	return value;
}

function doNext() {
	if (switchingPosition.value === undefined) {
		switchingPosition.value = Math.floor(Math.random() * (useIdealLogic.value ? length - primePosition.value : length));

		// chances logic -- start
		const totalChance = useIdealLogic.value ? length - primePosition.value : length;
		console.log(totalChance);

		const oldPrimeChances = [...chances.value[primePosition.value]];
		chances.value[primePosition.value].fill(0);
		chances.value[primePosition.value] = addMatrices(chances.value[primePosition.value], multiplyMatrixWithConstant(oldPrimeChances, 1 / totalChance));
		for (let position = length - 1; position >= length - totalChance; position--) {
			if (position === primePosition.value) continue;
			console.log(`${position} > ${length - totalChance}`);
			const oldPositionChances = [...chances.value[position]];
			// const newPositionChances = new Array<number>(length).fill(0);
			chances.value[position] = addMatrices(
				multiplyMatrixWithConstant(oldPositionChances, (totalChance - 1) / totalChance),
				multiplyMatrixWithConstant(oldPrimeChances, 1 / totalChance)
			);

			chances.value[primePosition.value] = addMatrices(chances.value[primePosition.value], multiplyMatrixWithConstant(oldPositionChances, 1 / totalChance));
			// chances[position] = newPositionChances.map((n, i) => {
			// 	const here = oldPositionChances[i];
			// 	const scan = oldPrimeChances[i];
			// 	return ((length - 1) / length * here) + (1 / length * scan);
			// });
		}
		// chances logic -- end
	} else {
		const temp = array.value[primePosition.value];
		array.value[primePosition.value] = array.value[switchingPosition.value];
		array.value[switchingPosition.value] = temp;
		chosens.value.push(switchingPosition.value);
		switchingPosition.value = undefined;

		primePosition.value++;
	}
}
</script>

<template>
	<main class="flex flex-col justify-center items-center min-h-screen bg-gray-200">
		<div class="mb-8">
			<div class="my-2">
				<button @click="useIdealLogic = false"
					:class="{ 'underline underline-offset-8 decoration-indigo-500 decoration-2': !useIdealLogic }"
					class="px-2 py-1 mr-2 border rounded"><span class="font-mono">Math.random() *
						length</span>
					logic</button>
				<button @click="useIdealLogic = true"
					:class="{ 'underline underline-offset-8 decoration-indigo-500 decoration-2': useIdealLogic }"
					class="px-2 py-1 border rounded"><span class="font-mono">Math.random() * (i +
						1)</span>
					logic</button>
			</div>
			<div>
				<label for="length">Element count</label>
				<input type="number" id="length" v-model="length" class="ml-2">
				<button class="px-2 py-1 ml-2 bg-indigo-500 text-white rounded" @click="reset">Reset Progress</button>
			</div>
			<div>
				<input type="checkbox" id="zoom" v-model="expandBar" class="mr-2">
				<label for="zoom">Zoom in as much as possible (keep scale)</label>
			</div>
			<div>
				<input type="checkbox" id="hide-zero" v-model="hideZero" class="mr-2">
				<label for="hide-zero">Hide zero chances</label>
			</div>
		</div>
		<div class="flex mb-6 overflow-scroll">
			<div v-for="(val, i) in array" :key="val" class="relative m-2">
				<TransitionGroup name="list">
					<div class="relative flex flex-col justify-center items-center w-20 h-20 ring-emerald-500 shadow-md rounded transition-all"
						:class="{
							'ring-4': switchingPosition === i,
							'bg-gray-300': chosens.includes(i),
							'bg-gray-100': !chosens.includes(i),
						}
							">
						<div v-if="i === primePosition"
							class="absolute top-0 right-0 translate-x-full text-center text-6xl text-rose-500 rounded-full">
							&larr;
						</div>
						<p class="absolute left-0 top-0 ml-1 font-semibold text-gray-400">
							{{ i }}
						</p>
						<p class="font-semibold text-6xl text-blue-500">
							{{ val }}
						</p>
					</div>
				</TransitionGroup>
				<div class="w-40 mt-4" :key="i">
					<TransitionGroup name="list">
						<p v-for="(c, j) in chances[i]" :key="j">
						<div v-if="!hideZero || c !== 0" class="relative my-2">
							<div class="absolute w-full h-full border border-gray-400 rounded">
								<div class="h-full bg-emerald-300/60 transition-[width] duration-500"
									:style="{ width: `${c / (expandBar ? maxChance : 1) * 100}%` }" />
							</div>
							<!-- <p>.</p> -->
							<p class="relative px-1">{{ j }}: {{ Math.round(c * 100 * 100) / 100 }}%</p>
							<!-- <p class="relative">{{ j }}: {{ Math.round(c * length * 100) / 100 }}/10</p> -->
						</div>
						</p>
					</TransitionGroup>
				</div>
			</div>
		</div>
		<div class="flex gap-4 ml-10 mb-6">
			<div v-for="(val, i) in array" :key="val">
				<!-- <div class="w-20">
					<p v-for="(c, j) in chances[i]">
					<div v-if="c !== 0" class="relative">
						<div class="absolute w-full h-full border border-black">
							<div></div>
						</div>
						<p>{{ j }}: {{ Math.round(c * length*100)/100 }}/10</p>
					</div>
					</p>
				</div> -->
			</div>
		</div>

		<button class="px-3 py-2 font-bold text-lg text-white rounded"
			:class="{ 'bg-rose-800': primePosition === length, 'bg-teal-500': primePosition !== length }" @click="doNext"
			:disabled="primePosition === length">
			{{ switchingPosition ? "Switch" : "Random" }}
		</button>
	</main>
</template>

<style scoped>
.list-move,
/* apply transition to moving elements */
.list-enter-active,
.list-leave-active {
	transition: all 500ms ease;
}

.list-enter-from,
.list-leave-to {
	opacity: 0;
	transform: translateX(30px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
	position: absolute;
}
</style>
