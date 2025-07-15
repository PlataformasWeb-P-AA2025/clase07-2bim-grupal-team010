<template>
  <div class="estudiante-detail-container">
    <h2>Detalle del Estudiante</h2>
    <p v-if="loading">Cargando detalles...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <div v-else-if="estudiante">
      <h3>{{ estudiante.nombre }} {{ estudiante.apellido }}</h3>
      <p><strong>Cédula:</strong> {{ estudiante.cedula }}</p>
      <p><strong>Correo:</strong> {{ estudiante.correo }}</p>

      <h4>Números Telefónicos:</h4>
      <ul v-if="numerosTelefonicos.length">
        <li v-for="(numero, index) in numerosTelefonicos" :key="numero.url">
          <div v-if="!editandoNumeros[index]">
            {{ numero.telefono }} ({{ numero.tipo }})
            <button @click="habilitarEdicion(index)">Editar</button>
            <button @click="eliminarNumero(numero, index)">Eliminar</button>
          </div>
          <div v-else>
            <input v-model="numero.telefono" placeholder="Número" type="text" />
            <input v-model="numero.tipo" placeholder="Tipo" type="text" />
            <button @click="guardarNumero(numero, index)">Guardar</button>
            <button @click="cancelarEdicion(index)">Cancelar</button>
            <button @click="eliminarNumero(numero, index)">Eliminar</button>
          </div>
        </li>
      </ul>
      <p v-else>No tiene números telefónicos registrados.</p>

      <router-link :to="{ name: 'EstudiantesList' }" class="back-button"
        >Volver al Listado</router-link
      >
    </div>
    <p v-else>Estudiante no encontrado.</p>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudianteDetail",
  props: ["estudianteUrl"], // <-- Ahora recibe 'estudianteUrl' como prop
  data() {
    return {
      estudiante: null,
      numerosTelefonicos: [],
      editandoNumeros: [],
      loading: true,
      error: null,
    };
  },
  async created() {
    // La URL viene codificada por el router, necesitamos decodificarla
    const decodedUrl = decodeURIComponent(this.estudianteUrl);
    await this.fetchEstudianteDetail(decodedUrl);
    await this.fetchNumerosTelefonicos(decodedUrl); // También pasamos la URL para filtrar
  },
  methods: {
    async fetchEstudianteDetail(url) {
      try {
        this.loading = true;
        this.error = null;
        // Usamos la URL completa directamente para la petición
        const response = await api.get(url);
        this.estudiante = response.data;
      } catch (err) {
        console.error(
          "Error al cargar detalle del estudiante:",
          err.response || err
        );
        this.error = "No se pudo cargar el detalle del estudiante.";
      } finally {
        this.loading = false;
      }
    },
    async fetchNumerosTelefonicos(estudianteApiUrl) {
      console.log(estudianteApiUrl);
      try {
        const response = await api.get("numerosts/");
        // Filtramos los números telefónicos que pertenecen a este estudiante usando la URL completa
        this.numerosTelefonicos = response.data.results.filter(
          (numero) => numero.estudiante === estudianteApiUrl
        );
      } catch (err) {
        console.error(
          "Error al cargar números telefónicos:",
          err.response || err
        );
      }
      this.editandoNumeros = this.numerosTelefonicos.map(() => false);
    },
    async guardarNumero(numero, index) {
      try {
        await api.put(numero.url, {
          telefono: numero.telefono,
          tipo: numero.tipo,
          estudiante: numero.estudiante,
        });
        alert("Número actualizado correctamente.");
        this.editandoNumeros[index] = false;
      } catch (error) {
        console.error("Error al actualizar número:", error.response || error);
        alert("Ocurrió un error al actualizar el número.");
      }
    },
    habilitarEdicion(index) {
      this.editandoNumeros[index] = true;
    },

    async cancelarEdicion(index) {
      this.editandoNumeros[index] = false;
      await this.fetchNumerosTelefonicos(
        decodeURIComponent(this.estudianteUrl)
      );
    },
    async eliminarNumero(numero, index) {
      if (!confirm("¿Estás seguro de que quieres eliminar este número?")) {
        return;
      }

      try {
        await api.delete(numero.url);
        alert("Número eliminado correctamente.");

        // Quitar del array local
        this.numerosTelefonicos.splice(index, 1);
        this.editandoNumeros.splice(index, 1);
      } catch (error) {
        console.error("Error al eliminar número:", error.response || error);
        alert("Ocurrió un error al eliminar el número.");
      }
    },
  },
};
</script>

<style scoped>
.estudiante-detail-container {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  text-align: left;
}

h2,
h3,
h4 {
  text-align: center;
  color: #333;
  margin-bottom: 15px;
}

p {
  margin-bottom: 10px;
}

ul {
  list-style: disc;
  padding-left: 20px;
  margin-bottom: 20px;
}

li {
  margin-bottom: 5px;
}

.back-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
  transition: background-color 0.3s ease;
}

.back-button:hover {
  background-color: #0056b3;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
