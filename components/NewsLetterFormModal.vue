<template>
  <modal-overlay
    id="newsletter-modal"
    :title="title"
    trigger-text="Subscribe To Newsletter55"
    class="newsletter-modal"
  >
    <news-letter-form />
  </modal-overlay>
</template>
<script>
import ModalOverlay from '~/components/ModalOverlay'
import NewsLetterForm from '~/components/NewsLetterForm'
export default {
  components: { ModalOverlay, NewsLetterForm },
  computed: {
    title() {
      return (
        this.$siteConfig.newsletter.heading || 'Subscribe To Our Newsletter44'
      )
    }
  }
}
</script>

<style lang="scss">
.newsletter-modal {
  .field.has-addons {
    justify-content: center;
  }
}
</style>
