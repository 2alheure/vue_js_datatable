<style lang="postcss">
</style>

<template>
  <div class="container w-full md:w-4/5 xl:w-3/5 mx-auto px-2">
    <select name="by-page" v-model="byPage">
      <option value="10">10</option>
      <option value="25">25</option>
      <option value="50">50</option>
      <option value="100">100</option>
    </select>

    <input
      type="search"
      name="search"
      class="border border-gray-400 rounded-lg"
      v-model="searchString"
      placeholder="Search"
    />

    <table class="stripe hover w-full py-4">
      <thead>
        <tr>
          <th
            v-for="header in headers"
            :key="header.linkTo+'-header'"
            v-html="header.html"
            @click="changeOrderBy(header.linkTo)"
          ></th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(content, index) in realContents.slice((this.page - 1) * this.byPage, this.page * this.byPage)"
          :key="index"
        >
          <td
            v-for="(header, headerIndex) in headers"
            :key="index+'-'+headerIndex"
            v-html="content[header.linkTo]"
          ></td>
        </tr>
      </tbody>
    </table>
    <small
      class="block text-left py-2"
    >Showing {{(this.page - 1) * this.byPage + 1}} to {{Math.min(this.page * this.byPage, realContents.length)}} of {{realContents.length}}.</small>
    <Pagination v-model="page" :maxPage="Math.ceil(realContents.length / byPage)" />
  </div>
</template>
		
<script>
import Pagination from "./Pagination.vue";

export default {
  name: "DataTable",
  components: {
    Pagination
  },
  props: {
    headers: Array,
    cellContents: Array
  },
  data() {
    return {
      contents: this.cellContents,
      page: 1,
      byPage: 10,
      searchString: null,
      orderBy: this.headers[0].linkTo,
      asc: true
    };
  },
  computed: {
    realContents() {
      return this.contents
        .map((e, i) => {
          // Get the order by and make an array to walk through with sort
          return { index: i, value: e[this.orderBy] };
        })
        .sort((a, b) => {
          // Sort by the orderBy asc or desc
          if (this.asc) {
            if (typeof a.value == "string")
              return a.value.localeCompare(b.value);
            else return a.value - b.value;
          } else {
            if (typeof b.value == "string")
              return b.value.localeCompare(a.value);
            else return b.value - a.value;
          }
        })
        .map(e => this.contents[e.index]) // Get the original values
    }
  },
  methods: {
    changeOrderBy(field) {
      if (this.orderBy == field) this.asc = !this.asc;
      else {
        this.orderBy = field;
        this.asc = true;
      }
    }
  }
};
</script>