<template>
  <country></country>
</template>
<script>
import country from './country'
export default {
  components: {
    country,
  },
  data() {
    return {
      title: '',
    }
  },
  created() {
    let { from } = this.$route.query
    this.title = '区号选择'
  },
}
</script>
<style lang="scss" scoped>
@import url('./area.scss');
</style>
