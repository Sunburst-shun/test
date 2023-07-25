<!--
 * @Author: vanexwh@163.com
 * @LastEditors: vanexwh@163.com
 * @Description: 
-->
<script setup lang="ts">
import { reactive } from 'vue';
import { getAssetsPointRealTimeData, ApiGetCombinePoints } from '/@/api/assets/point';
import { updateBatchAssetsPoints, updateBatchAssetsCombPoints } from '/@/api/assets/assets';
import { getRestPointsThatNotRelateAssetsAndRules, getRestCombPointsThatNotRelateAssetsAndRules } from '/@/api/monitor/others';
import { getDictLabel } from '/@/utils/getSysDict';

import PointHistory from '/@/components/modal/pointHistory.vue';
import Modal from '/@/components/modal/index.vue';
import Transfer from '/@/components/transfer/index.vue';

import M from 'moment';
import _ from 'lodash';

const props = defineProps<{
	id: string;
	visible: boolean;
}>();

const emits = defineEmits(['update:visible']);

const state = reactive({
	pointVisible: false,
	list: <any>{},
	points: {
		selected: [],
		unselected: [],
	},
	combPoints: {
		selected: [],
		unselected: [],
	},
});

const { pointVisible, list, points, combPoints } = toRefs(state);

const onClose = () => {
	emits('update:visible', pointVisible.value);
};

const transferRef1 = ref(null);
const transferRef2 = ref(null);
const onSubmit = async () => {
	await updateBatchAssetsPoints(
		props.id,
		transferRef1.value.data.map((v) => v.id)
	);
	await updateBatchAssetsCombPoints(
		props.id,
		transferRef2.value.data.map((v) => v.id)
	);
	pointVisible.value = false;
	_getPoint();
	emits('update:visible', pointVisible.value);
};

const _getPoint = async () => {
	const point: any = await getAssetsPointRealTimeData(props.id);
	list.value = point.list ? _.groupBy(point.list, 'point_type') : {};
	points.value.selected = point.list ? point.list : [];
	// 组合点
	const combPoint: any = await ApiGetCombinePoints({
		current: 1,
		page_size: 999,
		asset_id: props.id,
	});
	if (combPoint.list) {
		combPoints.value.selected = combPoint.list ? combPoint.list : [];
		list.value['组合监测点'] = _.map(combPoint.list, (c) => {
			return {
				id: c.id,
				abbreviation: c.name,
				result_value: c.result_value,
				alarm_rule_info: c.alarm_rule_info,
				isComb: true,
			};
		});
	}
};

// 获取全部监测点
const _getUnselectedPoint = async () => {
	const res1: any = await getRestPointsThatNotRelateAssetsAndRules({
		asset_bool: 0,
	});
	points.value.unselected = res1 ? res1 : [];

	const res2: any = await getRestCombPointsThatNotRelateAssetsAndRules({
		asset_bool: 0,
	});
	combPoints.value.unselected = res2 ? res2 : [];
};

watch(
	() => props.visible,
	(val) => {
		if (val) {
			_getUnselectedPoint().then(() => {
				pointVisible.value = val;
			});
		}
	}
);

watchEffect(async () => {
	if (props.id) {
		_getPoint();
	}
});

const history = ref({
	id: [],
	visible: false,
	isComb: false,
});

const onCheck = (id: string, isComb: boolean) => {
	history.value.id = [id];
	history.value.visible = true;
	history.value.isComb = isComb;
};

// 计算卡片高度
const calcCardHeight = () => {
	let height: string,
		len = _.size(list.value);
	height = 100 / Math.ceil(len / 2) + '%';
	return height;
};

const getPointValue = (point: any) => {
	if (point.point_category === 1) {
		let code: any;
		try {
			code = JSON.parse(point.yx_decode);
			return code[+point.result_value] || '-';
		} catch {
			return point.result_value;
		}
	} else {
		return point.result_value;
	}
};
</script>

<template>
	<div :class="['model-display-part-point', _.size(list) === 0 ? 'data' : '']">
		<div
			class="group"
			:style="{
				height: calcCardHeight(),
				width: _.size(list) === 1 ? '100%' : '48%',
				margin: _.size(list) === 1 ? '0' : _.size(list) === 2 ? '0 10px 0 0' : '0 10px 10px 0',
			}"
			v-for="(group, key) in list"
			:key="key"
		>
			<p class="title">
				<svg-icon class="mr5" icon="zaixianjiance" :size="20" />
				{{ getDictLabel('point_type', key) }}
			</p>
			<div class="group-list">
				<div class="group-item mb5">
					<p class="label">名称</p>
					<p class="value">值</p>
					<p class="unit">单位</p>
					<p class="result">结果</p>
				</div>
				<div class="group-item" v-for="item in group" :key="item.id" @click="onCheck(item.id, item.isComb)">
					<p class="label ellipsis">{{ item.abbreviation }}</p>
					<p class="value">{{ getPointValue(item) }}</p>
					<p class="unit">{{ item.unit || '-' }}</p>
					<p v-if="item?.alarm_rule_info?.rule_type != 4" :class="`result level-${item?.alarm_rule_info?.level}`">
						{{ getDictLabel('alarm_rule_type', item?.alarm_rule_info?.rule_type) }}
					</p>
					<p v-else class="result">无</p>
				</div>
			</div>
		</div>
		<twy-empty v-if="_.size(list) === 0" text="未关联监测点位" />
		<PointHistory v-model:visible="history.visible" :id="history.id" :isComb="history.isComb" />
	</div>
	<Modal v-model:visible="pointVisible" title="关联监测点" width="80vw" @close="onClose" @confirm="onSubmit">
		<Transfer
			ref="transferRef1"
			title="监测点"
			label="monitor_point_name"
			type="point_type"
			value="id"
			:unselected="points.unselected"
			:selected="points.selected"
			height="250px"
		/>
		<Transfer
			ref="transferRef2"
			title="组合监测点"
			label="name"
			value="id"
			:unselected="combPoints.unselected"
			:selected="combPoints.selected"
			height="250px"
		/>
	</Modal>
</template>

<style scoped lang="scss">
.data {
	box-shadow: 0px 8px 20px rgba(10, 18, 35, 0.6), inset 0px -4px 8px rgba(0, 199, 119, 0.4), inset 0px 4px 8px rgba(0, 199, 119, 0.4);
	border-radius: 8px;
}

.model-display-part-point {
	display: flex;
	flex-wrap: wrap;
	width: 100%;
	height: 100%;
	font-size: 14px;
	padding-bottom: 5px;
	color: #fff;
	overflow: auto;

	.group {
		box-shadow: 0px 8px 20px rgba(10, 18, 35, 0.6), inset 0px -4px 8px rgba(0, 199, 119, 0.4), inset 0px 4px 8px rgba(0, 199, 119, 0.4);
		backdrop-filter: blur(2px);
		border-radius: 8px;
		padding: 8px;

		.title {
			display: flex;
			align-items: center;
			font-size: 16px;
			font-style: italic;
			margin-bottom: 5px;
		}

		.group-list {
			width: 100%;
			height: calc(100% - 30px);
			overflow: auto;

			.group-item {
				width: 100%;
				display: flex;
				align-items: center;
				margin-bottom: 5px;
				cursor: pointer;

				&:hover {
					color: rgba(0, 199, 119, 0.8);
				}

				.label {
					width: 60%;
					text-indent: 10px;
				}

				.value {
					width: 15%;
					text-align: center;
				}

				.unit,
				.result {
					width: 12.5%;
					text-align: center;
				}
			}
		}
	}
}
</style>
