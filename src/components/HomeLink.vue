<template>
  <div @click="goHome" class="home-link">
    <router-link to="/" class="home" :class="{ invert }" event="">
      <font-awesome-icon class="icon" :icon="faHouse" fixed-width />
      <span class="text">Home</span>
    </router-link>
  </div>
</template>

<script>
import { faHouse } from '@fortawesome/free-solid-svg-icons';

export default {
  props: {
    invert: { type: Boolean, default: true },
  },
  data() {
    return {
      faHouse,
    };
  },
  methods: {
    goHome(e) {
      if (e.pointerType === 'touch') {
        // if touched, show animation first before going home (computers will see animation on hover)
        setTimeout(() => {
          this.$router.push('/');
        }, 130);
      } else {
        this.$router.push('/');
      }
    },
  },
};
</script>

<style lang="sass" scoped>
@import '@/styles/style.sass'

.home-link
  .home
    color: var(--secondary)
    border-radius: 100px
    padding: 7px
    transition: box-shadow .2s
    text-decoration: none
    display: flex
    align-items: center
    background-color: var(--background)
    z-index: 1
    transition: background-color .2s, color .2s

    .icon
      font-size: 1.5em

    .text
      display: block
      max-width: 0
      overflow: hidden
      transition: max-width .2s
      font-size: 1.1em

    &:hover
      +shadow

      .text
        max-width: 60px
        margin: 0 5px

    &.invert:hover
      color: white
      background-color: var(--color)

</style>
