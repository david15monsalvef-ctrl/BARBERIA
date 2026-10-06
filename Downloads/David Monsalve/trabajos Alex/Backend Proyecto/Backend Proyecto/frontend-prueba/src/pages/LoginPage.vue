<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useQuasar } from "quasar";
import { useAuthStore } from "../stores/auth";
import { mensajeError } from "../services/api";

const $q = useQuasar();
const router = useRouter();
const auth = useAuthStore();

const form = ref({ email: "", password: "" });
const verClave = ref(false);
const cargando = ref(false);

const entrar = async () => {
  cargando.value = true;
  try {
    await auth.login(form.value);
    router.push("/cursos");
  } catch (e) {
    $q.notify({ type: "negative", message: mensajeError(e) });
  } finally {
    cargando.value = false;
  }
};
</script>

<template>
  <div class="flex flex-center" style="min-height: 100vh; background-color: #0f172a;">
    <q-card class="q-pa-sm" style="width: 400px; max-width: 92vw; background-color: #1e293b; color: #f8fafc; border-radius: 16px; border: 1px solid rgba(255, 255, 255, 0.08); box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5), 0 10px 10px -5px rgba(0, 0, 0, 0.4);">
      
      <!-- Encabezado con ícono -->
      <q-card-section class="text-center q-pt-lg q-pb-md">
        <div class="q-pa-md inline-block q-mb-md" style="background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%); border-radius: 50%; width: 70px; height: 70px; display: flex; align-items: center; justify-content: center; margin: 0 auto; box-shadow: 0 10px 15px -3px rgba(59, 130, 246, 0.4);">
          <q-icon name="lock" size="32px" color="white" />
        </div>
        <div class="text-h5 text-weight-bold" style="color: #f8fafc;">Bienvenido</div>
        <div class="text-subtitle2" style="color: #94a3b8;">Inicia sesión para gestionar cursos y aprendices</div>
      </q-card-section>

      <q-form @submit.prevent="entrar">
        <q-card-section class="q-gutter-md q-px-lg">
          <q-input 
            v-model="form.email" 
            type="email" 
            label="Correo electrónico" 
            outlined 
            dense
            autofocus
            dark
            color="blue-5"
            label-color="grey-4"
            bg-color="#0f172a"
            style="border-radius: 8px;"
            :rules="[(v) => !!v || 'Escribe tu correo']"
          >
            <template #prepend>
              <q-icon name="email" color="blue-4" />
            </template>
          </q-input>

          <q-input 
            v-model="form.password" 
            :type="verClave ? 'text' : 'password'" 
            label="Contraseña" 
            outlined 
            dense
            dark
            color="blue-5"
            label-color="grey-4"
            bg-color="#0f172a"
            :rules="[(v) => !!v || 'Escribe tu contraseña']"
          >
            <template #prepend>
              <q-icon name="vpn_key" color="blue-4" />
            </template>
            <template #append>
              <q-icon 
                :name="verClave ? 'visibility_off' : 'visibility'" 
                class="cursor-pointer" 
                style="color: #94a3b8;" 
                @click="verClave = !verClave" 
              />
            </template>
          </q-input>
        </q-card-section>

        <q-card-actions class="q-px-lg q-pb-lg q-pt-sm">
          <q-btn 
            type="submit" 
            label="Entrar a la plataforma" 
            color="primary" 
            class="full-width q-py-sm text-weight-bold" 
            style="background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%); border-radius: 8px; text-transform: none; font-size: 16px;"
            :loading="cargando" 
            unelevated
          />
        </q-card-actions>
      </q-form>

      <!-- Pie de tarjeta -->
      <q-card-section class="text-center q-pt-none q-pb-md" style="font-size: 12px; color: #64748b;">
        Sistema de Gestión Académica
      </q-card-section>

    </q-card>
  </div>
</template>