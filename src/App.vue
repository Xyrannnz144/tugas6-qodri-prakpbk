<template>
  <div class="app-container">
    <h1>Manajemen Pegawai Xyrannnz Store</h1>
    <form @submit.prevent="save" class="form-container">
      <input type="text" v-model="form.nip" placeholder="NIP" class="form-input" required />
      <input type="text" v-model="form.name" placeholder="Nama Pegawai" class="form-input" required />
      <input type="number" v-model="form.gaji" placeholder="Gaji" class="form-input" required />
      <input type="number" v-model="form.umur" placeholder="Umur" class="form-input" required />
      <select v-model="form.divisi" class="form-select" required>
        <option disabled value="">Pilih Divisi</option>
        <option value="Dokumentasi">Dokumentasi</option>
        <option value="Developer">Developer</option>
        <option value="Editor">Editor</option>
      </select>
      <select v-model="form.posisi" class="form-select" required>
        <option disabled value="">Pilih Posisi</option>
        <option value="Internship">Internship</option>
        <option value="Manager">Manager</option>
        <option value="Staff">Staff</option>
      </select>
      <button type="submit" class="btn-save">{{ form.id? 'Ubah Data' : 'Simpan' }}</button>
      <button @click="resetForm" type="button" class="btn-cancel" v-if="form.id">Batal</button>
    </form>
    <ul class="employee-list">
      <li v-for="employee in paginatedEmployees" :key="employee.id" class="employee-item">
        <div class="card">
          <h2>{{ employee.name }}</h2>
          <p>NIP: {{ employee.nip }}</p>
          <p>Gaji: {{ employee.gaji }}</p>
          <p>Umur: {{ employee.umur }}</p>
          <p>Divisi: {{ employee.divisi }}</p>
          <p>Posisi: {{ employee.posisi }}</p>
          <button @click="edit(employee)" class="btn-edit">Ubah Data</button>
          <button @click="deleteEmployee(employee.id)" class="btn-delete">Hapus</button>
        </div>
      </li>
    </ul>
    <div class="pagination">
      <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
      <span>{{ currentPage }} of {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
    <button @click="load" class="btn-load">Tampilkan Semua Data</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive, computed } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      nip: '',
      name: '',
      gaji: '',
      umur: '',
      divisi: '',
      posisi: '',
    });

    const employees = ref([]);
    const currentId = ref(0);
    const currentPage = ref(1);
    const itemsPerPage = ref(4);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/employees');
        employees.value = response.data;

        if (employees.value.length > 0) {
          currentId.value = Math.max(...employees.value.map(employee => parseInt(employee.id, 10))) || 0;
        }
      } catch (error) {
        console.error('Error loading employees:', error);
      }
    }

    async function save() {
      if (!validateNIP(form.nip)) {
        alert("NIP harus berupa maksimal 10 digit angka saja, tanpa simbol dan huruf.");
        return;
      }

      if (!validateName(form.name)) {
        alert("Nama Pegawai hanya berupa huruf, tanpa simbol dan angka, kecuali koma dan titik.");
        return;
      }

      try {
        const url = form.id? `http://localhost:3000/employees/${form.id}` : 'http://localhost:3000/employees';
        const method = form.id? 'put' : 'post';

        if (!form.id) {
          currentId.value += 1;
          form.id = currentId.value.toString();  // Simpan ID sebagai string
        }

        const response = await axios[method](url, form);

        if (form.id && method === 'put') {
          employees.value = employees.value.map((employee) =>
            employee.id === form.id? response.data : employee
          );
        } else {
          employees.value.push(response.data);
        }

        resetForm();
      } catch (error) {
        console.error('Error saving employee:', error);
      }
    }

    function edit(employee) {
      form.id = employee.id;
      form.nip = employee.nip;
      form.name = employee.name;
      form.gaji = employee.gaji;
      form.umur = employee.umur;
      form.divisi = employee.divisi;
      form.posisi = employee.posisi;
    }

    function deleteEmployee(id) {
      if (confirm("Are you sure you want to delete this employee?")) {
        try {
          axios.delete(`http://localhost:3000/employees/${id}`);
          employees.value = employees.value.filter(employee => employee.id !== id);
          // Update the component's state and re-render the component
          this.$forceUpdate();
        } catch (error) {
          console.error('Error deleting employee:', error);
        }
      }
    }

    function resetForm() {
      form.id = null;
      form.nip = '';
      form.name = '';
      form.gaji = '';
      form.umur = '';
      form.divisi = '';
      form.posisi = '';
    }

    function validateNIP(nip) {
      const nipRegex = /^\d{1,10}$/;
      return nipRegex.test(nip);
    }

    function validateName(name) {
      const nameRegex = /^[a-zA-Z,\. ]+$/;
      return nameRegex.test(name);
    }

    const paginatedEmployees = computed(() => {
      const startIndex = (currentPage.value - 1) * itemsPerPage.value;
      return employees.value.slice(startIndex, startIndex + itemsPerPage.value);
    });

    const totalPages = computed(() => {
      return Math.ceil(employees.value.length / itemsPerPage.value);
    });

    function prevPage() {
      if (currentPage.value > 1) {
        currentPage.value -= 1;
      }
    }

    function nextPage() {
      if (currentPage.value < totalPages.value) {
        currentPage.value += 1;
      }
    }

    onMounted(load);

    return {
      form,
      employees,
      load,
      save,
      edit,
      deleteEmployee,
      resetForm,
      validateNIP,
      validateName,
      paginatedEmployees,
      currentPage,
      totalPages,
      prevPage,
      nextPage,
    };
  },
};
</script>

<style>
body {
  background: url('./assets/Gambar.jpg') no-repeat center center;
  background-size: cover;
}

.app-container {
  max-width: 800px;
  margin: 40px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.form-input {
  width: 100%;
  padding: 10px;
  margin: 10px;
  border: 1px solid #ccc;
}

.form-select {
  width: 100%;
  padding: 10px;
  margin: 10px;
  border: 1px solid #ccc;
}

.btn-save {
  background-color: #4CAF50;
  color: #fff;
  padding: 10px 20px;border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 40px;
}

.btn-cancel {
  background-color: #ccc;
  color: #333;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.employee-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.employee-item {
  margin-bottom: 20px;
}

.card {
  background-color: #fff;
  padding: 20px;
  border: 1px solid #ddd;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.btn-edit {
  background-color: #337ab7;
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-delete {
  background-color: #d9534f;
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-load {
  background-color: #337ab7;
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.pagination {
  margin-top: 20px;
  text-align: center;
}

.pagination button {
  margin: 0 10px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.pagination button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>