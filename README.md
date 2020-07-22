# Vue JS DataTable
A DataTable display for Vue JS

## Table of contents
- [Vue JS DataTable](#vue-js-datatable)
	- [Table of contents](#table-of-contents)
	- [Usage](#usage)
		- [Set up](#set-up)
		- [Basic Usage](#basic-usage)
		- [Template / Slot usage](#template--slot-usage)
	- [Props](#props)
		- [About the headers](#about-the-headers)
	- [Full Example](#full-example)

## Usage
### Set up
Download the component file and place it in your `components/` folder.  
You also need to download the [Pagination](https://github.com/2alheure/vue_js_pagination) component, since it is used by DataTable. Place it in the `components` folder too.  
Then, you can import DataTable with `import DataTable from "@/components/DataTable.vue";`.  

### Basic Usage
The minimum configuration usage is like following:  
```html
<DataTable :headers="headers" v-model="users" />

<script>
import DataTable from "@/components/DataTable.vue";

export default {
  components: {
    DataTable
  },
  data() {
    return {
      headers: [
        { html: "ID", linkTo: "id" },
        { html: "Gender", linkTo: "gender" },
        { html: "First Name", linkTo: "first_name" },
        { html: "Last Name", linkTo: "last_name" },
        { html: "Email", linkTo: "email" },
        { html: "IP Address", linkTo: "ip_address" }
      ],
      users: [
        {
          "id": 1,
          "first_name": "<b>Virginie</b>",
          "last_name": "Merrin",
          "email": "vmerrin0@howstuffworks.com",
          "gender": "Female",
          "ip_address": "86.87.125.205"
        },
        {
          "id": 2,
          "first_name": "<b>Mariana</b>",
          "last_name": "Dorgan",
          "email": "mdorgan1@indiegogo.com",
          "gender": "Female",
          "ip_address": "188.236.23.74"
        },
        {
          "id": 3,
          "first_name": "<b>Karalee</b>",
          "last_name": "Dod",
          "email": "kdod2@va.gov",
          "gender": "Female",
          "ip_address": "82.75.65.131"
        }
      ]      
    }
  }
};
</script>
```
Please note the `v-model` directive. The `users` object will be modified by the component. If needed, think of making a copy of it.  
Please note that in this configuration, the order of the `headers` array's matters. Columns will be displayed in that order.  
`headers.linkTo` property must correspond to a property of the objects displayed. It must points to a first level property (i.e. cannot point to a nested property such as `address.city`).  
The properties of `cellsContent` can be HTML elements. In that case, do not include the `<td>` tag, which is already written by the component.  
Unused properties of displayed objects are ignored.

### Template / Slot usage
You can also be more verbous and write your own rows' templates.  
Here is an example, corresponding to the one above:  
```html
<DataTable :headers="headers" v-model="users">
  <template #thead>
    <tr>
      <th>ID</th>
      <th>Gender</th>
      <th>First Name</th>
      <th>Last Name</th>
      <th>Email</th>
      <th>IP Address</th>
    </tr>
  </template>

  <tr v-for="(user, index) in users" :key="index">
	<td
      v-for="(header, headerIndex) in headers"
      :key="index+'-'+headerIndex"
      v-html="user[header.linkTo]"
	></td>
  </tr>
</DataTable>

<script>
import DataTable from "@/components/DataTable.vue";

export default {
  // Same as above
};
</script>
```
You can also specify the `<template #default>` tag but I am lazy so I don't (😜).  
The `<template #thead>` describes the content of the `<thead>` tag.  
The rest stands for the `<tbody>` tag.  
As the templates renders according to `users` and `users` is used in the `v-model`, it re-renders automatically with rows that need to be printed.  
This will produce the same result as the previous example, except that you won't have the column ordering since you do not have the `@click` here. To do so, you need to put a `ref` on the component and to call its `changeOrderBy` function in the `@click` like following:  
```html
<DataTable v-ref="myDT" ...>
  <template #thead>
    <tr>
      <th @click="$refs.myDT.changeOrderBy('id')">ID</th>

  <!-- The rest of the content -->
```
`changeOrderBy` takes a parameter which is the name of a property of the object.


## Props
You can pass several props to the component:
- `stripe` is a boolean, that defaults to `false`, and tells whether the body of the table should be striped (meaning one row out of two will have a slightly grey background).
- `hover` is a boolean, that defaults to `false`, and tells whether the rows should have an hovering (a slightly grey background).
- `noContentMessage` is a String that is printed when there is no row to output.
- `ascSymbol` is a String used as the symbol for columns when sorted ascendingly.
- `descSymbol` is a String used as the symbol for columns when sorted descendingly.
- `neutralSymbol` is a String used as the symbol for columns when not sorted.

### About the headers
You can pass a `headers` object containing headers with `linkTo` property to `null`. If so, when automatically displaying the `<thead>`, you will not have ordering available on this column. This also requires to manually display the rows since the component will not be able to get the right data without the information.

## Full Example
```html
<DataTable
  v-model="users"
  :headers="headers"
  :stripe="true"
  :hover="true"
  v-ref="myDT"
>
  <template #thead>
    <tr>
      <th @click="$refs.myDT.changeOrderBy('id')">ID</th>
      <th>Gender</th>
      <th>First Name</th>
      <th>Last Name</th>
      <th>Email</th>
      <th>IP Address</th>
    </tr>
  </template>

  <tr v-for="(user, index) in users" :key="index">
    <td
      v-for="(header, headerIndex) in headers"
      :key="index+'-'+headerIndex"
      v-html="user[header.linkTo]"
    ></td>
  </tr>
</DataTable>

<script>
import DataTable from "@/components/DataTable.vue";

export default {
  components: {
    DataTable
  },
  data() {
    return {
      users: [
        {
          "id": 1,
          "first_name": "<b>Virginie</b>",
          "last_name": "Merrin",
          "email": "vmerrin0@howstuffworks.com",
          "gender": "Female",
          "ip_address": "86.87.125.205"
        },
        {
          "id": 2,
          "first_name": "<b>Mariana</b>",
          "last_name": "Dorgan",
          "email": "mdorgan1@indiegogo.com",
          "gender": "Female",
          "ip_address": "188.236.23.74"
        },
        {
          "id": 3,
          "first_name": "<b>Karalee</b>",
          "last_name": "Dod",
          "email": "kdod2@va.gov",
          "gender": "Female",
          "ip_address": "82.75.65.131"
        }
      ],
      headers: [
        { html: "ID", linkTo: "id" },
        { html: "Gender", linkTo: "gender" },
        { html: "First Name", linkTo: "first_name" },
        { html: "Last Name", linkTo: "last_name" },
        { html: "Email", linkTo: "email" },
        { html: "IP Address", linkTo: "ip_address" }
      ]
    };
  }
};
</script>
```
