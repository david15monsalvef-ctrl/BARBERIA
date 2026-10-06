<script setup>
import { ref, onMounted } from "vue";
import { useQuasar } from "quasar";
import { cursosService } from "../services/cursos.service";
import { mensajeError } from "../services/api";

const $q = useQuasar();
const cursos = ref([]);
const cargando = ref(false);
const dialogo = ref(false);
const editandoId = ref(null);
const form = ref({ codigo: "", nombre: "", duracion: null });

const columnas = [
  { name: "codigo", label: "Código", field: "codigo", align: "left", sortable: true },
  { name: "nombre", label: "Nombre", field: "nombre", align: "left", sortable: true },
  { name: "duracion", label: "Duración (h)", field: "duracion", align: "right", sortable: true },
  { name: "status", label: "Estado", field: "status", align: "center" },
  { name: "acciones", label: "", align: "right" },
];

const cargar = async () => {
  cargando.value = true;
  try {
    cursos.value = (await cursosService.listar()).data;
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  } finally {
    cargando.value = false;
  }
};

const abrir = (c) => {
  editandoId.value = c?._id || null;
  form.value = c
    ? { codigo: c.codigo, nombre: c.nombre, duracion: c.duracion }
    : { codigo: "", nombre: "", duracion: null };
  dialogo.value = true;
};

const guardar = async () => {
  try {
    const payload = { ...form.value, duracion: Number(form.value.duracion) };
    const { data } = editandoId.value
      ? await cursosService.actualizar(editandoId.value, payload)
      : await cursosService.crear(payload);
    $q.notify({ type: "positive", message: data.msg });
    dialogo.value = false;
    cargar();
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  }
};

const cambiarEstado = async (c) => {
  try {
    const { data } = c.status === 0 ? await cursosService.desactivar(c._id) : await cursosService.activar(c._id);
    $q.notify({ type: "positive", message: data.msg });
    cargar();
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  }
};

onMounted(cargar);
</script>

<template>
  <q-table title="Cursos" :rows="cursos" :columns="columnas" row-key="_id" :loading="cargando"
    no-data-label="Aún no hay cursos. Crea el primero." rows-per-page-label="Filas por página">
    <template #top-right>
      <q-btn color="primary" icon="add" label="Nuevo curso" @click="abrir()" />
    </template>
    <template #body-cell-status="{ row }">
      <q-td class="text-center">
        <q-badge :color="row.status === 0 ? 'positive' : 'grey'" :label="row.status === 0 ? 'Activo' : 'Inactivo'" />
      </q-td>
    </template>
    <template #body-cell-acciones="{ row }">
      <q-td class="text-right">
        <q-btn flat round icon="edit" @click="abrir(row)"><q-tooltip>Editar</q-tooltip></q-btn>
        <q-btn flat round :icon="row.status === 0 ? 'toggle_on' : 'toggle_off'"
          :color="row.status === 0 ? 'positive' : 'grey'" @click="cambiarEstado(row)">
          <q-tooltip>{{ row.status === 0 ? "Desactivar" : "Activar" }}</q-tooltip>
        </q-btn>
      </q-td>
    </template>
  </q-table>

  <q-dialog v-model="dialogo">
    <q-card style="width: 420px; max-width: 92vw">
      <q-card-section class="text-h6">{{ editandoId ? "Editar curso" : "Nuevo curso" }}</q-card-section>
      <q-form @submit.prevent="guardar">
        <q-card-section class="q-gutter-md">
          <q-input v-model="form.codigo" label="Código" outlined :rules="[(v) => !!v || 'El código es obligatorio']" />
          <q-input v-model="form.nombre" label="Nombre" outlined :rules="[(v) => !!v || 'El nombre es obligatorio']" />
          <q-input v-model="form.duracion" type="number" label="Duración (horas)" outlined
            :rules="[(v) => v > 0 || 'La duración debe ser mayor a 0']" />
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" v-close-popup />
          <q-btn type="submit" color="primary" label="Guardar curso" />
        </q-card-actions>
      </q-form>
    </q-card>
  </q-dialog>
</template>
