<script setup lang="ts">
import { ref, reactive } from 'vue'
import GridItem from './GridItem.vue'

const initialState = reactive([
	[1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
	[0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
	[0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
	[0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
	[0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
])

const desiredState = reactive([
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0],
	[0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
	[0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
	[0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
	[0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
	[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
])

const currentPositionX = ref(9)
const currentPositionY = ref(9)

type Snapshot = {
	grid: number[][]
	x: number
	y: number
}

function cloneGrid(grid: number[][]): number[][] {
	return grid.map(row => [...row])
}

const state = ref(cloneGrid(initialState))
state.value[currentPositionY.value][currentPositionX.value] = 2

const history: Snapshot[] = []

function snapshot(): Snapshot {
	return {
		grid: cloneGrid(state.value),
		x: currentPositionX.value,
		y: currentPositionY.value,
	}
}

function restore(snap: Snapshot) {
	state.value = cloneGrid(snap.grid)
	currentPositionX.value = snap.x
	currentPositionY.value = snap.y
}

function checkLineOfSight(direction: 'up' | 'down' | 'left' | 'right') {
	if (direction === 'up') {
		for (let i = currentPositionY.value - 1; i >= 0; i--) {
			if (state.value[i][currentPositionX.value] === 1) {
				return { x: currentPositionX.value, y: i }
			}
		}
	} else if (direction === 'down') {
		for (let i = currentPositionY.value + 1; i < state.value.length; i++) {
			if (state.value[i][currentPositionX.value] === 1) {
				return { x: currentPositionX.value, y: i }
			}
		}
	} else if (direction === 'left') {
		for (let i = currentPositionX.value - 1; i >= 0; i--) {
			if (state.value[currentPositionY.value][i] === 1) {
				return { x: i, y: currentPositionY.value }
			}
		}
	} else if (direction === 'right') {
		for (let i = currentPositionX.value + 1; i < state.value[0].length; i++) {
			if (state.value[currentPositionY.value][i] === 1) {
				return { x: i, y: currentPositionY.value }
			}
		}
	}
	return null
}

function move(direction: 'up' | 'down' | 'left' | 'right') {
	const previous = snapshot()
	let oldPositionX = currentPositionX.value
	let oldPositionY = currentPositionY.value
	const lineOfSight = checkLineOfSight(direction)
	if (lineOfSight) {
		const temp = state.value[currentPositionY.value][currentPositionX.value]
		state.value[currentPositionY.value][currentPositionX.value] = state.value[lineOfSight.y][lineOfSight.x]
		state.value[lineOfSight.y][lineOfSight.x] = temp
		currentPositionX.value = lineOfSight.x
		currentPositionY.value = lineOfSight.y
	} else {
		if (direction === 'up' && currentPositionY.value > 0) {
			currentPositionY.value--
		} else if (direction === 'down' && currentPositionY.value < state.value.length - 1) {
			currentPositionY.value++
		} else if (direction === 'left' && currentPositionX.value > 0) {
			currentPositionX.value--
		} else if (direction === 'right' && currentPositionX.value < state.value[0].length - 1) {
			currentPositionX.value++
		}
		state.value[oldPositionY][oldPositionX] = 0
		state.value[currentPositionY.value][currentPositionX.value] = 2
	}
	if (currentPositionX.value !== previous.x || currentPositionY.value !== previous.y) {
		history.push(previous)
	}
}

function undo() {
	const previous = history.pop()
	if (!previous) return
	restore(previous)
}

function reset() {
	window.location.reload()
}

window.addEventListener('keydown', (event) => {
	if (event.key === 'ArrowUp' || event.key === 'w') {
		move('up')
	} else if (event.key === 'ArrowDown' || event.key === 's') {
		move('down')
	} else if (event.key === 'ArrowLeft' || event.key === 'a') {
		move('left')
	} else if (event.key === 'ArrowRight' || event.key === 'd') {
		move('right')
	} else if (event.key === 'z') {
		undo()
	} else if (event.key === 'r') {
		reset()
	}
})
</script>

<template>
  <div class="grid">
	<template v-for="(itemRow, rowIndex) in state" :key="rowIndex">
		<div class="grid-item-wrapper" v-for="(item, itemIndex) in itemRow" :key="itemIndex">
			<GridItem :currentItem="item" :desiresItem="desiredState[rowIndex]?.[itemIndex] ?? false" />
		</div>
	</template>
  </div>
  <div>
	<h2>Rules</h2>
	<p>
		All circles must be filled with black dots. You may move one space at a time, however, if you attempt to move toward a dot, you will instead swap places with it (regardless of distance). Use arrow keys or WASD to move.
	</p>
	<button @click="reset">Reset</button>
  </div>
</template>

<style scoped>
.grid {
	display: grid;
	grid-template-columns: repeat(11, 1fr);
	grid-template-rows: repeat(11, 1fr);

	width: 600px;
	height: 600px;

	border-right: 1px solid #aaa;
	border-bottom: 1px solid #aaa;
}

.grid-item-wrapper {
	display: flex;
	align-items: center;
	justify-content: center;

	aspect-ratio: 1;

	border-top: 1px solid #aaa;
	border-left: 1px solid #aaa;
}
</style>
