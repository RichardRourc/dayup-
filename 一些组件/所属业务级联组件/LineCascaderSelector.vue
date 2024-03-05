<template>
    <div class="selector">
        <el-cascader
            ref="cascader"
            :props="{ checkStrictly: !multiple, multiple, expandTrigger: 'hover', ...propsParam }"
            :options="options || fetchOptions"
            :size="size"
            :value="currentValue"
            :filterable="filterable"
            popper-class="selector-none-radio"
            :placeholder="multiple ? '' : isReach ? '请选择业务线' : '全部业务线'"
            style="width: 100%"
            :disabled="disabled"
            v-on="$listeners"
        >
            <template slot-scope="{ node, data }">
                <div @click="(e) => clickNode(node, data, e)">{{ data.label }}</div>
            </template>
        </el-cascader>
    </div>
</template>

<script>
import request from "@/utils/request";
export default {
    props: {
        isReach: {
            default: false,
            type: Boolean,
        },
        value: {
            type: Array,
            default: () => [],
        },
        options: {
            type: Array,
            default: () => null,
        },
        size: {
            type: String,
            default: "normal",
        },
        multiple: {
            type: Boolean,
            default: false,
        },
        changeOnSelect: {
            type: Boolean,
            default: true,
        },
        mode: {
            type: String,
            default: "radio",
        },
        disabled: {
            type: Boolean,
            default: false,
        },
        filterable: {
            type: Boolean,
            default: false,
        },
        propsParam: {
            type: Object,
            default: () => {},
        },
    },
    data() {
        return {
            fetchOptions: [],
            loading: false,
        };
    },
    computed: {
        currentValue() {
            if (!this.value.length) {
                return [""];
            } else {
                return this.value;
            }
        },
    },
    mounted() {
        this.loading = true;
        if (!this.options) {
            request({
                url: "/hivebrain/common/allLineList",
            }).then((res) => {
                const opt = (res.data || []).map((v) => ({
                    value: v.lineId,
                    label: v.lineName,
                    children: [],
                }));
                this.loading = false;
                if (!this.multiple && !this.isReach) {
                    opt.unshift({
                        value: "",
                        label: "全部业务线",
                    });
                }
                console.log(opt, "opt");
                this.fetchOptions = opt;
                this.fetchOptions.forEach((v) => {
                    this.handleItemChange([v.value]);
                });
            });
        }
    },
    methods: {
        clickNode(node, data) {
            const { level } = node;
            let value = [data.value];
            const panel = this.$refs.cascader.panel;
            if (!this.multiple) {
                if (data.value) {
                    if (node.parent) {
                        value.unshift(node.parent.data.value);
                    }
                } else {
                    value = [];
                }
                panel.handleCheckChange(value);
            } else {
                if (!node.hasChildren) {
                    node.doCheck(!node.checked);
                    panel.calculateMultiCheckedValue();
                }
            }
        },
        handleItemChange(val) {
            if (val.length === 1 && val[0]) {
                const target = this.fetchOptions.filter((v) => v.value === val[0])[0];
                if (target && target.children.length === 0) {
                    return request({
                        url: "/hivebrain/common/getByLine",
                        params: {
                            lineId: val[0],
                        },
                    }).then((res) => {
                        this.$set(
                            target,
                            "children",
                            (res.data || []).map((v) => ({
                                value: v.serviceId,
                                label: v.serviceName,
                            }))
                        );
                    });
                }
            }
        },
    },
};
</script>

<style lang="scss" scoped>
/deep/ .el-cascader .el-input .el-icon-arrow-down {
    font-size: 0.12rem;
    color: #8688a5;
}

/deep/ .el-cascader {
    .el-input.is-focus .el-input__inner {
        border-color: #1abfbf;
    }
}
</style>

// 全局样式，用于浮窗样式修改
<style lang="scss">
.selector-none-radio .el-radio {
    display: none;
}

// 所属业务级联自定义样式
.selector-none-radio.el-popper {
    background: #ffffff;
    .el-cascader-node {
        &.in-active-path,
        &.is-active,
        &.is-selectable.in-checked-path {
            color: #007979;
        }
    }

    .el-cascader-node:not(.is-disabled):hover {
        background-color: #daf5f5;
    }
}
</style>
