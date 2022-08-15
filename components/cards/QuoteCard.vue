<template>
  <div class="card quote-card">
    <div class="card-content">
      <p class="title">
        "The only real mistake is the one from which we learn nothing"
      </p>
      <p class="subtitle">
        ~ Henry Ford
      </p>
    </div>
    <footer class="card-footer">
      <div class="card-footer-item">
        <span>
          Also you can contact with me by the following form:
        </span>
      </div>
      <!--
      <div class="card-footer-item">
        <span> Share on <a href="#">Facebook</a> </span>
      </div>
      -->
    </footer>
    <div class="card-footer-item">
      <span>
        <form name="contact" method="POST" data-netlify="true">
          <p>
            <label>Email: <input type="email" name="email"/></label>
          </p>
          <p>
            <label>Message: <textarea name="message"></textarea></label>
          </p>
          <p>
            <button type="submit">Send</button>
          </p>
        </form>
      </span>
    </div>
  </div>
</template>
