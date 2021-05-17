# Vue.js DataTable
A DataTable display for Vue.js.

## Table of contents
- [Vue.js DataTable](#vuejs-datatable)
  - [Table of contents](#table-of-contents)
  - [What is this component made for](#what-is-this-component-made-for)
  - [Installation](#installation)
    - [Dependencies](#dependencies)
    - [Test](#test)
  - [Usage](#usage)
    - [Basic Usage](#basic-usage)
    - [Template / Slot usage](#template--slot-usage)
  - [Props](#props)
    - [About the `headers` prop](#about-the-headers-prop)
    - [About translations](#about-translations)
  - [Events](#events)

## What is this component made for
This component's purpose is to offer a Vue.js alternative for the jQuery plugin [DataTables](https://datatables.net/).  
It is a plugin that offers functionnalities for tables, such as ordering, search or pagination.

## Installation
First run `npm install @2alheure/vue-datatable`. Then you can import the component with `import DataTable from "@2alheure/vue-datatable";`.

### Dependencies
This package depends on two others:
- [@2alheure/vue-pagination](https://www.npmjs.com/package/@2alheure/vue-pagination)
- [lodash](https://www.npmjs.com/package/lodash)

### Test
The `test` command in this package uses some random users provided by [@pixelastic](https://github.com/pixelastic/fakeusers/blob/master/data/randomuser.json).   

## Usage

### Basic Usage
The minimum configuration usage is like following:  
```html
<DataTable :headers="headers" v-model="users" />

<script>
import DataTable from "@2alheure/vue-datatable";

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
Please note the `v-model` directive. The `users` object **will be modified** by the component. If needed, think of making a copy of your data, and not passing it directly to the component.  
Please note that in this configuration, the order of the `headers` prop **does matters**. Columns will be displayed in that order.  
`headers.linkTo` property must correspond to a property of the `v-model` linked object.   

### Template / Slot usage
You can also be more verbous and write your own rows' templates.  
Here is an example, corresponding to the one above:  
```html
<DataTable v-model="users">
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
import DataTable from "@2alheure/vue-datatable";

export default {
  components: {
    DataTable
  },
  data() {
    return {
      users: [
        // Same as above
      ]
    }
};
</script>
```
The `<template #thead>` describes the content of the `<thead>` tag of the table.  
The rest stands for the `<tbody>` tag. You can also specify the `<template #default>` tag but I am lazy so I don't (😜).  
As the templates renders according to `users` and `users` is used in the `v-model`, it re-renders automatically with rows that need to be printed.  
This will produce the same result as the previous example, except that you won't have the column ordering. To do so, you can use the `data-order-by` attribute on the header:  
```html
  <template #thead>
    <tr>
      <th data-order-by="id">ID</th>
      <th data-order-by="gender">Gender</th>
      <th data-order-by="first_name">First Name</th>
      <th data-order-by="last_name">Last Name</th>
      <th data-order-by="email">Email</th>
      <th data-order-by="ip_address">IP Address</th>
    </tr>
  </template>
```

## Props
| Name            |         Type         |        Default value        | Description                                                                                 |
| :-------------- | :------------------: | :-------------------------: | ------------------------------------------------------------------------------------------- |
| content         |        Array         |            none             | The prop linked to the `v-model` directive (mandatory).                                     |
| headers         |        Array         |            none             | Refer to the "[About the `headers` prop](#about-the-headers-prop)" section.                 |
| identifier      |        String        | A randomly generated string | The id of the component. Used by default to distinguish two different DataTable components. |
| ascSymbol       |        String        |          `&#8595;`          | The symbol when a column is ordered ascendingly.                                            |
| descSymbol      |        String        |          `&#8593;`          | The symbol when a column is ordered descendingly.                                           |
| neutralSymbol   |        String        |          `&#8597;`          | The symbol when a column is not ordered.                                                    |
| byPageOptions   |        Array         |     `[10, 25, 50, 100]`     | The possible values for the number of elements by page.                                     |
| paginationProps |        Object        |           `null`            | See [the Pagination component props](https://github.com/2alheure/vue_pagination#props).     |
| hover           |       Boolean        |           `false`           | Whether an hovered row should be styled to be a bit more visible.                           |
| stripe          |       Boolean        |           `false`           | Whether the table should style rows such as even rows are greyish.                          |
| translation     | String &vert; Object |            `en`             | Refer to the "[About translations](#about-translations)" section.                           |

### About the `headers` prop
The `headers` prop is **mandatory** when you use the DataTable without specifying the `<template #thead>`.  
It must be an array of objects, each one containing two keys:
- `html`: This will be the HTML content of the `<th>` tag
- `linkTo`: This is the property of the corresponding key in the `v-model` linked prop for the column.

### About translations
A translation can be a string, corresponding to one of the following values:
- `de`: german
- `en`: english
- `es`: spanish
- `fr`: french

You can also provide an object, which will replace the default translation sentences.  
The translation object looks like following (which is the english one, used by default):
```json
{
  "noContent": "There is nothing to print.",
  "showing": "Showing lines {0} to {1} of {2}.",
  "search": "Search"
}
```
If you provide an object, it must match this template, respecting the keys, and all keys are required.

## Events
| Name     | Value type | Value                                      | Description                                                   |
| :------- | :--------: | ------------------------------------------ | ------------------------------------------------------------- |
| change   |   Array    | The new value of the `v-model` linked prop | Emitted each time a change in made in the component           |
| page     |   Number   | The page number                            | Emitted each time the page changes (including when searching) |
| by-page  |   Number   | The number of elements by page             | Emitted each time the number of elements by page changes      |
| order-by |   String   | The column to order by                     | Emitted each time the order changes                           |
| search   |   String   | The search string                          | Emitted each time a search is made                            |
