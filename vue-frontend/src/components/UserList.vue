<script setup lang="ts">
import { onMounted, ref } from 'vue'

const userList = ref([]);

onMounted(async () => {
  const response = await fetch('http://localhost:3000/user');
  userList.value = await response.json();
});
</script>

<template>
  <p><strong>Users list</strong></p>
  <table>
    <thead>
    <tr><th>ID</th><th>First name</th><th>Last name</th></tr>
    </thead>
    <tbody>
    <tr v-for="(user, i) in userList" :key="i">
      <td>{{ user.id }}</td><td>{{ user.firstName }}</td><td>{{ user.lastName }}</td>
    </tr>
    </tbody>
  </table>
</template>

<style scoped>
table {
  border: 1px solid white;
  border-collapse: collapse;
}
table td, table th {
  border: 1px solid white;
  padding: 10px;
}
</style>