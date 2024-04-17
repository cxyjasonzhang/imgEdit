<template>
  <div>
    <div v-if="config.type === 'input'">
      <el-input v-model="value" placeholder="test"></el-input>
    </div>
    <div v-if="config.type === 'select'">
      <el-select v-model="value" placeholder="????" clearable>
        <el-option v-for="item in config.options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
    </div>
    <div v-if="config.type === 'datepicker'">
      <el-date-picker
        v-model="value"
        type="date"
        placeholder="选择日期">
      </el-date-picker>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FormItem',
  model: {
    prop: 'modelValue',
    event: 'change'
  },
  props: {
    config: {
      type: Object,
      default: () => ({})
    },
    modelValue: {
      type: null,
      default: ''
    }
  },
  data() {
    return {
      // value: this.modelValue
    }
  },
  computed: {
    value: {
      get() {
        return this.modelValue
      },
      set(val) {
        this.$emit('change', val)
      }
    }
  }
}
</script>

<style>

</style>