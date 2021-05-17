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

.dt table.hover tbody tr:hover {
  background-color: #f3f3f3;
}

.dt table.stripe tbody tr:nth-of-type(even) {
  background-color: #f6f6f6;
}

.dt table thead [data-order-by] {
  cursor: pointer;
}

.dt {
  overflow-y: auto;
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
  flex-wrap: wrap;
  flex-shrink: 0;
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
      <thead :id="identifier + '-thead'">
        <slot name="thead">
          <tr>
            <template v-for="(header, index) in headers">
              <th
                v-if="header.linkTo"
                :data-order-by="header.linkTo"
                :key="identifier + '-' + header.linkTo + '-header'"
                :id="identifier + '-' + header.linkTo + '-header'"
                v-html="header.html"
              ></th>
              <th
                v-else
                v-html="header.html"
                :key="identifier + '-' + index + 'header'"
                :id="identifier + '-' + index + 'header'"
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
              :headers="
                identifier + '-' + (header.linkTo || headerIndex) + '-header'
              "
              v-html="_get(c, header.linkTo || headerIndex)"
            ></td>
          </tr>
        </slot>

        <tr v-show="!hasContent">
          <td
            style="padding-top: 0.5rem; padding-bottom: 0.5rem"
            :colspan="headersLength"
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
import { get as _get, random as _random } from "lodash";

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
      default: function () {
        var minuscule = _random();
        if (minuscule)
          return (
            String.fromCharCode(_random(97, 122)) +
            Math.floor(Math.random() * 1000)
          );
        return (
          String.fromCharCode(_random(65, 90)) +
          Math.floor(Math.random() * 1000)
        );
      },
    },
    headers: {
      // If given, represents the headers for the table
      // If not given, calculated in mounted()
      type: Array,
      default: () => [],
    },
    content: {
      // Represents the data to put in the table
      // And to calculate on
      type: Array,
      default: () => [],
    },
    ascSymbol: {
      // The symbol when column is ordered asc
      type: String,
      default: "&#8595;",
    },
    descSymbol: {
      // The symbol when column is ordered desc
      type: String,
      default: "&#8593;",
    },
    neutralSymbol: {
      // The symbol when column is not ordered
      type: String,
      default: "&#8597;",
    },
    byPageOptions: {
      // The possible values for byPage
      type: Array,
      default: () => [10, 25, 50, 100],
      validator: (arr) => arr.join("").match(/\d+/),
    },
    paginationProps: Object, // The props of the Pagination component used
    hover: Boolean, // Whether this table should have the "hover" style
    stripe: Boolean, // Whether this table should have the "stripe" style
    translation: {
      // The language to use
      // Or the sentences to use
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
      initialContent: this.content || [], // A copy of the content, on which we can work
      page: 1, // The actual page
      byPage: this.byPageOptions[0], // The actual number of rows per page
      searchString: null, // The actual search string
      orderBy: null, // The actual column the order is done by
      asc: true, // Whether the order is ascending or descending
      maxIndex: 0, // The calculated number of rows to display in the table
      headersLength: 0,
    };
  },
  computed: {
    /**
     * Calculates the rows which will be printed
     */
    realContent() {
      if (this.initialContent.length > 0) {
        var rc = this.initialContent
          .map((e, i) => {
            // Get the orderBy and make an array to walk through with next functions
            return {
              index: i,
              value: _get(e, this.orderBy),
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
    /**
     * Returns whether the table has content or not
     */
    hasContent() {
      return this.content != null && this.content.length > 0;
    },
    /**
     * The index of the beginning row to print
     */
    from() {
      return (this.page - 1) * this.byPage + 1;
    },
    /**
     * The index of the ending row to print
     */
    to() {
      return Math.min(this.page * this.byPage, this.initialContent.length);
    },
    /**
     * The maximum value for page
     */
    maxPage() {
      return Math.ceil(this.maxIndex / this.byPage);
    },
    /**
     * The translation objet
     *
     * Can be the user given one
     * Or one of the component's default ones
     */
    translate() {
      if (typeof this.translation == "string")
        return translations[this.translation];
      else return this.translation;
    },
  },
  methods: {
    /**
     * Lodash _.get method
     */
    _get,
    /**
     * Returns a formatted string
     * Containing all the given arguments
     * Placed where placeholders stand
     */
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
    /**
     * Handles the change for orderBy
     */
    changeOrderBy(field) {
      if (field == null) return;
      if (this.orderBy == field) this.asc = !this.asc;
      else {
        this.orderBy = field;
        this.asc = true;
      }
      this.$emit("change", this.realContent);
    },
    /**
     * maxIndex setter
     */
    setMaxIndex(n) {
      this.maxIndex = n;
    },
    /**
     * Handler for the order by event
     */
    handleOrderable(event) {
      document
        .getElementById(this.identifier)
        .querySelectorAll("span.symbol")
        .forEach((el) => {
          el.innerHTML = this.neutralSymbol;
        });

      this.changeOrderBy(event.currentTarget.dataset.orderBy);

      document.querySelector(
        "#" +
          this.identifier +
          " [data-order-by=" +
          event.currentTarget.dataset.orderBy +
          "] .symbol"
      ).innerHTML = this.asc ? this.ascSymbol : this.descSymbol;
    },
  },
  watch: {
    byPage: function (newByPage, oldByPage) {
      // Sets the page in order to see the last rows which were printed
      this.page = Math.ceil(((this.page - 1) * oldByPage + 1) / newByPage);
    },
    searchString: function () {
      // Sets the page in order to correctly print the search results
      this.page = 1;
    },
    realContent: function () {
      // Automatic event trigger
      this.$emit("change", this.realContent);
    },
  },
  mounted() {
    // Calculating the headers size
    // In mounted because we use
    // the document object reference
    this.headersLength =
      this.headers.length ||
      document.getElementById(this.identifier + "-thead").firstElementChild
        .childElementCount;

    // Handling orderable columns
    document
      .getElementById(this.identifier)
      .querySelectorAll("[data-order-by]")
      .forEach((el) => {
        el.addEventListener("click", this.handleOrderable);
        el.innerHTML +=
          '<span class="symbol">' + this.neutralSymbol + "</span>";
      });

    // First calculation of realContent
    this.$emit("change", this.realContent);
  },
};
</script>
