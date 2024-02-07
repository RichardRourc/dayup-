<template>
  <div class="area-period-setting-dialog" v-if="dialogVisible">
    <el-dialog title="调整标签值展示" :visible.sync="dialogVisible" :before-close="handleClose" :width="'50%'">
      <div class="com-header">
        <div class="com-header-left">
          <div class="type-container">
            <div class="btn-type" :class="[curCustomType == 0 && 'btn-type-active']" @click="setCustomType(0)">
              默认
            </div>
            <div
              v-if="tagInfo && tagInfo.valueChooseType == 1"
              class="btn-type"
              :class="[curCustomType == 1 && 'btn-type-active']"
              @click="setCustomType(1)"
            >
              自定义
            </div>
          </div>
          <div class="tips">{{ descTips }}</div>
        </div>

        <el-form :model="formFast" ref="formFast" class="com-header-right" v-if="showFastSetting && curCustomType == 1">
          <div class="setting-container-shortcut-header">
            <el-form-item prop="start" :rules="startRules"
              ><el-input v-model.number="minValue" placeholder="开始值" @change="changeMinValue"></el-input
            ></el-form-item>
            <el-form-item prop="end" :rules="endRules"
              ><el-input v-model.number="maxValue" placeholder="结束值" @change="changeMaxValue"></el-input
            ></el-form-item>

            <el-input v-model.number="periodNum" placeholder="分段值(<=30)" @change="changePeriodNum"></el-input>
          </div>
          <div class="setting-container-shortcut-footer">
            <div class="btn-text" @click="resetFastSetting">重置</div>
            <div class="btn-text btn-text-confirm" @click="setFastSplit">确定</div>
          </div>
        </el-form>
        <!-- 占位符 -->
        <div class="empty-block" v-else></div>
      </div>
      <el-form ref="formContent" :model="formContent" class="setting-container" v-show="curCustomType == 1">
        <div class="setting-container-header">
          <div class="setting-label">设置区间</div>
          <div v-if="curCustomType == 1" class="btn-text-shortcut" @click="showFastSetting = !showFastSetting">
            <span class="point-setting" v-if="hasSetFast"></span>
            快速划分区间
          </div>
        </div>
        <div class="list-period-setting-wrap">
          <div class="list-period-setting-item" v-for="(item, key) in arr" :key="key">
            <div class="list-item-label">区间{{ key + 1 }}：</div>
            <!-- 根据循环的list的某个数组的字段的1和2来绑定值 -->
            <!-- 如果设置了快速划分则无法修改 -->
            <!-- TODO 先用个数组来短暂代替。 -->
            [
            <el-form-item :rules="checkInputRule" prop="test"
              ><el-input
                v-model.number="item.minValue"
                :disabled="hasSetFast"
                @focus="setFocusObj(key, 0)"
                @blur="clearFocusObj"
              ></el-input
            ></el-form-item>
            ,
            <el-form-item :rules="checkInputRule" prop="test1">
              <el-input
                v-model.number="item.maxValue"
                :disabled="hasSetFast"
                @focus="setFocusObj(key, 1)"
                @blur="clearFocusObj"
              ></el-input>
            </el-form-item>

            ]
            <svg-icon
              @click.native="deleteItem(key)"
              class="icon-delete el-icon-delete"
              v-if="arr && arr.length > 1 && !hasSetFast"
              :icon-class="'icon_subtract_normal'"
            />
            <!-- 栈式删除，出现条件是数组长度大于1 -->
          </div>
        </div>
        <div class="btn-add-period-wrap" @click="addPeriod" v-if="!hasSetFast && arr && arr.length < 30">
          <i class="el-icon-plus"></i>
          <span class="btn-add-period-text">增加区间</span>
        </div>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <div class="add_group">
          <el-button class="add_common add_cancel add_cancel--bg--w add_cancel" @click="handleClose">取 消</el-button>
          <el-button class="add_common add_ok" type="primary" @click="onConfirm">确 定</el-button>
        </div>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import lodash from 'lodash';

export default {
  props: {
    dialogVisible: {
      default: false,
      type: Boolean
    },
    customType: {
      default: 0,
      type: [String, Number]
    },
    tagInfo: {
      default: {},
      type: Object
    },
    fastObj: {
      default: '',
      type: [Object, String]
    }
  },
  computed: {
    descTips() {
      if (this.curCustomType == 0) {
        return '*展示业务既定区间';
      } else {
        return `*可设置的区间 ${this.minNumber}~${this.maxNumber}`;
      }
    },
    minNumber() {
      if (this.tagInfo) {
        return this.tagInfo.minValue;
      } else return 0;
    },
    maxNumber() {
      if (this.tagInfo) {
        return this.tagInfo.maxValue;
      } else return 100;
    }
  },
  data() {
    return {
      formFast: {},
      formContent: {},
      // 存储当前高亮的输入框所在一维和二维位置，用于规则校验。
      focusIndexObj: {
        one: '',
        two: ''
      },
      minValue: '',
      maxValue: '',
      periodNum: '',
      // 得校验值
      // 就按照这个形式来了
      arr: [{ minValue: '', maxValue: '', unitName: this.tagInfo.unitName }],
      curCustomType: 0,
      showFastSetting: false, // 快速设置区间显示的控制变量
      hasSetFast: false, // 判断是否设置了快速区间
      checkInputRule: [{ trigger: 'blur', message: '', validator: this.checkDIYInput }],
      startRules: [{ trigger: 'blur', message: '', validator: this.checkStartRules }],
      endRules: [{ trigger: 'blur', message: '', validator: this.checkEndRules }]
    };
  },
  created() {
    this.curCustomType = (this.tagInfo && this.tagInfo.customType) || 0;

    // 设置是否有自定义区间，有就填充，否则维持这个默认的空值
    if (this.tagInfo && this.tagInfo.customList && this.tagInfo.customList.length > 0) {
      this.arr = lodash.cloneDeep(this.tagInfo.customList);
      // this.hasSetFast = true;
      console.log(this.fastObj, 'fastObj', this.tagInfo);
      if (this.fastObj) {
        const { hasSetFast, minValue, maxValue, periodNum } = this.fastObj;
        this.hasSetFast = hasSetFast;
        this.minValue = minValue;
        this.maxValue = maxValue;
        this.periodNum = periodNum;
      }
    }
  },
  methods: {
    checkStartRules(rule, value, cb) {
      if (this.maxValue) {
        if (this.minValue >= this.maxValue) {
          cb(new Error('开始值不能大于结束值'));
        } else {
          cb();
        }
      } else {
        cb();
      }
    },
    checkEndRules(rule, value, cb) {
      if (this.minValue) {
        if (this.minValue >= this.maxValue) {
          cb(new Error('结束值不能小于开始值'));
        } else {
          cb();
        }
      } else {
        cb();
      }
    },
    setFocusObj(one, two) {
      this.focusIndexObj = { one, two };
    },
    clearFocusObj() {
      // this.focusIndexObj = { one: '', two: '' };
    },
    // 自定义区间输入框值的校验
    checkDIYInput(rule, value, cb) {
      const { one, two } = this.focusIndexObj;
      let conf = {
        0: 'minValue',
        1: 'maxValue'
      };
      // console.log(this.arr[one], 'this.arr[one]');
      // 值
      let curValue = parseInt(this.arr[one][conf[two + '']]);
      // 判断头尾

      // 最大最小值设置
      if (curValue <= this.minNumber) {
        this.arr[one][conf[two + '']] = this.minNumber;
      } else if (curValue >= this.maxNumber) {
        this.arr[one][conf[two + '']] = this.maxNumber;
      }

      // 判断右边的同行的
      let judgeOneRight = (cb, conf, one, curValue) => {
        if (this.arr[one][conf['1']]) {
          if (curValue >= this.arr[one][conf['1']]) {
            cb(new Error('开始值不可大于等于结束值'));
          } else {
            // 判断完右边的之后判断左边的
            // 上一组的判断，且有值才判断，别瞎判断
            if (one - 1 >= 0 && this.arr[one - 1]['maxValue']) {
              if (curValue <= this.arr[one - 1][conf['1']]) {
                cb(new Error('当前值不可小于上个区间的值'));
              } else {
                cb();
              }
            }
            // 都可以才可以
            else {
              cb();
              // 如果第一个通过了，去校验第二个？或者第二个通过了又去校验第一个？那不是无穷无尽了？或者整个表单校验，那更离谱。
            }
          }
        }
        // 如果第二个也值没有，直接校验通过
        else {
          cb();
        }
      };

      // TODO 先前后附近值对比吧，前面的也要判断的话就非常麻烦了。
      // 如果是第一个的
      // 校验顺序：上->下，左->右
      if (two == 0) {
        // 上一组的判断，且有值才判断，别瞎判断
        if (one - 1 >= 0 && this.arr[one - 1]['maxValue']) {
          if (curValue <= this.arr[one - 1][conf['1']]) {
            cb(new Error('当前值不可小于上个区间的值'));
          } else {
            // 如果通过了就开始判断右边的

            judgeOneRight(cb, conf, one, curValue);
          }
        }

        // 如果没有上一组，开始校验同行右边的
        else {
          judgeOneRight(cb, conf, one, curValue);
        }
      } else if (two == 1) {
        if (this.arr[one][conf['0']]) {
          // 如果是第二个的
          if (curValue <= this.arr[one][conf['0']]) {
            cb(new Error('结束值不可小于等于结束值'));
          }
          // 下一组的判断，且有最小值输入才判断
          else if (this.arr[one + 1] && this.arr[one + 1]['minValue']) {
            if (curValue >= this.arr[one + 1][conf['0']]) {
              cb(new Error('值不可大于下个区间的值'));
            } else {
              cb();
            }
          } else {
            cb();
          }
        } else {
          cb();
        }
      } else {
        cb();
      }

      // console.log(one, two, 'one,two');
      // cb(new Error('测试'));
    },
    changePeriodNum(val) {
      if (val >= 30) {
        this.periodNum = 30;
      }
    },
    changeMinValue(val) {
      if (val <= Number(this.minNumber)) {
        this.minValue = this.minNumber * 1;
      }
    },
    changeMaxValue(val) {
      if (val >= Number(this.maxNumber)) {
        this.maxValue = this.maxNumber * 1;
      }
    },
    divideRange(start, end, segments) {
      if (segments >= end - start) {
        // 如果段的数量大于等于范围的大小，直接返回整个范围
        return [{ minValue: start, maxValue: end, unitName: this.tagInfo.unitName }];
      }

      const range = end - start;
      const segmentSize = Math.floor(range / segments);
      const remainder = range % segments; // 余数，用于确保最后一个段覆盖剩余的范围

      const result = [];
      let currentStart = start;

      for (let i = 0; i < segments; i++) {
        let currentEnd = currentStart + segmentSize - 1; // 减去1确保每一段的结束值+1是下一段的开始值

        // 处理确保每个段都有足够的空间
        if (i < remainder) {
          // 在剩余的部分上增加一个单位的长度
          currentEnd++;
        }

        result.push({ minValue: currentStart, maxValue: currentEnd, unitName: this.tagInfo.unitName });
        currentStart = currentEnd + 1; // 更新当前段的起始值为下一段的开始值
      }

      // 确保最后一个段的结束值是范围的结束值
      result[segments - 1].maxValue = end;

      return result;
    },
    resetFastSetting() {
      this.clearDiyData();
      this.showFastSetting = false;
      this.arr = [{ minValue: '', maxValue: '', unitName: this.tagInfo.unitName }];
      // this.showFastSetting = false;
    },

    isNumeric(value) {
      // 使用正则表达式判断是否为数字或者可以转换为数字的字符串
      const numericRegex = /^(0|[1-9]\d*)$/;

      // 判断是否符合条件
      return numericRegex.test(value);
    },
    setFastSplit() {
      console.log(this.minValue, this.maxValue, this.periodNum, '(this.minValue, this.maxValue, this.periodNum)');
      //TODO 还要对三个输入框输入的值做限制的，现在先简单处理吧。
      if (this.isNumeric(this.minValue) && this.isNumeric(this.maxValue) && this.isNumeric(this.periodNum)) {
        if (this.$refs.formFast) {
          this.$refs.formFast.validate(res => {
            if (res) {
              this.hasSetFast = true;

              // 关闭快捷按钮的显示sss
              this.showFastSetting = false;
              // 分段
              // this.arr = new Array(this.periodNum);

              this.arr = [...this.divideRange(this.minValue, this.maxValue, this.periodNum)];
            }
          });
        }
      } else if (!this.isNumeric(this.minValue) && !this.isNumeric(this.maxValue) && !this.isNumeric(this.periodNum)) {
        // 如果都没值就清空
        this.clearDiyData();

        this.showFastSetting = false;
        this.arr = [{ minValue: '', maxValue: '', unitName: this.tagInfo.unitName }];
      } else {
        return;
      }
    },
    addPeriod() {
      this.arr.push({ minValue: '', maxValue: '', unitName: this.tagInfo.unitName });
    },
    deleteItem(index) {
      this.arr.splice(index, 1);
    },
    setCustomPeroid() {
      let dataObj = {
        customList: this.arr,
        customType: this.curCustomType,
        fastObj: {
          hasSetFast: this.hasSetFast,
          minValue: this.minValue,
          maxValue: this.maxValue,
          periodNum: this.periodNum
        },
        tagInfo: this.tagInfo
      };
      // 如果是默认的需要在传过去时，自定义的传空字符串
      if (this.curCustomType == 0) {
        dataObj.customList = '';
      }
      this.$emit('setCustomPeriod', dataObj);
    },

    // 在这个函数里对值整体操作然后会向上抛出方法重新请求这个函数
    // 确认后对item值进行更新
    onConfirm() {
      // 非快速划分才需要校验
      if (this.customType == 1 && !this.hasSetFast && this.$refs.formContent) {
        this.$refs.formContent.validate(res => {
          // console.log(res, 'res?');
          if (res) {
            this.setCustomPeroid();
            this.handleClose();
          }
        });
      } else {
        this.setCustomPeroid();
        this.handleClose();
      }
    },
    handleClose() {
      this.$emit('handleCloseSetting');
    },
    clearDiyData() {
      this.hasSetFast = false;
      this.arr = [];
      this.minValue = '';
      this.maxValue = '';
      this.periodNum = '';
    },
    // 不清空数据了，只切换。
    setCustomType(val) {
      this.curCustomType = val;

      // 如果切换到了默认，需要清空所有数据和操作
      // if (val == 0) {
      //   this.clearDiyData();
      // } else {
      //   // 切换到自定义需要至少保留一个
      //   this.arr = [{ minValue: '', maxValue: '', unitName: this.tagInfo.unitName }];
      // }
    }
  }
};
</script>

<style lang="scss" scoped>
@import '~@/views/personas/components/usergrouping/el-dialog.scss';
@import '~@/views/personas/components/usergrouping/addUserPackage/common.scss';

.list-item-label {
  width: 60px;
}

/deep/ .el-icon-plus {
  font-weight: bolder;
  font-size: 12px;
  width: 12px;
  height: 12px;
}

.add_group {
  padding-bottom: 0 !important;
}

/deep/ .el-form-item {
  margin-bottom: 0;

  .el-form-item__content {
    line-height: 100%;
  }
  .el-form-item__error {
    position: absolute;
    padding: 0.01rem;
  }
}
.icon-delete {
  margin-left: 16px;
}
.list-period-setting-wrap {
  max-height: 256px;
  overflow: scroll;
}
.point-setting {
  margin-right: 8px;
  width: 6px;
  height: 6px;
  background: #1abfbf;
  border-radius: 50%;
}
.btn-add-period-wrap {
  display: flex;
  align-items: center;
  margin-top: 16px;

  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 500;
  color: #007979;

  cursor: pointer;
}
.list-period-setting-item {
  display: flex;
  align-items: center;

  margin-bottom: 0.2rem;
  &:not(:last-of-type) {
    margin-bottom: 16px;
  }

  /deep/ .el-input {
    margin: 0 8px;
    width: 158px;
    height: 32px;
    border: 1px solid #bac2cd;
    overflow: hidden;

    border-radius: 4px;

    .el-input__inner {
      width: 158px;
      height: 32px;
      border: 1px solid #bac2cd;

      border: none;
    }
  }
}
.btn-text {
  font-size: 14px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #6a6e8c;

  &-confirm {
    margin-left: 16px;
    color: #007979;
  }
}
.setting-container-header {
  display: flex;
  justify-content: space-between;

  .setting-label {
    margin-bottom: 16px;

    font-size: 16px;
    font-family: PingFang SC, PingFang SC;
    font-weight: 500;
    color: #6a6e8c;
  }

  .btn-text-shortcut {
    display: flex;
    align-items: center;

    font-size: 14px;
    font-family: PingFang SC, PingFang SC;
    font-weight: 500;
    color: #007979;

    cursor: pointer;
  }
}
.tips {
  margin-top: 16px;
  margin-bottom: 24px;

  font-size: 12px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: #9399ae;
}
.type-container {
  display: flex;
}
.com-header-right {
  box-sizing: border-box;
  padding: 16px;
  padding-bottom: 8px;

  width: 522px;
  height: 84px;

  border: 1px solid rgba(140, 223, 223, 1);
  border-radius: 2px;
}
.setting-container-shortcut-header {
  display: flex;
  justify-content: center;

  /deep/ .el-input {
    margin-right: 8px;
    width: 158px;
    height: 32px;

    .el-input__inner {
      width: 158px;
      height: 32px;

      &:focus {
        border-color: #1abfbf;
      }
    }
  }
}
.com-header {
  display: flex;
  justify-content: space-between;
}
.setting-container-shortcut-footer {
  display: flex;
  justify-content: flex-end;

  margin-top: 8px;
}
.btn-type {
  margin-right: 24px;

  width: 120px;
  height: 40px;
  border-radius: 2px;
  text-align: center;
  line-height: 40px;

  font-size: 16px;
  font-family: PingFang SC, PingFang SC;
  font-weight: 400;
  color: rgba(106, 110, 140, 1);

  border: 1px solid rgba(186, 194, 205, 1);

  cursor: pointer;

  &-active {
    color: rgba(0, 121, 121, 1);
    border-color: rgba(26, 191, 191, 1);
  }
}
/deep/ .el-dialog {
  left: 50%;
  top: 50%;
  position: relative;
  margin: 0 !important;
  transform: translate(-50%, -50%);

  width: 804px;
  height: fit-content;
  border: none;
  box-shadow: 0px 0px 6px 0px rgba(0, 0, 0, 0.12);

  .el-dialog__header {
    position: relative;
    padding: 26px 30px 20px;
    .el-dialog__title {
      font-family: PingFangSC-Semibold;
      font-size: 18px;
      color: #9cdeff;
      letter-spacing: 0;
      line-height: 18px;
    }
    .el-icon-close:before {
      content: '';
      width: 30px;
      height: 30px;
      background: url('~@/assets/img/siteSel/recommend_close.svg') no-repeat center;
      background-size: 100% 100%;
      display: inline-block;
      cursor: pointer;
    }
    &::before {
      content: '';
      position: absolute;
      bottom: 0;
      display: inline-block;
      // width: 100%;
      width: 93%;
      height: 1px;
      // background-image: linear-gradient(90deg, #9cdeff 0%, rgba(156, 222, 255, 0) 95%);
    }
  }

  .el-dialog__body {
    padding: 24px 30px 0px;
  }

  .el-dialog__footer {
    padding: 40px 30px 20px;
    padding-top: 24px;
    .el-button--default {
      border: none;
      background-color: transparent;
      font-family: PingFangSC-Medium;
      font-size: 16px;
      color: #d2f0ff;
      letter-spacing: 0;
      text-align: center;
    }
    .el-button--primary {
      width: 120px;
      height: 40px;
      text-align: center;
      font-family: PingFangSC-Medium;
      font-size: 16px;
      color: #28fff5;
      letter-spacing: 0;
      background: url('~@/assets/img/personas/downLoad.svg') no-repeat center;
      background-size: 100% 100%;
      border: none;
    }
  }
}
</style>
