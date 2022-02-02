<template>
  <div>
    <form
      target="_blank"
      method="post"
      :action="formAction"
      @submit="$emit('submit', email)"
    >
      <div class="field has-addons is-fullwidth">
        <p class="control has-icons-left">
          <!-- <input
            v-model="email"
            class="input"
            type="email"
            name="EMAIL"
            placeholder="EmailXXXXXXX"
          /> -->
          <!--
            Esto es un icono
          <span class="icon is-small is-left-asds">
            <font-awesome-icon icon="envelope" />
            <i class="fas fa-envelope-asds"></i>
          </span>
          -->
        </p>
        <div class="control">
          <p>
            By using our website you agree to our use of cookies
          </p>
          <!--
            Esto es el botón de ok
           <button type="button" class="button is-primary">
             {{ $siteConfig.newsletter.btnText || 'Subscribe' }}
           </button>
          -->
        </div>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: ''
    }
  },
  computed: {
    formAction() {
      if (this.$siteConfig.newsletter.mailchimp.on) {
        return this.$siteConfig.newsletter.mailchimp.formAction
      }
      return this.$siteConfig.newsletter.custom.formAction
    }
  }
}
</script>
