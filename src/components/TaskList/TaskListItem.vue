<template>
<div class="elastigantt__task-list-item" :style="root.state.taskList.styles.row">
  <item-column v-for="column in root.state.taskList.columns" :key="column._id" :column="column" :task="task">
    <task-list-expander v-if="column.expander" :tasks="[task]" :style="expanderStyle"></task-list-expander>
  </item-column>
</div>
</template>
<script>
import TaskListExpander from './Expander.vue';
import ItemColumn from './ItemColumn.vue';

export default {
  components: {
    'task-list-expander': TaskListExpander,
    'item-column': ItemColumn
  },
  inject: ['root'],
  props: ['task', 'expanderStyle'],
  data() {
    return {};
  },
}
</script>
