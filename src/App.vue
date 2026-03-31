<template>
  <div class="app">
    <header>
      <h1>Root CA Certificate Fetcher</h1>
      <p class="subtitle">Download root CA certificates for any domain</p>
    </header>

    <main>
      <CertificateForm />
    </main>

    <footer>
      <p>For educational and administrative purposes only</p>
    </footer>
  </div>
</template>

<script setup lang="ts">
import CertificateForm from './components/CertificateForm.vue'
</script>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  color: #ffffff;
}

header {
  text-align: center;
  padding: 2rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

h1 {
  margin: 0 0 0.5rem 0;
  font-size: 2rem;
  font-weight: 600;
}

.subtitle {
  margin: 0;
  color: #a0a0a0;
  font-size: 1rem;
}

main {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 2rem;
}

footer {
  text-align: center;
  padding: 1.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  color: #666;
  font-size: 0.875rem;
}
</style>
