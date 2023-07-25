<script setup lang="ts">
import { IApiAssetsType } from '/@/interface/api/assetsType';
import { getDictLabel } from '/@/utils/getSysDict';
import { useRouter } from 'vue-router';

import picPre from '/@/components/upload/picPre.vue';

const router = useRouter();

const props = defineProps<{
  data: IApiAssetsType
}>();

const onJumpToDetail = () => {
  router.push(`/device/detail/${props.data?.id}`)
};

</script>

<template>
  <div class="comp-assets-type-card" @click="onJumpToDetail">
    <div class="comp-assets-type-card-name line ellipsis">{{ props.data?.name }}</div>
    <div class="comp-assets-type-card-category line">类别：{{ getDictLabel('asset_category', props.data?.asset_category) }}
    </div>
    <div class="comp-assets-type-card-path line">路径：<p>{{ props.data?.path }}</p>
    </div>
    <div class="comp-assets-type-card-count">关联资产数：{{ props.data?.asset_count }}</div>
    <div class="comp-assets-type-card-pic">
      <picPre v-if="props.data.asset_avatar" :path="props.data.asset_avatar" :preview="false" />
      <img v-else src="/images/assets_type.png" />
    </div>
  </div>
</template>

<style scoped lang="scss">
.comp-assets-type-card {
  position: relative;
  width: 100%;
  height: 160px;
  padding: 8px;
  background-image: linear-gradient(10deg, #1c2a48 20%, #193050 70%);
  margin: 0 10px 10px 0;
  color: #fff;
  font-size: 14px;
  border-radius: 4px;
  transition: 0.3s;
  cursor: pointer;

  &:hover {
    transform: translateY(3px);
    transition: 0.3s;
  }

  &:active {
    opacity: 0.7;
  }

  .line {
    margin-bottom: 6px;
  }

  &-name {
    font-size: 18px;
    letter-spacing: 1px;
    font-weight: 600;
    margin-bottom: 12px !important;
    user-select: none;
  }

  &-category,
  &-path {
    display: flex;
    width: 100%;
    font-size: 12px;
    color: #aaa;

    p {
      width: 80%;
    }
  }

  &-count {
    font-size: 12px;
    color: var(--color-success);
    user-select: none;
  }

  &-count {
    position: absolute;
    bottom: 10px;
  }

  &-pic {
    position: absolute;
    right: 15px;
    bottom: 10px;
    width: 60px;
    user-select: none;

    img {
      width: 100%;
    }
  }

}
</style>