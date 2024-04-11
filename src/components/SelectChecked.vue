<template>
    <!-- 下拉多选框 -->
    <el-select 
    v-model="selectedOptions"
    multiple
    clearable
    placeholder="请选择"
    :popper-apped-to-body="false"
    @remove-tag="removeTag"
    style="width:100%"
    >
        <el-option 
        v-for="item in options" 
        :key="item.value" 
        :label="item.label" 
        :value="item.value"
        >
          <el-checkbox v-model="item.check" @change="isChecked(item)">.
            {{ item.label }}
          </el-checkbox>
        </el-option>
        <div class="is-all">
            <div @click="selectAll(true)">全选</div>
            <div @click="selectAll(false)">反选</div>
        </div>
    </el-select>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
//定义下拉数据类型
type SelectIem = {
    value: string|number,
    label: string,
    check: boolean,
}

//接受父组件参数
let props = defineProps({
    options: {
        type: Array<SelectIem>,
        required: true,
    },
    width: {
        type: Number,
        default() {
            return 220;
        }
    }
})
//注册事件
const emit = defineEmits(['selected'])
//下拉的数据
let selectedOptions = ref<Array<string | number>>([])
//下拉的点击事件
const isChecked = (item: SelectIem) => {
    if (item.check && selectedOptions.value.indexOf(item.value)==-1) {
        selectedOptions.value.push(item.value)
    } else if (!item.check) {
        selectedOptions.value.forEach((elm, idx) => {
            if (elm == item.value) {
                selectedOptions.value.splice(idx, 1)
            }
        })
    }
    emit("selected", selectedOptions.value)
}
//删除tag
const removeTag = (value: any) => {
    props.options.forEach((elm) => {
        if (elm.value == value) {
            elm.check = false
        }
    })
    emit("selected", selectedOptions.value)
}
// 全选或反选
const selectAll = (isAll: boolean) => {
    if (isAll) {
        selectedOptions.value = [];
        props.options.forEach((item: SelectIem) => {
            item.check = true;
            selectedOptions.value.push(item.value);
        });
    } else {
        // 反选逻辑简化如下
        const currentlySelected = new Map(props.options.map((option) => [option.value, option.check]));
        //奖励所有，将check属性设为反值
        props.options.forEach((item: SelectIem) => {
            item.check = !currentlySelected.get(item.value);
        });
        //重新计算 selectedOptions.value，仅包含当前选中的选项值
        selectedOptions.value = props.options
            .filter((item) => item.check)
            .map((item) => item.value);
    }
    emit("selected", selectedOptions.value);
};

//清空下拉的数据
const clear = () => {
  selectedOptions.value=[];
}
//暴露出去，给外部组件使用
defineExpose({
  clear
})
</script>
<style lang="scss" scoped>
.is-all{
    display: flex;
    padding-left:10px;
    div{
        cursor:pointer;
        margin:6px 10px;
        transition:0.2s;
        &:hover{
            opacity: 0.7;
        }
    }
}

</style>