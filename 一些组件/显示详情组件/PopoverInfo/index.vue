<template>
  <el-popover
    id="popover"
    ref="popover"
    popper-class="popover-info"
    :trigger="triggerType"
    placement="right"
    :visible-arrow="false"
    @show="checkShow"
    @hide="resetOverflow"
  >
    <!-- :popper-options="popperOptions" -->
    <div slot="reference">
      <slot name="default">
        触发按钮地方
      </slot>
    </div>
    <!-- :style="{ position: 'fixed', top: `${remPosition.top}rem`, left: `${remPosition.left}rem` }" -->
    <div class="popover-info-container" ref="popoverContainer">
      <div class="info-title-wrap flex-between">
        <div class="info-name">
          {{ infoData.name }}
        </div>
        <div class="info-type">
          {{ labelType }}
        </div>
      </div>
      <div class="prop-wrap">
        {{ infoData.propDesc }}
      </div>
      <div class="number-cover-wrap flex-start">
        <div class="info-prop-title">
          覆盖人数<el-tooltip class="item" placement="bottom" popper-class="data-addpackageTip count-type__tooltip">
            <div slot="content">在该标签维度下可识别出的人数</div>
            <i class="explain"></i> </el-tooltip
          >：
        </div>
        <div class="number-info-wrap">{{ formatNum(infoData.number) }}({{ infoData.percent }})</div>
      </div>
      <div class="caliber-desc-wrap flex-start">
        <div class="info-prop-title info-prop-title-caliber">口径描述：</div>
        <div
          id="textContainer"
          class="caliber-info-wrap text-container"
          ref="textContainer"
          v-show="isTextContainerVisible"
        >
          <div
            class="celiber-info-content text-content"
            id="textContent"
            ref="textContent"
            v-html="formatContent(infoData.caliberDesc) || '-'"
          ></div>
          <span ref="ellipsis" v-if="isOverflow && !hasExpand" class="ellipsis" id="ellipsis">...</span>
          <button ref="expandButton" v-if="isOverflow" class="expand-button" id="expandButton" @click="toggleExpand">
            {{ (hasExpand && '收起') || '展开' }}
          </button>
        </div>
      </div>
    </div>
  </el-popover>
</template>

<script>
import { numFormat } from '@/utils';

export default {
  props: {
    triggerType: {
      default: 'hover',
      type: String
    },
    infoData: {
      default: () => ({
        type: 'int',
        name: '性别',
        caliberDesc: '大河向东流啊，天上的星星参北斗啊',
        number: '11111万',
        percent: '26.73%',
        propDesc: 'propDesc sex'
      }),
      type: Object
    }
  },
  computed: {
    labelType() {
      const conf = {
        1: 'string',
        2: 'date',
        3: 'int',
        4: 'bigInt',
        5: 'float'
      };

      return conf[this.infoData.type + ''];
    }
  },
  data() {
    return {
      remPosition: { top: 0, left: 0 },
      popperOptions: {
        modifiers: {
          preventOverflow: {
            boundariesElement: 'viewport' // 设置边界为视口
          },
          flip: {
            enabled: true
          }
        }
      },
      hasExpand: false,
      isTextContainerVisible: false,
      isOverflow: false
    };
  },
  methods: {
    manualPositioning() {
      const reference = this.$refs.popover.$el;

      const rect = reference.getBoundingClientRect();
      const topRem = rect.bottom / parseFloat(getComputedStyle(document.documentElement).fontSize);
      const leftRem = rect.left / parseFloat(getComputedStyle(document.documentElement).fontSize);

      this.remPosition = { top: topRem, left: leftRem };
    },
    isElementInViewport(element) {
      var rect = element && element.getBoundingClientRect();

      return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
        rect.right <= (window.innerWidth || document.documentElement.clientWidth)
      );
    },
    formatContent(text) {
      // console.log('text', JSON.stringify(text));
      if (!text) {
        return '-';
      }
      const str = JSON.stringify(text);
      return str.replace(/\\n/g, '<br/>').replace(/"/g, '');
    },
    resetOverflow() {
      this.$refs.textContainer.classList.remove('expanded');
      this.hasExpand = false;
    },
    formatNum(num) {
      return numFormat(num === undefined ? 0 : num);
    },
    checkShow() {
      this.isTextContainerVisible = true;
      this.$nextTick(() => {
        this.checkOverflow();
      });
    },
    checkOverflow() {
      const textContainer = this.$refs.textContainer;
      const textContent = this.$refs.textContent;

      this.isOverflow = textContent.offsetHeight > textContainer.offsetHeight;
    },
    toggleExpand() {
      this.hasExpand = !this.hasExpand;
      this.$refs.textContainer.classList.toggle('expanded');
      // console.log(this.$refs.textContainer, 'this.$refs.textContainer');

      // 如果展开后内容超出屏幕需要重新计算
      // if (this.hasExpand) {
      //   this.$nextTick(() => {
      //     this.$nextTick(() => {
      //       console.log(this.isElementInViewport(this.$refs.popoverContainer), 'hi?');
      //       if (!this.isElementInViewport(this.$refs.popoverContainer)) {
      //         // this.$refs.popover.updatePopper();
      //         this.manualPositioning();
      //       }
      //     });
      //   });
      // }

      // const textContainer = document.querySelector('.text-container');
      // textContainer.classList.toggle('expanded');
    }
  }
  // watch: {
  //   infoData: {
  //     immediate: true,
  //     handler(v) {
  //       if (v) {
  //         this.$nextTick(() => {
  //           this.checkOverflow();
  //         });
  //       }
  //     }
  //   }
  // }
};
</script>

<style lang="scss" scoped>
.text-container {
  // max-height: calc(1.2em * 3); /* 3行文本的高度，可以根据需要调整 */
  overflow: hidden;
  position: relative;
}

.text-content {
  width: 100%;
  // height: 100%;
  height: auto;
  margin-right: -20px; /* 调整按钮与文本之间的空白 */
}

.ellipsis {
  text-align: center;
  width: 0.18rem;
  height: 20px;
  position: absolute;
  bottom: 0;
  right: 0.3rem; /* 调整省略号与按钮之间的距离 */
  background: white; /* 确保省略号不会被文本遮挡 */
}

.expand-button {
  text-align: right;
  padding: 0;
  width: 0.3rem;
  height: 20px;
  line-height: 20px;

  position: absolute;
  bottom: 0.02rem;
  right: 0;
  background: none;
  border: none;
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #07a5ff;
  cursor: pointer;
  background: rgba(255, 255, 255, 1); /* 确保省略号不会被文本遮挡 */
}

.expanded {
  max-height: none !important; /* 展开状态下，显示全部文本 */
}

.explain {
  position: relative;
  width: 0.16rem;
  height: 0.16rem;
  line-height: 0.16rem;
  background: url('~@/assets/img/siteSel/period-explain.svg') no-repeat center;
  background-size: 100% 100%;
  display: inline-block;
  margin-left: 4px;
  // margin-right: 16px;
  vertical-align: middle;
  transform: translateY(-1px);
}
.flex-between {
  display: flex;
  justify-content: space-between;
}
.flex-start {
  display: flex;
  justify-content: flex-start;
}
.info-title-wrap {
  display: flex;
  height: 20px;
}
.info-name {
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #383b55;
  line-height: 20px;
}
.info-type {
  box-sizing: border-box;
  padding: 0 8px;
  // width: 33px;
  height: 20px;
  background: rgba(134, 136, 165, 0.08);
  border-radius: 2px 2px 2px 2px;
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #6a6e8c;
}
.prop-wrap {
  margin-bottom: 16px;
  height: 20px;
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #6a6e8c;
}
.number-cover-wrap {
  margin-bottom: 8px;
  height: 20px;
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #6a6e8c;
}
.info-prop-titless {
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #6a6e8c;

  &-caliber {
    width: 90px;
  }
}

.caliber-info-wrap {
  flex: 1;
  max-height: 60px;

  // overflow: hidden;
  // text-overflow: ellipsis;
}
</style>
