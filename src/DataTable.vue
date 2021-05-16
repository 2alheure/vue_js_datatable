<style lang="postcss" scoped>
.dt > form > select,
.dt > form > input {
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

.dt form small {
  display: block;
  text-align: left;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  width: auto;
}

.dt form {
  display: flex;
  justify-content: space-between;
}
</style>

<template>
  <div class="dt" :id="identifier">
    <form>
      <select name="by-page" v-model="byPage">
        <option
          v-for="opt in byPageOptions"
          :value="opt"
          :key="identifier + '-' + opt + 'by-page'"
        >
          {{ opt }}
        </option>
      </select>

      <input
        type="search"
        name="search"
        v-model="searchString"
        :placeholder="translate.search"
      />
    </form>

    <table :class="[hover ? 'hover' : null, stripe ? 'stripe' : null]">
      <thead>
        <slot name="thead">
          <tr>
            <template v-for="(header, index) in headers">
              <th
                v-if="header.linkTo"
                :key="identifier + '-' + header.linkTo + '-header'"
                :id="header.linkTo + '-header'"
                class="orderable"
                v-html="header.html + '&nbsp;' + symbol(header.linkTo)"
                @click="changeOrderBy(header.linkTo)"
              ></th>
              <th
                v-else
                v-html="header.html"
                :key="identifier + '-header-' + index"
              ></th>
            </template>
          </tr>
        </slot>
      </thead>

      <tbody>
        <slot>
          <tr v-for="(c, index) in content" :key="identifier + '-' + index">
            <td
              v-for="(header, headerIndex) in headers"
              :key="identifier + '-' + index + '-' + headerIndex"
              :headers="header.linkTo"
              v-html="resolve(c, header.linkTo)"
            ></td>
          </tr>
        </slot>

        <tr v-show="!hasContent">
          <td
            style="padding-top: 0.5rem; padding-bottom: 0.5rem"
            :colspan="headers.length"
          >
            {{ translate.noContent }}
          </td>
        </tr>
      </tbody>
    </table>

    <form v-show="hasContent && maxPage > 1">
      <small>{{ sprintf(translate.showing, from, to, maxIndex) }}</small>
      <Pagination v-model="page" :maxPage="maxPage" v-bind="paginationProps" />
    </form>
  </div>
</template>
		
<script>
import Pagination from "@2alheure/vue-pagination";
import translations from "./translations.json";

export default {
  name: "DataTable",
  components: {
    Pagination,
  },
  model: {
    event: "change",
    prop: "content",
  },
  props: {
    identifier: {
      // Used to prefix all keys if multiple DataTables in the same page
      type: String,
      default:
        String.fromCharCode(Math.floor(Math.random() * 52 + 65)) +
        Math.floor(Math.random() * 1000),
    },
    headers: Array,
    content: {
      type: Array,
      default: () => [],
    },
    ascSymbol: {
      type: String,
      default: "&#8595;",
    },
    descSymbol: {
      type: String,
      default: "&#8593;",
    },
    neutralSymbol: {
      type: String,
      default: "&#8597;",
    },
    byPageOptions: {
      type: Array,
      default: () => [10, 25, 50, 100],
      validator: (arr) => arr.join("").match(/\d+/),
    },
    paginationProps: Object,
    hover: Boolean,
    stripe: Boolean,
    translation: {
      type: [String, Object],
      default: () => translations.en,
      validator: (value) => {
        if (
          (typeof value == "string" && translations[value] !== undefined) ||
          (value instanceof Object &&
            JSON.stringify(Object.keys(value).sort()) ==
              JSON.stringify(Object.keys(translations.en).sort()))
        )
          return true;

        console.error(
          '`translation` prop must be an Object with keys: "' +
            Object.keys(translations.fr).join('", "') +
            '" or one String in: "' +
            Object.keys(translations).join('", "') +
            '". Received:',
          value
        );
        return false;
      },
    },
  },
  data() {
    return {
      initialContent: this.content || [],
      page: 1,
      byPage: 10,
      searchString: null,
      orderBy: null,
      asc: true,
      maxIndex: 0,
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
              value: this.resolve(e, this.orderBy),
              toStr: JSON.stringify(e),
            };
          })
          .filter((e) =>
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
          .map((e) => this.initialContent[e.index]);

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
    },
    translate() {
      if (typeof this.translation == "string")
        return translations[this.translation];
      else return this.translation;
    },
  },
  methods: {
    resolve(object, path, defaultValue) {
      if (path === undefined || path === null || !path.length) return null;
      if (object === undefined || object === null) return null;

      defaultValue = defaultValue || null;

      return path
        .split(".")
        .reduce((o, p) => (o ? o[p] : defaultValue), object);
    },
    sprintf(string) {
      if (typeof string == "string" && string.length) {
        var i = arguments.length;

        while (--i) {
          string = string.replace(
            new RegExp("\\{" + (i - 1) + "\\}", "gm"),
            arguments[i]
          );
        }
      }

      return string;
    },
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
    },
  },
  watch: {
    // page: function() {
    //   this.$emit("change", this.realContent);
    // },
    byPage: function (newByPage, oldByPage) {
      // Sets the page in order to see the last rows which were printed
      this.page = Math.ceil(((this.page - 1) * oldByPage + 1) / newByPage);
      // this.$emit("change", this.realContent);
    },
    searchString: function () {
      this.page = 1;
      // this.$emit("change", this.realContent);
    },
    realContent: function () {
      this.$emit("change", this.realContent);
    },
  },
  mounted() {
    this.$emit("change", this.realContent);
  },
};
</script>
