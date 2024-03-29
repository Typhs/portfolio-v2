<script lang="ts" setup>
import TheHeaderConstellation from "@/views/Home/TheHeaderConstellation.vue";
import TheHeaderMoon from "@/views/Home/TheHeaderMoon.vue";

import { onMounted, ref } from "vue";
import { templateRef } from "@vueuse/core";

// ========= Scroll Parallax =========
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import anime from "animejs";

gsap.registerPlugin(ScrollTrigger);
const parallaxContainerEl = templateRef<HTMLElement>("parallax-container");
onMounted(() => {
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: parallaxContainerEl.value,
      start: "top top",
      end: "bottom top",
      scrub: true,
    },
  });

  gsap.utils.toArray(".parallax-layer").forEach((layer: any) => {
    const depth = layer.dataset.depth;
    const movement = -(layer.offsetHeight * depth);
    tl.to(layer, { y: movement, ease: "none" }, 0);
  });
});
// ========= Scroll Parallax =========

const darkeningIntensity = ref(0); // used in Styles v-bind

const speedTimeout = ref<undefined | ReturnType<typeof setTimeout>>(undefined);
const constellationSpeed = ref(1);

function speedUpConstellation() {
  clearTimeout(speedTimeout.value);
  constellationSpeed.value = 5;
  speedTimeout.value = setTimeout(() => {
    constellationSpeed.value = 1;
  }, 100);
}

// ========= Mounted animation =========
const moonContainerEl = templateRef<HTMLElement>("moon-container");
function animateHeaderInit() {
  document.documentElement.scrollTop = 270;
  const timeline = anime.timeline({
    autoplay: false,
  });
  timeline.add(
    {
      targets: moonContainerEl.value,
      opacity: [0, 1],
      scale: [0, 1],
      easing: "spring(1, 50, 20, 16)",
    },
    0,
  );

  timeline.add(
    {
      targets: parallaxContainerEl.value.querySelectorAll(
        ".constellation-container",
      ),
      duration: 1300,
      easing: "spring(3, 50, 20, 1)",
      scale: [0, 1],
    },
    0,
  );

  timeline.add(
    {
      targets: parallaxContainerEl.value.querySelectorAll(".init-move-up"),
      duration: 1000,
      translateY: [500, 0],
      delay: anime.stagger(400),
      easing: "easeOutQuad",
    },
    0,
  );
  timeline.play();
}

onMounted(() => {
  animateHeaderInit();
});
// ========= Mounted animation =========
</script>

<template>
  <div>
    <div class="parallax-container" ref="parallax-container">
      <div class="parallax-layer" data-depth="0.10">
        <div class="parallax-background" />
      </div>

      <div class="parallax-layer" data-depth="0.85">
        <div class="bg-shadow-overlay" />
      </div>

      <div class="parallax-layer" data-depth="0.10">
        <div class="constellation-container">
          <the-header-constellation
            class="h-100 pointer-events-all"
            :speedModifier="constellationSpeed"
          />
        </div>
      </div>

      <div class="parallax-layer" data-depth="0.1">
        <div class="bg-fade-wave init-move-up" />
      </div>

      <div class="parallax-layer h-100" data-depth="0.40">
        <div>
          <div class="center-middle">
            <div ref="moon-container">
              <the-header-moon
                class="pointer-events-all"
                v-model="darkeningIntensity"
                @onHover="speedUpConstellation()"
                @onClick="speedUpConstellation()"
              />
            </div>
          </div>
        </div>
      </div>

      <div class="parallax-layer" data-depth="0.3">
        <div class="bg-back-buildings init-move-up" />
      </div>

      <div class="parallax-layer" data-depth="0.5">
        <div class="bg-front-buildings init-move-up" />
      </div>

      <div class="parallax-layer" data-depth="1">
        <div class="bg-railing-transition" />
      </div>
    </div>

    <div class="after-parallax">
      <div class="min-height-100vh">
        <div>
          <slot name="default"></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
$parallaxHeight: 130vh;

$darkeningIntensity: v-bind(darkeningIntensity);

@mixin darkenLayer {
  filter: brightness(min(calc(1 / ($darkeningIntensity / 25)), 1));
}

.parallax-container {
  height: $parallaxHeight;
  overflow: hidden;
  position: relative;
  max-width: 1920px;
  margin: 0 auto;
  z-index: 0;
  background-color: #12172d;
}
.after-parallax {
  position: relative;
  background-color: $background;
}
.parallax-layer {
  background-position: bottom center;
  background-size: auto;
  background-repeat: no-repeat;
  width: 100%;
  height: $parallaxHeight;
  position: fixed;
  pointer-events: none;
  z-index: -1;
  > * {
    height: 100%;
    width: 100%;
    background-position: bottom center;
    background-size: auto;
    background-repeat: no-repeat;
  }
}

.parallax-background {
  background-image: url("@/assets/heroHeader/header-background.png");
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  opacity: 0.5;
}

.bg-fade-wave {
  background-image: url("@/assets/heroHeader/bg-fade-wave.png");
  background-position: left bottom;
  background-size: 100% auto;
}
.bg-back-buildings {
  background-image: url("@/assets/heroHeader/back-buildings-RESIZED.png");
  background-position: left bottom;
  background-size: 100% auto;
  @include darkenLayer;
}
.bg-front-buildings {
  background-image: url("@/assets/heroHeader/front-buildings-RESIZED.png");
  background-position: bottom;
  background-size: 100% auto;
  @include darkenLayer;
}
.bg-railing-transition {
  background-image: url("@/assets/heroHeader/transition-railing.png");
  background-position: right bottom;
  background-size: 100% auto;
  box-shadow: 0px 100px 0px 0px $background; // this covers the seams that sometimes appear after the end of Parllax Layers
}
.bg-shadow-overlay {
  background-position: bottom;
  background-size: cover;
  background-position: center;
  background-repeat: repeat;
  box-shadow: inset 0 0 calc(#{$darkeningIntensity} * 10vw)
    calc(#{$darkeningIntensity} * 10px) rgba(0, 0, 0, 0.5);
}
.pointer-events-all {
  pointer-events: all;
}

.center-middle {
  position: absolute;
  top: 50vh;
  left: 50%;
  transform: translate(-50%, -50%);
}

.min-height-100vh {
  min-height: 100vh;
}
</style>
