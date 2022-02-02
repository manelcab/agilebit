<template>
  <footer class="site-footer">
    {{ $siteConfig.siteName }} &copy; {{ new Date().getFullYear() }} - All
    Rights Reserved
    <br />
    Made with <span style="color: #e25555;">&#9829;</span> with Nuxt.js and
    Netlify
    <br />
    <div class="seperator">
      <h5>
        <span>Metadata</span>
      </h5>
    </div>
    Unsplash content
    <a href="" @click="gotomyUrl()"> Everest </a>
    <br />
    <a href="" @click="gotomyAwake()"> Thanks to awake template</a>
  </footer>
</template>

<script>
export default {
  name: 'SiteFooter',
  myUrl: 'https://www.ti.com',
  methods: {
    gotomyUrl() {
      window.open('https://unsplash.com/photos/9wg5jCEPBsw')
    },
    gotomyAwake() {
      window.open('https://github.com/danielkellyio/awake-template')
    }
  }
}
</script>

<style scoped>
footer {
  background: #eee;
  padding: 20px 10px;
  text-align: center;
  transition: 0.5s ease all;
}
</style>
