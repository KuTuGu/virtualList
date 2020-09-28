<template>
  <List
    :data="listData"
  >
    <template v-slot="scope">
      {{scope.row.content}}
    </template>
  </List>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import List, { ListDataProps } from './components/List.vue'

export default defineComponent({
  name: 'App',
  components: {
    List
  },
  setup() {
    const listData = ref(getListData());

    return {
      listData
    };
    
    function getListData(): Array<ListDataProps> {
      const arr = [];
      const len = 1000;

      for (let i = 0;i < len;i++) {
        arr.push({
          id: i,
          content: `${i}. ${"t".repeat(i)}`
        })
      }

      return arr;
    }
  }
});
</script>