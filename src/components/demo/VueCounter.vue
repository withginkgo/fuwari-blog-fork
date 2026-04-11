<script setup lang="ts">
import { computed, ref } from "vue";

const props = withDefaults(
	defineProps<{
		headline?: string;
		initialCount?: number;
	}>(),
	{
		headline: "Vue island ready",
		initialCount: 6,
	},
);

const count = ref(props.initialCount);

const summary = computed(() => {
	if (count.value === props.initialCount) {
		return "Hydrated with the server-provided initial state.";
	}

	if (count.value > props.initialCount) {
		return "Client-side reactivity is working after hydration.";
	}

	return "State changed locally inside the Vue component.";
});

function increment() {
	count.value += 1;
}

function decrement() {
	count.value -= 1;
}

function reset() {
	count.value = props.initialCount;
}
</script>

<template>
	<section class="vue-counter">
		<p class="vue-counter__eyebrow">Vue 3 Island</p>
		<h3>{{ headline }}</h3>
		<p class="vue-counter__summary">{{ summary }}</p>

		<div class="vue-counter__meter">
			<span class="vue-counter__label">Current count</span>
			<strong>{{ count }}</strong>
		</div>

		<div class="vue-counter__actions">
			<button type="button" @click="decrement">-1</button>
			<button type="button" @click="increment">+1</button>
			<button type="button" class="vue-counter__reset" @click="reset">
				Reset
			</button>
		</div>
	</section>
</template>

<style scoped>
.vue-counter {
	display: grid;
	gap: 1rem;
	padding: 1.25rem;
	border-radius: 1rem;
	border: 1px solid rgb(15 23 42 / 0.08);
	background:
		linear-gradient(160deg, rgb(255 255 255 / 0.96), rgb(248 250 252 / 0.9));
}

.vue-counter__eyebrow {
	margin: 0;
	font-size: 0.75rem;
	font-weight: 700;
	letter-spacing: 0.16em;
	text-transform: uppercase;
	color: var(--primary);
}

h3 {
	margin: 0;
	font-size: 1.4rem;
	line-height: 1.15;
	color: rgb(15 23 42 / 0.88);
}

.vue-counter__summary {
	margin: 0;
	line-height: 1.7;
	color: rgb(15 23 42 / 0.68);
}

.vue-counter__meter {
	display: flex;
	align-items: end;
	justify-content: space-between;
	gap: 1rem;
	padding: 1rem;
	border-radius: 0.9rem;
	background: rgb(15 23 42 / 0.05);
}

.vue-counter__label {
	font-size: 0.88rem;
	color: rgb(15 23 42 / 0.58);
}

strong {
	font-size: clamp(1.9rem, 4vw, 2.8rem);
	line-height: 1;
	letter-spacing: -0.05em;
	color: rgb(15 23 42 / 0.92);
}

.vue-counter__actions {
	display: flex;
	flex-wrap: wrap;
	gap: 0.75rem;
}

button {
	appearance: none;
	border: 0;
	padding: 0.75rem 1rem;
	border-radius: 999px;
	font: inherit;
	font-weight: 600;
	color: rgb(248 250 252 / 0.95);
	background: rgb(15 23 42 / 0.88);
	cursor: pointer;
	transition:
		transform 0.2s ease,
		background-color 0.2s ease;
}

button:hover {
	transform: translateY(-1px);
	background: rgb(15 23 42 / 0.96);
}

.vue-counter__reset {
	background: rgb(14 116 144 / 0.9);
}

.vue-counter__reset:hover {
	background: rgb(8 145 178 / 0.95);
}

:global(.dark) .vue-counter {
	border-color: rgb(255 255 255 / 0.08);
	background:
		linear-gradient(160deg, rgb(30 41 59 / 0.94), rgb(15 23 42 / 0.88));
}

:global(.dark) h3 {
	color: rgb(248 250 252 / 0.92);
}

:global(.dark) .vue-counter__summary {
	color: rgb(226 232 240 / 0.7);
}

:global(.dark) .vue-counter__meter {
	background: rgb(255 255 255 / 0.05);
}

:global(.dark) .vue-counter__label {
	color: rgb(226 232 240 / 0.56);
}

:global(.dark) strong {
	color: rgb(248 250 252 / 0.94);
}
</style>
