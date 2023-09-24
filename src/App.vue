<template>
  <div>
    <h1 class="titulo">Hola Vite</h1>

    <!-- Subida y envío de archivo -->
    <label for="file_input" class="archivo">Seleccione un archivo en formato CSV </label>
    <input id="file_input" type="file" ref="fileInput">
    <div>
      <button @click="handleClick">Enviar</button>
    </div>

    <!-- Tabla -->
    <div>
      <h2 class="tituloTabla">Tabla de Contactos</h2>
      <table class="tabla">
        <thead>
          <tr>
            <th>Nombre</th>
            <th>Email</th>
            <th>Teléfono</th>
          </tr>
        </thead>
        <tbody>

          <!-- Agregar array a la tabla -->
          <tr v-for="contact in contacts" :key="contact.name">
            <td>{{ contact.name }}</td>
            <td>{{ contact.email }}</td>
            <td>{{ contact.phone }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

// Definir componente para que TypeScript entienda
interface MyComponent {
  jsonData: string;
  //contactData: [];
  contacts: any[];
  handleClick(): Promise<void>;
  convertFileToJson(file: File): Promise<any[]>;
  uploadData(jsonData: any[]): Promise<void>;
}

export default defineComponent({
  data() {
    return {
      jsonData: " ", // Aquí se guarda mi CSV convertido a JSON
      contacts: [] as any[],
    } as MyComponent; // Asignar tipo al componente
  },

  // Métodos
  methods: {
    // Método de evento para enviar el archivo a su conversión
    async handleClick() {
      const fileInput = this.$refs.fileInput as HTMLInputElement;
      const file = fileInput.files?.[0];

      //this.contactData = await getData();
      //this.contactData = hola.items;
      //this.contacts = await hola.json();
          //this.contactData = await hola.json(); 
         //let perro = this.contactData.items;
          //console.log("hola:" + hola.items);
          //console.log("contacData:" + this.contactData);

      if (file) {
        this.convertFileToJson(file).then((jsonData) => {
          this.jsonData = JSON.stringify(jsonData, null, 2);
          this.contacts = jsonData;
          this.uploadData(jsonData); // Este es para el fetch
        });
      } else {
        alert('Por favor, seleccione un archivo válido.');
      }
    },

    // Conversión de CSV a JSON
    async convertFileToJson(file: File): Promise<any[]> {
      return new Promise((resolve, reject) => {
        const jsonArray: any[] = [];
        const reader = new FileReader();

        reader.onload = (event) => {
          const result = event.target?.result as string;

          const lines = result.split('\n');
          lines.forEach((line) => {
            const [name, phone, email] = line.split(',');
            if (name && phone && email) {
              let formattedPhone = phone;

              const zerosToAdd = 10 - formattedPhone.length;

              for (let i = 0; i < zerosToAdd; i++) {
                formattedPhone = `0${formattedPhone}`;
              }

              formattedPhone = formattedPhone.replace(/(\d{3})(\d{3})(\d{4})/, '$1-$2-$3');

              jsonArray.push({ name, email, phone: formattedPhone });
            }
          });

          resolve(jsonArray);
        };

        reader.onerror = (event) => {
          reject(event.target?.error);
        };

        reader.readAsText(file);
      });
    },

    // Aquí me aseguro que el POST funcione
    async uploadData(jsonData: any) {
      for (const item of jsonData) {
        try {
          const response = await postData(item);
          console.log(`Contacto ${item.name} subido con éxito. Respuesta del servidor:`, response);
        } catch (error) {
          console.error(`Error al subir el contacto ${item.name}:`, error);
        }
      }
      alert('Contactos cargados correctamente');

      //let hola = await fetch(endpoint);
      //this.contacts = await hola.json();
    },
  },
});

const endpoint = "https://8j5baasof2.execute-api.us-west-2.amazonaws.com/production/tests/trucode/items";

async function postData(data = {}) {
  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  };

  // Realizar la solicitud POST
  const response = await fetch(endpoint, options);
  return response.json();
}

  // Aquí se hace el GET
  fetch(endpoint)
  .then(response => {
    if (!response.ok) {
      throw new Error(`La solicitud falló con estado ${response.status}`);
    }
    return response.json();
  })
  .then(data => {
    // Mensajito en consola del GET, no necesario
    console.log("Datos obtenidos:", data);
  })
  .catch(error => {
    console.error("Error al obtener datos:", error);
  })

/*async function getData() {
  try {
    const response = await fetch(endpoint);

    if (!response.ok) {
      throw new Error(`La solicitud falló con estado ${response.status}`);
    }

    const data = await response.json();

    // Hacer algo con los datos obtenidos
    console.log("Datos obtenidos:", data);

    return data;
  } catch (error) {
    console.error("Error al obtener datos:", error);
    throw error; // Puedes re-lanzar el error para que el código que llame a esta función pueda manejarlo
  }
}*/
</script>



<style>
.titulo { 
  color: #b0e57c;
}

.archivo {
  color: #f5f5dc;
  display: block;
  margin-bottom: 10px;
  font-size: 18px;
}

#file_input {
  color: #f5f5dc;
}

button {
  padding: 10px 20px;
  margin-top: 20px;
  color: white;
  border: none;
}

.tituloTabla {
  color: #d1c0ff;  
}

.tabla {
  width: 100%;
  margin: 0 auto;
  color: #f5f5dc;
  text-align: center;
}

.tabla th{
  border: 1px solid #a6d9fc;
  padding: 8px;
  color: #ffd3e6;
}

.tabla td {
  border: 1px solid #a6d9fc;
  padding: 8px;
}
</style>