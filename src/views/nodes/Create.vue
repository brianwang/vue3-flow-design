<template>
  <div>
    <h1>Job <el-button @click="rootAdd">+</el-button></h1>
    <hr />
    <el-form :model="form" label-width="auto" style="max-width: 800px">
      <el-form-item :label="field.name" v-for="field in nodes" :key="field.name">
        <div v-if="field.type == 'argument'">
          <el-input v-model="field.value" :name="field.name" v-if="field.format == 'string'" />
          <el-input-number
            v-model="field.value"
            :name="field.name"
            v-if="field.format == 'number'"
          />
          <el-switch v-model="field.value" class="ml-2" v-if="field.format == 'boolean'" />
        </div>
        <div v-if="field.type == 'option'">
          <el-form-item v-for="(option, index) in field.options" :key="index">
            <el-input v-model="field.options[index]" />
            <el-button @click="clickAdd(option)">+</el-button>
            <el-button @click="field.options.splice(index, 1)">-</el-button>
          </el-form-item>
          <div v-for="(o, i) in field.options" :key="o.id"> {{ i }}-{{ o }} </div>
        </div>
        <el-button @click.prevent="clickAdd(field)" v-if="field.type != 'argument'">+</el-button>
        <el-button @click.prevent="clickRm(field)">-</el-button>
      </el-form-item>
    </el-form>

    <el-dialog
      v-model="CreatedNodeDialog"
      title="Node List"
      width="500"
      :before-close="handleCloseCreateNode"
    >
      <div
        v-for="n in getCreatedNode()"
        :key="n.id"
        style="display: flex; flex-direction: column; border: solid 0.5px #ddd; border-radius: 15px"
      >
        <div v-for="(a, i) in n" :key="i"> {{ i }}:{{ a }} </div>
        <el-button @click.prevent="clickAdd2Option(n)">添加到option</el-button>
      </div>
    </el-dialog>
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
          <el-button @click="addOption()">+</el-button>
        </el-form-item>
        <el-button @click="save">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
  import { onMounted, ref, watch } from 'vue';
  import { utils } from '/@/utils/common';
  type ModuleTypes = 'argument' | 'option' | 'procedure';
  interface NodeArgument {
    id: string;
    repeatable?: boolean | false;
    required?: boolean | false;
    type: ModuleTypes | 'argument';
    name: string;
    value?: string;
    help?: string;
    default?: string;
    format?: string;
  }

  interface NodeProcedure extends NodeArgument {
    // [key: string]: NodeArgument;
    params: Array<NodeArgument>;
  }
  type NodeChoice = NodeProcedure | NodeArgument;

  interface NodeOption extends NodeArgument {
    options: Array<NodeChoice>;
  }

  type NodeType = NodeArgument | NodeProcedure | NodeOption;
  let nodes = ref<any>({});
  let dialogVisible = ref(false);
  let form = ref<NodeArgument | NodeProcedure | NodeOption>({
    id: utils.getId(),
    name: '',
    type: 'argument',
  });

  let config = ref({
    types: ['argument', 'option', 'procedure'],
    format: ['string', 'number', 'boolean', 'object', 'array'],
  });

  let curNode = ref<NodeType>();

  function handleClose() {
    console.log('close');
    dialogVisible.value = false;
  }

  onMounted(() => {
    //console.log('mounted');
  });

  function getCreatedNode(): Array<NodeType> {
    let retList = Array<NodeType>();
    for (const k in nodes.value) {
      let n = nodes.value[k];
      console.log(n);
      if (n['type'] === 'argument' || n['type'] === 'procedure') {
        retList.push(n);
      }
    }
    console.log('已经生成的Node:', retList);
    return retList;
  }

  const CreatedNodeDialog = ref(false);
  //关闭
  function handleCloseCreateNode() {
    CreatedNodeDialog.value = !CreatedNodeDialog.value;
  }

  //根结点增加
  function rootAdd() {
    console.log('add');
    dialogVisible.value = true;
  }
  function save() {
    console.log(form.value);
    let cNode = Object.assign({}, form.value);
    cNode.id = utils.getId();
    nodes.value[form.value.name] = cNode;
    dialogVisible.value = false;
    curNode.value = cNode;
    console.log(nodes.value);
  }

  function cancel() {
    dialogVisible.value = false;
  }

  function clickAdd(node) {
    console.log('add：', node);
    curNode.value = node;
    dialogVisible.value = true;
  }

  function clickRm(node) {
    console.log(node);
    nodes.value = nodes.value.filter((item) => item.id !== node.id);
    console.log('rm');
  }

  function addOption() {
    CreatedNodeDialog.value = true;
  }

  function clickAdd2Option(node) {
    // if (curNode.value != undefined) {
    form.value.options.push(node);
    // }
    CreatedNodeDialog.value = false;
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
