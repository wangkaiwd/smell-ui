<template>
  <div
    class="smile-carousel"
    @mouseenter="pause"
    @mouseleave="doAutoPlay"
    @touchstart="onTouchStart"
    @touchend="onTouchEnd"
  >
    <slot></slot>
    <div class="smile-carousel-arrow">
      <span
        class="smile-carousel-arrow-left"
        @click="updateSelect(activeChildIndex-1)"
      >
        <smile-icon icon="left"></smile-icon>
      </span>
      <span
        class="smile-carousel-arrow-right"
        @click="updateSelect(activeChildIndex+1)"
      >
        <smile-icon icon="right"></smile-icon>
      </span>
    </div>
    <ul class="smile-carousel-controls" v-if="controls">
      <li
        v-for="i in childLength"
        :key="i"
        @click="updateSelect(i-1)"
      >
        <span
          class="smile-carousel-controls-inner"
          :class="{active: i === activeChildIndex+1}"
        >
        </span>
      </li>
    </ul>
  </div>
</template>

<script>
  /**
   * 用户操作的一种特殊情况：
   *    1. 如果用户将鼠标移入到carousel中刷新页面，会导致移入事件直接触发，之后继续执行autoPlay
   *    2. 此时当鼠标移出的时候，会继续执行autoPlay，而此时timerId是存在的
   *    3. 所以为了免去一些不必要的问题，在每次autoPlay之前要判断一下timerId是否存在，防止重复触发定时器
   *    4. 对应的也要在移除定时器的时候将timerId置为null
   */
  export default {
    name: 'SmileCarousel',
    props: {
      select: {
        type: String,
        required: true
      },
      autoPlay: {
        type: Boolean,
        default: false
      },
      controls: {
        type: Boolean,
        default: true
      },
      interval: {
        type: Number,
        default: 2000
      }
    },
    data () {
      return {
        childLength: 0,
        timerId: null,
        startX: null,
        startY: null
      };
    },
    computed: {
      activeChildIndex () {
        return this.names.indexOf(this.select);
      },
      // this.$children: 当前实例的直接子组件。
      // slot中的组件和非slot中的组件都会算入其中，比如这里使用到的icon组件，所以这里要只需要carouseItem组件
      // 这里也体现了为一个组件添加name属性的用途
      names () {
        return this.$children.filter(vm => vm.$options.name === 'SmileCarouselItem')
          .map(vm => vm.name);
      }
    },
    mounted () {
      this.doAutoPlay();
      this.childLength = this.names.length;
    },
    methods: {
      doAutoPlay () {
        if (!this.autoPlay) return;
        // 若果有timerId的话就不会再次进行自动播放
        if (this.timerId) return;
        this.timerId = setInterval(() => {
          // 每次都要重新获取当前的索引
          let index = this.activeChildIndex;
          index++;
          this.updateSelect(index);
        }, this.interval);
      },
      /**
       * 动画方向分析：
       *  0-2: 逆向动画
       *  2-0: 正向动画
       * 根据当前索引和下一个要进入item的索引，来确定所有的item的动画执行方向
       */
      setChildReverse (reverse) {
        this.$children.map(vm => vm.reverse = reverse);
      },
      updateSelect (index) {
        this.setChildReverse(index < this.activeChildIndex);
        if (index < 0) {
          index = this.names.length - 1;
        }
        if (index > this.names.length - 1) {
          index = 0;
        }
        // 切换动画在方向变化之前执行了，要在dom更新完之后再进行渲染更新select
        this.$nextTick(() => {
          this.$emit('update:select', this.names[index]);
        });
      },
      pause () {
        if (this.timerId) {
          clearInterval(this.timerId);
          this.timerId = null;
        }
      },
      /**
       * 移动端的轮播处理：
       *    判断滑动方向：a.左划 b.右划
       *  常用的触摸事件
       *    touchstart: 当用户在触摸平面上放置了一个触点时触发。事件的目标`element`将是触点位置上
       *                的那个目标element
       *    touchend: 当一个触点被用户从触摸平面上移除（当用户将一个手指离开）时触发。当触点移出
       *              触摸平面的边界时也将触发。例如用户将手指划出屏幕边缘。已经被从触摸平面上移除的触点，
       *              可以在changedTouches属性定义的TouchList中找到
       *    touchmove: 当用户在触摸平面上移动触点时触发。
       *
       *  触摸事件相关的3个属性：
       *    touches: 包含了所有当前接触触摸平面的触点的Touch对象，无论它们起始于哪个element上，也无论它们状态是否发生了变化
       *    targetTouches: 触摸事件起始于当前事件的目标element上，并且仍然没有离开触摸平面的触点
       *    changedTouches: 包含了代表所有从上一次触摸事件到此次事件过程中，状态发生了变化的触点的Touch对象
       */
      onTouchStart (e) {
        this.pause();
        const touch = e.targetTouches[0]; // 获取触摸目标元素上的Touch对象
        this.startX = touch.clientX;
        this.startY = touch.clientY;
      },
      onTouchEnd (e) {
        const touch = e.changedTouches[0]; // 离开触摸平面的Touch对象
        const endX = touch.clientX;
        const endY = touch.clientY;
        const rate = Math.abs(endY - this.startY) / Math.abs(endX - this.startX);
        if (rate < 1) {
          if (endX > this.startX) {
            this.updateSelect(this.activeChildIndex - 1);
          } else {
            this.updateSelect(this.activeChildIndex + 1);
          }
        }
        this.doAutoPlay();
      },
    }
  };
</script>

<style lang="scss" scoped>
  @import "~styles/vars";
  @import "~styles/mixins";

  .smile-carousel {
    position: relative;
    display: inline-block;
    width: 100%;
    overflow: hidden;
    &-controls {
      position: absolute;
      bottom: $space-md;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      align-items: center;
      li {
        cursor: pointer;
        padding: $space-xs 0;
        margin: 0 $space-xs;
      }
      &-inner {
        display: block;
        width: 30px;
        height: 3px;
        background-color: $secondary;
        &.active {
          background-color: $white;
        }
      }
    }
    &-arrow-left,
    &-arrow-right {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      color: $secondary;
      font-size: 30px;
      &:hover {
        cursor: pointer;
        color: $white;
      }
    }
    &-arrow-left {
      left: $space-lg;
    }
    &-arrow-right {
      right: $space-md;
    }
  }
</style>
