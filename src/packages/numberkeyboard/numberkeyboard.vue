<template>
  <transition name="nut-board-slide-up" @after-enter="afterEnter" @after-leave="afterLeave">
    <div class="nut-numberkeyboard" v-show="visible">
      <div class="number-board-header">
        <h3 class="tit">{{ title }}</h3>
        <span class="keyboard-close" @click="() => closeBoard()" v-if="type == 'default'">完成</span>
      </div>
      <div class="number-board-body">
        <div class="number-board">
          <div class="key-board-wrapper" v-for="item of keysList" :key="'key' + item.id">
            <div
              :class="['key', { active: item.id == clickKeyIndex }, { lock: item.type == 'lock' }, { delete: item.type == 'delete' }]"
              @touchstart="event => onTouchstart(item, event)"
              @touchmove="event => onTouchMove(item, event)"
              @touchend="onTouchEnd"
            >
              <template v-if="item.type == 'number' || item.type == 'custom'">{{ item.id }}</template>
              <img
                v-if="item.type == 'lock'"
                src="https://img11.360buyimg.com/imagetools/jfs/t1/146371/38/8485/738/5f606425Eca239740/14f4b4f5f20d8a68.png"
              />
              <img
                v-if="item.type == 'delete'"
                src="https://img11.360buyimg.com/imagetools/jfs/t1/129395/8/12735/2030/5f61ac37E70cab338/fb477dc11f46056c.png"
              />
            </div>
          </div>
        </div>
        <div class="number-board-sidebar" v-if="type == 'rightColumn'">
          <div class="key-board-wrapper">
            <div
              :class="['key', { active: clickKeyIndex == 'delete' }]"
              @touchstart="event => onTouchstart({ id: 'delete', type: 'delete' }, event)"
              @touchmove="event => onTouchMove({ id: 'delete', type: 'delete' }, event)"
              @touchend="onTouchEnd"
            >
              <img src="https://img11.360buyimg.com/imagetools/jfs/t1/129395/8/12735/2030/5f61ac37E70cab338/fb477dc11f46056c.png" />
            </div>
          </div>
          <div class="key-board-wrapper" @click="() => closeBoard()">
            <div :class="['key', 'finish', { activeFinsh: clickKeyIndex == 'finish' }]">
              完成
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>
<script>
export default {
  name: 'nut-numberkeyboard',
  props: {
    title: {
      type: String,
      default: ''
    },
    visible: {
      type: Boolean,
      default: false
    },
    type: {
      type: String,
      default: 'default'
    },
    customKey: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    keysList() {
      if (this.type == 'rightColumn') {
        return this.genCustomKeys();
      }
      return this.defaultKey();
    }
  },
  data() {
    return {
      once: 0,
      keys: [],
      clickKeyIndex: undefined
    };
  },
  beforeDestroy() {
    document.getElementsByTagName('body')[0].removeEventListener('click', () => {});
  },
  mounted() {
    let that = this;
    this.$nextTick(() => {
      let targetArea = that.$el;
      document.getElementsByTagName('body')[0].addEventListener('click', function(event) {
        let e = event || window.event;
        let elem = e.target;
        if (targetArea.contains(elem)) {
          // console.log("在区域内");
        } else {
          // console.log("在区域外");
          if (that.once == 0 && that.visible) {
            that.once += 1;
            return false;
          }
          that.closeBoard();
        }
      });
    });
  },
  methods: {
    getBasicKeys() {
      const keys = [];
      for (let i = 1; i <= 9; i++) {
        keys.push({ id: i, type: 'number' });
      }
      return keys;
    },
    defaultKey() {
      return [...this.getBasicKeys(), { id: 'lock', type: 'lock' }, { id: 0, type: 'number' }, { id: 'delete', type: 'delete' }];
    },
    genCustomKeys() {
      const keys = this.getBasicKeys();
      const { customKey } = this;
      const customKeys = Array.isArray(customKey) ? customKey : [customKey];
      if (customKeys.length === 1) {
        keys.push({ id: 0, type: 'number' }, { id: customKeys[0], type: 'custom' });
      } else if (customKeys.length === 2) {
        keys.push({ id: customKeys[0], type: 'custom' }, { id: 0, type: 'number' }, { id: customKeys[1], type: 'custom' });
      }
      return keys;
    },
    afterEnter() {
      this.$emit('enter');
    },
    afterLeave() {
      this.$emit('leave');
    },
    onTouchstart(item, event) {
      event.stopPropagation();
      this.clickKeyIndex = item.id;
      if (item.type == 'number' || item.type == 'custom') {
        this.$emit('input', item.id);
      }
      if (item.type == 'lock') {
        this.closeBoard();
      }
      if (item.type == 'delete') {
        this.$emit('delete');
      }
    },
    onTouchMove(id, event) {
      event.stopPropagation();
      // this.clickKeyIndex = id;
    },
    onTouchEnd() {
      this.clickKeyIndex = undefined;
    },
    closeBoard() {
      if (this.once > 0) {
        this.once = 0;
        this.$emit('close');
      }
    }
  }
};
</script>
