<script setup>
import { ref, computed, onMounted } from "vue";
import { useQuasar } from "quasar";
import { aprendicesService } from "../services/aprendices.service";
import { cursosService } from "../services/cursos.service";
import { mensajeError } from "../services/api";

const $q = useQuasar();
const aprendices = ref([]);
const cursos = ref([]);
const filtroCurso = ref(null);
const cargando = ref(false);
const dialogo = ref(false);
const editandoId = ref(null);
const form = ref({ documento: "", nombre: "", email: "", curso: null });

const opcionesCurso = computed(() =>
  cursos.value.filter((c) => c.status === 0).map((c) => ({ label: `${c.codigo} - ${c.nombre}`, value: c._id }))
);

const columnas = [
  { name: "documento", label: "Documento", field: "documento", align: "left", sortable: true },
  { name: "nombre", label: "Nombre", field: "nombre", align: "left", sortable: true },
  { name: "email", label: "Correo", field: "email", align: "left" },
  { name: "curso", label: "Curso", field: (r) => r.curso?.nombre || "Sin curso", align: "left", sortable: true },
  { name: "status", label: "Estado", field: "status", align: "center" },
  { name: "acciones", label: "", align: "right" },
];

const cargar = async () => {
  cargando.value = true;
  try {
    const { data } = filtroCurso.value
      ? await aprendicesService.porCurso(filtroCurso.value)
      : await aprendicesService.listar();
    aprendices.value = data;
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  } finally {
    cargando.value = false;
  }
};

const abrir = (a) => {
  editandoId.value = a?._id || null;
  form.value = a
    ? { documento: a.documento, nombre: a.nombre, email: a.email, curso: a.curso?._id || null }
    : { documento: "", nombre: "", email: "", curso: null };
  dialogo.value = true;
};

const guardar = async () => {
  try {
    const { data } = editandoId.value
      ? await aprendicesService.actualizar(editandoId.value, form.value)
      : await aprendicesService.crear(form.value);
    $q.notify({ type: "positive", message: data.msg });
    dialogo.value = false;
    cargar();
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  }
};

const cambiarEstado = async (a) => {
  try {
    const { data } = a.status === 0 ? await aprendicesService.desactivar(a._id) : await aprendicesService.activar(a._id);
    $q.notify({ type: "positive", message: data.msg });
    cargar();
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  }
};

onMounted(async () => {
  try {
    cursos.value = (await cursosService.listar()).data;
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  }
  cargar();
});
</script>

<template>
  <q-table title="Aprendices" :rows="aprendices" :columns="columnas" row-key="_id" :loading="cargando"
    no-data-label="No hay aprendices para mostrar." rows-per-page-label="Filas por página">
    <template #top-right>
      <q-select v-model="filtroCurso" :options="opcionesCurso" emit-value map-options clearable dense outlined
        label="Filtrar por curso" style="min-width: 240px" class="q-mr-md" @update:model-value="cargar" />
      <q-btn color="primary" icon="add" label="Nuevo aprendiz" @click="abrir()" />
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
      <q-card-section class="text-h6">{{ editandoId ? "Editar aprendiz" : "Nuevo aprendiz" }}</q-card-section>
      <q-form @submit.prevent="guardar">
        <q-card-section class="q-gutter-md">
          <q-input v-model="form.documento" label="Documento" outlined :rules="[(v) => !!v || 'El documento es obligatorio']" />
          <q-input v-model="form.nombre" label="Nombre" outlined :rules="[(v) => !!v || 'El nombre es obligatorio']" />
          <q-input v-model="form.email" type="email" label="Correo" outlined :rules="[(v) => !!v || 'El correo es obligatorio']" />
          <q-select v-model="form.curso" :options="opcionesCurso" emit-value map-options label="Curso" outlined
            :rules="[(v) => !!v || 'Selecciona un curso']" />
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" v-close-popup />
          <q-btn type="submit" color="primary" label="Guardar aprendiz" />
        </q-card-actions>
      </q-form>
    </q-card>
  </q-dialog>
</template>
