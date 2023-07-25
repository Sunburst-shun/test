<!--
 * @Author: vanexwh@163.com
 * @LastEditors: vanexwh@163.com
 * @Description: 
-->
<script setup lang="ts">
import { IApiAssetsType } from '/@/interface/api/assetsType';
import { getDictLabel } from '/@/utils/getSysDict';
import { useRouter } from 'vue-router';

const router = useRouter();


const props = defineProps<{
  data: IApiAssetsType[]
}>();

const onJumpToDetail = (id: string) => {
  router.push(`/device/detail/${id}`)
};

const tabRowClassName = (row: any) => {
  let index = row.rowIndex;
  if (index % 2 == 0) {
    return 'odd-row';
  } else {
    return 'even-row';
  }
};

</script>

<template>
  <div class="comp-assets-type-list">
    <el-table class="mb5" :data="props.data" height="80vh"
      :header-cell-style="{ backgroundColor: 'rgba(38, 56, 86, 1)', color: '#fff' }" :row-class-name="tabRowClassName">
      <el-table-column label="序号" prop="sort" show-overflow-tooltip width="100" align="left" />
      <el-table-column label="类别" prop="asset_category">
        <template #default="scope">
          {{ getDictLabel('asset_category', scope.row.asset_category) }}
        </template>
      </el-table-column>
      <el-table-column label="名称" prop="name" />
      <el-table-column label="路径" prop="path" align="center" width="500" />
      <el-table-column label="使用范围" prop="path" align="center">
        <template #default="scope">
          <el-tag>{{ getDictLabel('is_private', scope.row.state) }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="150">
        <template #default="scope">
          <el-button size="small" type="success" @click="onJumpToDetail(scope.row.id)">查看</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<style scoped lang="scss">
.comp-assets-type-list {
  width: 100%;
}
</style>