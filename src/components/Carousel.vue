<template>
  <div
    class="ssr-carousel"
    v-if="$slots.default() && $slots.default().length"
    :key="$slots.default().length"
    :data-ssrc-id="scopeId"
    @keyup.tab="onTab"
  >
    <span v-html="instanceStyles" />
    <div class="ssr-carousel-slides">
      <div class="ssr-peek-values" ref="peekValues" :style="peekStyles"></div>
      <div
        class="ssr-carousel-mask"
        ref="mask"
        v-on="maskListeners"
        :class="{
          pressing,
          disabled,
          'no-mask': overflowVisible,
          'not-draggable': noDrag,
        }"
      >
        <CarouselTrack
          ref="track"
          v-bind="{
            dragging,
            trackTranslateX,
            slideOrder,
            activeSlides,
            leftPeekingSlideIndex,
            rightPeekingSlideIndex,
          }"
        >
          <template #default>
            <slot></slot>
          </template>

          <template v-if="hasPeekClones" #clones>
            <slot></slot>
          </template>
        </CarouselTrack>
      </div>
      <CarouselArrows
        v-if="showArrows"
        v-bind="{ index, pages, shouldLoop }"
        @back="back"
        @next="next"
      >
        <template #back="props">
          <slot name="back-arrow" v-bind="props"></slot>
        </template>
        <template #next="props">
          <slot name="next-arrow" v-bind="props"></slot>
        </template>
      </CarouselArrows>
      <CarouselDots v-if="showDots" v-bind="{ boundedIndex, pages }" @goto="gotoDot">
        <template #dot="props">
          <slot name="dot" v-bind="props"></slot>
        </template>
      </CarouselDots>
      <div class="ssr-carousel-visually-hidden" aria-live="polite" aria-atomic="true">
        {{ currentSlideMessage }}
      </div>
    </div>
  </div>
</template>

<script>
import CarouselArrows from '@/components/CarouselArrows.vue'
import CarouselDots from '@/components/CarouselDots.vue'
import CarouselTrack from '@/components/CarouselTrack.vue'

import accessibility from '@/lib/accessibility'
import autoplay from '@/lib/autoplay'
import dimensions from '@/lib/dimensions'
import dragging from '@/lib/dragging'
import feathering from '@/lib/feathering'
import focus from '@/lib/focus'
import gutters from '@/lib/gutters'
import looping from '@/lib/looping'
import pagination from '@/lib/pagination'
import peeking from '@/lib/peeking'
import responsive from '@/lib/responsive'
import tweening from '@/lib/tweening'
import variableWidth from '@/lib/variable-width'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Carousel',
  mixins: [
    accessibility,
    autoplay,
    dimensions,
    dragging,
    feathering,
    focus,
    gutters,
    looping,
    pagination,
    responsive,
    peeking, // After `responsive` so prop can access `gutter` prop
    tweening,
    variableWidth,
  ],
  components: {
    CarouselArrows,
    CarouselDots,
    CarouselTrack,
  },
  emits: [
    'update:modelValue',
    'tween:start',
    'tween:end',
    'drag:start',
    'drag:end',
    'change',
    'release',
    'press',
  ],
  props: {
    showArrows: {
      type: Boolean,
    },
    showDots: {
      type: Boolean,
    },
  },
  computed: {
    // Combine the different factors that come together to determine the x
    // transform of the track.  We don't return a value until the carousel
    // width is measured since the calculation depends on that.
    trackTranslateX() {
      if (!(this.carouselWidth && !this.disabled)) return
      return (
        this.currentX + // The value from tweening or dragging
        this.trackLoopOffset + // Offset from re-ordering slides for looping
        this.peekLeftPx // Offset slides for the left peek
      )
    },
    watchesHover() {
      return this.autoplayDelay > 0
    },
    maskListeners() {
      if (this.disabled) return {}
      return {
        ...(this.noDrag
          ? {}
          : {
              mousedown: this.onPointerDown,
              touchstart: this.onPointerDown,
            }),
        ...(!this.watchesHover
          ? {}
          : {
              mouseenter: this.onEnter,
              mouseleave: this.onLeave,
            }),
      }
    },
  },
}
</script>

<style lang="scss">
.ssr-carousel {
  touch-action: pan-y;

  * {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
     box-sizing: border-box;

    &:before, &:after {
      box-sizing: border-box;
    }
  }
}

.ssr-carousel-slides {
  position: relative;
}

.ssr-peek-values {
  position: absolute;
}

.ssr-carousel-mask {
  position: relative;

  &:not(.no-mask) {
    overflow: hidden;
  }

  &:not(.disabled):not(.not-draggable) {
    cursor: grab;
  }

  &:not(.disabled):not(.not-draggable).pressing {
    cursor: grabbing;
  }
}

.ssr-carousel-visually-hidden {
  border: 0;
  clip: rect(0 0 0 0);
  clip-path: inset(50%);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
  white-space: nowrap;
}
</style>
