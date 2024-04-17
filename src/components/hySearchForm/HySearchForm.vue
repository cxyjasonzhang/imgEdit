<template>
  <div>
    <template v-for="(item, index) in searchFormConfigModel">
      <FormItem v-model="item.value" :key="index" :config="item"></FormItem>
    </template>
    <el-button type="warming" size="default" @click="restForm">重置</el-button>
    <el-button type="primary" size="default" @click="handlerSearch">搜索</el-button>
  </div>
</template>

<script>
import FormItem from './components/FormItem.vue'
export default {
 name: "HySearchForm",
 props: {
  searchFormConfig: {
    type: Array,
    default: () => []
  }
 },
 data() {
  return {
    fromParams: {},
    searchFormConfigModel: []
  }
 },
 watch: {
  searchFormConfig: {
    handler(newVal, oldVal) {
      console.log(newVal, oldVal, '值变了')
    },
    deep: true
  }
 },
 components: {
  FormItem
 },
 mounted() {
  console.log('挂载')
  this.searchFormConfigModel = this.searchFormConfig
  this.init()
 },
 methods: {
  init() {
    this.searchFormConfig.forEach(item => {
      this.fromParams[item.name] = ''
    })
    console.log(this.fromParams, '****');
  },
  getFormData() {
    this.searchFormConfig.forEach(item => {
      this.fromParams[item.name] = item.value || ''
    })
  },
  restForm() {
    const keys = Object.keys(this.fromParams)
    keys.forEach(key => {
      const columnForm = this.searchFormConfigModel.find(item => item.name === key)
      columnForm.value = ''
    })
    // this.searchFormConfigModel = this.searchFormConfigModel.map(item => {
    //   item.value = ''
    //   return item
    // })
    this.$emit('restForm')
  },
  handlerSearch() {
    this.getFormData()
    console.log('joke',  this.fromParams)
    this.$emit('handlerSearch', this.fromParams)
  }
 }
}
</script>

<style>

</style>