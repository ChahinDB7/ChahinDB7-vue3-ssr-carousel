<template>
  <div class="parent">
    <Carousel show-arrows show-dots>
      <Slide :index="1" />
      <Slide :index="2" />
      <Slide :index="3" />
    </Carousel>
  </div>
</template>

<script setup>
import Carousel from '@/components/Carousel.vue'
import Slide from '@/components/Slide.vue'
</script>

<style lang="scss" scoped>
.parent {
  max-width: 1280px;
  margin: 2rem auto;
  padding: 0 20px;
}
</style>
