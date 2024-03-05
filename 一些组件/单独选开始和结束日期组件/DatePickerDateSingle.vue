<template>
  <div class="daterangeSelect">
    <el-date-picker
      v-model="startValue"
      class="currentPick monthStyle daterange--start"
      type="date"
      popper-class="dateSelectClassNew"
      :picker-options="pickerOptionsStart"
      placeholder="开始时间"
      :disabled="disabled"
      :clearable="false"
      @change="pickRangeBtn"
    />
    至
    <el-date-picker
      ref="pickerEnd"
      v-model="endValue"
      class="currentPick monthStyle daterange--end"
      popper-class="dateSelectClassNew"
      :picker-options="pickerOptionsEnd"
      type="date"
      placeholder="结束时间"
      :clearable="canClear ? true : false"
      :value-format="valueFormatEnd"
      :format="formatEnd"
      :disabled="startPeriodType == 'long' || (startPeriodType == 'limited' && !startValue)"
      @change="pickRangeBtn"
    />
  </div>
</template>

<script>
import moment from 'moment';

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
    }
  },
  data() {
    return {
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
        return 'yyyy-MM-dd';
      }
    },
    formatEnd() {
      if (this.startPeriodType == 'long') {
        return '-';
      } else {
        return 'yyyy-MM-dd';
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
    }
  },
  created() {
    this.setQueryDate('daterange');
  },
  methods: {
    getDafaultDate(dateType) {
      return this.defaultValArray ? this.defaultValArray[this.dateArray.findIndex(_ => _ === dateType)] : '';
    },
    setQueryDate(dateType, value) {
      // 进来没有默认值，除非有设置才默认值设置

      if (dateType === 'daterange') {
        console.log(this.value, 'this.value');
        // 进来如果要回显的话，说明第一个必须是要有值的
        // TODO 不知道为啥空字符串也可以判定进来了
        if (parseInt(this.value[0]) || parseInt(this.value[1])) {
          // 自定义默认选择今天
          // let today = moment().format("YYYY-MM-DD");
          // 自定义默认选择昨天
          // Now.subtract(1, "day");
          // const subDay = Now.format('YYYY-MM-DD');
          // this.queryStartValue = [subDay, subDay];
          // this.queryStartValue = [today, today];
          // this.startValue = subDay;
          // this.startValue = today;
          // this.endValue = subDay;
          // this.endValue = today;
          // 2020-03-19 新增default array功能

          // 如果有值时的回显
          console.log(this.value[0], 'this.value[0]');
          if (parseInt(this.value[0])) {
            console.log(';hi');
            this.startValue = moment(this.value[0]).format('YYYY-MM-DD');
          }

          // debugger;
          console.log(this.startValue, 'startValue');
          this.queryStartValue[0] = this.startValue;
          if (parseInt(this.value[1])) {
            this.endValue = moment(this.value[1]).format('YYYY-MM-DD');
            this.queryStartValue[1] = this.endValue;
          }

          // const defaultVal = this.getDafaultDate(dateType);
          // if (defaultVal && RegExp('~').test(defaultVal) === true) {
          //   const [ startValue, endValue ] = defaultVal.split('~');
          //   this.queryStartValue = [ startValue, endValue ];
          //   // 对外界的值进行更新
          //   // this.$emit("update:value", this.queryStartValue);
          //   this.startValue = startValue;
          //   this.endValue = endValue;
          // }
        }
      }

      this.pickCurrent(this.queryStartValue);
    },
    pickRangeBtn() {
      const { startValue, endValue } = this;
      // let queryType = this.queryTypes.findIndex(item => filterDateType === item) + 1;
      const queryType = 'daterange';
      let queryStartValue = '';
      let queryEndValue = '';
      if (startValue || endValue) {
        // if (moment(endValue).diff(moment(startValue), 'days') > 90) {
        //   window.root.$message({
        //     showClose: true,
        //     message: '自定义日期跨度不能大于90天',
        //     type: 'error'
        //   })
        //   return
        // }
        queryStartValue = moment(startValue).format('YYYY-MM-DD');
        queryEndValue = moment(endValue).format('YYYY-MM-DD');
        console.log(queryStartValue, queryEndValue, 'queryStartValue,queryEndValue 没更新成功吗');
        // TODO 这里怎么就不用this了
        this.$emit('update:value', [queryStartValue, queryEndValue]);

        this.selectcall({
          queryType,
          queryStartValue,
          queryEndValue,
          //   ...this.showDateText(),
          dateStr: 'daterange'
          // ranNum,
          //   ...this.showRate(filterDateType, queryStartValue)
        });
      }
    },

    pickCurrent(value) {
      const queryType = 'daterange';
      let queryStartValue = '';
      let queryEndValue = '';
      if (Array.isArray(value)) {
        queryStartValue = moment(value[0]).format('YYYY-MM-DD');
        queryEndValue = moment(value[1]).format('YYYY-MM-DD');
      } else {
        queryStartValue = moment(value)
          .clone()
          .startOf('daterange')
          .format('YYYY-MM-DD');
        queryEndValue = moment(value)
          .clone()
          .endOf('daterange')
          .format('YYYY-MM-DD');
      }

      this.$emit('update:value', [queryStartValue, queryEndValue]);
      this.selectcall({
        queryType,
        queryStartValue,
        // ...this.showDateText(),
        queryEndValue,
        dateStr: 'daterange'

        // ...this.showRate(filterDateType, queryStartValue)
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.daterangeSelect {
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

  &:hover {
    border-color: #c0c4cc;
  }

  /deep/ .el-icon-date:before {
    content: '';
  }

  .el-date-editor.el-input {
    // padding-left: 0.8rem;
    padding-left: 5px;
    // width: 5.8rem;
    width: 82px;

    .el-input__inner {
      width: 82px;
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
