<script setup lang="ts">
import { reactive } from 'vue';
import { getAssetsRelateCameras, updateAssetsRelateCamera, getCameras } from '/@/api/assets/camera';
import Video from '/@/components/video/index.vue';
import Modal from '/@/components/modal/index.vue';
import Transfer from '/@/components/transfer/index.vue';

import { Swiper, SwiperSlide } from 'swiper/vue';
import { Navigation, Autoplay } from 'swiper';
import 'swiper/css';
import 'swiper/css/pagination';
import 'swiper/css/navigation';
import 'swiper/css/virtual';

const swiperModules = [Navigation, Autoplay];

const props = defineProps<{
	id: string;
	visible: boolean;
}>();

const emits = defineEmits(['update:visible']);

const state = reactive({
	videos: [],
	cameraVisible: false,
	relateIds: [],
	relateVideos: [],
});

const { videos, relateVideos, cameraVisible, relateIds } = toRefs(state);

const _getCameras = async () => {
	const res: any = await getCameras(1, 999);
	videos.value = res.list ? res.list : [];
};

const onClose = () => {
	emits('update:visible', cameraVisible.value);
};

const transferRef = ref(null);
const onSubmit = async () => {
	await updateAssetsRelateCamera(
		props.id,
		transferRef.value.data.map((v) => v.id)
	);
	_getRelateCameras();
	cameraVisible.value = false;
	emits('update:visible', cameraVisible.value);
};

const _getRelateCameras = async () => {
	const res: any = await getAssetsRelateCameras(props.id);
	relateVideos.value = res ? res : [];
};

watch(
	() => props.visible,
	(val) => {
		_getCameras().then(() => {
			cameraVisible.value = val;
		});
	}
);

watchEffect(async () => {
	if (props.id) {
		_getRelateCameras();
	}
});
</script>

<template>
	<div class="model-display-part-camera flexCC">
		<Swiper :spaceBetween="10" :navigation="true as undefined" :modules="swiperModules">
			<SwiperSlide v-for="video in relateVideos" :key="video.id">
				<div class="video">
					<Video :camera="video" :show-del="false" :show-zoom="false" />
				</div>
			</SwiperSlide>
		</Swiper>
		<twy-empty text="暂无摄像头" v-if="!relateVideos.length" />
	</div>
	<Modal v-model:visible="cameraVisible" title="关联摄像头" width="80vw" @close="onClose" @confirm="onSubmit">
		<Transfer ref="transferRef" title="摄像头" label="camera_name" value="id" :unselected="videos" :selected="relateVideos" height="350px" />
	</Modal>
</template>

<style scoped lang="scss">
.model-display-part-camera {
	width: 480px;
	height: 100%;
	box-shadow: 0px 8px 20px rgba(10, 18, 35, 0.6), inset 0px -4px 8px rgba(0, 199, 119, 0.4), inset 0px 4px 8px rgba(0, 199, 119, 0.4);
	backdrop-filter: blur(2px);
	border-radius: 8px;
	padding: 16px;
  overflow: auto;

	.video {
		display: flex;
		align-items: center;
		width: 100%;
		min-height: 300px;
	}
}
</style>
