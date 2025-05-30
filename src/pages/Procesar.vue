<template>
  <div class="container-process pt-4 sm:ml-64">
    <div class="container mx-auto px-4 py-8">
      <h1 class="text-xl md:text-2xl font-semibold text-gray-600 mb-6 pb-4">
        PROCESAMIENTO DE DOCUMENTOS
      </h1>

      <div v-if="!processed" class="bg-white rounded-lg shadow-sm p-6 mb-8">
        <h2 class="text-lg font-medium text-gray-600 mb-4">
          AÑADIR NUEVOS DOCUMENTOS
        </h2>

        <div
          class="border-2 border-dashed border-gray-300 rounded-lg p-6 flex flex-col items-center justify-center text-center bg-gray-50"
          @drop.prevent="handleDrop"
          @dragover.prevent
        >
          <p v-if="!fileSelected && !processing" class="text-gray-500 mb-4">
            Arrastre archivos aquí o ...
          </p>

          <input
            ref="fileInput"
            type="file"
            class="hidden"
            @change="handleFileChange"
            accept=".pdf"
            :disabled="fileSelected"
          />

          <button
            v-if="!fileSelected && !processing"
            class="bg-gradient-to-r from-blue-500 to-indigo-500 text-white font-semibold py-2 px-6 rounded-lg shadow-md hover:from-blue-600 hover:to-indigo-600 focus:outline-none focus:ring-4 focus:ring-blue-300 focus:ring-opacity-60 transition transform hover:scale-105 active:scale-95 flex items-center gap-2"
            @click="selectFile"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
              stroke-width="2"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M12 4v16m8-8H4"
              />
            </svg>
            Seleccionar archivos
          </button>

          <div v-if="loading" class="w-full mt-6">
            <div class="w-full bg-gray-200 rounded-full h-4">
              <div
                class="bg-blue-500 h-4 rounded-full transition-all duration-300"
                :style="{ width: progress + '%' }"
              ></div>
            </div>
            <p class="text-blue-500 mt-2">Cargando archivo...</p>
          </div>

          <div v-if="fileName" class="mt-4 text-gray-700">
            Archivo seleccionado: <strong>{{ fileName }}</strong>
          </div>

          <button
            v-if="fileSelected && !processed && !loading"
            class="mt-4 bg-gradient-to-r from-green-500 to-emerald-500 hover:from-green-600 hover:to-emerald-600 text-white font-semibold py-2 px-6 rounded-lg shadow-md transition transform hover:scale-105 active:scale-95 focus:outline-none focus:ring-4 focus:ring-green-300 focus:ring-opacity-60 flex items-center gap-2"
            @click="iniciarProcesamiento"
            :disabled="processing"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5"
              viewBox="0 0 24 24"
              fill="currentColor"
            >
              <path
                d="M12 4V1L8 5l4 4V6c3.31 0 6 2.69 6 6 0 1.08-.29 2.1-.8 2.98l1.46 1.46A7.938 7.938 0 0020 12c0-4.42-3.58-8-8-8zm-6.66.8L3.88 6.26A7.938 7.938 0 004 12c0 4.42 3.58 8 8 8v3l4-4-4-4v3c-3.31 0-6-2.69-6-6 0-1.08.29-2.1.8-2.98L5.34 4.8z"
              />
            </svg>
            Procesar
          </button>

          <div
            v-if="processing"
            class="flex items-center justify-center gap-2 mt-4"
          >
            <svg
              class="animate-spin h-8 w-8 text-blue-600"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
            >
              <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
              ></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8v8H4z"
              ></path>
            </svg>
            <span class="text-blue-600 font-semibold"
              >Procesando, espere por favor...</span
            >
          </div>
        </div>
        <p class="text-gray-500 mt-4 text-sm md:text-base text-center">
          Formatos soportados : PDF
        </p>
      </div>

      <div v-if="processed" class="bg-white rounded-lg shadow-sm p-6">
        <div class="text-center">
          <p class="text-gray-600 font-medium mb-4">¡Documento procesado!</p>
          <span class="text-blue-700 font-semibold"
            >procesado-{{ fileName }}</span
          ><br />
          <button
            @click="handleDescargarMarkdown"
            class="inline-flex items-center justify-center bg-green-500 hover:bg-green-600 text-white font-medium py-3 px-6 rounded-md transition duration-300 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50 mt-4"
            style="margin-left: 10px"
          >
            <span class="mr-2 text-blue-200">
              <img
                src="../assets/descargar.png"
                alt="download-icon"
                class="w-8 h-8"
              />
            </span>
            <span class="text-center">Descargar</span>
          </button>
        </div>

        <!-- Procesar un nuevo documento -->
        <div class="mt-6 border-t-1 border-gray-200 pt-2">
          <button
            class="bg-gradient-to-r from-blue-400 to-blue-600 text-white font-semibold py-2 px-6 rounded-xl shadow-lg hover:from-blue-500 hover:to-blue-700 focus:outline-none focus:ring-4 focus:ring-blue-300 focus:ring-opacity-50 transition-transform transform hover:scale-105 active:scale-95"
            @click="resetProcess"
          >
            Procesar otro PDF
          </button>
        </div>
      </div>

      <!-- show urls extracted -->
      <div v-if="processed" class="bg-white rounded-lg shadow-sm p-6 mt-6">
        <div v-if="estado.urls && estado.urls.length">
          <h3 class="text-lg font-semibold text-gray-700 mb-3">
            URLs extraídas:
          </h3>
          <ul class="list-disc list-inside space-y-1">
            <li
              v-for="(url, idx) in estado.urls"
              :key="idx"
              class="text-blue-700 hover:underline break-all"
            >
              <a :href="url" target="_blank" rel="noopener noreferrer">{{
                url
              }}</a>
            </li>
          </ul>
        </div>
        <div v-else>
          <pre
            class="bg-gray-100 rounded p-4 text-sm text-gray-700 overflow-x-auto"
            >{{ estado }}</pre
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import {
  procesar_pdf,
  descargarMarkdown,
  obtenerUrlsExtraidas,
  upload_pdf_with_progress,
  consultarEstadoProcesamiento,
} from "../api";

const prompt = ref("simple");
const batchSize = ref(3);
const pauseSeconds = ref(30);
const estado = ref("");
const fileInput = ref(null);
const fileName = ref("");
const fileSelected = ref(false);
const loading = ref(false);
const processing = ref(false);
const processed = ref(false);
const progress = ref(0);

/* función para resetear input */
function resetInput() {
  if (fileInput.value) {
    fileInput.value.value = null;
  }
}

async function iniciarProcesamiento() {
  if (processing.value) return;
  processing.value = true;
  processed.value = false;

  try {
    const resultado = await procesar_pdf({
      filename: fileName.value,
      prompt: prompt.value,
      batchSize: batchSize.value,
      pauseSeconds: pauseSeconds.value,
    });

    estado.value = resultado;

    let nombreBase = fileName.value.replace(/\.pdf$/i, "");

    // Si ya estaba procesado, cargamos directamente las URLs
    if (resultado.status === "ya_procesado") {
      const urls = await obtenerUrlsExtraidas(nombreBase);
      estado.value.urls = urls;
      processed.value = true;
      return;
    }

    // Espera para polling
    await new Promise((r) => setTimeout(r, 1500));

    // Polling hasta que termine (opcional si no haces polling, puedes quitar esto)
    const urls = await obtenerUrlsExtraidas(nombreBase);
    estado.value.urls = urls;
    processed.value = true;
  } catch (e) {
    console.error("Error en el procesamiento:", e);
    estado.value = { status: "error", message: "Error al procesar el PDF" };
  } finally {
    processing.value = false;
  }
}

async function handleDescargarMarkdown() {
  const blob = await descargarMarkdown(fileName.value.replace(/\.pdf$/i, ""));
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = `procesado-${fileName.value.replace(/\.pdf$/i, "")}.md`;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

async function handleFileChange(event) {
  const file = event.target.files[0];
  if (file && file.type === "application/pdf") {
    if (!nameFileValidate(file.name)) {
      alert(
        "⚠️ El nombre del archivo no es válido, debe seguir el formato: ddmmyyyy (ejemplo: 22042025)"
      );
      resetInput();
      return;
    }
    loading.value = true;
    progress.value = 0;
    try {
      const res = await upload_pdf_with_progress(file, (percent) => {
        progress.value = percent;
      });
      fileName.value = res.filename;
      fileSelected.value = true;
    } catch (err) {
      console.error("Error al subir el archivo PDF:", err);
    } finally {
      loading.value = false;
      resetInput();
    }
  } else {
    alert("Solo se permiten archivos PDF.");
    resetInput();
  }
}

async function handleDrop(event) {
  const file = event.dataTransfer.files[0];
  if (file && file.type === "application/pdf") {
    if (!nameFileValidate(file.name)) {
      alert(
        "⚠️ El nombre del archivo no es válido, debe seguir el formato: ddmmyyyy (ejemplo: 22042025)"
      );
      return;
    }
    loading.value = true;
    progress.value = 0;
    try {
      const res = await upload_pdf_with_progress(file, (percent) => {
        progress.value = percent;
      });
      fileName.value = res.filename;
      fileSelected.value = true;
    } catch (err) {
      console.error("Error al subir el archivo PDF:", err);
    } finally {
      loading.value = false;
    }
  } else {
    alert("Solo se permiten archivos PDF.");
  }
}

function nameFileValidate(name) {
  return /^\d{8}\.pdf$/i.test(name);
}

function selectFile() {
  if (fileInput.value) {
    fileInput.value.click();
  }
}

function resetProcess() {
  fileName.value = "";
  fileSelected.value = false;
  processed.value = false;
  estado.value = "";
  progress.value = 0;
  processing.value = false;
}
</script>

<style scoped>
/* Para evitar seleccionar texto accidentalmente en la zona de drop */
.container-process .border-dashed {
  user-select: none;
}
</style>
