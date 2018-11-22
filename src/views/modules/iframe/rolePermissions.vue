<template>
  <iframe-component :iframeSrc="iframeSrc"></iframe-component>
</template>
<script>
import IframeComponent from './iframeComponent.vue'
export default {
  components: {
    IframeComponent
  },
  data () {
    return {
      iframeSrc: 'rolePermissions'
    }
  }
}
</script>
