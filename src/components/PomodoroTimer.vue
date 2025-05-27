<script setup>
import { ref, computed } from 'vue';

// Time states (in seconds)
const durations = {
	work: 1500, // 25 minutes
	short: 300, // 5 minutes
	long: 900 // 15 minutes
};

const timeLeft = ref(durations.work);
const mode = ref('work');
const isRunning = ref(false);
const wasPaused = ref(false);
const pomodoroCount = ref(0);
const intervalId = ref(null);

// Compute time as MM:SS string
const timer = computed(() => {
	const minutes = Math.floor(timeLeft.value / 60);
	const seconds = timeLeft.value % 60;
	return `${minutes}:${seconds.toString().padStart(2, '0')}`;
});

// Label based on current mode
const modeLabel = computed(() => {
	switch (mode.value) {
		case 'work':
			return 'Time to focus!';
		case 'short':
			return 'Quick break time!';
		case 'long':
			return 'Take a long break!';
		default:
			return '';
	}
});

// Colors for visual feedback
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

function setMode(newMode) {
	stopTimer();
	mode.value = newMode;
	timeLeft.value = durations[newMode];
	isRunning.value = false;
	wasPaused.value = false;
}

function toggleTimer() {
	if (isRunning.value) {
		stopTimer();
		wasPaused.value = true;
		isRunning.value = false;
	} else {
		startTimer();
		wasPaused.value = false;
		isRunning.value = true;
	}
}

function startTimer() {
	intervalId.value = setInterval(() => {
		if (timeLeft.value > 0) {
			timeLeft.value--;
		} else {
			handleCycle();
		}
	}, 1000);
}

function stopTimer() {
	clearInterval(intervalId.value);
}

function resetTimer() {
	stopTimer();
	timeLeft.value = durations[mode.value];
	isRunning.value = false;
	wasPaused.value = false;
}

function handleCycle() {
	stopTimer();
	isRunning.value = false;

	if (mode.value === 'work') {
		pomodoroCount.value++;
		if (pomodoroCount.value % 4 === 0) {
			setMode('long');
		} else {
			setMode('short');
		}
	} else {
		setMode('work');
	}

	startTimer();
	isRunning.value = true;
}
</script>

<template>
	<div class="timerBox">
		<div class="mode-buttons btnContainer">
			<button class="btn btnWork" :class="{ active: mode === 'work' }" @click="setMode('work')">
				Pomodoro
			</button>
			<button class="btn btnShort" :class="{ active: mode === 'short' }" @click="setMode('short')">
				Short Break
			</button>
			<button class="btn btnLong" :class="{ active: mode === 'long' }" @click="setMode('long')">
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
		<p class="pomodoroCounter">Pomodoros completed: {{ pomodoroCount }}</p>
	</div>
</template>

<style scoped lang="scss">
.timerBox {
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	font-size: 2em;

	.timer {
		font-size: 4em;
	}

	.pomodoroCounter {
		font-size: 1rem;
		margin-top: 1rem;
		color: #555;
	}
}

.active {
	border-color: currentColor;
	color: currentColor;
	box-shadow: 0 0 6px currentColor;
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
		max-width: 100%;
	}

	button:hover {
		cursor: pointer;
	}
}

.btn {
	border: 2px solid var(--btn-color);
}
.btn:hover {
	box-shadow: 0 0 6px var(--btn-color);
}
.btnWork {
	--btn-color: #e63946;
}
.btnShort {
	--btn-color: #2a9d8f;
}
.btnLong {
	--btn-color: #457b9d;
}

.mode {
	text-align: center;
	margin-top: 1rem;
	font-size: 1em;
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
</style>
