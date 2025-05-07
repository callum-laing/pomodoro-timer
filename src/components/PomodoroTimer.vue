<script setup>
import { ref, computed } from 'vue';

const timeLeft = ref(1500);
const intervalId = ref(null);
const mode = ref('work');
const isRunning = ref(false);
const wasPaused = ref(false);

const durations = {
	work: 1500,
	short: 300,
	long: 900
};

function setMode(newMode) {
	stopTimer();
	mode.value = newMode;
	timeLeft.value = durations[newMode];
	isRunning.value = false;
}

const timer = computed(() => {
	const minutes = Math.floor(timeLeft.value / 60);
	const seconds = timeLeft.value % 60;
	return `${minutes}:${seconds.toString().padStart(2, '0')}`;
});

const modeLabel = computed(() => {
	if (mode.value === 'work') return 'Time to focus!';
	if (mode.value === 'short') return 'Quick break time!';
	return 'Take a long break!';
});

function startTimer() {
	intervalId.value = setInterval(() => {
		if (timeLeft.value > 0) {
			timeLeft.value--;
		} else {
			if (mode.value === 'work') {
				mode.value = 'break';
				timeLeft.value = 300;
			} else {
				stopTimer();
			}
		}
	}, 1000);
}

function toggleTimer() {
	if (isRunning.value) {
		stopTimer();
		isRunning.value = false;
		wasPaused.value = true;
	} else if (wasPaused.value) {
		startTimer();
		isRunning.value = true;
		wasPaused.value = false;
	} else {
		startTimer();
		isRunning.value = true;
	}
}

function stopTimer() {
	clearInterval(intervalId.value);
}

function resetTimer() {
	clearInterval(intervalId.value);
	isRunning.value = false;
	wasPaused.value = false;
	timeLeft.value = durations[mode.value];
}

const currentColor = computed(() => {
	switch (mode.value) {
		case 'work':
			return '#e63946';
		case 'short':
			return '#2a9d8f';
		case 'long':
			return '#457b9d';
		default:
			return '#ccc';
	}
});

const glowButtonStyle = computed(() => ({
	borderColor: currentColor.value,
	color: currentColor.value,
	boxShadow: `0 0 10px ${currentColor.value}`
}));
</script>

<template>
	<div class="timerBox">
		<div class="mode-buttons btnContainer">
			<button class="btnWork" :class="{ active: mode === 'work' }" @click="setMode('work')">
				Pomodoro
			</button>
			<button class="btnShort" :class="{ active: mode === 'short' }" @click="setMode('short')">
				Short Break
			</button>
			<button class="btnLong" :class="{ active: mode === 'long' }" @click="setMode('long')">
				Long Break
			</button>
		</div>

		<p class="timer">{{ timer }}</p>

		<div class="btnContainer">
			<button class="btnAction" @click="toggleTimer" :style="glowButtonStyle">
				{{ isRunning ? 'Pause' : wasPaused ? 'Continue' : 'Start' }}
			</button>
			<button
				class="btnAction"
				v-if="isRunning || wasPaused"
				@click="resetTimer"
				:style="glowButtonStyle"
			>
				Reset
			</button>
		</div>
		<p class="mode">{{ modeLabel }}</p>
	</div>
</template>

<style lang="scss" scoped>
.timerBox {
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	font-size: 2em;

	.timer {
		font-size: 4em;
	}
}

.active {
	border-color: currentColor;
	color: currentColor;
	box-shadow: 0 0 6px currentColor;
}

.btnContainer {
	button {
		border-radius: 5px;
		padding: 2px 10px;
		margin: 10px;
		background-color: transparent;
		font-weight: bold;
	}
	button:hover {
		cursor: pointer;
	}
}

.btnWork {
	border: 2px solid #e63946;
}

.btnShort {
	border: 2px solid #2a9d8f;
}

.btnLong {
	border: 2px solid #457b9d;
}

.timerBox {
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	font-size: 2em;
	padding: 1rem;

	.timer {
		font-size: 4em;
		margin: 1rem 0;
	}

	.mode {
		text-align: center;
		margin-top: 1rem;
		font-size: 1em;
	}
}

.btnContainer {
	display: flex;
	flex-wrap: wrap;
	justify-content: center;
	gap: 10px;

	button {
		border-radius: 5px;
		padding: 10px 20px;
		margin: 5px;
		background-color: transparent;
		font-weight: bold;
		font-size: 1em;
		min-width: 100px;
	}
}

@media (max-width: 380px) {
	.timerBox {
		font-size: 0.9em;

		.timer {
			font-size: 5em;
		}
		.mode {
			font-size: 2em;
		}
	}

	.btnContainer {
		flex-wrap: nowrap;
		button {
			padding: 5px;
		}
	}
}

@media (min-width: 390px) and (max-width: 768px) {
	.timerBox {
		font-size: 0.9em;

		.timer {
			font-size: 6em;
		}
		.mode {
			font-size: 2rem;
		}
	}

	.btnContainer {
		flex-wrap: nowrap;
		button {
			padding: 5px;
		}
	}
}
</style>
