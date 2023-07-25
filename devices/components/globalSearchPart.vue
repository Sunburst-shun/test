<!--
 * @Author: vanexwh@163.com
 * @LastEditors: vanexwh@163.com
 * @Description: 
-->
<script setup lang="ts">
import { reactive } from 'vue';
import { Search } from '@element-plus/icons-vue';
import { getAssetsList } from '/@/api/assets/assets';

const emits = defineEmits(['search']);

const state = reactive({
  search: {
    like: '',
    results: []
  }
});

const { search } = toRefs(state);

const onInput = () => {
  if (search.value.like) {
    _getAssets().then(res => {
      search.value.results = res;
    });
  } else {
    emits('search', '');
    search.value.results = [];
  }
};

const onSearch = (id: string) => {
  emits('search', id);
};

const _getAssets = async () => {
  const res: any = await getAssetsList(1, 999, '', '', search.value.like);
  return res.list ? res.list : [];
};

</script>

<template>
  <p class="model-search-part-tips">
    <span>目前仅支持对</span>
    <span class="search-key">资产</span>
    <span>的检索</span>
  </p>
  <el-input :prefix-icon="Search" class="eleW100" v-model="search.like" clearable placeholder="输入关键词以检索"
    @input="onInput" />
  <p v-if="search.like" class="model-search-part-tips">查询结果：</p>
  <div class="model-search-part-results">
    <div class="item" v-for="item in search.results" :key="item.id" @click="onSearch(item.id)">
      <p class="text">{{ item.asset_name }}</p>
    </div>
    <twy-empty v-if="!search.results.length && search.like" text="未查询到数据" />
  </div>
</template>

<style scoped lang="scss">
.model-search-part-tips {
  font-size: 12px;
  color: var(--color-text);
  margin: 10px 0;
  user-select: none;
  .search-key {
    font-weight: 600;
  }
}

.model-search-part-results {
  height: calc(100% - 100px);
  padding: 10px 0;
  overflow: auto;

  .item {
    width: 100%;
    display: flex;
    margin-bottom: 5px;
    cursor: pointer;

    .text {
      color: #aaa;
      font-size: 18x;
      text-indent: 10px;
      &:hover {
        color: #ddd;
      }
    }
  }
}
</style>