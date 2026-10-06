<script setup>
import { ref } from "vue";
import { useQuasar } from "quasar";
import { usuariosService } from "../services/usuarios.service";
import { mensajeError } from "../services/api";

const $q = useQuasar();
const vacio = () => ({ nombre: "", email: "", password: "" });
const form = ref(vacio());
const cargando = ref(false);

const guardar = async () => {
  cargando.value = true;
  try {
    const { data } = await usuariosService.crear(form.value);
    $q.notify({ type: "positive", message: data.msg });
    form.value = vacio();
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  } finally {
    cargando.value = false;
  }
};
</script>

<template>
  <q-card style="max-width: 460px">
    <q-card-section class="text-h6">Crear usuario</q-card-section>
    <q-form @submit.prevent="guardar">
      <q-card-section class="q-gutter-md">
        <q-input v-model="form.nombre" label="Nombre" outlined :rules="[(v) => !!v || 'El nombre es obligatorio']" />
        <q-input v-model="form.email" type="email" label="Correo" outlined :rules="[(v) => !!v || 'El correo es obligatorio']" />
        <q-input v-model="form.password" type="password" label="Contraseña" outlined
          :rules="[(v) => (v && v.length >= 6) || 'Mínimo 6 caracteres']" />
      </q-card-section>
      <q-card-actions align="right">
        <q-btn type="submit" color="primary" label="Crear usuario" :loading="cargando" />
      </q-card-actions>
    </q-form>
  </q-card>
</template>
