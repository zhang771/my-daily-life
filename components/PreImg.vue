<script lang='ts' setup>
interface Props {
  src: string
  height?: number | string
  width?: number | string
}

const props = defineProps<Props>()

const { src } = toRefs(props)
const modalVisible = ref(false)
</script>

<template>
  <div>
    <nuxt-img
      cursor-pointer
      :src="src"
      @click="modalVisible = true"
    />
    <Modal v-model:model-visible="modalVisible" :bg-transparent="true">
      <img
        :src="src"
      >
    </Modal>
  </div>
</template>

<style scoped></style>
