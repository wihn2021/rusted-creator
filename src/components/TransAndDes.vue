<template>
<span @click="changeShow">{{ showTrans?trans:des }}: </span>
</template>

<script>
export default {
  props: {
    trans: {
      type: String,
      value: "未添加翻译"
    },
    des: {
      type:String,
      value: "未添加描述"
    }
  },
  name: "TransAndDes",
  data() {
    return {
      showTrans: true
    }
  },
  methods: {
    changeShow() {
      let h = document.querySelector("#hint")
      h.innerHTML = this.$props.des
    }
  }
}
</script>

<style scoped>

</style>
