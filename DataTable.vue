<style lang="postcss" scoped>
.dt > p > select,
.dt > p > input {
  height: 2.25rem;
  font-size: 1rem;
  vertical-align: middle;
  border: 1px solid #cbd5e0;
  border-radius: 5px;
  padding: 0.5rem;
  width: auto;
}

.dt table {
  width: 100%;
  margin-top: 1rem;
  margin-bottom: 1rem;
  table-layout: auto;
}

.dt table thead tr th.orderable {
  cursor: pointer;
}

.dt table.hover tbody tr:hover {
  background-color: #f3f3f3;
}

.dt table.stripe tbody tr:nth-of-type(even) {
  background-color: #f6f6f6;
}

/* For compatibility without Tailwind CSS */
.dt p.flex small {
  display: block;
  text-align: left;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  width: auto;
}

.dt .flex {
  display: flex;
  justify-content: space-between;
}
</style>

<template>
  <div class="dt">
    <p class="flex">
      <select name="by-page" v-model="byPage">
        <option value="10">10</option>
        <option value="25">25</option>
        <option value="50">50</option>
        <option value="100">100</option>
      </select>

      <input type="search" name="search" v-model="searchString" placeholder="Search" />
    </p>

    <table :class="[(hover ? 'hover' : null), (stripe ? 'stripe' : null)]">
      <thead>
        <slot name="thead">
          <tr>
            <template v-for="(header, index) in headers">
              <th
                v-if="header.linkTo"
                :key="keyPrefix+'-'+header.linkTo+'-header'"
                :id="header.linkTo+'-header'"
                class="orderable"
                v-html="header.html+'&nbsp;'+symbol(header.linkTo)"
                @click="changeOrderBy(header.linkTo)"
              ></th>
              <th v-else v-html="header.html" :key="keyPrefix+'-header-'+index"></th>
            </template>
          </tr>
        </slot>
      </thead>

      <tbody>
        <slot>
          <tr v-for="(c, index) in content" :key="keyPrefix+'-'+index">
            <td
              v-for="(header, headerIndex) in headers"
              :key="keyPrefix+'-'+index+'-'+headerIndex"
              :headers="header.linkTo"
              v-html="c[header.linkTo]"
            ></td>
          </tr>
        </slot>

        <tr v-show="!hasContent">
          <td
            style="padding-top:.5rem;padding-bottom:.5rem"
            :colspan="headers.length"
          >{{noContentMessage}}</td>
        </tr>
      </tbody>
    </table>

    <p class="flex" v-show="hasContent">
      <small>Showing {{from}} to {{to}} of {{maxIndex}}.</small>
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
  model: {
    event: "change",
    prop: "content"
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
    content: {
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
      initialContent: this.content || [],
      page: 1,
      byPage: 10,
      searchString: null,
      orderBy: null,
      asc: true,
      maxIndex: 0
    };
  },
  computed: {
    realContent() {
      if (this.initialContent.length > 0) {
        var rc = this.initialContent
          .map((e, i) => {
            // Get the orderBy and make an array to walk through with sort
            return {
              index: i,
              value: e[this.orderBy],
              toStr: JSON.stringify(e)
            };
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
          .map(e => this.initialContent[e.index]);

        this.setMaxIndex(rc.length);

        return rc.slice(this.from - 1, this.to);
      } else return null;
    },
    hasContent() {
      return this.content != null && this.content.length > 0;
    },
    from() {
      return (this.page - 1) * this.byPage + 1;
    },
    to() {
      return Math.min(this.page * this.byPage, this.initialContent.length);
    },
    maxPage() {
      return Math.ceil(this.maxIndex / this.byPage);
    }
  },
  methods: {
    changeOrderBy(field) {
      if (field == null) return;
      if (this.orderBy == field) this.asc = !this.asc;
      else {
        this.orderBy = field;
        this.asc = true;
      }
      this.$emit("change", this.realContent);
    },
    symbol(header) {
      if (header == this.orderBy) {
        if (this.asc)
          return '<span class="symbol">' + this.ascSymbol + "</span>";
        else return '<span class="symbol">' + this.descSymbol + "</span>";
      }

      return '<span class="symbol">' + this.neutralSymbol + "</span>";
    },
    setMaxIndex(n) {
      this.maxIndex = n;
    }
  },
  watch: {
    // page: function() {
    //   this.$emit("change", this.realContent);
    // },
    byPage: function(newByPage, oldByPage) {
      // Sets the page in order to see the last rows which were printed
      this.page = Math.ceil(((this.page - 1) * oldByPage + 1) / newByPage);
      // this.$emit("change", this.realContent);
    },
    searchString: function() {
      this.page = 1;
      // this.$emit("change", this.realContent);
    },
    realContent: function() {
      this.$emit("change", this.realContent);
    }
  },
  mounted() {
    this.$emit("change", this.realContent);
  }
};
</script>
