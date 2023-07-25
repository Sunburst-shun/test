<script setup lang="ts">

</script>

<template>
	<div class="page-container">
		<router-view v-slot="{ Component }">
			<component :is="Component" />
		</router-view>
	</div>
</template>

<style scoped lang="scss">
.page-container {
	width: 100%;
	height: 100%;
	padding: 16px;
}
</style>