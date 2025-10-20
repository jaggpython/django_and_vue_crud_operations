<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

// 🔗 Django API base URL
const API_URL = 'http://127.0.0.1:8000/api/users/'

// 🔸 Reactive variables
const users = ref<User[]>([])
const newUser = ref<User>({ id: 0, name: '', email: '' })
const isEditing = ref(false)
const editingId = ref<number | null>(null)
const message = ref('')
const showMessage = ref(false)
const messageType = ref<'success' | 'error'>('success')
const confirmVisible = ref(false)
const confirmUserId = ref<number | null>(null)

const openConfirm = (id: number) => {
  confirmUserId.value = id
  confirmVisible.value = true
}

const confirmDelete = async () => {
  if (confirmUserId.value !== null) {
    await deleteUser(confirmUserId.value)
    confirmVisible.value = false
    confirmUserId.value = null
  }
}

const cancelDelete = () => {
  confirmVisible.value = false
  confirmUserId.value = null
}

// 🌈 Background image change every 15 seconds
const images = ['/bg1.jpg', '/bg4.jpg', '/bg2.png', '/bg3.jpg']
let index = ref(0)
onMounted(() => {
  const bg = document.querySelector('.background') as HTMLElement
  setInterval(() => {
    index.value = (index.value + 1) % images.length
    bg.style.backgroundImage = `url(${images[index.value]})`
  }, 15000)
  fetchUsers()
})

// 🎯 Popup message
const showPopup = (msg: string, type: 'success' | 'error' = 'success') => {
  message.value = msg
  messageType.value = type
  showMessage.value = true
  setTimeout(() => (showMessage.value = false), 2000)
}

// 🧩 Fetch all users (GET)
const fetchUsers = async () => {
  try {
    const res = await fetch(API_URL)
    if (!res.ok) {
      const errData = await res.json()
      throw new Error(errData.detail || 'Failed to fetch users')
    }
    users.value = await res.json()
  } catch (err: any) {
    showPopup(`⚠️ ${err.message}`, 'error')
  }
}

// ➕ Add user (POST)
const addUser = async () => {
  if (!newUser.value.name || !newUser.value.email) {
    showPopup('Please fill all fields', 'error')
    return
  }
  try {
    const res = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: newUser.value.name,
        email: newUser.value.email,
      }),
    })

    if (!res.ok) {
      const errData = await res.json()
      throw new Error(
        Object.values(errData).flat().join(', ') || 'Failed to add user'
      )
    }

    await fetchUsers()
    newUser.value = { id: 0, name: '', email: '' }
    showPopup('✅ User added successfully!')
  } catch (err: any) {
    showPopup(`⚠️ ${err.message}`, 'error')
  }
}

// ✏️ Edit user
const editUser = (user: User) => {
  isEditing.value = true
  editingId.value = user.id
  newUser.value = { ...user }
}


// ♻️ Update user (PUT)
const updateUser = async () => {
  if (!editingId.value) return
  try {
    const res = await fetch(`${API_URL}${editingId.value}/`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: newUser.value.name,
        email: newUser.value.email,
      }),
    })

    if (!res.ok) {
      const errData = await res.json()
      throw new Error(
        Object.values(errData).flat().join(', ') || 'Failed to update user'
      )
    }

    await fetchUsers()
    cancelEdit()
    showPopup('✅ User updated successfully!')
  } catch (err: any) {
    showPopup(`⚠️ ${err.message}`, 'error')
  }
}

// ❌ Cancel editing
const cancelEdit = () => {
  isEditing.value = false
  editingId.value = null
  newUser.value = { id: 0, name: '', email: '' }
  showPopup('❌ Edit cancelled', 'error')
}

// 🗑️ Delete user (DELETE)
const deleteUser = async (id: number) => {
  try {
    const res = await fetch(`${API_URL}${id}/`, { method: 'DELETE' })
    if (!res.ok) {
      const errData = await res.json()
      throw new Error(errData.detail || 'Failed to delete user')
    }
    await fetchUsers()
    showPopup('🗑️ User deleted successfully!')
  } catch (err: any) {
    showPopup(`⚠️ ${err.message}`, 'error')
  }
}

</script>

<template>
  <div class="background">
    <div class="overlay">
      <h1 class="title">CRUD Operations</h1>

      <!-- Popup -->
      <transition name="fade">
        <div
          v-if="showMessage"
          class="popup"
          :class="messageType === 'success' ? 'popup-success' : 'popup-error'"
        >
          {{ message }}
        </div>
      </transition>

      <!-- Delete Confirmation Modal -->
    <transition name="fade">
    <div v-if="confirmVisible" class="confirm-overlay">
        <div class="confirm-box">
        <h3>🗑️ Confirm Deletion</h3>
        <p>Are you sure you want to delete this user?</p>
        <div class="confirm-buttons">
            <button class="confirm-yes" @click="confirmDelete">Yes, Delete</button>
            <button class="confirm-no" @click="cancelDelete">Cancel</button>
        </div>
        </div>
    </div>
    </transition>


      <!-- Table -->
      <div class="table-container">
        <table border="1" class="user-table">
          <thead>
            <tr>
              <th>#</th>
              <th>Name</th>
              <th>Email</th>
              <th id="action-bt">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="users.length === 0">
              <td colspan="4" class="no-data">No users found.</td>
            </tr>
            <tr v-for="(user, index) in users" :key="user.id">
              <td>{{ index + 1 }}</td>
              <td>{{ user.name }}</td>
              <td>{{ user.email }}</td>
              <td>
                <button @click="editUser(user)" class="action-btn edit">Edit</button>
                <button @click="openConfirm(user.id)" class="action-btn delete">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Form -->
      <div class="form-container">
        <input v-model="newUser.name" placeholder="Enter name" class="input-field" />
        <input v-model="newUser.email" placeholder="Enter email" class="input-field" />

        <button v-if="!isEditing" @click="addUser" class="add-btn">
          ➕ Add User
        </button>

        <div v-else class="button-group">
          <button class="update-btn" @click="updateUser">✅ Update</button>
          <button class="cancel-btn" @click="cancelEdit">❌ Cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>


<style scoped>

.background {
  background-image: url('/bg3.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: auto;
  transition: background-image 5s ease-in-out;
}

.overlay {
  min-height: 100vh;
  padding: 20px;
}

.title {
  width: 80%; /* match your .user-table width */
  margin: 30px auto 10px; /* centers it horizontally */
  text-align: center;
  color: #fff;
  font-size: 30px;
  font-weight: 700;
  background: linear-gradient(135deg, #2c67ef, #6a11cb);
  padding: 14px 0;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
  letter-spacing: 1px;
  text-transform: uppercase;
  font-family: 'Poppins', 'Times New Roman', Times, serif;
  backdrop-filter: blur(6px);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.title:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.35);
}



/* Popup Message */
.popup {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 14px 20px;
  border-radius: 8px;
  color: white;
  font-weight: 600;
  font-size: 14px;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  animation: fadeInOut 2s ease forwards;
}

.popup-success {
  background-color: #28a745;
}

.popup-error {
  background-color: #dc3545;
}

/* Fade transition for popup */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Popup animation */
@keyframes fadeInOut {
  0% { opacity: 0; transform: translateY(-20px); }
  10% { opacity: 1; transform: translateY(0); }
  90% { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-20px); }
}


.add-btn {
  background-color: #28a745;
  color: white;
}

.add-btn:hover {
  background-color: #218838;
}

.button-group {
  display: flex;
  gap: 10px;
}

.update-btn {
  flex: 1;
  background-color: #007bff;
  color: white;
}

.update-btn:hover {
  background-color: #0056b3;
}

.cancel-btn {
  flex: 1;
  background-color: #6c757d;
  color: white;
}

.cancel-btn:hover {
  background-color: #5a6268;
}

.table-container {
  display: flex;
  justify-content: center;
  margin-top: 40px;
  padding: 10px;
}

.user-table {
  width: 80%;
  border-collapse: collapse;
  border-radius: 8px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 6px 25px rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(8px);
  transition: all 0.3s ease;
}

.user-table:hover {
  transform: scale(1.01);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.25);
}

/* Header */
.user-table th {
  background: linear-gradient(135deg, #2b65ec, #1e3c72);
  color: #fff;
  padding: 14px;
  text-align: center;
  font-size: 16px;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

/* Table Cells */
.user-table td {
  padding: 14px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  text-align: center;
  color: #333;
  font-size: 15px;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

/* Zebra effect */
.user-table tbody tr:nth-child(even) {
  background: rgba(240, 240, 240, 0.5);
}

/* Hover row highlight */
.user-table tbody tr:hover {
  background: rgba(44, 103, 239, 0.1);
  transform: scale(1.01);
}

/* Action column */
.user-table td:last-child {
  display: flex;
  justify-content: center;
  gap: 10px;
}

/* Buttons */
.user-table td:last-child .action-btn {
  width: 90px;
  text-align: center;
  padding: 8px 0;
  font-weight: 600;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  color: #fff;
  transition: all 0.25s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
}

/* Edit Button */
.action-btn.edit {
  background: linear-gradient(135deg, #ffce00, #ffa600);
  color: #000;
}

.action-btn.edit:hover {
  background: linear-gradient(135deg, #ffa600, #ff8700);
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 6px 15px rgba(255, 166, 0, 0.5);
}

/* Delete Button */
.action-btn.delete {
  background: linear-gradient(135deg, #ff4b5c, #dc3545);
}

.action-btn.delete:hover {
  background: linear-gradient(135deg, #e63946, #c82333);
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 6px 15px rgba(220, 53, 69, 0.5);
}


.no-data {
  text-align: center;
  padding: 16px;
  font-style: italic;
  color: #666;
}

.form-container {
  position: relative; /* allows layering */
  margin: 30px auto;
  width: 360px;
  padding: 25px 30px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(12px) saturate(180%);
  -webkit-backdrop-filter: blur(12px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.25);
  box-shadow: 0 8px 32px rgba(31, 38, 135, 0.25);
  transition: all 0.3s ease-in-out;
  text-align: center;
  z-index: 1; /* make form above background layers */
}

.form-container:hover {
  transform: translateY(-5px) scale(1.03);
  box-shadow: 0 12px 36px rgba(0, 0, 0, 0.25);
}

/* Background overlay, placed behind form content */
.form-container::before {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: 16px;
  background: linear-gradient(145deg, rgba(255,255,255,0.15), rgba(255,255,255,0.05));
  z-index: -1; /* push behind form elements */
  pointer-events: none; /* ensures click-through */
}

/* Inputs */
.input-field {
  width: 100%;
  padding: 12px;
  margin-bottom: 14px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.4);
  background: rgba(255, 255, 255, 0.7);
  font-size: 15px;
  outline: none;
  color: #333;
  transition: all 0.2s ease;
}

.input-field:focus {
  border-color: #007bff;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}

/* Buttons */
.add-btn,
.update-btn,
.cancel-btn {
  padding: 10px;
  border-radius: 8px;
  font-weight: 600;
  font-size: 15px;
  border: none;
  transition: all 0.25s ease;
}

.add-btn:hover,
.update-btn:hover,
.cancel-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.25);
}

/* 🌐 Responsive Design */
@media (max-width: 1024px) {
  .title,
  .user-table {
    width: 90%;
  }

  .form-container {
    width: 80%;
  }
}

@media (max-width: 768px) {
  .title {
    width: 90%;
    font-size: 24px;
    padding: 10px;
  }

  .user-table {
    width: 95%;
    font-size: 14px;
  }

  .user-table th,
  .user-table td {
    padding: 10px;
    font-size: 13px;
  }

  .form-container {
    width: 90%;
    padding: 20px;
  }

  .input-field {
    font-size: 14px;
    padding: 10px;
  }

  .add-btn,
  .update-btn,
  .cancel-btn {
    font-size: 14px;
    padding: 10px;
  }

  .popup {
    width: 80%;
    left: 10%;
    right: 10%;
    text-align: center;
  }
}

@media (max-width: 480px) {
  .title {
    width: 95%;
    font-size: 20px;
    letter-spacing: 0.5px;
    padding: 8px;
  }

  .user-table {
    width: 100%;
    display: block;
    overflow-x: auto;
    border-radius: 8px;
  }

  .user-table th,
  .user-table td {
    padding: 8px;
    font-size: 12px;
  }

  .user-table td:last-child {
    flex-direction: column;
    gap: 6px;
  }

  .action-btn {
    width: 100%;
    font-size: 13px;
    padding: 6px;
  }

  .form-container {
    width: 95%;
    padding: 15px;
  }

  .input-field {
    padding: 10px;
    font-size: 13px;
  }

  .add-btn,
  .update-btn,
  .cancel-btn {
    font-size: 13px;
    padding: 8px;
  }

  .popup {
    font-size: 12px;
    padding: 10px;
    width: 90%;
    left: 5%;
  }
}

/* 🧊 Confirmation Modal */
.confirm-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.55);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  backdrop-filter: blur(4px);
  animation: fadeIn 0.3s ease;
}

.confirm-box {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 30px 40px;
  text-align: center;
  backdrop-filter: blur(10px) saturate(180%);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.25);
  border: 1px solid rgba(255, 255, 255, 0.3);
  animation: scaleIn 0.3s ease;
}

.confirm-box h3 {
  color: #fff;
  margin-bottom: 10px;
  font-size: 20px;
}

.confirm-box p {
  color: #eee;
  font-size: 14px;
  margin-bottom: 20px;
}

.confirm-buttons {
  display: flex;
  justify-content: center;
  gap: 15px;
}

.confirm-yes,
.confirm-no {
  padding: 10px 18px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  font-weight: 600;
  font-size: 14px;
  transition: all 0.3s ease;
}

.confirm-yes {
  background: linear-gradient(135deg, #ff4b5c, #dc3545);
  color: white;
}
.confirm-yes:hover {
  background: linear-gradient(135deg, #e63946, #c82333);
  transform: translateY(-2px);
}

.confirm-no {
  background: linear-gradient(135deg, #6c757d, #495057);
  color: white;
}
.confirm-no:hover {
  background: linear-gradient(135deg, #5a6268, #343a40);
  transform: translateY(-2px);
}

/* Animations */
@keyframes scaleIn {
  from { transform: scale(0.9); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}


</style>