<template>
  <div
    class="daterangeSelect"
    @mouseenter="checkDateData"
    @mouseleave="hideClose"
  >
    <el-date-picker
      v-model="startValue"
      class="currentPick monthStyle daterange--start"
      :type="dateType"
      popper-class="dateSelectClassNew"
      :picker-options="pickerOptionsStart"
      placeholder="开始时间"
      :disabled="disabled"
      :clearable="false"
      @change="onBtnClick"
      :value-format="valueFormatStart"
      :format="formatStart"
    />
    至
    <el-date-picker
      ref="pickerEnd"
      v-model="endValue"
      class="currentPick monthStyle daterange--end"
      popper-class="dateSelectClassNew"
      :picker-options="pickerOptionsEnd"
      :type="dateType"
      placeholder="结束时间"
      :clearable="false"
      :value-format="valueFormatEnd"
      :format="formatEnd"
      :disabled="
        disabled ||
        startPeriodType == 'long' ||
        (startPeriodType == 'limited' && !startValue)
      "
      @change="onBtnClick"
    />
    <i
      class="icon-clear-date"
      v-if="!disabled && closeVisible"
      @click="clearDate"
    ></i>
  </div>
</template>

<script>
import moment from "moment";

import _isEqual from "lodash/isEqual";
import cloneDeep from "lodash/cloneDeep";

export default {
  props: {
    // 禁用
    disabled: {
      type: Boolean,
      default: false,
    },
    // 选中的回调
    selectcall: {
      type: Function,
      default: function () {},
    },
    // 期间类型
    startPeriodType: {
      type: String,
      default: "limited", // limited-固定期间 long-长期
    },
    // 绑定的值
    value: {
      type: Array,
      default: () => ["", ""],
    },
    // 是否需要从今天开始选
    startFromToday: {
      type: Boolean,
      default: true,
    },
    // 是否可以清空
    canClear: {
      type: Boolean,
      default: false,
    },
    dateType: {
      type: String,
      default: "date",
    },
    formatStartString: {
      type: String,
      default: "yyyy-MM-dd",
    },
    formatEndString: {
      type: String,
      default: "yyyy-MM-dd",
    },
  },
  computed: {
    // computed 里面放一个函数搞不起来啊，没响应的？要用到变量和模板化的话，还是函数赋值给变量好？。可以的。。。是computed书写不规范，其他地方写了。。。
    pickerOptionsStart() {
      return this.generatePickerOption(
        "start",
        this.startFromToday,
        this.value
      );
    },
    pickerOptionsEnd() {
      return this.generatePickerOption("end", this.startFromToday, this.value);
    },

    valueFormatStart() {
      if (this.startPeriodType == "long") {
        return "-";
      } else {
        return this.formatStartString;
      }
    },
    formatStart() {
      if (this.startPeriodType == "long") {
        return "-";
      } else {
        return this.formatStartString;
      }
    },

    valueFormatEnd() {
      if (this.startPeriodType == "long") {
        return "-";
      } else {
        return this.formatEndString;
      }
    },
    formatEnd() {
      if (this.startPeriodType == "long") {
        return "-";
      } else {
        return this.formatEndString;
      }
    },
  },
  data() {
    return {
      // 关闭按钮是否显示
      closeVisible: false,
      queryStartValue: ["", ""], // 用于给外部回调函数当数据用 ????
      startValue: "",
      endValue: "",
    };
  },

  watch: {
    startPeriodType(v) {
      if (v == "long") {
        this.endValue = "-";
      } else {
        this.endValue = "";
      }
    },
    // 为了模拟v-model双向修改，除了emit向上改变，还需要上到下，下层监听prop改变，并判断两者是否相等，然后再赋值
    value: {
      deep: true,
      // immediate: true,
      handler(v, ov) {
        if (!_isEqual(v, [this.startValue, this.endValue])) {
          this.startValue = cloneDeep(v[0]);
          this.endValue = cloneDeep(v[1]);
        }
      },
    },
  },
  created() {
    this.initDateData();
  },
  methods: {
    // 生成开始和结束的pickerOption选项
    generatePickerOption(type, startFromToday, value) {
      let resObj = {};
      resObj.firstDayOfWeek = 1;
      resObj.disabledDate = function (time) {
        let nowTime = "";

        // 设置一个属性，当天以前的是否可选
        if (startFromToday) {
          if (time.getTime() < moment().startOf("day").valueOf()) {
            return true;
          }
        }

        const isStart = type == "start" ? true : false;
        const compareValue = isStart ? value[1] : value[0];

        if (parseInt(compareValue)) {
          nowTime = moment(compareValue);
          // 如果有选中结束日期了，需要小于等于选中日期
          if (isStart) {
            return time.getTime() > nowTime.startOf("day").valueOf();
          } else {
            // 如果有开始日期了，需要大于等于选中日期
            return time.getTime() < nowTime.startOf("day").valueOf();
          }
        }
      };

      return resObj;
    },
    clearDate() {
      this.startValue = "";
      this.endValue = "";
      this.$emit("update:value", ["", ""]);
      this.selectcall({
        queryType: "daterange",
        queryStartValue: "",
        queryEndValue: "",
        dateStr: "daterange",
      });

      // 关闭清空图标的显示
      this.hideClose();
    },
    hideClose() {
      this.closeVisible = false;
    },
    // 鼠标划入时判断组件是否有数据，有数据则显示可清空按钮
    checkDateData() {
      if (Array.isArray(this.value)) {
        if (this.startValue && this.startValue !== "Invalid date") {
          this.closeVisible = true;
        } else if (this.endValue && this.endValue !== "Invalid date") {
          this.closeVisible = true;
        }
      }
    },

    initDateData() {
      this.setQueryDate();
    },
    // 初始化数据或者回显数据处理
    setQueryDate() {
      // 进来没有默认值，除非有设置才默认值设置

      // 进来如果要回显的话，说明第一个必须是要有值的
      if (parseInt(this.value[0]) || parseInt(this.value[1])) {
        // 如果有值时的回显

        if (parseInt(this.value[0])) {
          this.startValue = this.queryStartValue[0] =
            this.replaceToMomentFormatString(this.value[0]);
        }

        if (parseInt(this.value[1])) {
          this.endValue = this.queryStartValue[1] =
            this.replaceToMomentFormatString(this.value[1]);
        }
      }
    },
    // 这里是为了兼容moment的字符串日期格式，moment后面日期是DD。element和浏览器的是dd
    replaceToMomentFormatString(value) {
      return moment(value).format(
        this.formatStartString.replace("yyyy-MM-dd", "yyyy-MM-DD")
      );
    },
    // 监控日期组件按钮点击
    onBtnClick(val) {
      const { startValue, endValue } = this;

      let queryStartValue = "";
      let queryEndValue = "";

      // 点击清除按钮的时候，需要把值置为空
      if (!val) {
        this.clearDate();
        return false;
      }

      if (startValue) {
        // if (moment(endValue).diff(moment(startValue), 'days') > 90) {
        //   window.root.$message({
        //     showClose: true,
        //     message: '自定义日期跨度不能大于90天',
        //     type: 'error'
        //   })
        //   return
        // }

        queryStartValue = this.replaceToMomentFormatString(startValue);
      }

      if (endValue) {
        queryEndValue = this.replaceToMomentFormatString(endValue);
      }

      // 双向绑定修改
      this.$emit("update:value", [queryStartValue, queryEndValue]);

      // 可能需要用到的回调函数的方式传其他数据
      this.selectcall({
        queryStartValue,
        queryEndValue,
        dateStr: "daterange",
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.daterangeSelect {
  position: relative;
  height: 32px;
  width: 243px;
  line-height: 32px;
  background-color: #fff;

  border-radius: 2px 2px 2px 2px;
  border: 1px solid #bac2cd;
  transition: all 0.2s;

  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  font-size: 14px;
  color: #9399ae;

  overflow: hidden;

  &:hover {
    border-color: #c0c4cc;
  }

  .icon-clear-date {
    display: inline-block;
    position: absolute;
    right: 0.09rem;
    top: 0;
    bottom: 0;
    margin: auto;
    width: 14px;
    height: 14px;
    background-color: #9399ae;
    // background: url('~@/assets/img/icon-clear-date.svg') no-repeat center;
    background-size: 100% 100%;
    cursor: pointer;
    //
  }

  // /deep/ .el-icon-date:before {
  //   content: '';
  // }
  .el-date-editor.el-input {
    // padding-left: 0.8rem;
    padding-left: 5px;
    // width: 5.8rem;
    // width: 82px;
    // width: 106px;
    width: 110px;
    height: 100%;

    ::v-deep .el-input__inner {
      border: none !important;
      // width: 82px;
      width: 100% !important;
      height: 95%;
    }

    ::v-deep .el-input__icon {
      display: flex;
      align-items: center;
    }
  }

  .el-date-editor.daterange--start {
    ::v-deep .el-input__inner {
      text-indent: 20px;
    }
  }
  .el-date-editor.daterange--end {
    width: 100px;
    ::v-deep .el-input__inner {
      width: 100px;
    }
    ::v-deep .el-icon-date:before {
      content: "";
    }
    ::v-deep .el-input__suffix {
      right: 0;
    }
    ::v-deep .el-input__icon {
      right: 0;
      width: 12.25px;
    }
  }

  ::v-deep .el-input__inner {
    padding: 0;
    // width: 5.8rem;
    background-color: transparent !important;
    border: none;
    font-size: 0.8rem;
    font-weight: 600;
  }
}
</style>
