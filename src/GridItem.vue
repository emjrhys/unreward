<script setup lang="ts">
import { defineProps } from 'vue'

const props = defineProps<{
	currentItem: number,
	desiresItem: boolean,
}>()

</script>

<template>
	<div class="grid-item">
		<span v-if="currentItem === 2" style="font-size: 36px;">⛄︎</span>
		<span v-else-if="desiresItem">
			<span v-if="currentItem === 1" style="font-size: 20px">◉</span>
			<span v-else style="font-size: 24px;">◎</span>
		</span>
		<span v-else-if="currentItem === 1" style="font-size: 30px;">●</span>
	</div>
</template>

<style scoped></style>