<template>
  <main>
    <div>
      <h2>With props</h2>
      <DataTable
        :headers="headers"
        v-model="users"
        style="max-width: 70%; margin: auto; border: 1px solid red"
      />
    </div>
    <div>
      <h2>With <code>&lt;slot&gt;</code></h2>
      <DataTable
        v-model="usersCopy"
        style="max-width: 70%; margin: auto; border: 1px solid purple"
        :paginationProps="{ textColor: 'red' }"
        translation="fr"
      >
        <template #thead>
          <tr>
            <th data-order-by="username">Username</th>
            <th>First Name</th>
            <th>Last Name</th>
            <th>Gender</th>
            <th>Email</th>
            <th>Phone</th>
            <th>City</th>
          </tr>
        </template>

        <tr v-for="(user, index) in usersCopy" :key="index">
          <td style="color: blue">{{ user.username }}</td>
          <td>{{ user.first_name }}</td>
          <td>{{ user.last_name }}</td>
          <td style="color: green">{{ user.gender }}</td>
          <td>
            <a :href="'mailto:' + user.email">{{ user.email }}</a>
          </td>
          <td>{{ user.phone_number }}</td>
          <td @click="console.log('test')">{{ user.location.city }}</td>
        </tr>
      </DataTable>
    </div>
  </main>
</template>

<script>
import DataTable from "../src/DataTable.vue";
import users from "./users.json";

export default {
  components: {
    DataTable,
  },
  data() {
    return {
      headers: [
        { html: "Username", linkTo: "username" },
        { html: "First Name", linkTo: "first_name" },
        { html: "Last Name", linkTo: "last_name" },
        { html: "Gender", linkTo: "gender" },
        { html: "Email", linkTo: "email" },
        { html: "Phone", linkTo: "phone_number" },
        { html: "City", linkTo: "location.city" },
      ],
      users: users,
      usersCopy: users,
    };
  },
};
</script>