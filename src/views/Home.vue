<template>
    <div>
        <el-button type="info" @click="next">退出</el-button>
    </div>
</template>

<script>
export default {
  methods: {
    next () {
      sessionStorage.removeItem('token')
      this.$router.push('/login')
    }
  }
}
</script>

<style>

</style>
