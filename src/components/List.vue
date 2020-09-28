<template>
  <ul :style="`max-height: ${containerHeightMax}px`" ref="listContainerRef">
    <div id="itemContainerRef" ref="itemContainerRef">
      <!-- 占位符最大高度要保证 rowHeightMin * preRenderSize，即初始不能进入可视区域 -->
      <span id="upPlaceholderRef" ref="upPlaceholderRef" :style="`height: ${rowHeightMin * preRenderSize}px`"></span>
      <li v-for="(i, index) in renderData" :key="i.id || index" :style="`min-height: ${rowHeightMin}px`">
        <slot :row="i">
          {{ i.content || i }}
        </slot>
      </li>
      <span id="downPlaceholderRef" ref="downPlaceholderRef" :style="`height: ${rowHeightMin * preRenderSize}px`"></span>
    </div>
    <div class="heightPlaceholder" :style="`height: ${dataHeight}px`"></div>
  </ul>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, onBeforeUnmount } from 'vue'

export interface ListDataProps {
  id: Number;
  content: String;
}

export default defineComponent({
  name: 'List',
  props: {
    data: {
      type: Array,
      default: []
    },
    containerHeightMax: {
      type: Number,
      default: 420
    },
    rowHeightMin: {
      type: Number,
      default: 30
    },
    // 预渲染数据大小，避免白屏问题；也是每次更新数据的步长
    preRenderSize: {
      type: Number,
      default: 3
    }
  },
  setup(props) {
    const { data, containerHeightMax, rowHeightMin, preRenderSize } = props;
    // 计算一屏真实渲染数据的最大长度
    const RenderSize = computed(() => Math.ceil(containerHeightMax / rowHeightMin));
    // 数据高度最小值，作为占位符高度
    const dataHeight = computed(() => data.length * rowHeightMin);
    // 渲染区域
    const start = ref(0);
    const end = computed(() => start.value + RenderSize.value);
    const renderData = computed(() => data.slice(
      Math.max(start.value - preRenderSize, 0),
      Math.min(end.value + preRenderSize, data.length)
    ));
    // 上下占位符标记dom引用
    const upPlaceholderRef = ref(null);
    const downPlaceholderRef = ref(null);
    const itemContainerRef = ref(null);
    const listContainerRef = ref(null);
    // 监听者
    const observer = ref(null);

    onMounted(() => {
      if ("IntersectionObserver" in window) {
        observer.value = new IntersectionObserver(arr => {
          arr.forEach(ob => {
            // 上下边界进入可视区域，可视为对应方向的移动
            if (ob.isIntersecting) {
              /*
                特定偏移量，产生滑动假象，具体解释见下；
                这里只是一个预估最小值，要精确滑动需要获取具体高度，影响性能
              */
              const moveLength = rowHeightMin * preRenderSize;
              if (ob.target.id === "upPlaceholderRef") {
                /*
                  此时，下方最小有 preRenderSize 数量的dom节点进入不可视区域；
                  所以应该将下方dom节点移至上方，并插入上方 preRenderSize 数量的新数据；
                  从另一个角度看，不移动dom只改变数据，即是将数据切片前移 preRenderSize 大小。(当然从底层来看Vue的Diff过程就是对dom的位置移动)
                  同时为了保证dom节点的位置一致，对整体列表做向上特定值的偏移。
                */
                start.value = Math.max(0, start.value - preRenderSize);
                itemContainerRef.value.style.top = `${Math.max(0, parseInt(itemContainerRef.value.style.top || 0) - moveLength)}px`;
              } else {
                // 这里可以引入分页懒加载功能，不修改 start 根据下标读取而是直接发起请求
                start.value = Math.min(data.length - 1, start.value + preRenderSize);
                itemContainerRef.value.style.top = `${parseInt(itemContainerRef.value.style.top || 0) + moveLength}px`;
              }
            }
          })
        }, {
          root: listContainerRef.value
        });

        observer.value.observe(upPlaceholderRef.value);
        observer.value.observe(downPlaceholderRef.value);
      } else {
        // IE兼容处理，采用 polyfill 或者 scroll 监听
        console.error('IE兼容性问题，请使用其他浏览器');
      }
    })

    // 清除监听
    onBeforeUnmount(() => {
      observer.value.disconnect && observer.value.disconnect();
    })

    return {
      renderData,
      containerHeightMax,
      rowHeightMin,
      dataHeight: dataHeight.value,
      upPlaceholderRef,
      downPlaceholderRef,
      itemContainerRef,
      listContainerRef
    };
  }
});
</script>

<style lang="less" scoped>
  @row-border-color: #e8e8e8;
  
  ul {
    position: relative;
    left: 0;
    top: 0;
    width: 400px;
    border: solid 1px @row-border-color;
    border-radius: 2px;
    list-style-type: none;
    padding: 20px;
    overflow-y: scroll;

    li {
      display: flex;
      align-items: center;
      text-align: left;
      border-bottom: solid 1px @row-border-color;
      padding: 5px 10px;
      word-break: break-all;
    }
    li:last-child {
      border-bottom: none;
    }

    .heightPlaceholder {
      position: absolute;
      left: 0;
      top: 0;
      right: 0;
      z-index: -1;
    }

    #itemContainerRef {
      position: relative;
      top: 0;
      left: 0;
    }

    #upPlaceholderRef, #downPlaceholderRef {
      position: absolute;
      left: 0;
      width: 100%;
      z-index: -1;
    }
    #upPlaceholderRef {
      top: 0;
    }
    #downPlaceholderRef {
      bottom: 0;
    }
  }
</style>

