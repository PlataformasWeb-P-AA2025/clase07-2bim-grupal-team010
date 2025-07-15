<template>
  <div class="estudiantes-list-container">
    <h2>Listado de Estudiantes</h2>
    <p v-if="loading">Cargando estudiantes...</p>
    <p v-if="error" class="error-message">{{ error }}</p>

    <ul v-else-if="estudiantes.length">
      <li
        v-for="(estudiante, index) in estudiantes"
        :key="estudiante.url"
        class="estudiante-item"
      >
        <div v-if="!editandoEstudiantes[index]">
          <router-link
            :to="{
              name: 'EstudianteDetail',
              params: { estudianteUrl: estudiante.url },
            }"
          >
            {{ estudiante.nombre }} {{ estudiante.apellido }} (Cédula:
            {{ estudiante.cedula }})
          </router-link>
          <div class="actions">
            <button @click="habilitarEdicion(index)">Editar</button>
            <button @click="eliminarEstudiante(estudiante.url, index)">
              Eliminar
            </button>
          </div>
        </div>

        <div v-else>
          <input v-model="estudiante.nombre" placeholder="Nombre" />
          <input v-model="estudiante.apellido" placeholder="Apellido" />
          <input v-model="estudiante.cedula" placeholder="Cédula" />
          <input v-model="estudiante.correo" placeholder="Correo" />
          <button @click="guardarCambios(estudiante, index)">Guardar</button>
          <button @click="cancelarEdicion(index)">Cancelar</button>
        </div>
      </li>
    </ul>

    <p v-else>No hay estudiantes registrados.</p>

    <router-link to="/estudiantes/nuevo" class="add-button"
      >Agregar Nuevo Estudiante</router-link
    >
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudiantesList",
  data() {
    return {
      estudiantes: [],
      editandoEstudiantes: [],
      loading: true,
      error: null,
    };
  },
  async created() {
    await this.fetchEstudiantes();
  },
  methods: {
    async fetchEstudiantes() {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get("estudiantes/");
        this.estudiantes = response.data.results || response.data;
        this.editandoEstudiantes = this.estudiantes.map(() => false);
      } catch (err) {
        console.error("Error al cargar estudiantes:", err.response || err);
        this.error =
          "No se pudieron cargar los estudiantes. Asegúrate de estar logueado.";
      } finally {
        this.loading = false;
      }
    },
    habilitarEdicion(index) {
      this.editandoEstudiantes[index] = true;
    },
    cancelarEdicion(index) {
      this.editandoEstudiantes[index] = false;
      this.fetchEstudiantes(); // recarga para descartar cambios locales
    },
    async guardarCambios(estudiante, index) {
      try {
        await api.put(estudiante.url, {
          nombre: estudiante.nombre,
          apellido: estudiante.apellido,
          cedula: estudiante.cedula,
          correo: estudiante.correo,
        });
        alert("Estudiante actualizado correctamente.");
        this.editandoEstudiantes[index] = false;
      } catch (err) {
        console.error("Error al guardar cambios:", err.response || err);
        alert("No se pudo actualizar el estudiante.");
      }
    },
    async eliminarEstudiante(url, index) {
      if (!confirm("¿Seguro que deseas eliminar este estudiante?")) return;

      try {
        await api.delete(url);
        this.estudiantes.splice(index, 1);
        this.editandoEstudiantes.splice(index, 1);
        alert("Estudiante eliminado.");
      } catch (err) {
        console.error("Error al eliminar estudiante:", err.response || err);
        alert("No se pudo eliminar el estudiante.");
      }
    },
  },
};
</script>

<style scoped>
.estudiantes-list-container {
  max-width: 800px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

.estudiante-item {
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.estudiante-item:last-child {
  border-bottom: none;
}

.estudiante-item a {
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
}

.estudiante-item a:hover {
  text-decoration: underline;
}

.actions {
  margin-top: 5px;
  display: flex;
  gap: 10px;
}

.actions button {
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: white;
}

.actions button:first-child {
  background-color: #ffc107;
}

.actions button:last-child {
  background-color: #dc3545;
}

input {
  display: block;
  margin: 5px 0;
  padding: 6px;
  width: 100%;
  max-width: 400px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.add-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
  transition: background-color 0.3s ease;
}

.add-button:hover {
  background-color: #218838;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
