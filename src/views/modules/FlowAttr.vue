<template>
  <a-tabs size="small" :activeKey="activeKey">
    <!-- 流程属性 -->
    <a-tab-pane :key="ActiveTypeEnum.CANVAS" v-if="activeKey === ActiveTypeEnum.CANVAS">
      <template #tab>
        <span>
          <component :is="'ClusterOutlined'" />
          流程属性
        </span>
      </template>
      <a-form layout="vertical">
        <a-form-item label="流程id">
          <a-input :value="flowData.attr.id" disabled />
        </a-form-item>
      </a-form>
    </a-tab-pane>

    <!-- 节点属性 -->
    <a-tab-pane :key="ActiveTypeEnum.NODE" v-if="activeKey === ActiveTypeEnum.NODE">
      <template #tab>
        <span>
          <component :is="'ProfileOutlined'" />
          节点属性
        </span>
      </template>
      <a-form layout="vertical">
        <a-form-item label="类型">
          <a-tag color="purple">{{ currentSelect.type }}</a-tag>
        </a-form-item>
        <a-form-item label="id">
          <a-input :value="currentSelect.id" disabled />
        </a-form-item>
        <a-form-item label="名称" v-if="isAllowChange(currentSelect.type as NodesType)">
          <a-input
            placeholder="请输入节点名称"
            :value="(currentSelect as INode)?.nodeName"
            @change="nodeNameChange"
          />
        </a-form-item>
        <a-form-item label="JSON输出">
          <json-viewer :value="(currentSelect as INode).data" :expand-depth="3" boxed copyable />

          <!-- <Codemirror
            :value="yaml.dump((currentSelect as INode).data)"
            :options="cmOptions"
            ref="cmRef"
            height="400"
            @change="onChange"
            @input="onInput"
            @ready="onReady"
          /> -->
          <!-- <a-textarea v-model:value="(currentSelect as INode).data" /> -->
        </a-form-item>
        <a-form-item label="YAML输出">
          <a-textarea :value="yaml.dump((currentSelect as INode).data)" style="height: 200px" />
        </a-form-item>
      </a-form>
    </a-tab-pane>

    <!-- 连线属性 -->
    <a-tab-pane :key="ActiveTypeEnum.CONNECTION" v-if="activeKey === ActiveTypeEnum.CONNECTION">
      <template #tab>
        <span>
          <component :is="'BranchesOutlined'" />
          连线属性
        </span>
      </template>
      <a-form layout="vertical">
        <a-form-item label="id">
          <a-input :value="currentSelect.id" disabled />
        </a-form-item>
        <a-form-item label="源节点">
          <a-input :value="(currentSelect as ILink)?.sourceId" disabled />
        </a-form-item>
        <a-form-item label="目标节点">
          <a-input :value="(currentSelect as ILink)?.targetId" disabled />
        </a-form-item>
        <a-form-item label="文本">
          <a-input :value="(currentSelect as ILink)?.label" @change="linkLabelChange" />
        </a-form-item>
      </a-form>
    </a-tab-pane>
  </a-tabs>
</template>

<script lang="ts" setup>
  import { ChangeEvent } from 'ant-design-vue/lib/_util/EventInterface';
  import { ref, watch, unref, PropType, reactive } from 'vue';
  import { JsonViewer } from 'vue3-json-viewer';
  import 'vue3-json-viewer/dist/index.css';
  // import 'codemirror/mode/javascript/javascript.js';
  // import Codemirror from 'codemirror-editor-vue3';
  import yaml from 'js-yaml';
  // import type { CmComponentRef } from 'codemirror-editor-vue3';
  // import type { Editor, EditorConfiguration } from 'codemirror';

  import { INode, ILink, NodesType } from '/@/type/index';
  import { CommonNodeTypeEnum, ActiveTypeEnum } from '/@/type/enums';
  // const cmOptions: EditorConfiguration = reactive({
  //   mode: 'text/x-yaml',
  //   theme: 'default',
  //   readOnly: false,
  //   lineNumbers: true,
  //   lineWiseCopyCut: true,
  //   gutters: ['CodeMirror-lint-markers'],
  //   lint: true,
  // });
  // const onChange = (val: string, cm: Editor) => {
  //   console.log(val);
  //   console.log(cm.getValue());
  // };

  // const onInput = (val: string) => {
  //   console.log(val);
  // };

  // const onReady = (cm: Editor) => {
  //   console.log(cm.focus());
  // };

  const props = defineProps({
    plumb: {
      type: Object,
      default: () => ({}),
    },
    flowData: {
      type: Object,
      default: () => ({}),
    },
    select: {
      type: Object as PropType<INode | ILink>,
      default: () => ({}),
    },
  });

  const emits = defineEmits(['update:select']);

  const currentSelect = ref<INode | ILink>(props.select);

  const activeKey = ref<string>(ActiveTypeEnum.CANVAS);

  // 修改节点名称
  function nodeNameChange(e: ChangeEvent) {
    (currentSelect.value as INode).nodeName = e.target.value ?? '';
  }

  // 是否可以修改节点名称
  function isAllowChange(type: NodesType) {
    return ![
      CommonNodeTypeEnum.START,
      CommonNodeTypeEnum.END,
      CommonNodeTypeEnum.GATEWAY,
      CommonNodeTypeEnum.EVENT,
    ].includes(type as CommonNodeTypeEnum);
  }

  // 修改连接文本
  function linkLabelChange(e: ChangeEvent) {
    let label = e.target.value ?? '';
    (currentSelect.value as ILink).label = label;
    let conn = props.plumb.getConnections({
      source: (unref(currentSelect) as ILink).sourceId,
      target: (unref(currentSelect) as ILink).targetId,
    })[0];
    let link_id = conn.canvas.id;
    let labelHandle = (e: Event) => {
      e.stopPropagation();
      currentSelect.value = props.flowData.linkList.find((l: ILink) => l.id === link_id);
    };

    if (label !== '') {
      conn.setLabel({
        label: label,
        cssClass: `linkLabel ${link_id}`,
      });
      // 添加label点击事件
      document.querySelector('.' + link_id)?.addEventListener('click', labelHandle);
    } else {
      // 移除label点击事件
      document.querySelector('.' + link_id)?.removeEventListener('click', labelHandle);

      let labelOverlay = conn.getLabelOverlay();
      if (labelOverlay) conn.removeOverlay(labelOverlay.id);
    }
  }

  watch(
    () => props.select,
    (val) => {
      currentSelect.value = val;
      if (!unref(currentSelect)?.type) {
        activeKey.value = ActiveTypeEnum.CANVAS;
      } else if (unref(currentSelect)?.type === 'link') {
        activeKey.value = ActiveTypeEnum.CONNECTION;
      } else {
        activeKey.value = ActiveTypeEnum.NODE;
      }
    },
    { deep: true },
  );

  watch(
    () => currentSelect.value,
    (currentSelect) => {
      console.log('select change!');
      let strnode = JSON.stringify(currentSelect);
      console.log(strnode);
      console.log('currentSelect value:', currentSelect);
      emits('update:select', currentSelect);
    },
  );
</script>
