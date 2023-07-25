<!--
 * @Author: vanexwh@163.com
 * @LastEditors: vanexwh@163.com
 * @Description: 
-->
<script setup lang="ts">
import { getAssetsTypes } from '/@/api/assets/assetType';
import { getAssetsList } from '/@/api/assets/assets';

import AssetsTypeCard from './components/assetsTypeCard.vue';
import AssetsTypeList from './components/assetsTypeList.vue';
import AssetsScore from './components/assetsScore.vue';
import Modal from '/@/components/modal/index.vue';

import _ from 'lodash';
import { Search } from '@element-plus/icons-vue';

const state = reactive({
  dataList: [],
  assets: [],
  isToggle: false,
  visible: false,
  filterKey: ''
});

const { dataList, assets, isToggle, visible, filterKey } = toRefs(state);

const onToggle = () => {
  isToggle.value = !isToggle.value;
};

const onSearch = () => {
  _getAssetsTypes(filterKey.value);
};

const _getAssetsTypes = async (name?: string) => {
  const res: any = await getAssetsTypes({
    current: 1,
    page_size: 999,
    name,
  });
  // 1.取到分类关联资产数量>0；2.按降序排序；
  dataList.value = res.list ? _.filter(_.sortBy(res.list, ['sort', 'create_time'], ['asc']), item => {
    if (item.asset_count > 0) {
      return item;
    }
  }) : [];
};

const _getAssets = async () => {
  const res: any = await getAssetsList(1, 999);
  assets.value = res.list ? res.list : [];
};

onMounted(() => {
  _getAssetsTypes();
  _getAssets();
});

</script>

<template>
  <div class="page-template-container">
    <div class="page-template-container-header">
      <div class="page-template-container-header-sizer">
        <el-input v-model="filterKey" placeholder="输入名称检索" :prefix-icon="Search" clearable @input="onSearch" />
      </div>
      <div class="page-template-container-header-icon flexCC">
        <el-tooltip placement="top" content="查看资产健康度">
          <svg-icon class="mr10" icon="score" :size="32" @click="visible = true" />
        </el-tooltip>
        <el-tooltip v-if="isToggle" placement="top" content="切换成视图">
          <svg-icon icon="online" :size="36" @click="onToggle" />
        </el-tooltip>
        <el-tooltip v-else placement="top" content="切换成列表">
          <svg-icon icon="liebiao" :size="48" @click="onToggle" />
        </el-tooltip>
      </div>
    </div>
    <div class="page-template-container-cards" v-if="!isToggle && _.size(dataList) > 0">
      <div class="card" v-for="item in dataList" :key="item.id">
        <AssetsTypeCard :data="item" />
      </div>
    </div>
    <div clalss="page-template-container-list" v-if="isToggle">
      <AssetsTypeList :data="dataList" />
    </div>
    <twy-empty v-if="_.size(dataList) == 0" />
  </div>
  <Modal v-model:visible="visible" title="资产健康一览图" width="60vw" :show-footer="false">
    <AssetsScore :data="assets" />
  </Modal>
</template>

<style scoped lang="scss">
.page-template-container {
  width: 100%;
  height: 100%;

  &-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    height: 56px;
    border-bottom: 2px dashed #aaa;
    margin-bottom: 15px;
  }


  &-cards {
    display: grid;
    align-content: start;
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
    gap: 0px 10px;
    width: 100%;
    height: calc(100% - 15px - 56px);
    overflow: auto;
  }

  &-list {
    width: 100%;
  }
}
</style>