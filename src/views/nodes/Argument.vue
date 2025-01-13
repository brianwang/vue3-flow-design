<template>
  <div>
    <h1>Job <el-button @click="rootAdd">+</el-button></h1>
    <hr />
    <el-form :model="form" label-width="auto" style="max-width: 800px">
      <el-form-item :label="field.name" v-for="field in fields" :key="field.name">
        <el-input v-model="field.value" :name="field.name" v-if="field.format == 'string'" />
        <el-input-number v-model="field.value" :name="field.name" v-if="field.format == 'number'" />
        <el-switch v-model="field.value" class="ml-2" v-if="field.format == 'boolean'" />
        <el-button @click.prevent="clickAdd(field)" v-if="field.type != 'argument'">+</el-button>
        <el-button @click.prevent="clickRm(field)">-</el-button>
      </el-form-item>
    </el-form>

    <el-dialog v-model="dialogVisible" title="Create Node" width="500" :before-close="handleClose">
      <el-form :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item label="type">
          <el-select v-model="form.type" placeholder="请选择一个Module类型">
            <el-option v-for="t in config.types" :label="t" :value="t" :key="t" />
          </el-select>
        </el-form-item>
        <el-form-item label="name">
          <el-input v-model="form.name" />
        </el-form-item>
        <el-form-item label="format">
          <el-select v-model="form.format" placeholder="请选择一个Format类型">
            <el-option v-for="t in config.format" :label="t" :value="t" :key="t" />
          </el-select>
        </el-form-item>
        <el-form-item label="repeatable">
          <el-switch v-model="form.repeatable" />
        </el-form-item>
        <el-form-item label="required">
          <el-switch v-model="form.required" />
        </el-form-item>
        <el-form-item label="options" v-if="form.type === 'option'">
          <el-button @click="addOption">+</el-button>
        </el-form-item>
        <el-button @click="save">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
  import { onMounted, ref } from 'vue';

  interface Field {
    id: number;
    type: string;
    name: string;
    value: string;
    format: string;
    options: Array<any>;
  }

  interface Form {
    id: number;
    type: ModuleTypes;
    name: string;
    value: string;
    format: string;
    repeatable: boolean;
    required: boolean;
    options?: string[];
  }

  type ModuleTypes = 'argument' | 'option' | 'procedure';

  let fields = ref<Field[]>([]);
  let dialogVisible = ref(false);
  let form = ref<Form>({
    id: 0,
    type: 'argument',
    name: '',
    format: 'string',
    value: '',
    repeatable: false,
    required: false,
    options: [],
  });

  let config = ref({
    types: ['argument', 'option', 'procedure'],
    format: ['string', 'number', 'boolean', 'object', 'array'],
  });

  function handleClose() {
    console.log('close');
    dialogVisible.value = false;
  }

  onMounted(() => {
    //console.log('mounted');
  });

  //根结点增加
  function rootAdd() {
    console.log('add');
    dialogVisible.value = true;
  }
  function save() {
    console.log(form.value);
    fields.value.push({
      id: fields.value.length + 1,
      type: form.value.type,
      name: form.value.name,
      value: form.value.value,
      format: form.value.format,
      options: form.value.options,
    });
    dialogVisible.value = false;
  }

  function cancel() {
    dialogVisible.value = false;
  }

  function clickAdd(node) {
    console.log('add：', node);
    if (node.type === 'option') {
      form.value.options = [];
    }
    dialogVisible.value = true;
  }

  function clickRm(node) {
    console.log(node);
    fields.value = fields.value.filter((item) => item.id !== node.id);
    console.log('rm');
  }

  function addOption() {
    if (!form.value.options) {
      form.value.options = [];
    }
    form.value.options.push('');
  }
</script>

<style scoped lang="less">
  .el-form {
    .el-form-item {
      :deep(.el-form-item__content) {
        flex-wrap: nowrap !important;
        // button {
        //   margin-left: 10px;
        // }
      }
    }
  }
</style>
