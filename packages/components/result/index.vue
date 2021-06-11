<template>
  <div
    class="key-board-result"
    :style="{ color }"
    v-if="(status === 'CN' && data.code) || status === 'handwrite'"
  >
    <div class="key-board-code-show" v-if="status === 'CN'">
      {{ data.code }}
    </div>
    <div class="key-board-result-show">
      <div class="hg-candidate-box">
        <div class="hg-candidate-box-prev"
             :class="{'hg-candidate-box-btn-active': showIndex + 1 > 1}"
             @click="upper"></div>
        <ul class="hg-candidate-box-list">
          <li class="hg-candidate-box-list-item" v-for="(key, index) in showList[showIndex]"
              :key="index"
              @click="selectWord(key)"
          >{{ key }}</li>
        </ul>
        <div class="hg-candidate-box-next"
             :class="{'hg-candidate-box-btn-active': showIndex + 1 < showList.length}"
             @click="lower"></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import type { PropType } from "vue";
import { groupSplitArray } from "@/utils";
import { IResultData, IValue } from "@/typings";
import { getInject } from "@/context/keyboardContext";
import useEventEmitter from "@/hooks/useEventEmitter";
import {
  watch,
  toRefs,
  computed,
  reactive,
  onMounted,
  onUnmounted,
  defineComponent,
} from "vue";
export default defineComponent({
  props: {
    data: Object as PropType<IValue>,
  },
  emits: ["change"],
  setup(props: { data: IValue }, { emit }) {
    const injectData = getInject();
    const getStyle = computed(() => ({
      borderTopColor: injectData?.color,
    }));
    const resultData = reactive<IResultData>({
      status: "",
      valueList: [],
      showList: [],
      showIndex: 0,
    });

    /**
     * @description 上一页
     */
    function upper() {
      if (resultData.showIndex === 0) return;
      resultData.showIndex -= 1;
    }

    /**
     * @description 下一页
     */
    function lower() {
      if (resultData.showIndex === resultData.showList.length - 1) return;
      resultData.showIndex += 1;
    }

    /**
     * @description 重置
     */
    function reset() {
      resultData.showIndex = 0;
      resultData.showList = [];
      resultData.valueList = [];
      useEventEmitter.emit("resultReset");
    }

    /**
     * @description 上一页
     */
    function selectWord(word) {
      reset();
      emit("change", word);
    }

    // 监听data的变化
    watch(
      () => props.data,
      (newVal) => {
        resultData.showIndex = 0;
        resultData.valueList = newVal?.value?.split("") || [];
        if (resultData.valueList.length === 0) {
          resultData.showList = [];
          return;
        }
        resultData.showList = groupSplitArray(resultData.valueList, 11);
      },
      {
        immediate: true,
      }
    );

    onMounted(() => {
      useEventEmitter.on("keyBoardChange", (status: string) => {
        // 会引起高度变化 需要重新计算画板
        useEventEmitter.emit("updateBound");
        resultData.status = status;
        reset();
      });

      useEventEmitter.on("getWordsFromServer", (serverData: string) => {
        const _valueList = Array.from(
          new Set(serverData.replace(/\s+/g, "").split(""))
        );
        resultData.valueList = _valueList;
        resultData.showList = groupSplitArray(_valueList, 11);
      });
    });

    onUnmounted(() => {
      useEventEmitter.remove("keyBoardChange");
      useEventEmitter.remove("getWordsFromServer");
    });

    return {
      color: injectData?.color,
      upper,
      lower,
      getStyle,
      selectWord,
      ...toRefs(resultData),
    };
  },
});
</script>

<style scoped lang='less'>
.key-board-result {
  width: 100%;
  height: 139px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  position: absolute;
  top: -145px;
  margin-left: 20px;

  .key-board-code-show {
    height: 40px;
    background: #f5f5f5;
    padding: 5px;
    border-radius: 5px 5px 0 0;
    font-size: 40px;
    font-weight: 400;
    line-height: 40px;
    transform: translateY(-6px);
    z-index: -1;
  }

  .key-board-result-show {
    display: flex;
    align-items: center;
    flex-wrap: nowrap;
    flex: 1;
    font-size: 50px;

    .hg-candidate-box {
      background:#ececec;
      border-bottom:2px solid #b5b5b5;
      border-radius:5px;
      display:inline-flex;
      margin-top:-10px;
      user-select:none
    }
    ul.hg-candidate-box-list {
      display:flex;
      flex:1;
      list-style:none;
      margin:0;
      padding:0
    }
    li.hg-candidate-box-list-item {
      align-items:center;
      display:flex;
      height:100px;
      justify-content:center;
      width:100px
    }
    li.hg-candidate-box-list-item:hover {
      background:rgba(0,0,0,.03);
      cursor:pointer
    }
    li.hg-candidate-box-list-item:active {
      background:rgba(0,0,0,.1)
    }
    .hg-candidate-box-prev:before {
      content:"◄"
    }
    .hg-candidate-box-next:before {
      content:"►"
    }
    .hg-candidate-box-next,
    .hg-candidate-box-prev {
      align-items:center;
      background:#d0d0d0;
      color:#969696;
      cursor:pointer;
      display:flex;
      padding:0 10px
    }
    .hg-candidate-box-next {
      border-bottom-right-radius:5px;
      border-top-right-radius:5px
    }
    .hg-candidate-box-prev {
      border-bottom-left-radius:5px;
      border-top-left-radius:5px
    }
    .hg-candidate-box-btn-active {
      color:#444
    }
  }
}
</style>
