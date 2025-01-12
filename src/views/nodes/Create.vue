<template>
  <div>
    <h1>Job</h1>
    <el-button @click="clickAdd">+</el-button>
    <div v-for="field in fields" :key="field.name">
      <label>{{ field.name }}:</label>
      <input :type="field.type" :id="field.id" :name="field.name" :value="field.value" />
      <button @click.prevent="clickAdd">+</button>
      <button @click.prevent="clickRm">-</button>
    </div>

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
import { ref } from 'vue';

interface Field {
  type: string;
  id: string;
  name: string;
  value: string;
}

interface Form {
  type: ModuleTypes;
  name: string;
  format: string;
  repeatable: boolean;
  required: boolean;
  options?: string[];
}

type ModuleTypes = 'argument' | 'option' | 'procedure';

let fields = ref<Field[]>([]);
let dialogVisible = ref(false);
let form = ref<Form>({
  type: 'argument',
  name: '',
  format: '',
  repeatable: false,
  required: false,
});

let config = ref({
  types: ['argument', 'option', 'procedure'],
  format: ['string', 'number', 'boolean', 'object', 'array'],
});

function handleClose() {
  console.log('close');
  dialogVisible.value = false;
}

function save() {
  console.log(form.value);
  fields.value.push({
    type: form.value.type,
    id: form.value.name, // 假设 id 和 name 相同，可以根据实际情况调整
    name: form.value.name,
    value: ''
  });
  dialogVisible.value = false;
}

function cancel() {
  dialogVisible.value = false;
}

function clickAdd() {
  console.log('add');
  dialogVisible.value = true;
}

function clickRm() {
  console.log('rm');
}

function addOption() {
  if (!form.value.options) {
    form.value.options = [];
  }
  form.value.options.push('');
}
</script>