<script setup lang="ts">
import { getAsset, updateAssets } from '/@/api/assets/assets';
import { getDictLabel, getSysDict } from '/@/utils/getSysDict';

import M from 'moment';
import Score from '/@/components/waveProcess/index.vue';
import picPre from '/@/components/upload/picPre.vue';
import Modal from '/@/components/modal/index.vue';


const props = defineProps<{
  id: string,
  source: string,
  visible: boolean,
}>();

const emits = defineEmits(['update:visible']);

const info = ref<any>({});


const state = reactive({
  assetsVisible: false,
  form: {
    id: '',
    asset_name: '',
    asset_model: '',
    asset_code: '',
    asset_state: 1,
    manu_facturer: '',
    factory_number: '',
    supplier: '',
    production_at: +M().format('x'),
    operation_at: +M().format('x'),
    operation_location: '',
    current_status_at: +M().format('x'),
    warranty_at: +M().format('x'),
    asset_type_id: '',
    parent_id: '',
    e_file: '',
    data: ''
  },
  rules: {
    asset_name: [{ required: true, message: '请输入资产名称', trigger: 'blur' }],
    asset_code: [{ required: true, message: '请输入资产编号', trigger: 'blur' }],
    asset_model: [{ required: true, message: '请输入资产型号', trigger: 'blur' }],
    operation_location: [{ required: true, message: '请选择运行地点', trigger: 'change' }],
  },
  expand_fields: {
    assets: {
      voltage_grade: '',
      path: '',
    },
    robot: {
      robot_id: '',
      robot_address: '',
      robot_supplier: null,
      authorization: ''
    }
  },
});

const { assetsVisible, form, rules, expand_fields } = toRefs(state);

const onClose = () => {
  emits('update:visible', assetsVisible.value);
};

const formRef = ref(null);
const onSubmit = () => {
  const { id, parent_id, asset_name, asset_model, asset_code, asset_state, manu_facturer, factory_number, supplier, production_at, operation_at, operation_location, current_status_at, warranty_at, data } = form.value;
  formRef.value.validate(async (valid: boolean) => {
    if (valid) {
      await updateAssets(id, asset_code, asset_name, asset_model, asset_state, info.value.asset_category, operation_location,
        manu_facturer, factory_number, supplier, production_at, operation_at, current_status_at, warranty_at,
        info.value.asset_type_id, parent_id, data);
    }
    assetsVisible.value = false;
    emits('update:visible', assetsVisible.value);
    _getAssetsInfo();
  });
};

const _getAssetsInfo = async () => {
  const res: any = await getAsset(props.id);
  info.value = res ? res : {};
};

watch(() => props.visible, val => {
  assetsVisible.value = val;
  if (val) {
    form.value = {
      id: info.value.id,
      asset_name: info.value.asset_name,
      asset_model: info.value.asset_model,
      asset_code: info.value.asset_code,
      asset_state: info.value.asset_state,
      manu_facturer: info.value.manu_facturer,
      factory_number: info.value.factory_number,
      supplier: info.value.supplier,
      production_at: info.value.production_at,
      operation_at: info.value.operation_at,
      operation_location: info.value.operation_location,
      current_status_at: info.value.current_status_at,
      warranty_at: info.value.warranty_at,
      asset_type_id: info.value.asset_type_id,
      parent_id: info.value.parent_id,
      e_file: info.value.e_file,
      data: info.value.data
    };
    if (info.value.asset_category === 1) {
      expand_fields.value.assets = JSON.parse(info.value.data);
    } else if (info.value.asset_category === 2) {
      expand_fields.value.robot = JSON.parse(info.value.data);
    }
  }
});


watchEffect(async () => {
  info.value = {};

  if (props.id) {
    _getAssetsInfo();
  }
});

</script>

<template>
  <div class="model-display-part-assets">
    <!-- <div class="header" v-if="props.id">
      <div class="score">
        <Score :process="+info?.score" />
      </div>
      <div class="source">
        <picPre v-if="info.asset_avatar" :path="info.asset_avatar" :preview="true" />
        <twy-empty v-else text="未设置图片" />
      </div>
    </div> -->
    <div class="info" v-if="props.id">
      <p class="line">
        <span class="label">资产类目：</span>
        <span class="value">{{ getDictLabel('asset_category', info.asset_category) }}</span>
      </p>
      <p class="line">
        <span class="label">资产编码：</span>
        <span class="value">{{ info?.asset_code || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">资产名称：</span>
        <span class="value">{{ info?.asset_name || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">资产型号：</span>
        <span class="value">{{ info?.asset_model || '-' }}</span>
      </p>
      <p class="line" v-if="info.asset_category == 1">
        <span class="label">资产路径：</span>
        <span class="value">{{ info.data ? JSON.parse(info.data).path || '-' : '-' || '-' }}</span>
      </p>
      <p class="line" v-if="info.asset_category == 1">
        <span class="label">电压等级：</span>
        <span class="value">{{ info.data ? JSON.parse(info.data).voltage_grade || '-' : '-' || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">运行地点：</span>
        <span class="value">{{ info?.operation_location || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">出厂编号：</span>
        <span class="value">{{ info?.factory_number || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">生产厂商：</span>
        <span class="value">{{ info?.manu_facturer || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">供应厂商：</span>
        <span class="value">{{ info?.supplier || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">出厂日期：</span>
        <span class="value">{{ M(info?.production_at).format('YYYY-MM-DD') || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">投运日期：</span>
        <span class="value">{{ M(info?.operation_at).format('YYYY-MM-DD') || '-' }}</span>
      </p>
      <p class="line">
        <span class="label">保修日期：</span>
        <span class="value">{{ M(info?.warranty_at).format('YYYY-MM-DD') || '-' }}</span>
      </p>
      <p class="line" v-if="info.asset_category == 2">
        <span class="label">机器人id：</span>
        <span class="value">{{ info.data ? JSON.parse(info.data).robot_id || '-' : '-' || '-' }}</span>
      </p>
      <p class="line" v-if="info.asset_category == 2">
        <span class="label">访问地址：</span>
        <span class="value">{{ info.data ? JSON.parse(info.data).robot_address || '-' : '-' || '-' }}</span>
      </p>
      <p class="line" v-if="info.asset_category == 2">
        <span class="label">供应厂商：</span>
        <span class="value">{{ info.data ? JSON.parse(info.data).robot_supplier || '-' : '-' || '-' }}</span>
      </p>
    </div>
    <twy-empty text="暂无信息" v-else />
  </div>
  <Modal ref="formRef" v-model:visible="assetsVisible" title="编辑资产" width="40vw" @close="onClose" @confirm="onSubmit">
    <el-form ref="formRef" :model="form" :rules="rules" label-width="100px">
      <el-form-item label="资产名称：" prop="asset_name">
        <el-input v-model="form.asset_name" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item label="资产编号：" prop="asset_code">
        <el-input v-model="form.asset_code" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item label="资产型号：" prop="asset_model">
        <el-input v-model="form.asset_model" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item label="运行地点：" prop="operation_location">
        <el-select style="width:100%;" v-model="form.operation_location">
          <el-option v-for="l in getSysDict('assets_location')" :key="l.id" :value="l.dict_value" :label="l.dict_label" />
        </el-select>
      </el-form-item>
      <el-form-item label="生产厂家：">
        <el-input v-model="form.manu_facturer" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item label="出厂编号：">
        <el-input v-model="form.factory_number" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item label="供应厂商：">
        <el-input v-model="form.supplier" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item label="出厂日期：">
        <el-date-picker style="width:100%;" placeholder="请选择" value-format="x" v-model="form.production_at" clearable />
      </el-form-item>
      <el-form-item label="投运日期：">
        <el-date-picker style="width:100%;" placeholder="请选择" value-format="x" v-model="form.operation_at" clearable />
      </el-form-item>
      <el-form-item label="保修日期：">
        <el-date-picker style="width:100%;" placeholder="请选择" value-format="x" v-model="form.warranty_at" clearable />
      </el-form-item>
      <p class="expand">扩展信息：</p>
      <!-- 电力资产扩展字段 -->
      <el-form-item v-if="info.asset_category === 1" label="电压等级：" prop="voltage_grade">
        <el-select style="width:100%;" v-model="expand_fields.assets.voltage_grade">
          <el-option v-for="g in getSysDict('voltage_grade')" :key="g.id" :value="g.dict_value" :label="g.dict_label" />
        </el-select>
      </el-form-item>
      <el-form-item v-if="info.asset_category === 1" label="资产路径：">
        <el-input v-model="expand_fields.assets.path" placeholder="请输入" clearable />
      </el-form-item>
      <!-- 机器人扩展字段 -->
      <el-form-item v-if="info.asset_category === 2" label="机器人id：">
        <el-input v-model="expand_fields.robot.robot_id" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item v-if="info.asset_category === 2" label="访问地址：">
        <el-input v-model="expand_fields.robot.robot_address" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item v-if="info.asset_category === 2" label="供应厂商：">
        <el-select class="eleW100" v-model="expand_fields.robot.robot_supplier">
          <el-option v-for="dict in getSysDict('robot_supplier')" :key="dict.id" :value="+dict.dict_value"
            :label="dict.dict_label" />
        </el-select>
      </el-form-item>
      <el-form-item v-if="info.asset_category === 2" label="令牌：">
        <el-input v-model="expand_fields.robot.authorization" placeholder="请输入" clearable />
      </el-form-item>
    </el-form>
  </Modal>
</template>

<style scoped lang="scss">
.model-display-part-assets {
  width: 480px;
  height: 100%;
  box-shadow: 0px 8px 20px rgba(10, 18, 35, 0.6), inset 0px -4px 8px rgba(0, 199, 119, 0.4), inset 0px 4px 8px rgba(0, 199, 119, 0.4);
  backdrop-filter: blur(2px);
  border-radius: 8px;
  padding: 16px 8px 8px 8px;
  margin-bottom: 10px;

  .header {
    width: 100%;
    height: 200px;
    display: flex;
    align-items: center;
    border-bottom: 2px dashed #fff;
    padding-bottom: 5px;
    margin-bottom: 10px;

    .score {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 160px;
      border-right: 2px dashed #fff;
      margin-right: 10px;
    }

    .source {
      width: calc(100% - 160px);
      height: 100%;
      padding: 4px;
    }
  }

  .info {
    display: flex;
    flex-wrap: wrap;
    width: 100%;
    height: 100%;
    overflow: auto;
    color: #fff;

    .line {
      display: flex;
      align-items: center;
      width: 100%;
      margin-bottom: 8px;
      font-size: 14px;
      padding-left: 10px;

      .label {
        width: 20%;
        min-width: 80px;
        color: #bbb;
      }

      .value {
        width: 65%;
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
      }
    }
  }
}
</style>