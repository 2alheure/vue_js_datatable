# Vue JS DataTable
A DataTable displaying for Vue JS

## Table of contents
- [Vue JS DataTable](#vue-js-datatable)
	- [Table of contents](#table-of-contents)
	- [Usage](#usage)
		- [Set up](#set-up)
		- [Basic Usage](#basic-usage)
		- [Template / Slot usage](#template--slot-usage)
	- [Props](#props)
		- [Actions](#actions)
	- [Full Example](#full-example)

## Usage
### Set up
Download the component file and place it in your `components/` folder.  
You also need to download the [Pagination](https://github.com/2alheure/vue_js_pagination) component, since it is used by DataTable. Place it in the `components` folder too.  
Then, you can import DataTable with `import DataTable from "@/components/DataTable.vue";`.  

### Basic Usage
The minimum configuration usage is like following:  
```html
<DataTable :headers="headers" :cellsContent="users" />

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
Please note that in this configuration, the order of the `headers` array's matter. Columns will be displayed in that order.  
`headers linkTo` property must correspond to a property of the objects displayed. It must points to a first level property (i.e. cannot point to a nested property such as `address.city`).  
The properties of `cellsContent` can be HTML elements. In that case, do not include the `<td>` tag, which is already written by the component.  
Unused properties of displayed objects are ignored.

### Template / Slot usage
You can also be more verbous and write your own rows' templates.  
Here is an example, corresponding to the one above:  
```html
<DataTable :headers="headers">
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
      :headers="header.linkTo"
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
      headers: [
        { linkTo: "id" },
        { linkTo: "gender" },
        { linkTo: "first_name" },
        { linkTo: "last_name" },
        { linkTo: "email" },
        { linkTo: "ip_address" }
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
This will produce the same result as the previous example.  
You can also specify the `<template #default>` tag but I am lazy so I don't (😜).  
The `<template #thead>` describes the content of the `<thead>` tag.  
The rest stands for the `<tbody>` tag.  
Though it is more readable than the previous example, it is not performance-friendly as the component has to read your content to extract the information it needs to compute.


## Props
You can pass several props to the component:
- `stripe` is a boolean, that defaults to `false`, and tells whether the body of the table should be striped (meaning one row out of two will have a slightly grey background).
- `hover` is a boolean, that defaults to `false`, and tells whether the rows should have an hovering (a slightly grey background).
- `actions` is an array of objects. See [below](#actions) for more details.
- `noContentMessage` is a String that is printed when there is no row to output.
- `ascSymbol` is a String used as the symbol for columns when sorted ascendingly.
- `descSymbol` is a String used as the symbol for columns when sorted descendingly.
- `neutralSymbol` is a String used as the symbol for columns when not sorted.

### Actions
Eventhough you can write your own templates, `@event` properties won't work. This is because of the internal computing of the component.  
I still wanted to offer a way to put action buttons and make them reactive, so I implemented the `actions` prop.  
It simply is an array of objects, each one describing a button which can handle an `onclick` event.  
The `actions` objects properties are:
- `html`: a String that represents the HTML content of your button. Do not include the `<button>` tag.
- `class`: a String, Object or Function. The class(es) to apply to your button.
- `style`: a String, Object or Function. The style(s) to apply to your button.
- `atClick`: a Function, taking two arguments and processing whatever you want when the button is clicked. The two arguments are the object content of your row (keyed with the `linkTo` property of the `headers` object) and the Vue JS event object.  
Example for a simple action button:  
```js
actions: [{
  html: `<i class="fas fa-exclamation-circle"></i>`,  // A Font Awesome icon
  class: "bg-red-600 text-white rounded-full",  // Tailwind CSS classes
  atClick: (user, event) => alert(user.first_name+' '+user.last_name+' was triggered !')
}]
```
When you use the `action` prop, you can also provide two more props:
- `actionsHeader`: Actions are put in an additionnal column (do not write it in your `<template #thead>` !). This prop gives the html of the heading of the column.
- `actionsAtEnd`: a Boolean that defaults to `false` and tells where to put the actions. Set it to `true` if you want the buttons at the end of the rows.

## Full Example
```html
<DataTable
  :headers="headers"
  :actions="actions"
  :actionsHeader="Functions"
  :actionsAtEnd="true"
  :stripe="true"
  :hover="true"
>
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
      ],
      actions: [
        {
          html: "<i><b>CLICK ME !</b></i>",
          atClick: (u, e) => {
            alert(u.first_name+' '+u.last_name+' clicked me ! =D');
          },
          class: "bg-green-700 text-white p-2",
          style: "border: 5px solid red"
        }
      ]
    };
  }
};
</script>
```
