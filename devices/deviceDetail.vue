<script setup lang="ts">
import AssetsInfo from './components/assetsInfo.vue';
import AssetsPoint from './components/assetsPoint.vue';
import AssetsCamera from './components/assetsCamera.vue';
import Modal from '/@/components/modal/index.vue';
import AssetsScore from './components/assetsScore.vue';

import { reactive } from 'vue';
import { getAssetsList } from '/@/api/assets/assets';
import { useRoute, useRouter } from 'vue-router';
import {useStore} from '/@/store';

const store = useStore();
const route = useRoute();
const router = useRouter();

const state = reactive({
  params: {
    typeId: '',
    assetsId: ''
  },
  currentAssetsId: '',
  source: '',
  list: [],
  visible: {
    score: false,
    info: false,
    camera: false,
    point: false
  },
});

const { params, currentAssetsId, list, source, visible } = toRefs(state);

const goBack = () => {
  router.push('/device/type');
};

const onOpenModal = (type: string) => {
  visible.value[type] = true;
};

onMounted(() => {
  // 路由取参
  params.value = {
    typeId: route?.params?.id as string ?? '',
    assetsId: route?.query?.assetId as string ?? ''
  };
  currentAssetsId.value = params.value.assetsId;
  _getAssets();
});

const _getAssets = async () => {
  const res: any = await getAssetsList(1, 999, params.value.typeId);
  list.value = res.list ? res.list : [];
  // 默认数据
  if (res.total&&!currentAssetsId.value) {
    currentAssetsId.value = res.list[0].id;
  }
};

</script>

<template>
  <div class="page-template-container">
    <div class="page-template-container-header">
      <div class="item">
        <svg-icon class="mr10" icon="fanhui" @click="goBack" />
        <el-select class="eleW30" v-model="currentAssetsId" placeholder="请选择资产" :filterable="list.length > 5">
          <el-option v-for="item in list" :key="item.id" :value="item.id" :label="item.asset_name" />
        </el-select>
        <twy-button  v-if="store.config.getDeloy" text="编辑资产" type="success" @click="onOpenModal('info')" />
        <twy-button  v-if="store.config.getDeloy" text="关联摄像头" type="success" @click="onOpenModal('camera')" />
        <twy-button  v-if="store.config.getDeloy" text="关联监测点" type="success" @click="onOpenModal('point')" />
      </div>
      <el-tooltip placement="top" content="查看资产健康度">
        <svg-icon class="mr10" icon="score" :size="32" @click="visible.score = true" />
      </el-tooltip>
    </div>
    <div class="page-template-container-main">
      <div class="left">
        <div class="assets mb10">
          <AssetsInfo v-model:visible="visible.info" :id="currentAssetsId" :source="source" />
        </div>
        <div class="camera">
          <AssetsCamera v-model:visible="visible.camera" :id="currentAssetsId" />
        </div>
      </div>
      <div class="right">
        <AssetsPoint class="points" v-model:visible="visible.point" :id="currentAssetsId" />
      </div>
    </div>
  </div>
  <Modal v-model:visible="visible.score" title="资产健康一览图" width="60vw" :show-footer="false">
    <AssetsScore :data="list" />
  </Modal>
</template>

<style scoped lang="scss">
.page-template-container {
  width: 100%;
  height: 100%;
  &-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    height: 56px;
    background-color: var(--color-primary);
    padding: 0 15px;
    margin-bottom: 15px;
    border-radius: 8px;

    .item {
      display: flex;
      align-items: center;
      width: 50%;
    }
  }

  &-main {
    display: flex;
    width: 100%;
    height: calc(100% - 60px);

    .left {
      width: 480px;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      margin-right: 10px;

      .camera,
      .assets {
        height: 50%;
      }
    }

    .right {
      width: calc(100% - 480px);
      height: 100%;
    }
  }
}
</style>