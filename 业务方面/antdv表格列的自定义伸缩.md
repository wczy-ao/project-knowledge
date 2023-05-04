# antdv表格列的自定义伸缩

ant-design-vue是不具备这个功能的，element-ui具有这个功能；



为了在antdv上实现这个功能，我们需要一个依赖，`vue-draggable-resizable`



### 安装：

```
yarn add vue-draggable-resizable
```



### 使用：

`dragTableColum.js`

```js
import Vue from 'vue';
import VueDraggableResizable from 'vue-draggable-resizable';

Vue.component('vue-draggable-resizable', VueDraggableResizable);
const columns = [
  {
    title: '序号',
    scopedSlots: {
      customRender: 'index',
    },
    key: 'index',
    align: 'center',
    width: 50,
  },
  {
    title: '点位名称',
    dataIndex: 'name',
    width: 100,
    align: 'center',
    key: 'name',
  },
  {
    title: '操作',
    scopedSlots: {
      customRender: 'operate',
    },
    width: 100,
    ellipsis: true,
    key: 'operate',
    align: 'center',
  },
];
const draggableTable = {
  header: {
    cell: (h, props, children) => {
      const { key, ...restProps } = props;

      const col = columns.find(col => {
        const k = col.dataIndex || col.key;
        return k === key;
      });

      if (!col || !col.width) {
        return h('th', { ...restProps }, [...children]);
      }

      const dragProps = {
        key: col.dataIndex || col.key,
        class: 'table-draggable-handle',

        attrs: {
          w: 4,
          x: col.width,
          z: 20,
          axis: 'x',
          draggable: true,
          resizable: false,
        },
        on: {
          dragging: x => {
            col.width = Math.max(x, 1);
          },
        },
      };
      const drag = h('vue-draggable-resizable', { ...dragProps });
      return h('th', { ...restProps, class: 'resize-table-th' }, [...children, drag]);
    },
  },
};

export { draggableTable, columns };

```



- 需要注意的是`draggableTable`不用修改。照着写就行
- `column`的列必须具有`dataIndex`和`width`必须是整数



`组件中使用`

```vue
<template>
<div class="center">
  <a-table
    :scroll="{ y: 'calc(100% - 28px)' }"
    :components="components"
    :columns="columns"
    :data-source="dataMessage"
    :rowKey="row => row.id"
    :rowClassName="rowClassName"
    :pagination="false"
  >
    <template slot="index" slot-scope="text, record, index">
      {{ rowIndex + index + 1 }}
    </template>
    <template slot="imageState" slot-scope="text, record">
      <span :style="`color:${addDrawColor(record.navigationId)}`">
          {{ addDrawPoint(record.navigationId) }}
        </span>
      </template>
      <template slot="operate" slot-scope="text, record">
        <span
        class="operate-choice"
        :class="{ 'last-deploy': record.id === lastDeployId }"
       	@click="choicePointDeploy(record)">
       选择
     </span>
   </template>
 </a-table>
</template>
<script>

import { draggableTable, columns } from './pointDeployList.js';
export default {
  data() {
    this.components = draggableTable;
    return {
      columns,
    };
  },
};
</script>

<style lang="scss" scoped>
 .center {
   position: relative;
   width: 100%;
   height: 100%;
   height: 400px;
   overflow: hidden;
   .operate-choice {
     color: rgb(87, 165, 255);
     cursor: pointer;
     &:hover {
       color: green;
     }
   }

   .resize-table-th {
     position: relative;
   }

   .table-draggable-handle {
     transform: none;
     position: absolute;
     height: 100% !important;
     left: 0px !important;
     cursor: col-resize;
     touch-action: none;
     border: none;
     top: 0px;
     height: 26px;
     background-color: #f0f0f0;
   }
   .ant-table-wrapper {
     width: 100%;
     height: calc(100% - 50px);
   }
 }
</style>

```

- `a-table`添加`:components="components"`
- 样式设置，要不然会出问题