<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Persistencia de los datos
const servicios = useLocalStorage('barberia_servicios_v1', [
  {
    id: 1,
    cliente: 'Carlos Gómez',
    servicio: 'Corte + Barba',
    barbero: 'Don Ramiro',
    fecha: '2026-08-28T10:30',
    precio: 35000,
    metodoPago: 'efectivo',
    estadoPago: 'pagado',
    calificacion: 5,
    observaciones: 'Cliente habitual, degradado bajo.'
  },
  {
    id: 2,
    cliente: 'Andrés Morales',
    servicio: 'Corte moderno',
    barbero: 'Mateo',
    fecha: '2026-08-28T11:15',
    precio: 25000,
    metodoPago: 'transferencia',
    estadoPago: 'fiado',
    calificacion: 4,
    observaciones: 'Pendiente comprobante Nequi.'
  }
])

// Estados para modales y control
const mostrarModal = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)
const servicioAEliminar = ref(null)

// Formulario reactivo
const formulario = ref({
  cliente: '',
  servicio: 'Corte clásico',
  barbero: 'Don Ramiro',
  fecha: '',
  precio: 20000,
  metodoPago: 'efectivo',
  estadoPago: 'pagado',
  calificacion: 5,
  observaciones: ''
})

function abrirModalCrear() {
  modoEdicion.value = false
  idEdicion.value = null
  const ahora = new Date()
  ahora.setMinutes(ahora.getMinutes() - ahora.getTimezoneOffset())
  
  formulario.value = {
    cliente: '',
    servicio: 'Corte clásico',
    barbero: 'Don Ramiro',
    fecha: ahora.toISOString().slice(0, 16),
    precio: 20000,
    metodoPago: 'efectivo',
    estadoPago: 'pagado',
    calificacion: 5,
    observaciones: ''
  }
  mostrarModal.value = true
}

function abrirModalEditar(item) {
  modoEdicion.value = true
  idEdicion.value = item.id
  formulario.value = { ...item }
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
}

function guardarServicio() {
  if (!formulario.value.cliente.trim() || formulario.value.precio <= 0) {
    alert('Por favor ingrese el nombre del cliente y un precio válido.')
    return
  }

  if (modoEdicion.value) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === idEdicion.value) {
        servicios.value[i] = { ...formulario.value, id: idEdicion.value }
        break
      }
    }
  } else {
    servicios.value.unshift({
      ...formulario.value,
      id: Date.now()
    })
  }
  cerrarModal()
}

function pedirConfirmacionEliminar(servicio) {
  servicioAEliminar.value = servicio
}

function borrarServicio() {
  if (servicioAEliminar.value) {
    servicios.value = servicios.value.filter(s => s.id !== servicioAEliminar.value.id)
    servicioAEliminar.value = null
  }
}
</script>

<template>
  <div class="contenedor">
    <header class="header">
      <div>
        <h1>💈 Barbería Don Ramiro</h1>
        <p>Control diario de cortes y cobros</p>
      </div>
      <button class="btn btn-primario" @click="abrirModalCrear">+ Registrar Servicio</button>
    </header>

    <div v-if="servicios.length === 0" class="vacio">
      <p>No hay servicios registrados hoy.</p>
    </div>

    <main v-else class="grid">
      <div 
        v-for="s in servicios" 
        :key="s.id" 
        class="card"
        :class="{ 'card-fiado': s.estadoPago === 'fiado' }"
      >
        <div class="card-head">
          <h3>{{ s.cliente }}</h3>
          <span class="badge" :class="s.estadoPago">
            <span v-if="s.estadoPago === 'pagado'">✅ Pagado</span>
            <span v-else-if="s.estadoPago === 'pendiente'">⏳ Pendiente</span>
            <span v-else>⚠️ Fiado</span>
          </span>
        </div>

        <div class="card-body">
          <p><strong>Servicio:</strong> {{ s.servicio }}</p>
          <p><strong>Barbero:</strong> {{ s.barbero }}</p>
          <p><strong>Fecha:</strong> {{ new Date(s.fecha).toLocaleString() }}</p>
          <p><strong>Precio:</strong> ${{ s.precio.toLocaleString() }}</p>
          <p>
            <strong>Pago:</strong> 
            <span v-if="s.metodoPago === 'efectivo'">💵 Efectivo</span>
            <span v-else-if="s.metodoPago === 'transferencia'">📱 Transferencia</span>
            <span v-else>💳 Tarjeta</span>
          </p>
          <p>
            <strong>Calificación:</strong> 
            <span :class="{ 'mala-nota': s.calificacion <= 2 }">
              {{ '⭐'.repeat(s.calificacion) }} ({{ s.calificacion }}/5)
            </span>
          </p>
          <p v-if="s.observaciones"><strong>Notas:</strong> {{ s.observaciones }}</p>
        </div>

        <div class="card-acciones">
          <button class="btn btn-secundario" @click="abrirModalEditar(s)">✏️ Editar</button>
          <button class="btn btn-peligro" @click="pedirConfirmacionEliminar(s)">🗑️ Eliminar</button>
        </div>
      </div>
    </main>

    <!-- Modal Formulario -->
    <div v-if="mostrarModal" class="modal-bg" @click.self="cerrarModal">
      <div class="modal-body">
        <h2>{{ modoEdicion ? 'Editar Servicio' : 'Nuevo Servicio' }}</h2>
        <form @submit.prevent="guardarServicio">
          <label>Cliente:
            <input type="text" v-model="formulario.cliente" required />
          </label>
          <label>Servicio:
            <select v-model="formulario.servicio">
              <option value="Corte clásico">Corte clásico</option>
              <option value="Corte moderno">Corte moderno</option>
              <option value="Barba">Barba</option>
              <option value="Corte + Barba">Corte + Barba</option>
              <option value="Cejas">Cejas</option>
              <option value="Tinte">Tinte</option>
            </select>
          </label>
          <label>Barbero:
            <select v-model="formulario.barbero">
              <option value="Don Ramiro">Don Ramiro</option>
              <option value="Mateo">Mateo</option>
              <option value="Camilo">Camilo</option>
            </select>
          </label>
          <label>Fecha y Hora:
            <input type="datetime-local" v-model="formulario.fecha" required />
          </label>
          <label>Precio:
            <input type="number" v-model.number="formulario.precio" min="0" required />
          </label>
          <label>Método de Pago:
            <select v-model="formulario.metodoPago">
              <option value="efectivo">Efectivo</option>
              <option value="transferencia">Transferencia</option>
              <option value="tarjeta">Tarjeta</option>
            </select>
          </label>
          <label>Estado del Pago:
            <select v-model="formulario.estadoPago">
              <option value="pagado">Pagado</option>
              <option value="pendiente">Pendiente</option>
              <option value="fiado">Fiado</option>
            </select>
          </label>
          <label>Calificación (1 a 5):
            <input type="number" v-model.number="formulario.calificacion" min="1" max="5" />
          </label>
          <label>Observaciones:
            <textarea v-model="formulario.observaciones"></textarea>
          </label>
          <div class="modal-btns">
            <button type="button" class="btn btn-secundario" @click="cerrarModal">Cancelar</button>
            <button type="submit" class="btn btn-primario">Guardar</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal Eliminar -->
    <div v-if="servicioAEliminar" class="modal-bg" @click.self="servicioAEliminar = null">
      <div class="modal-body">
        <h3>¿Eliminar servicio?</h3>
        <p>Confirma eliminar el registro de <strong>{{ servicioAEliminar.cliente }}</strong>.</p>
        <div class="modal-btns">
          <button class="btn btn-secundario" @click="servicioAEliminar = null">Cancelar</button>
          <button class="btn btn-peligro" @click="borrarServicio">Eliminar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.contenedor { max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif; }
.header { display: flex; justify-content: space-between; align-items: center; background: #2c3e50; color: white; padding: 15px 20px; border-radius: 8px; margin-bottom: 20px; }
.header h1 { margin: 0; font-size: 1.5rem; }
.header p { margin: 5px 0 0 0; color: #bdc3c7; }
.vacio { text-align: center; padding: 40px; background: #f8f9fa; border-radius: 8px; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); gap: 15px; }
.card { border: 1px solid #ddd; border-radius: 8px; padding: 15px; background: white; }
.card-fiado { border-left: 5px solid #e74c3c; background: #fdf2f2; }
.card-head { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #eee; padding-bottom: 8px; }
.card-head h3 { margin: 0; }
.badge { padding: 3px 8px; border-radius: 4px; font-size: 0.8rem; font-weight: bold; }
.badge.pagado { background: #d4edda; color: #155724; }
.badge.pendiente { background: #fff3cd; color: #856404; }
.badge.fiado { background: #f8d7da; color: #721c24; }
.card-body p { margin: 6px 0; font-size: 0.9rem; }
.mala-nota { color: #e74c3c; font-weight: bold; }
.card-acciones { display: flex; gap: 8px; margin-top: 12px; }
.btn { padding: 8px 12px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; }
.btn-primario { background: #2980b9; color: white; }
.btn-secundario { background: #bdc3c7; color: #2c3e50; }
.btn-peligro { background: #e74c3c; color: white; }
.modal-bg { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; }
.modal-body { background: white; padding: 20px; border-radius: 8px; width: 90%; max-width: 400px; max-height: 85vh; overflow-y: auto; }
.modal-body form { display: flex; flex-direction: column; gap: 10px; }
.modal-body label { display: flex; flex-direction: column; font-size: 0.85rem; font-weight: bold; gap: 4px; }
.modal-body input, .modal-body select, .modal-body textarea { padding: 8px; border: 1px solid #ccc; border-radius: 4px; }
.modal-btns { display: flex; justify-content: flex-end; gap: 8px; margin-top: 15px; }
</style>