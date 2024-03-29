<template>
  <div class="opts_wrapper">
    <div class="opts_name">基础信息</div>
    <div class="basic-opt-content">
      <el-row>
        <el-col :span="24">
          <div class="item">
            <div class="label">分群名称</div>

            <div class="value package-name new-value value-detail">
              <el-tooltip class="item" placement="bottom" popper-class="packageName-tip">
                <div slot="content">
                  {{ packageInfo.packageName }}
                </div>
                <!-- 去掉编辑界面的单独可编辑 -->
                <div class="edit-btn" v-if="editBtnVisible">
                  {{ packageInfo.packageName }}
                </div>
              </el-tooltip>
              <span class="edit-btn" v-if="!editBtnVisible && (isAdmin || packageInfo.isOwner)">
                <el-input v-model="currentPackageName" maxlength="25"></el-input>

                <span class="numTip">{{
                  25 - currentPackageName.length > 0 ? 25 - currentPackageName.length : 0
                }}</span>
                <div @click="onAffirm" class="icon-affirm">
                  <el-tooltip class="item" placement="bottom" popper-class="data-adress">
                    <div slot="content">确认修改</div>
                    <svg-icon :icon-class="'affirm'" class="affirm_icon"></svg-icon>
                  </el-tooltip>
                </div>
              </span>
            </div>
          </div>
        </el-col>
      </el-row>
      <el-row>
        <el-col>
          <div class="item  item-flex">
            <div class="label">分群描述</div>
            <!-- TODO 分群描述 -->
            <div id="textContainer" class="caliber-info-wrap text-container" ref="textContainer">
              <div
                class="celiber-info-content text-content  value value-detail value-detail-desc"
                id="textContent"
                ref="textContent"
                v-html="formatContent(packageInfo.pkgDesc || '-')"
              ></div>
              <span ref="ellipsis" v-if="isOverflow && !hasExpand" class="ellipsis" id="ellipsis">...</span>
              <button
                ref="expandButton"
                v-if="isOverflow"
                class="expand-button"
                id="expandButton"
                @click="toggleExpand"
              >
                {{ (hasExpand && '收起') || '展开' }}
              </button>
            </div>
          </div>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="6" v-if="packageInfo.bussinessName"
          ><div class="item">
            <div class="label">应用业务</div>

            <div class="value new-value len value-detail value-detail-app">{{ packageInfo.bussinessName }}</div>
          </div></el-col
        >
        <el-col :span="4" v-if="packageInfo.appType"
          ><div class="item">
            <div class="label">应用类型</div>

            <div class="value new-value len value-detail value-detail-app">{{ appTypeOpt[packageInfo.appType] }}</div>
          </div></el-col
        >
      </el-row>
      <!-- <basic-corner /> -->
    </div>
  </div>
</template>

<script>
import { BasicCorner } from '../../../index.js';
import { parseTime, showMsg } from '@/utils/index';
import { mapGetters } from 'vuex';
import { UpdateCabinetPackage, UpdatePackage } from '@/api/usergroup';

export default {
  name: 'BasicDetail',
  components: {
    BasicCorner
  },
  props: {
    packageInfo: {
      type: Object,
      required: true,
      default: () => {}
    },
    highlightName: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      // 展开收起用的数据
      hasExpand: false,
      isTextContainerVisible: true,
      isOverflow: false,

      editBtnVisible: true,
      currentPackageName: '',
      userprofileType:
        this.$route.path.split('/')[2] == 'consumer' ? 2 : this.$route.path.split('/')[2] == 'cabinet' ? 3 : 1,
      appTypeOpt: {
        1: '系统调用',
        2: '个人用途'
      }
    };
  },
  computed: {
    ...mapGetters(['isAdmin'])
  },
  mounted() {},
  watch: {
    packageInfo: {
      deep: true,
      handler: function() {
        this.currentPackageName = this.packageInfo.packageName;
        const name = this.userprofileType === 3 ? '柜机包' : '用户包';
        document.title = `丰巢大脑【${name}` + this.packageInfo.id + '】' + this.packageInfo.packageName;

        this.checkShow();
      }
    }
  },
  destroyed() {
    document.title = '丰巢大脑';
  },
  methods: {
    // 预览用的函数
    checkShow() {
      this.isTextContainerVisible = true;
      this.$nextTick(() => {
        this.checkOverflow();
      });
    },
    checkOverflow() {
      const textContainer = this.$refs.textContainer;
      const textContent = this.$refs.textContent;

      // 判断两者都存在
      if (textContainer && textContent) {
        this.isOverflow = textContent.offsetHeight > textContainer.offsetHeight;
      }
    },
    toggleExpand() {
      this.hasExpand = !this.hasExpand;
      this.$refs.textContainer.classList.toggle('expanded');
    },
    // --------
    formatContent(text) {
      if (!text) {
        return '-';
      }
      const str = JSON.stringify(text);
      if (typeof str === 'string') {
        return str.replace(/\\n/g, '<br/>').replace(/"/g, '');
      } else {
        return '-';
      }
    },
    formatterTime(n) {
      return parseTime(n, '{y}-{m}-{d} {h}:{i}:{s}');
    },
    onEdit() {
      if (this.isAdmin || this.packageInfo.isOwner) {
        this.editBtnVisible = false;
      }
    },
    onAffirm() {
      if (this.currentPackageName.match(/^[ ]*$/)) {
        this.currentPackageName = this.packageInfo.packageName;
        this.userprofileType == 3 && showMsg('柜机包名称不能为空', 'warning');
        this.userprofileType != 3 && showMsg('用户包名称不能为空', 'warning');
        return;
      }

      this.currentPackageName = this.currentPackageName.replace(/(^\s*)|(\s*$)/g, '');

      if (this.currentPackageName == this.packageInfo.packageName) {
        this.editBtnVisible = true;
        return;
      }
      const params = {
        id: this.packageInfo.id,
        packageName: this.currentPackageName
      };

      this.userprofileType == 3 &&
        UpdateCabinetPackage(params).then(res => {
          if (res && res.code === '0') {
            this.editBtnVisible = true;
            showMsg('修改柜机包名称成功', 'success');
            this.$emit('updataPackage');
          } else {
            showMsg(res.msg, 'warning');
          }
        });

      this.userprofileType != 3 &&
        UpdatePackage(params).then(res => {
          if (res && res.code === '0') {
            this.editBtnVisible = true;
            showMsg('修改用户包名称成功', 'success');
            this.$emit('updataPackage');
          } else {
            showMsg(res.msg, 'warning');
          }
        });
    }
    // CopyConfiguratio(text) {
    //   let transfer = document.createElement('input');
    //   document.body.appendChild(transfer);
    //   transfer.value = text; // 这里表示想要复制的内容
    //   transfer.focus();
    //   transfer.select();
    //   try {
    //     //进行复制到剪切板
    //     if (document.execCommand('Copy', 'false', null)) {
    //       //如果复制成功
    //       showMsg('复制成功', 'success');
    //     } else {
    //       showMsg('复制失败', 'warning');
    //     }
    //   } catch (err) {
    //     showMsg('复制错误', 'warning');
    //   }

    //   transfer.blur();
    //   document.body.removeChild(transfer);
    // }
  }
};
</script>

<style lang="scss" scoped>
@import '~@/views/personas/components/usergrouping/addUserPackage/BasicOpt.scss';

.item-flex {
  display: flex;
  width: 100%;
}

.text-container {
  margin-left: 0.06rem;
  // max-height: calc(1.2em * 3); /* 3行文本的高度，可以根据需要调整 */
  overflow: hidden;
  position: relative;

  max-height: 0.6rem;

  width: 72%;
  // max-width: 72%;
  // min-width: 0.1rem;
}

.text-content {
  width: auto !important;
  max-width: 100% !important;

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

.opts_name {
  margin-bottom: 16px !important;
  height: auto;
}

// .el-row {
//   &:not(:last-of-type) {
//     margin-bottom: 16px !important;
//   }
// }

.el-col {
  margin-bottom: 16px !important;
  min-height: 20px !important;
  line-height: 20px !important;
  // min-height: 0 !important;
}

.value-detail {
  font-size: 14px !important;
  font-family: MiSans, MiSans !important;
  font-weight: 500 !important;
  color: #383b55 !important;
}

.value-detail-app {
  cursor: auto !important;
}

.basic-opt-content {
  padding: 0;
}

.opts_wrapper {
  .basic-opt-content {
    // padding: 0.16rem 0.3rem 0.16rem;
    margin-bottom: 8px;
    .el-col {
      min-height: 40px;
      line-height: 40px;
      height: fit-content;
      margin-bottom: 0;
    }
    .item {
      width: 100%;
      .label {
        display: inline-block;
        // min-width: 100px;
        margin-right: 8px;

        font-size: 14px;
        font-family: MiSans, MiSans;
        font-weight: 400;
        color: #6a6e8c;

        // color: #9cdeff;

        vertical-align: middle;
      }
      .value {
        display: inline-block;
        font-size: 14px;
        font-family: PingFangSC-Medium, PingFang SC;

        font-weight: 300;
        // color: #ffffff;
        width: 68%;

        min-height: 20px;
        line-height: 20px;
        .id {
          font-size: 18px;
          font-weight: 300;
          font-family: Fragma-Light, Fragma;
          // color: #c7d6f3;
          margin-left: 8px;
        }
        &.num {
          font-size: 18px;
          font-family: Fragma-Light, Fragma;
        }
        &.extra {
          width: 1000px;
          min-height: 54px;
          background: rgba(19, 30, 48, 0.3);
          box-shadow: 0px 0px 24px 0px rgba(43, 88, 165, 0.2);
          border-radius: 8px;
          border: 1px solid #33557f;
          padding: 10px 16px 0 16px;
          display: block;
          margin-left: 100px;
          margin-bottom: 8px;

          .item {
            min-height: 28px;
            line-height: 28px;
          }
          .label {
            font-size: 16px;
            font-family: Fragma-Medium, Fragma;

            &.chinese {
              font-size: 14px;
              font-family: PingFangSC-Medium, PingFang SC;
            }
            font-weight: 500;
            color: #c7d6f3;
            min-width: 70px;
            vertical-align: unset;
          }
          .value {
            font-size: 16px;
            font-family: Fragma-Medium, Fragma;

            &.chinese {
              font-size: 14px;
              font-family: PingFangSC-Medium, PingFang SC;
            }
            font-weight: 500;
            color: #9bdeff;
            background-color: transparent;
            border: none;
            width: 80%;
            min-height: 28px;
            line-height: 28px;
            .copy {
              width: 16px;
              height: 16px;
              display: inline-block;
              background: url('~@/assets/img/personas/copy.svg') no-repeat center;
              background-size: 100% 100%;
              margin-left: 8px;
              transform: translateY(4px);
              cursor: pointer;
            }
          }

          .basicInfo {
            border-top: 1px solid rgba(155, 222, 255, 0.15);
            margin-top: 6px;
            .item {
              width: fit-content;
              min-width: 20%;
              display: inline-block;
              transform: translateY(-1px);
              .label {
                font-size: 16px;
                font-family: Fragma-Medium, Fragma;
                font-weight: 500;
                color: #5a8ac6;

                &.chinese {
                  font-size: 14px;
                  font-family: PingFangSC-Medium, PingFang SC;
                }
              }
              .value {
                font-size: 16px;
                font-family: Fragma-Medium, Fragma;
                font-weight: 500;
                color: #97a5c0;
                width: fit-content;
                margin-right: 40px;
                &.chinese {
                  font-size: 14px;
                  font-family: PingFangSC-Medium;
                }
                .extra {
                  font-size: 12px;
                  font-family: PingFangSC-Medium, PingFang SC;
                  font-weight: 500;
                  color: #97a5c0;
                }
              }
            }
          }
        }

        &.package-name {
          height: 20px;
          line-height: 20px;
          width: 300px;
          .svg-icon {
            margin-left: 10px;
            cursor: pointer;
          }
          .edit-btn {
            width: 20px;
            line-height: 20px;
          }
          .edit_icon {
            font-size: 16px;
          }
          .affirm_icon {
            font-size: 24px;
          }
          .icon-affirm {
            display: inline-block;
            width: 24px;
            height: 24px;
            transform: translate(-10px, 3px);
            .svg-icon {
              margin-left: 0;
            }
          }
          span {
            margin-right: 0;
          }
          .numTip {
            display: inline-block;

            width: 12px;
            height: 40px;
            font-size: 16px;
            font-family: Fragma-Medium, Fragma;
            font-weight: 500;
            color: #97a5c0;
            line-height: 40px;

            transform: translateX(-25px);
          }

          /deep/.el-input {
            // width: 220px;
            // height: 30px;
            line-height: 30px;
            background-color: #05306a;
            border: 1px solid rgba(129, 179, 247, 0.6);
            position: relative;

            .el-input__inner {
              height: 92%;
              font-size: 16px;
              transform: translateY(1px);
            }
          }
        }
      }
      .create {
        width: auto;
        margin-left: 40px;
        font-size: 14px;
        font-weight: 500;
        font-family: Fragma-Medium, Fragma;
        &.first {
          margin-left: 0;
        }
      }
      .new-value {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        vertical-align: middle;
        cursor: pointer;
        &.len {
          width: 160px;
        }
      }
    }
  }
}
</style>

<style lang="scss">
.packageName-tip {
  &.el-tooltip__popper.is-dark {
    background: rgba(10, 40, 68, 0.95);
    border: 1px solid rgba(156, 222, 255, 0.6);
    border-radius: 0;
    font-family: PingFangSC-Regular, PingFang SC;
    font-size: 14px;
    line-height: 20px;
    color: rgba(156, 222, 255, 1);

    &[x-placement^='top'] .popper__arrow {
      border-top-color: rgba(156, 222, 255, 1);
      &::after {
        border-top-color: rgba(20, 49, 84, 1);
      }
    }

    &[x-placement^='bottom'] .popper__arrow {
      border-bottom-color: rgba(156, 222, 255, 1);
      &::after {
        border-bottom-color: rgba(20, 49, 84, 1);
      }
    }
  }
}
</style>
