<!-- 空组件 -->
<template>
  <div class="empty-box">
    <div class="img-box">
      <p>{{ !props.isSearch ? '这里空空如也' : '没有找到您搜索的内容' }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
const props = defineProps({
  isSearch: {
    type: Boolean,
    default: false
  }
})
</script>

<style scoped lang="less">
.empty-box {
  text-align: center;
  margin: 20px 0;

  p {
    color: #818693;
  }
}
</style>
