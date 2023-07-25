<!--
 * @Author: vanexwh@163.com
 * @LastEditors: vanexwh@163.com
 * @Description: 
-->
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router';

import Chart from '/@/components/chart/index.vue';

const router = useRouter();
const route = useRoute();

const props = defineProps<{
  data: any[]
}>();

const DATA = markRaw(props?.data || []);

const options = ref({
  xAxis: {
    type: 'value',
  },
  yAxis: {
    type: 'category',
    data: DATA.map(v => v.asset_name),
  },
  tooltip: {
    trigger: 'axis',
  },
  axisLabel: {
    show: true
  },
  series: [
    {
      type: 'bar',
      data: DATA.map(v => {
        return {
          id: v.id,
          type_id: v.asset_type_id,
          value: v.score,
          itemStyle: {
            color: v.score >= 80 ? '#00c777' : v.score < 80 && v.score >= 60 ? '#FAAD14' : '#FA4D4D'
          }
        }
      }),
      colorBy: 'series'
    },
  ],
  dataZoom: [
    {
      type: 'slider',
      xAxisIndex: [0, 10],
      filterMode: 'none'
    },
    {
      type: 'slider',
      yAxisIndex: 0,
      filterMode: 'none'
    },
    {
      type: 'inside',
      xAxisIndex: 0,
      filterMode: 'none'
    },
    {
      type: 'inside',
      yAxisIndex: 0,
      filterMode: 'none'
    }
  ],
});

const getChartItem = (params) => {
  if (route?.name === 'assetsDetail') return;
  router.push(`/assets/detail/${params?.data?.type_id}?assetId=${params?.data?.id}`);
};

</script>

<template>
  <Chart title="资产健康一览图" :option="options" height="80vh" @get-item="getChartItem" />
</template>

<style scoped lang="scss"></style>