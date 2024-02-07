<template>
  <div class="preview-wave-ball">
    <div class="preview-num-count">
      <span class="preview-num-select">
        {{ (selectCount && formatNum(selectCount)) || '-' }}
      </span>
      <span class="seperate-line seperate-num">/</span>

      <span class="preview-num-all">
        {{ (allCount && formatNum(allCount)) || '-' }}
      </span>
    </div>
    <div class="preview-desc-tips">
      <span class="desc-tips-select">
        选中用户
      </span>
      <span class="measure-text" v-if="tabType === '1'">(预估)</span>
      <span class="seperate-line seperate-text"> /</span>

      <span class="desc-tips-all">
        总合并用户id<el-tooltip popper-class="data-adress explain-popper" placement="bottom">
          <div slot="content">
            1. 统计口径：累计注册的丰巢user_id，经过规则合并，并剔除账户状态为已删除、无
            <br />效的user_id
            <br />
            2. user_id合并规则：(1) 手机号相同，(2) 绑定了微信且微信渠道号相同，(3) 绑定了支付宝<br />且支付宝渠道号相同，(4)
            对以上三种场景中同时出现两次或两次以上的user_id再合并；<br />合并后的id为更新时间最新的user_id，
            如更新时间相同，则比较创建时间
          </div>
          <svg-icon icon-class="icon-tooltip" class="icon-tooltip"></svg-icon>
        </el-tooltip>
      </span>
    </div>
    <!-- 球状体 -->

    <div class="ball-wrap" :class="{ 'ball-wrap-disabled': disabled }">
      <!-- 没有数据时的预览的球体状态 -->
      <div
        class="status-empty ball-inner"
        :class="{ 'ball-inner--disabled': disabled, 'ball-inner--disabled-init': disabled }"
        v-if="!hasPreview"
      >
        <el-button
          class="btn-preview"
          :class="{ 'btn-preview--disabled': disabled, 'btn-preview--disabled-init': disabled }"
          @click="getPreviewData"
          >预览</el-button
        >
      </div>
      <div class="status-data ball-inner" v-else>
        <!-- loading效果 -->
        <template v-if="loading">
          <LoadingPersonas />
        </template>
        <template v-else>
          <WaveCom :percent="selectCount / allCount" />
          <div class="ball-center">
            <div class="value">{{ ((selectCount / allCount) * 100).toFixed(2) }}%</div>
            <div class="value-txt">占全部用户</div>
          </div></template
        >
      </div>
    </div>
    <!-- 非详情页才显示刷新预览 -->
    <div class="refresh-wrap" v-if="hasPreview && status !== 'detail'" @click="refresh">
      <i class="el-icon-refresh"></i>
      刷新预览
    </div>
  </div>
</template>

<script>
import { numFormat } from '@/utils';

import WaveCom from './WaveCom';
import LoadingPersonas from '@/components/Loading/LoadingPersonas';

export default {
  props: {
    selectCount: {
      type: [String, Number],
      default: ''
    },
    allCount: {
      type: [String, Number],
      default: '-'
    },
    loading: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    status: {
      type: String,
      default: ''
    },
    tabType: {
      type: String,
      default: '0' //0-离线，1-实时
    }
    // toReset: {
    //   type: Boolean,
    //   default: false
    // }
  },
  computed: {},
  components: {
    WaveCom,
    LoadingPersonas
  },
  data() {
    return {
      innerDisabled: false, // 外部操作后内部禁止用
      initialDisabled: false,
      hasPreview: false // 是否点击了预览
    };
  },
  created() {
    if (!this.selectCount) {
      this.initialDisabled = true;
    }
  },
  methods: {
    formatNum(num) {
      return numFormat(num === undefined ? 0 : num);
    },
    toReset() {
      this.hasPreview = false;
    },
    // TODO 发起刷新请求 .
    refresh() {
      this.$emit('onPreview');
    },
    getPreviewData() {
      if (this.disabled) return true;
      // 解除默认禁止
      this.initialDisabled = false;
      this.$emit('onPreview');
      this.hasPreview = true;
    }
  },
  watch: {
    selectCount: {
      immediate: true,
      handler(v) {
        if (parseInt(v) >= 0) {
          this.hasPreview = true;
        }
      }
    }
    // toReset: {
    //   handler(v) {
    //     if (v) {
    //       // 重置
    //       this.hasPreview = false;
    //     }
    //   }
    // }
  }
};
</script>

<style lang="scss" scoped>
.preview-wave-ball {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.preview-num-count {
  margin-top: 8px;
  margin-bottom: 9px;
}
.preview-desc-tips {
  margin-bottom: 8px;
}
.seperate-line {
  margin: 0 8px;
}
.preview-num-select {
  font-size: 18px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #383b55;
}
.preview-num-all {
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #6a6e8c;
}
.seperate-num {
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #6a6e8c;
}
.seperate-text {
  font-size: 12px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #9399ae;
}
.desc-tips-select {
  font-size: 12px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #6a6e8c;
}
.desc-tips-all {
  font-size: 12px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #9399ae;
}
.refresh-wrap {
  margin-top: 17px;
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #069595;
  cursor: pointer;
}
.el-icon-refresh {
  margin-right: 8px;
}

.ball-wrap {
  width: 140px;
  height: 140px;
  border-radius: 50%;
  border: 2px solid #8cdfdf;
  overflow: hidden;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  &-disabled {
    border-color: #a8abb220;
  }
}

.ball-inner {
  position: relative;

  &--disabled {
    cursor: not-allowed;

    &-init {
      background-color: #a8abb220 !important;
    }
  }
}

.ball-center {
  position: absolute;
  left: 0%;
  right: 0%;
  top: 0%;
  bottom: 0%;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  border: 3px solid #fff;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.value {
  font-size: 16px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #6a6e8c;

  text-align: center;
}

.value-txt {
  margin-top: 8px;
  font-size: 12px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #8688a5;
}

.status-empty {
  width: 100%;
  height: 100%;
  background: rgba(26, 191, 191, 0.2);
  border: 4px solid #ffffff;
  border-radius: 50%;

  display: flex;
  justify-content: center;
  align-items: center;
}

.btn-preview {
  padding: 0 16px;
  height: 28px;
  line-height: 28px;
  background: #1abfbf;
  border-radius: 2px 2px 2px 2px;

  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #ffffff;

  &--disabled {
    cursor: not-allowed;

    &-init {
      background-color: #a8abb2;
    }
  }
}

.measure-text {
  font-size: 12px;
  font-family: MiSans, MiSans;
  font-weight: 400;
  color: #9399ae;
}
</style>
