<script setup>
import { onMounted, reactive, ref } from 'vue'

const apiUrl = 'http://localhost:3000/api/borrow-records'

const records = ref([])
const editingId = ref(null)
const message = ref('')

const form = reactive({
  equipment_name: '',
  borrower_name: '',
  borrow_date: '',
  status: 'Borrowed'
})

async function loadRecords() {
  const response = await fetch(apiUrl)

  if (!response.ok) {
    throw new Error('Could not load records')
  }

  records.value = await response.json()
}

function resetForm() {
  form.equipment_name = ''
  form.borrower_name = ''
  form.borrow_date = ''
  form.status = 'Borrowed'
  editingId.value = null
  message.value = ''
}

function editRecord(record) {
  editingId.value = record.id
  form.equipment_name = record.equipment_name
  form.borrower_name = record.borrower_name
  form.borrow_date = record.borrow_date
  form.status = record.status
  message.value = ''
  window.scrollTo(0, 0)
}

async function saveRecord() {
  const isEditing = editingId.value !== null

  try {
    const response = await fetch(
      isEditing ? `${apiUrl}/${editingId.value}` : apiUrl,
      {
        method: isEditing ? 'PUT' : 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(form)
      }
    )

    const data = await response.json()

    if (!response.ok) {
      throw new Error(data.error || 'Could not save record')
    }

    resetForm()
    await loadRecords()
    message.value = isEditing ? 'Record updated' : 'Record added'
  } catch (error) {
    message.value = error.message
  }
}

async function deleteRecord(record) {
  if (!window.confirm(`Delete ${record.equipment_name}?`)) return

  try {
    const response = await fetch(`${apiUrl}/${record.id}`, {
      method: 'DELETE'
    })

    const data = await response.json()

    if (!response.ok) {
      throw new Error(data.error || 'Could not delete record')
    }

    if (editingId.value === record.id) resetForm()
    await loadRecords()
    message.value = 'Record deleted'
  } catch (error) {
    message.value = error.message
  }
}

onMounted(async () => {
  try {
    await loadRecords()
  } catch (error) {
    message.value = error.message
  }
})
</script>

<template>
  <main class="page">
    <h1>Equipment Borrowing Tracker</h1>
    <p>Add and manage equipment borrowing records.</p>

    <section class="card">
      <h2>{{ editingId === null ? 'Add record' : 'Edit record' }}</h2>

      <form @submit.prevent="saveRecord">
        <div class="form-grid">
          <label>
            Equipment name
            <input v-model="form.equipment_name" required placeholder="Projector" />
          </label>

          <label>
            Borrower name
            <input v-model="form.borrower_name" required placeholder="Student name" />
          </label>

          <label>
            Borrow date
            <input v-model="form.borrow_date" type="date" required />
          </label>

          <label v-if="editingId !== null">
            Status
            <select v-model="form.status">
              <option value="Borrowed">Borrowed</option>
              <option value="Returned">Returned</option>
            </select>
          </label>
        </div>

        <div class="actions">
          <button type="submit">
            {{ editingId === null ? 'Add record' : 'Save changes' }}
          </button>
          <button v-if="editingId !== null" type="button" class="secondary" @click="resetForm">
            Cancel
          </button>
        </div>
      </form>
    </section>

    <p v-if="message" class="message">{{ message }}</p>

    <section class="card">
      <h2>Borrowing records</h2>
      <p v-if="records.length === 0">No records yet.</p>

      <div v-else class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Equipment</th>
              <th>Borrower</th>
              <th>Date</th>
              <th>Status</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="record in records" :key="record.id">
              <td>{{ record.equipment_name }}</td>
              <td>{{ record.borrower_name }}</td>
              <td>{{ record.borrow_date }}</td>
              <td>{{ record.status }}</td>
              <td>
                <button type="button" @click="editRecord(record)">Edit</button>
                <button type="button" class="danger" @click="deleteRecord(record)">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </main>
</template>

<style scoped>
.page {
  max-width: 900px;
  margin: 40px auto;
  padding: 0 20px;
}

h1 {
  color: #275c3c;
}

.card {
  background: white;
  padding: 24px;
  margin: 20px 0;
  border-radius: 10px;
  box-shadow: 0 2px 10px #00000012;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

label {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

input, select {
  padding: 10px;
  border: 1px solid #bbb;
  border-radius: 6px;
}

.actions {
  display: flex;
  gap: 8px;
  margin-top: 20px;
}

button {
  padding: 9px 13px;
  border: 0;
  border-radius: 6px;
  background: #357a50;
  color: white;
  cursor: pointer;
}

button.secondary {
  background: #777;
}

button.danger {
  background: #b84444;
  margin-left: 6px;
}

.message {
  color: #275c3c;
}

.table-wrap {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px;
  border-bottom: 1px solid #ddd;
  text-align: left;
}

@media (max-width: 650px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
}
</style>