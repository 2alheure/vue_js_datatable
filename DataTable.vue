<style lang="postcss" scoped>
.dt > p > select,
.dt > p > input {
  height: 2.25rem;
  font-size: 1rem;
  vertical-align: middle;
  border: 1px solid #cbd5e0;
  border-radius: 5px;
  padding: 0.5rem;
}

.dt table thead tr th {
  cursor: pointer;
}

.dt table.hover tbody tr:hover {
  background-color: #f3f3f3;
}

.dt table.stripe tbody tr:nth-of-type(even) {
  background-color: #f6f6f6;
}
</style>

<template>
  <div class="dt">
    <p class="flex justify-between">
      <select name="by-page" v-model="byPage">
        <option value="10">10</option>
        <option value="25">25</option>
        <option value="50">50</option>
        <option value="100">100</option>
      </select>

      <input type="search" name="search" v-model="searchString" placeholder="Search" />
    </p>

    <table
      :class="['w-full my-4 table-auto', (hover ? 'hover' : null), (stripe ? 'stripe' : null)]"
    >
      <thead>
        <tr>
          <th v-if="actions.length > 0 && !actionsAtEnd" v-html="actionsHeader"></th>
          <th
            v-for="header in headers"
            :key="keyPrefix+'-'+header.linkTo+'-header'"
            :id="header.linkTo+'-header'"
            v-html="header.html+'&nbsp;'+symbol(header.linkTo)"
            @click="changeOrderBy(header.linkTo)"
          ></th>
          <th v-if="actions.length > 0 && actionsAtEnd" v-html="actionsHeader"></th>
        </tr>
      </thead>

      <tbody v-if="contents.length">
        <tr v-for="(content, index) in realContents.slice(from-1, to)" :key="keyPrefix+'-'+index">
          <td v-if="actions.length > 0 && !actionsAtEnd">
            <button
              v-for="(action, actionIndex) in actions"
              :key="keyPrefix+'-'+index+'-button-'+actionIndex"
              :class="action.class"
              :style="action.style"
              v-html="action.html"
              @click="action.atClick(content)"
            ></button>
          </td>

          <td
            v-for="(header, headerIndex) in headers"
            :key="keyPrefix+'-'+index+'-'+headerIndex"
            :headers="header.linkTo"
            v-html="content[header.linkTo]"
          ></td>

          <td v-if="actions.length > 0 && actionsAtEnd">
            <button
              v-for="(action, actionIndex) in actions"
              :key="keyPrefix+'-'+index+'-button-'+actionIndex"
              :class="action.class"
              :style="action.style"
              v-html="action.html"
              @click="action.atClick(content) || null"
            ></button>
          </td>
        </tr>

        <tr v-show="!hasContent">
          <td
            class="py-2"
            :colspan="headers.length + (actions.length > 0 ? 1 : 0)"
          >{{noContentMessage}}</td>
        </tr>
      </tbody>

      <tbody v-else>
        <slot></slot>
      </tbody>
    </table>

    <p class="flex justify-between" v-show="hasContent">
      <small class="block text-left py-2">Showing {{from}} to {{to}} of {{maxIndex}}.</small>
      <Pagination v-model="page" :maxPage="maxPage" />
    </p>
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
    keyPrefix: {
      // Used to prefix all keys if multiple DataTables in the same page
      type: String,
      default:
        String.fromCharCode(Math.floor(Math.random() * 52 + 65)) +
        Math.floor(Math.random() * 1000)
    },
    headers: Array,
    actions: {
      type: Array,
      default: () => []
    },
    actionsAtEnd: Boolean,
    actionsHeader: {
      type: String,
      default: "Actions"
    },
    cellsContent: {
      type: Array,
      default: () => []
    },
    ascSymbol: {
      type: String,
      default: "&#8595;"
    },
    descSymbol: {
      type: String,
      default: "&#8593;"
    },
    neutralSymbol: {
      type: String,
      default: "&#8597;"
    },
    noContentMessage: {
      type: String,
      default: "Il n'y a aucune ligne à afficher."
    },
    hover: Boolean,
    stripe: Boolean
  },
  data() {
    return {
      contents: this.cellsContent,
      realContents: [],
      rows: [],
      page: 1,
      byPage: 10,
      searchString: null,
      orderBy: null,
      asc: true
    };
  },
  computed: {
    dataByProps() {
      return this.cellsContent.length > 0;
    },
    hasContent() {
      return this.realContents.length > 0;
    },
    from() {
      return (this.page - 1) * this.byPage + 1;
    },
    to() {
      return Math.min(this.page * this.byPage, this.realContents.length);
    },
    maxIndex() {
      return this.realContents.length;
    },
    maxPage() {
      return Math.ceil(this.realContents.length / this.byPage);
    }
  },
  methods: {
    createContentArray() {
      for (var header of this.headers) {
        if (!header.linkTo) {
          console.error(
            "`headers` property objects MUST contain a `linkTo` property"
          );
          return;
        }
      }

      this.contents = [];

      var usrTRs = this.$el.querySelectorAll("table > tbody > tr");

      itereTR: usrTRs.forEach((elem, index) => {
        this.rows.push({ index: index, html: elem.outerHTML });

        var usrTDs = this.$el.querySelectorAll(
          "table > tbody > tr:nth-child(" + (index + 1) + ") > td"
        );
        if (!usrTDs.length) return;
        var contentRow = {};

        usrTDs.forEach((e, i) => {
          contentRow[this.headers[i].linkTo] = e.innerText || null; // We must fill empty fields with null values
        });
        this.contents.push(contentRow);
      });
    },
    updateRealContents() {
      this.realContents = this.contents
        .map((e, i) => {
          // Get the orderBy and make an array to walk through with sort
          return { index: i, value: e[this.orderBy], toStr: JSON.stringify(e) };
        })
        .filter(e =>
          this.searchString
            ? // Keep only the ones who contain the searchString if the search field is not empty
              e.toStr.toLowerCase().includes(this.searchString)
            : true
        )
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
        .map(e => this.contents[e.index]);
    },
    changeOrderBy(field) {
      if (this.orderBy == field) this.asc = !this.asc;
      else {
        this.orderBy = field;
        this.asc = true;
      }
      this.updateRealContents();
    },
    symbol(header) {
      if (header == this.orderBy) {
        if (this.asc)
          return '<span class="symbol">' + this.ascSymbol + "</span>";
        else return '<span class="symbol">' + this.descSymbol + "</span>";
      }

      return '<span class="symbol">' + this.neutralSymbol + "</span>";
    }
  },
  watch: {
    byPage: function(newByPage, oldByPage) {
      // Sets the page in order to see the last rows which were printed
      this.page = Math.ceil(((this.page - 1) * oldByPage + 1) / newByPage);
      this.updateRealContents();
    },
    searchString: function(older, newer) {
      this.page = 1;
      this.updateRealContents();
    }
  },
  mounted() {
    if (!this.dataByProps) this.createContentArray();
    this.updateRealContents();
  }
};
</script>
