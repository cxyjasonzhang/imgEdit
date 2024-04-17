<template>
  <div>
    洒水卡斯卡金阿奎
    <HySearchForm :searchFormConfig="searchFormConfig"></HySearchForm>
  </div>
</template>

<script>
import HySearchForm from '../components/hySearchForm/HySearchForm.vue'
export default {
  name: 'SearchForm',
  data() {
    return {
      searchFormConfig: [
        {
          type: 'input',
          name: 'name',
          value: '',
          label: '用户名'
        },
        {
          type: 'input',
          name: 'realname',
          value: '',
          label: '姓名'
        },
        {
          type: 'select',
          name: 'gender',
          value: '',
          options: [
            {
              label: '男',
              value: '1'
            },
            {
              label: '女',
              value: '2'
            }
          ],
          label: '性别'
        },
        {
          type: 'datepicker',
          name: 'date',
          value: '',
          label: '入职时间'
        }
      ]
    }
  },
  components: {
    HySearchForm
  }
}
</script>

<style>

</style>