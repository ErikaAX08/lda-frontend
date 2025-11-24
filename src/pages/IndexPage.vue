<template>
  <q-page class="page-container">
    <div class="content-wrapper">
      <!-- Header Section -->
      <div class="text-center q-mb-xl">
        <h3 class="text-h3 text-weight-bold q-mb-md gradient-text">Análisis de Tópicos con LDA</h3>
        <p class="text-h6 text-grey-7">Extrae tópicos de tus libros usando Muestreo de Gibbs</p>
      </div>

      <!-- Upload Section -->
      <q-card class="upload-card" flat bordered>
        <q-card-section>
          <div class="text-h6 q-mb-md">
            <q-icon name="cloud_upload" size="sm" class="q-mr-sm" />
            Cargar Documento PDF
          </div>

          <!-- Drop Zone -->
          <div
            class="drop-zone"
            :class="{ 'drop-zone-active': isDragging }"
            @drop.prevent="onDrop"
            @dragover.prevent="isDragging = true"
            @dragleave.prevent="isDragging = false"
          >
            <q-icon
              :name="pdfFile ? 'check_circle' : 'cloud_upload'"
              :color="pdfFile ? 'positive' : 'primary'"
              size="64px"
              class="q-mb-md"
            />

            <div v-if="!pdfFile">
              <p class="text-h6 q-mb-sm">Arrastra tu archivo PDF aquí</p>
              <p class="text-grey-6">o</p>
              <q-btn
                rounded
                color="primary"
                label="Seleccionar archivo"
                icon="attachment"
                class="q-mt-sm"
                @click="$refs.fileInput.pickFiles()"
              />
            </div>

            <div v-else>
              <p class="text-h6 text-positive q-mb-sm">
                {{ pdfFile.name }}
              </p>
              <p class="text-grey-6">
                {{ formatFileSize(pdfFile.size) }}
              </p>
              <q-btn
                flat
                rounded
                color="negative"
                label="Cambiar archivo"
                icon="close"
                class="q-mt-sm"
                @click="pdfFile = null"
              />
            </div>
          </div>

          <!-- Hidden File Input -->
          <q-file ref="fileInput" v-model="pdfFile" accept=".pdf" style="display: none" />
        </q-card-section>

        <!-- Parameters Section -->
        <q-card-section v-if="pdfFile">
          <div class="text-h6 q-mb-md">
            <q-icon name="tune" size="sm" class="q-mr-sm" />
            Parámetros del Modelo
          </div>

          <div class="row q-col-gutter-md">
            <!-- Number of Topics -->
            <div class="col-12 col-md-6">
              <div class="parameter-card">
                <div class="text-subtitle1 q-mb-sm">
                  Número de Tópicos
                  <q-icon name="info" size="xs" class="q-ml-xs">
                    <q-tooltip max-width="300px">
                      Cantidad de temas a extraer del documento (2-20)
                    </q-tooltip>
                  </q-icon>
                </div>
                <q-slider
                  v-model="params.numTopics"
                  :min="2"
                  :max="20"
                  :step="1"
                  label
                  label-always
                  color="primary"
                  class="q-mt-md"
                />
              </div>
            </div>

            <!-- Number of Iterations -->
            <div class="col-12 col-md-6">
              <div class="parameter-card">
                <div class="text-subtitle1 q-mb-sm">
                  Iteraciones
                  <q-icon name="info" size="xs" class="q-ml-xs">
                    <q-tooltip max-width="300px">
                      Más iteraciones = mejor convergencia pero más tiempo (10-1000)
                    </q-tooltip>
                  </q-icon>
                </div>
                <q-slider
                  v-model="params.numIterations"
                  :min="10"
                  :max="500"
                  :step="10"
                  label
                  label-always
                  color="secondary"
                  class="q-mt-md"
                />
              </div>
            </div>

            <!-- Alpha Parameter -->
            <div class="col-12 col-md-6">
              <div class="parameter-card">
                <div class="text-subtitle1 q-mb-sm">
                  Alpha (α)
                  <q-icon name="info" size="xs" class="q-ml-xs">
                    <q-tooltip max-width="300px">
                      Concentración doc-tópico. Menor = tópicos más específicos
                    </q-tooltip>
                  </q-icon>
                </div>
                <q-slider
                  v-model="params.alpha"
                  :min="0.01"
                  :max="1.0"
                  :step="0.01"
                  label
                  label-always
                  color="deep-orange"
                  class="q-mt-md"
                />
              </div>
            </div>

            <!-- Beta Parameter -->
            <div class="col-12 col-md-6">
              <div class="parameter-card">
                <div class="text-subtitle1 q-mb-sm">
                  Beta (β)
                  <q-icon name="info" size="xs" class="q-ml-xs">
                    <q-tooltip max-width="300px">
                      Concentración tópico-palabra. Menor = vocabulario más enfocado
                    </q-tooltip>
                  </q-icon>
                </div>
                <q-slider
                  v-model="params.beta"
                  :min="0.001"
                  :max="0.1"
                  :step="0.001"
                  label
                  label-always
                  color="teal"
                  class="q-mt-md"
                />
              </div>
            </div>
          </div>
        </q-card-section>

        <!-- Action Button -->
        <q-card-section v-if="pdfFile">
          <q-btn
            unelevated
            rounded
            color="primary"
            label="Analizar Documento"
            icon="psychology"
            size="lg"
            class="full-width"
            :loading="loading"
            @click="analyzePDF"
          />
        </q-card-section>
      </q-card>

      <!-- Results Section -->
      <div v-if="results" class="q-mt-xl">
        <!-- Metadata -->
        <q-card class="result-card q-mb-lg" flat bordered>
          <q-card-section>
            <div class="text-h5 q-mb-md">
              <q-icon name="info" class="q-mr-sm" />
              Información del Análisis
            </div>
            <div class="row q-col-gutter-md">
              <div class="col-6 col-md-3">
                <q-chip square color="primary" text-color="white" icon="article">
                  {{ results.metadata.num_documents }} Documentos
                </q-chip>
              </div>
              <div class="col-6 col-md-3">
                <q-chip square color="secondary" text-color="white" icon="topic">
                  {{ results.metadata.num_topics }} Tópicos
                </q-chip>
              </div>
              <div class="col-6 col-md-3">
                <q-chip square color="deep-orange" text-color="white" icon="repeat">
                  {{ results.metadata.num_iterations }} Iteraciones
                </q-chip>
              </div>
              <div class="col-6 col-md-3">
                <q-chip square color="teal" text-color="white" icon="abc">
                  {{ results.metadata.vocabulary_size }} Palabras
                </q-chip>
              </div>
            </div>
          </q-card-section>
        </q-card>

        <!-- Topics -->
        <div class="text-h5 q-mb-md">
          <q-icon name="auto_stories" class="q-mr-sm" />
          Tópicos Encontrados
        </div>
        <div class="row q-col-gutter-md">
          <div v-for="topic in results.topics" :key="topic.topic_id" class="col-12 col-md-6">
            <q-card class="topic-card" flat bordered>
              <q-card-section>
                <div class="text-h6 q-mb-md">Tópico {{ topic.topic_id + 1 }}</div>
                <div class="word-chips">
                  <q-chip
                    v-for="(word, idx) in topic.top_words"
                    :key="word.word"
                    :color="getChipColor(idx)"
                    text-color="white"
                    class="word-chip"
                  >
                    {{ word.word }}
                    <q-badge
                      color="white"
                      :text-color="getChipColor(idx)"
                      :label="`${(word.probability * 100).toFixed(1)}%`"
                      class="q-ml-xs"
                    />
                  </q-chip>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <!-- Convergence Chart -->
        <q-card class="result-card q-mt-lg" flat bordered>
          <q-card-section>
            <div class="text-h5 q-mb-md">
              <q-icon name="show_chart" class="q-mr-sm" />
              Convergencia del Modelo
            </div>
            <canvas ref="convergenceChart" style="max-height: 400px"></canvas>
          </q-card-section>
        </q-card>

        <!-- Mathematical Process -->
        <q-card class="result-card q-mt-lg" flat bordered>
          <q-card-section>
            <div class="text-h5 q-mb-md">
              <q-icon name="functions" class="q-mr-sm" />
              Proceso Matemático
            </div>

            <q-expansion-item
              v-for="(formula, key) in results.mathematical_process.formulas"
              :key="key"
              :label="key"
              icon="calculate"
              class="q-mb-sm formula-item"
            >
              <q-card flat bordered>
                <q-card-section class="bg-grey-2">
                  <code class="formula-code">{{ formula }}</code>
                </q-card-section>
              </q-card>
            </q-expansion-item>

            <div class="q-mt-lg">
              <div class="text-h6 q-mb-sm">Perplejidad Final</div>
              <q-linear-progress
                :value="1 - results.convergence.final_perplexity / 1000"
                color="positive"
                size="25px"
                rounded
              >
                <div class="absolute-full flex flex-center">
                  <q-badge
                    color="white"
                    text-color="positive"
                    :label="results.convergence.final_perplexity.toFixed(2)"
                  />
                </div>
              </q-linear-progress>
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, nextTick } from 'vue'
import axios from 'axios'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const pdfFile = ref(null)
const isDragging = ref(false)
const loading = ref(false)
const results = ref(null)
const convergenceChart = ref(null)

const params = ref({
  numTopics: 5,
  numIterations: 100,
  alpha: 0.1,
  beta: 0.01,
})

const onDrop = (e) => {
  isDragging.value = false
  const files = e.dataTransfer.files
  if (files.length > 0 && files[0].type === 'application/pdf') {
    pdfFile.value = files[0]
  }
}

const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return Math.round((bytes / Math.pow(k, i)) * 100) / 100 + ' ' + sizes[i]
}

const getChipColor = (index) => {
  const colors = ['primary', 'secondary', 'deep-orange', 'teal', 'purple', 'pink', 'indigo']
  return colors[index % colors.length]
}

const analyzePDF = async () => {
  loading.value = true

  const formData = new FormData()
  formData.append('pdf_file', pdfFile.value)
  formData.append('num_topics', params.value.numTopics)
  formData.append('num_iterations', params.value.numIterations)
  formData.append('alpha', params.value.alpha)
  formData.append('beta', params.value.beta)

  try {
    const response = await axios.post('http://localhost:8000/api/analyze-pdf/', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    })

    results.value = response.data

    await nextTick()
    renderConvergenceChart()
  } catch (error) {
    console.error('Error al analizar:', error)
    alert('Error al analizar el documento. Por favor intenta nuevamente.')
  } finally {
    loading.value = false
  }
}

const renderConvergenceChart = () => {
  if (!convergenceChart.value) return

  const ctx = convergenceChart.value.getContext('2d')
  const perplexityData = results.value.convergence.perplexity_history

  new Chart(ctx, {
    type: 'line',
    data: {
      labels: perplexityData.map((d) => `Iter ${d.iteration}`),
      datasets: [
        {
          label: 'Perplejidad',
          data: perplexityData.map((d) => d.perplexity),
          borderColor: 'rgb(102, 126, 234)',
          backgroundColor: 'rgba(102, 126, 234, 0.1)',
          tension: 0.4,
          fill: true,
          pointRadius: 5,
          pointHoverRadius: 7,
        },
      ],
    },
    options: {
      responsive: true,
      maintainAspectRatio: true,
      plugins: {
        title: {
          display: true,
          text: 'Evolución de la Perplejidad',
          font: {
            size: 16,
          },
        },
        legend: {
          display: false,
        },
      },
      scales: {
        y: {
          beginAtZero: false,
          title: {
            display: true,
            text: 'Perplejidad',
          },
        },
        x: {
          title: {
            display: true,
            text: 'Iteraciones',
          },
        },
      },
    },
  })
}
</script>

<style scoped>
.page-container {
  padding: 40px 20px;
  background: linear-gradient(to bottom, #f8f9fa 0%, #ffffff 100%);
  min-height: 100vh;
}

.content-wrapper {
  max-width: 1200px;
  margin: 0 auto;
}

.gradient-text {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.upload-card {
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
}

.upload-card:hover {
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.12);
}

.drop-zone {
  border: 3px dashed #e0e0e0;
  border-radius: 16px;
  padding: 60px 20px;
  text-align: center;
  transition: all 0.3s ease;
  background: #fafafa;
  cursor: pointer;
}

.drop-zone:hover {
  border-color: #667eea;
  background: #f5f7ff;
}

.drop-zone-active {
  border-color: #667eea;
  background: #f0f3ff;
  transform: scale(1.02);
}

.parameter-card {
  padding: 20px;
  border-radius: 12px;
  background: #f8f9fa;
  border: 1px solid #e0e0e0;
}

.result-card {
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.06);
}

.topic-card {
  border-radius: 16px;
  transition: all 0.3s ease;
  border-left: 4px solid #667eea;
}

.topic-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.word-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.word-chip {
  border-radius: 12px;
  font-weight: 500;
}

.formula-item {
  border-radius: 12px;
  overflow: hidden;
}

.formula-code {
  display: block;
  padding: 16px;
  font-size: 14px;
  color: #1a1a1a;
  background: white;
  border-radius: 8px;
  overflow-x: auto;
}

@media (max-width: 768px) {
  .page-container {
    padding: 20px 10px;
  }

  .drop-zone {
    padding: 40px 15px;
  }
}
</style>
