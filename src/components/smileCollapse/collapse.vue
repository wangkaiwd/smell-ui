<template>
  <div class="smile-collapse">
    <slot></slot>
  </div>
</template>

<script>
  export default {
    name: 'SmileCollapse',
    props: {
      selected: {
        type: Array,
        default: () => []
      },
      single: {
        type: Boolean,
        default: false
      },
    },
    methods: {
      updateSelected (newSelected) {
        this.$emit('update:selected', newSelected);
      }
    }
  };
</script>

<style lang="scss" scoped>
  @import "~styles/vars";
  @import "~styles/mixins";
  .smile-collapse {
    border: $border-gray;
    border-radius: $border-radius-md;
    &-item:last-child {
      /deep/ div:last-child {
        border: none;
      }
    }
  }
</style>
