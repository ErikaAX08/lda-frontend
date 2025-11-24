<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-gradient-primary">
      <q-toolbar>
        <q-toolbar-title class="text-weight-bold">
          <q-icon name="auto_stories" size="sm" class="q-mr-sm" />
          LDA Topic Analyzer
        </q-toolbar-title>

        <!-- Redes Sociales -->
        <div class="row q-gutter-sm">
          <q-btn
            flat
            round
            dense
            icon="mdi-instagram"
            href="https://instagram.com/tu-usuario"
            target="_blank"
            class="social-btn"
          >
            <q-tooltip>Instagram</q-tooltip>
          </q-btn>

          <q-btn
            flat
            round
            dense
            icon="mdi-github"
            href="https://github.com/tu-usuario"
            target="_blank"
            class="social-btn"
          >
            <q-tooltip>GitHub</q-tooltip>
          </q-btn>

          <q-btn
            flat
            round
            dense
            icon="mdi-linkedin"
            href="https://linkedin.com/in/tu-usuario"
            target="_blank"
            class="social-btn"
          >
            <q-tooltip>LinkedIn</q-tooltip>
          </q-btn>

          <q-btn
            flat
            round
            dense
            icon="mdi-web"
            href="https://tu-sitio-web.com"
            target="_blank"
            class="social-btn"
          >
            <q-tooltip>Mi Sitio Web</q-tooltip>
          </q-btn>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup></script>

<style scoped>
.bg-gradient-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.social-btn {
  transition: all 0.3s ease;
}

.social-btn:hover {
  transform: translateY(-2px);
  background: rgba(255, 255, 255, 0.1);
}
</style>
