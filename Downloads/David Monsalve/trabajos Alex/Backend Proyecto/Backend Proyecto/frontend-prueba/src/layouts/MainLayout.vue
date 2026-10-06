<script setup>
import { useRouter } from "vue-router";
import { useAuthStore } from "../stores/auth";

const auth = useAuthStore();
const router = useRouter();

const salir = () => {
  auth.logout();
  router.push("/login");
};
</script>

<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated class="bg-secondary">
      <q-toolbar>
        <q-toolbar-title>Cursos y Aprendices</q-toolbar-title>
        <q-tabs shrink stretch>
          <q-route-tab to="/cursos" label="Cursos" icon="menu_book" />
          <q-route-tab to="/aprendices" label="Aprendices" icon="school" />
          <q-route-tab to="/usuarios" label="Usuarios" icon="manage_accounts" />
        </q-tabs>
        <q-separator vertical inset class="q-mx-md" />
        <span class="gt-xs q-mr-sm">{{ auth.usuario?.nombre }}</span>
        <q-btn flat round icon="logout" @click="salir"><q-tooltip>Cerrar sesión</q-tooltip></q-btn>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <q-page padding><router-view /></q-page>
    </q-page-container>
  </q-layout>
</template>
