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
          类型：{{ field.type }}
          <el-form-item v-for="(option, index) in field.options" :key="index">
            选项:
            <div>
              {{ option.name }}
            </div>
          </el-form-item>
        </div>
        <div v-if="field.type == 'procedure'">
          类型:{{ field.type }}
          <el-form-item v-for="(param, index) in field.params" :key="index">
            参数：
            <div>
              {{ param.name }}
            </div>
          </el-form-item>
        </div>
        <el-button @click.prevent="clickAdd(field)" v-if="field.type != 'argument'">+</el-button>
        <el-button @click.prevent="clickRm(field)">-</el-button>
      </el-form-item>
    </el-form>

    <el-dialog
      v-model="CreatedNodeDialog"
      title="Option List"
      width="500"
      :before-close="handleCloseCreateNode"
    >
      <div style="display: flex">
        <div
          v-for="n in getCreatedNode()"
          :key="n.id"
          style="
            display: flex;
            width: 200px;
            margin: 0 10px;
            flex-direction: column;
            border: solid 0.5px #ddd;
            border-radius: 5px;
            justify-content: center;
          "
        >
          <div>name: {{ n.name }}</div>
          <div>type: {{ n.type }}</div>
          <div>format: {{ n.format }}</div>
          <el-button @click.prevent="clickAdd2Option(n)">添加到option</el-button>
        </div>
      </div>
    </el-dialog>

    <el-dialog
      v-model="addParamDialog"
      title="Parameter List"
      width="500"
      :before-close="handleCloseAddParam"
    >
      <div style="display: flex">
        <div
          v-for="n in getCreatedNode()"
          :key="n.id"
          style="
            display: flex;
            width: 200px;
            margin: 0 10px;
            flex-direction: column;
            border: solid 0.5px #ddd;
            border-radius: 5px;
            justify-content: center;
          "
        >
          <div>name: {{ n.name }}</div>
          <div>type: {{ n.type }}</div>
          <div>format: {{ n.format }}</div>

          <el-button @click.prevent="add2Procedure(n)">添至Procedure</el-button>
        </div>
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
        <el-form-item label="format" v-if="form.type === 'argument'">
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
        <div v-if="form.type === 'option'" style="display: flex">
          <div
            v-for="o in (form as NodeOption).options"
            :key="o.id"
            style="border: solid 0.5px #ddd; margin: 0 10px; padding: 5px"
          >
            <div>{{ o.name }}</div>
            <div>{{ o.type }}</div>
            <el-button @click.prevent="rmOption(o)">-</el-button>
          </div>
        </div>

        <el-form-item label="parameter" v-if="form.type === 'procedure'">
          <el-button @click="addParams()">+</el-button>
        </el-form-item>
        <div style="display: flex" v-if="form.type === 'procedure'">
          <div
            v-for="o in (form as NodeProcedure).params"
            :key="o.id"
            style="border: solid 0.5px #ddd; margin: 0 10px; padding: 5px"
          >
            <div>{{ o.name }}</div>
            <div>{{ o.type }}</div>
            <el-button @click.prevent="rmParam(o)">x</el-button>
          </div>
        </div>
        <el-button @click="save">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
  import { onMounted, ref, watch } from 'vue';
  import { utils } from '/@/utils/common';

  const props = defineProps({
    data: {
      type: Object,
      default: () => ({}),
    },
    yaml: {
      type: Object,
      default: () => ({}),
    },
  });

  type ModuleTypes = 'argument' | 'option' | 'procedure';
  interface NodeArgument {
    id: string;
    repeatable?: boolean | false;
    required?: boolean | false;
    type: ModuleTypes | 'argument';
    name: string;
    value?: string | boolean;
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

  const emits = defineEmits(['update:data']);

  type NodeType = NodeArgument | NodeProcedure | NodeOption;
  let nodes = ref<any>(props.data);
  let dialogVisible = ref(false);
  const addParamDialog = ref(false);
  let form = ref<NodeArgument | NodeProcedure | NodeOption>({
    id: utils.getId(),
    name: '',
    type: 'argument',
    repeatable: false,
    required: false,
    format: 'string',
  });

  let config = ref({
    types: ['argument', 'option', 'procedure'],
    format: ['string', 'number', 'boolean'],
  });

  let curNode = ref<NodeType>();

  function handleClose() {
    console.log('close');
    dialogVisible.value = false;
  }

  onMounted(() => {
    //console.log('mounted');
  });

  function handleCloseAddParam() {
    addParamDialog.value = false;
  }
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

  watch(
    () => props.data,
    (val) => {
      nodes.value = val;
      // props.yaml =
    },
    { deep: true },
  );

  function rmOption(node) {
    console.log('删除 procedure 的参数:', node);
    let n_option = form.value as NodeOption;
    n_option.options = n_option.options.filter((i) => i.id !== node.id);
    form.value = n_option;
  }

  function rmParam(node) {
    console.log('删除 procedure 的参数:', node);
    let n_proc = form.value as NodeProcedure;
    n_proc.params = n_proc.params.filter((i) => i.id !== node.id);
    form.value = n_proc;
  }
  function add2Procedure(node) {
    let n_proc = form.value as NodeProcedure;
    if (n_proc.params == undefined) {
      n_proc.params = [];
    }
    let node_obj = Object.assign({}, node);
    n_proc.params.push(node_obj);
    form.value = n_proc;
    addParamDialog.value = false;
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
    if (cNode.type == 'procedure') {
      delete cNode.options;
    }
    if (cNode.type == 'option') {
      delete cNode.params;
    }
    cNode.id = utils.getId();
    nodes.value[form.value.name] = cNode;
    dialogVisible.value = false;
    curNode.value = cNode;
    console.log(nodes.value);
    emits('update:data', nodes);
  }

  function cancel() {
    dialogVisible.value = false;
  }

  //添加参数
  function addParams() {
    addParamDialog.value = true;
  }

  function clickAdd(node) {
    console.log('add：', node);
    curNode.value = node;
    dialogVisible.value = true;
  }

  function clickRm(node) {
    console.log(node);
    delete nodes.value[node.name];
    // nodes.value = nodes.value.filter((item) => item.id !== node.id);
    console.log('rm');
  }

  function addOption() {
    CreatedNodeDialog.value = true;
  }

  function clickAdd2Option(node) {
    let n_option = form.value as NodeOption;
    if (n_option.options == undefined) {
      n_option.options = [];
    }

    let node_obj = Object.assign({}, node);
    // if (curNode.value != undefined) {
    n_option.options.push(node_obj);
    // }
    form.value = n_option;
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
