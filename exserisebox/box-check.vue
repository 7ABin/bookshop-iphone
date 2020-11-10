<template>
  <div>
    <multi-select
      :allTags="allTags"
            :checked="checked"
            @onChange="onChange" ></multi-select>
  </div>
</template>
<script>
import MultiSelect from '../../components/common/multi-select'
export default {
  components: {
    MultiSelect
  },
  data () {
    return {
      allTags: [
        {
          value: '1',
          text: '全部'
        },
        {
          value: '2',
          text: '不朽者'
        },
        {
          value: '3',
          text: '步兵'
        },
        {
          value: '4',
          text: '工兵'
        },
        {
          value: '5',
          text: '朽者'
        },
        {
          value: '6',
          text: '枪王'
        },
        {
          value: '7',
          text: '火箭筒'
        }
      ],
      checked: []
    }
  },
  methods: {
    onChange (value) {
      // 当勾选标签时触发 value 格式是：['1', '2', '3']
      console.log(value, '父主键的value')
      this.checked = value
    }
  }
}
</script>
