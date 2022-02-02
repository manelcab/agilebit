<template>
  <div id="about-page" class="page-wrapper about-page content-page">
    <site-hero
      title="About Us"
      subtitle=";)"
      image="/uploads/about-jonas-stolle-6rhdGmD6wfw-unsplash.jpg"
    ></site-hero>
    <main-section theme="sidebar-right">
      <template v-slot:default>
        <div class="content">
          <h3>Hi .... ;)</h3>
          <p>
            I am a curious mind, I like to discover, understand how it works,
            connect the dots, try and learn. At the beginning discovering the
            exciting world of electronics, then diving between
            telecommunications and computing, looking for ways to link
            technologies, IOT, kubernetes, maker, lora, frontend and backend
            development, 3D printing, mqtt, etc... . Enjoying devices like
            arduinos, raspberry, nodemcus, esp8266, ..., adopting Agile
            methodologies, Management 3.0, ... Currently researching the world
            of cryto, NFTs, DEX, Blockchain, ... and whatever comes next.
            <br />
            Thank you for visiting my blog, if you want to contact me @Lenambac_
          </p>
          <quote-card />
        </div>
      </template>

      <template v-slot:sidebar>
        <h3 class="subtitle is-4">
          Latest Posts
        </h3>
        <posts-grid :per-row="1" :number="2" />
      </template>
    </main-section>
  </div>
</template>
<script>
import QuoteCard from '~/components/cards/QuoteCard'
export default {
  head() {
    return {
      title: `About | ${this.$siteConfig.siteName}`
    }
  },
  components: { QuoteCard }
}
</script>
