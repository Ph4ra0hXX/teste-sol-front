<template>
  <div id="app">
    <div class="container">
      <header class="header">
        <h1 class="title">Monitoramento de Inversores</h1>
        <p class="subtitle">Potência atual e status dos sistemas Growatt</p>
      </header>

      <div class="search-container">
        <input
          v-model="searchTerm"
          type="text"
          placeholder="Buscar por nome ou status..."
          class="search-input"
        />
        <div class="stats">
          <span class="stat">Total: {{ filteredUsers.length }} plantas</span>
        </div>
      </div>

      <div class="table-container">
        <table class="users-table desktop-table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Nome</th>
              <th>Potência Atual (W)</th>
              <th>Status</th>
              <th>Última Atualização</th>
              <th>Ações</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="user in filteredUsers" :key="user.id" class="table-row">
              <td>{{ user.id }}</td>
              <td>{{ user.accountName }}</td>
              <td>{{ user.pac }} W</td>
              <td>
                <span
                  :class="[
                    'status-badge',
                    user.status === '1' ? 'ativo' : 'inativo',
                  ]"
                >
                  {{ user.status === "1" ? "Online" : "Offline" }}
                </span>
              </td>
              <td>{{ formatDate(user.lastUpdateTime) }}</td>
              <td>
                <button class="action-btn edit" @click="editUser(user)">
                  ✏️
                </button>
                <button class="action-btn delete" @click="deleteUser(user.id)">
                  🗑️
                </button>
              </td>
            </tr>
          </tbody>
        </table>

        <!-- Layout para mobile -->
        <div class="mobile-cards">
          <div v-for="user in filteredUsers" :key="user.id" class="user-card">
            <div class="card-header">
              <h3 class="card-name">{{ user.accountName }}</h3>
              <span
                :class="[
                  'status-badge',
                  user.status === '1' ? 'ativo' : 'inativo',
                ]"
              >
                {{ user.status === "1" ? "Online" : "Offline" }}
              </span>
            </div>

            <div class="card-details">
              <div class="detail-item">
                <span class="detail-label">ID:</span>
                <span class="detail-value">{{ user.id }}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Potência:</span>
                <span class="detail-value">{{ user.pac }} W</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Atualizado em:</span>
                <span class="detail-value">{{
                  formatDate(user.lastUpdateTime)
                }}</span>
              </div>
            </div>

            <div class="card-actions">
              <button class="action-btn edit" @click="editUser(user)">
                Editar
              </button>
              <button class="action-btn delete" @click="deleteUser(user.id)">
                Excluir
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="filteredUsers.length === 0" class="empty-state">
        <p>Nenhum inversor encontrado</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";

interface InverterData {
  id: string;
  pac: number;
  accountName: string;
  status: string;
  lastUpdateTime: string;
}

const searchTerm = ref("");
const users = ref<InverterData[]>([]);

const formatDate = (dateString: string) => {
  const date = new Date(dateString);
  return (
    date.toLocaleDateString("pt-BR") + " " + date.toLocaleTimeString("pt-BR")
  );
};

const filteredUsers = computed(() => {
  if (!searchTerm.value) return users.value;

  return users.value.filter(
    (user) =>
      user.id.includes(searchTerm.value.toLowerCase()) ||
      user.accountName.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
      (user.status === "1" ? "online" : "offline").includes(
        searchTerm.value.toLowerCase()
      )
  );
});

const editUser = (user: InverterData) => {
  alert(`Editando planta: ${user.accountName}`);
};

const deleteUser = (id: string) => {
  if (confirm("Tem certeza que deseja remover essa planta?")) {
    users.value = users.value.filter((user) => user.id !== id);
  }
};

onMounted(async () => {
  try {
    const res = await fetch(
      "https://teste-sol-git-master-ph4ra0hxxs-projects.vercel.app/"
    );
    const data = await res.json();
    users.value = data;
  } catch (error) {
    console.error("Erro ao buscar dados da API:", error);
  }
});
</script>
<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 0.5rem;
}

.subtitle {
  font-size: 1.125rem;
  color: #6b7280;
  margin: 0;
}

.search-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  gap: 1rem;
}

.search-input {
  flex: 1;
  max-width: 400px;
  padding: 0.75rem 1rem;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s ease;
}

.search-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.stats {
  color: #6b7280;
  font-weight: 500;
}

.table-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.users-table {
  width: 100%;
  border-collapse: collapse;
}

.users-table th {
  background: #f8fafc;
  padding: 1rem;
  text-align: left;
  font-weight: 600;
  color: #374151;
  border-bottom: 2px solid #e5e7eb;
}

.table-row {
  transition: background-color 0.2s ease;
}

.table-row:hover {
  background: #f8fafc;
}

.users-table td {
  padding: 1rem;
  border-bottom: 1px solid #e5e7eb;
  vertical-align: middle;
}

.user-id {
  font-weight: 600;
  color: #6b7280;
}

.name-container {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #3b82f6;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 1rem;
}

.user-name {
  font-weight: 600;
  color: #1f2937;
}

.user-email {
  color: #6b7280;
}

.department-tag {
  background: #e0f2fe;
  color: #0369a1;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
}

.status-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
}

.status-badge.ativo {
  background: #dcfce7;
  color: #15803d;
}

.status-badge.inativo {
  background: #fee2e2;
  color: #dc2626;
}

.user-actions {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  justify-content: center;
}

.action-btn {
  padding: 0.5rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.action-btn.edit {
  background: #e0f2fe;
  color: #0369a1;
}

.action-btn.edit:hover {
  background: #bae6fd;
}

.action-btn.delete {
  background: #fee2e2;
  color: #dc2626;
  margin-top: 5px;
}

.action-btn.delete:hover {
  background: #fecaca;
}

.empty-state {
  text-align: center;
  padding: 3rem;
  color: #6b7280;
  font-size: 1.125rem;
}

/* Mobile Cards Layout */
.mobile-cards {
  display: none;
}

.user-card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e7eb;
  transition: all 0.2s ease;
}

.user-card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
  gap: 1rem;
}

.card-header .name-container {
  flex: 1;
}

.user-info {
  flex: 1;
}

.card-name {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 0.25rem 0;
}

.card-email {
  color: #6b7280;
  font-size: 0.875rem;
  margin: 0;
}

.card-details {
  display: grid;
  gap: 0.75rem;
  margin-bottom: 1.5rem;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0;
  border-bottom: 1px solid #f3f4f6;
}

.detail-item:last-child {
  border-bottom: none;
}

.detail-label {
  font-weight: 500;
  color: #6b7280;
  font-size: 0.875rem;
}

.detail-value {
  font-weight: 600;
  color: #1f2937;
  font-size: 0.875rem;
}

.card-actions {
  display: flex;
  gap: 0.75rem;
  justify-content: flex-end;
}

.card-actions .action-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 8px;
}

@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }

  .title {
    font-size: 2rem;
  }

  .search-container {
    flex-direction: column;
    align-items: stretch;
  }

  .search-input {
    max-width: none;
  }

  /* Hide desktop table on mobile */
  .desktop-table {
    display: none;
  }

  /* Show mobile cards */
  .mobile-cards {
    display: block;
  }

  .card-header .name-container {
    gap: 0.75rem;
  }

  .card-header .avatar {
    width: 48px;
    height: 48px;
    font-size: 1.125rem;
  }

  .card-actions {
    flex-direction: column;
    gap: 0.5rem;
  }

  .card-actions .action-btn {
    justify-content: center;
    width: 100%;
  }
}

@media (min-width: 769px) and (max-width: 1024px) {
  .users-table th,
  .users-table td {
    padding: 0.75rem 0.5rem;
    font-size: 0.875rem;
  }

  .avatar {
    width: 36px;
    height: 36px;
    font-size: 0.875rem;
  }

  .action-btn {
    padding: 0.375rem;
  }
}
</style>
