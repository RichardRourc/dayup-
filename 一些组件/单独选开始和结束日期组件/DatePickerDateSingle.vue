<template>
  <div class="daterangeSelect" @mouseenter="checkData" @mouseleave="hideClose">
    <el-date-picker
      v-model="startValue"
      class="currentPick monthStyle daterange--start"
      :type="dateType"
      popper-class="dateSelectClassNew"
      :picker-options="pickerOptionsStart"
      placeholder="开始时间"
      :disabled="disabled"
      :clearable="false"
      @change="pickRangeBtn"
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
      :disabled="disabled || startPeriodType == 'long' || (startPeriodType == 'limited' && !startValue)"
      @change="pickRangeBtn"
    />
    <i class="icon-clear-date" v-if="!disabled && closeVisible" @click="clearDate"></i>
  </div>
</template>

<script>
import moment from 'moment';

import _isEqual from 'lodash/isEqual';
import cloneDeep from 'lodash/cloneDeep';

export default {
  props: {
    // 禁用
    disabled: {
      type: Boolean,
      default: false
    },
    // 选中的回调
    selectcall: {
      type: Function,
      default: function() {}
    },
    // 期间类型
    startPeriodType: {
      type: String,
      default: 'limited' // limited-固定期间 long-长期
    },
    // 绑定的值
    value: {
      type: Array,
      default: () => ['', '']
    },
    // 是否需要从今天开始选
    startFromToday: {
      type: Boolean,
      default: false
    },
    // 是否可以清空
    canClear: {
      type: Boolean,
      default: false
    },
    dateType: {
      type: String,
      default: 'date'
    },
    formatStartString: {
      type: String,
      default: 'yyyy-MM-dd'
    },
    formatEndString: {
      type: String,
      default: 'yyyy-MM-dd'
    }
  },
  data() {
    return {
      // 关闭按钮是否显示
      closeVisible: false,
      queryStartValue: ['', ''],
      startValue: '',
      endValue: '',
      pickerOptionsStart: {
        firstDayOfWeek: 1,
        disabledDate: time => {
          let nowTime = '';
          // 小于当天的都不可以选吧

          // 设置一个属性，当天以前的是否可选
          if (this.startFromToday) {
            if (
              time.getTime() <
              moment()
                .startOf('day')
                .valueOf()
            ) {
              return true;
            }
          }

          // 如果有选中结束日期了，需要小于等于选中日期
          if (parseInt(this.value[1])) {
            nowTime = moment(this.value[1]);

            return time.getTime() > nowTime.startOf('day').valueOf();
          }
        }
      },
      pickerOptionsEnd: {
        firstDayOfWeek: 1,
        // 目前是这个月的第一天以及今天，实际上应该是上一个日期以及后面？总之后面不设限？可是如果第一个还没选呢？我也disabled?
        disabledDate: time => {
          let nowTime = '';

          if (this.startFromToday) {
            // 小于当天的不可选
            if (
              time.getTime() <
              moment()
                .startOf('day')
                .valueOf()
            ) {
              return true;
            }
          }

          // 如果有选中开始日期了，需要大于等于选中日期
          if (parseInt(this.value[0])) {
            nowTime = moment(this.value[0]);

            return time.getTime() < nowTime.startOf('day').valueOf();
          }
        }
      }
    };
  },
  computed: {
    valueFormatEnd() {
      if (this.startPeriodType == 'long') {
        return '-';
      } else {
        return this.formatEndString;
      }
    },
    formatEnd() {
      if (this.startPeriodType == 'long') {
        return '-';
      } else {
        return this.formatEndString;
      }
    },
    valueFormatStart() {
      if (this.startPeriodType == 'long') {
        return '-';
      } else {
        return this.formatStartString;
      }
    },
    formatStart() {
      if (this.startPeriodType == 'long') {
        return '-';
      } else {
        return this.formatStartString;
      }
    }
  },
  watch: {
    startPeriodType(v) {
      if (v == 'long') {
        this.endValue = '-';
      } else {
        this.endValue = '';
      }
    },
    // TODO 为了模拟v-model，除了emit向上改变，还需要上到下，下层监听prop改变，并判断两者是否相等，然后再赋值
    value: {
      deep: true,
      // immediate: true,
      handler(v, ov) {
        console.log(v, ov, this.startValue, this.endValue, 'value');
        if (!_isEqual(v, [this.startValue, this.endValue])) {
          this.startValue = cloneDeep(v[0]);
          this.endValue = cloneDeep(v[1]);
        }
      }
    }
  },
  created() {
    this.setQueryDate('daterange');
  },
  methods: {
    clearDate() {
      this.startValue = '';
      this.endValue = '';
      this.$emit('update:value', ['', '']);
      this.selectcall({ queryType: 'daterange', queryStartValue: '', queryEndValue: '', dateStr: 'daterange' });

      // 关闭自己
      this.closeVisible = false;
    },
    hideClose() {
      this.closeVisible = false;
    },
    checkData() {
      if (Array.isArray(this.value)) {
        if (this.startValue && this.startValue !== 'Invalid date') {
          this.closeVisible = true;
        } else if (this.endValue && this.endValue !== 'Invalid date') {
          this.closeVisible = true;
        }
      }
    },
    getDafaultDate(dateType) {
      return this.defaultValArray ? this.defaultValArray[this.dateArray.findIndex(_ => _ === dateType)] : '';
    },
    // 这个设置是为了啥，设置初始值
    setQueryDate(dateType, value) {
      // 进来没有默认值，除非有设置才默认值设置

      if (dateType === 'daterange') {
        // 进来如果要回显的话，说明第一个必须是要有值的
        // TODO 不知道为啥空字符串也可以判定进来了
        if (parseInt(this.value[0]) || parseInt(this.value[1])) {
          // 如果有值时的回显
          console.log(this.value[0], 'this.value[0]');
          if (parseInt(this.value[0])) {
            this.startValue = moment(this.value[0]).format(this.formatStartString);
          }

          console.log(this.startValue, 'startValue');
          this.queryStartValue[0] = this.startValue;
          if (parseInt(this.value[1])) {
            this.endValue = moment(this.value[1]).format(this.formatEndString);
            this.queryStartValue[1] = this.endValue;
          }
        }
      }

      this.pickCurrent(this.queryStartValue);
    },
    pickRangeBtn(val) {
      console.log('test clear', val);

      const { startValue, endValue } = this;
      // let queryType = this.queryTypes.findIndex(item => filterDateType === item) + 1;
      const queryType = 'daterange';
      let queryStartValue = '';
      let queryEndValue = '';

      // 点击清除按钮的时候，需要把值置为空
      if (!val) {
        this.startValue = '';
        this.endValue = '';
        this.$emit('update:value', ['', '']);
        this.selectcall({ queryType, queryStartValue, queryEndValue, dateStr: 'daterange' });
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
        queryStartValue = moment(startValue).format(this.formatStartString.replace('yyyy-MM-dd', 'yyyy-MM-DD'));
      }

      if (endValue) {
        queryEndValue = moment(endValue).format(this.formatEndString.replace('yyyy-MM-dd', 'yyyy-MM-DD'));
      }

      console.log(queryStartValue, queryEndValue, 'queryStartValue,queryEndValue 没更新成功吗');
      // TODO 这里怎么就不用this了
      this.$emit('update:value', [queryStartValue, queryEndValue]);

      this.selectcall({
        queryType,
        queryStartValue,
        queryEndValue,
        dateStr: 'daterange'
      });
    },

    pickCurrent(value) {
      const queryType = 'daterange';
      let queryStartValue = '';
      let queryEndValue = '';
      if (Array.isArray(value)) {
        // 需要有值且不是无效日期才可以正确赋值
        if (value[0] && value[0] !== 'Invalid date') {
          queryStartValue = moment(value[0]).format(this.formatStartString);
        } else {
          queryStartValue = '';
        }

        if (value[1] && value[1] !== 'Invalid date') {
          queryEndValue = moment(value[1]).format(this.formatEndString);
        } else {
          queryEndValue = '';
        }
      } else {
        queryStartValue = moment(value)
          .clone()
          .startOf('daterange')
          .format(this.formatStartString);
        queryEndValue = moment(value)
          .clone()
          .endOf('daterange')
          .format(this.formatEndString);
      }

      this.$emit('update:value', [queryStartValue, queryEndValue]);
      this.selectcall({
        queryType,
        queryStartValue,
        // ...this.showDateText(),
        queryEndValue,
        dateStr: 'daterange',
        initTag: true
        // ...this.showRate(filterDateType, queryStartValue)
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.daterangeSelect {
  position: relative;
  height: 0.32rem;
  width: 243px;
  line-height: 0.32rem;
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
    background: url('~@/assets/img/icon-clear-date.svg') no-repeat center;
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

    /deep/ .el-input__inner {
      border: none !important;
      // width: 82px;
      width: 100% !important;
      height: 95%;
    }

    /deep/ .el-input__icon {
      display: flex;
      align-items: center;
    }
  }

  .el-date-editor.daterange--start {
    /deep/ .el-input__inner {
      text-indent: 20px;
    }
  }
  .el-date-editor.daterange--end {
    width: 100px;
    /deep/ .el-input__inner {
      width: 100px;
    }
    /deep/ .el-icon-date:before {
      content: '';
    }
    /deep/ .el-input__suffix {
      right: 0;
    }
    /deep/ .el-input__icon {
      right: 0;
      width: 12.25px;
    }
  }

  /deep/ .el-input__inner {
    padding: 0;
    // width: 5.8rem;
    background-color: transparent !important;
    border: none;
    font-size: 0.8rem;
    font-weight: 600;
  }
}
</style>
