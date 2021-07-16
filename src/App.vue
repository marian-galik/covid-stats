<template>
  <div id="app">
    <Header />
    <div class="container">
      <div v-if="loading"
        class="text-center">
        <b-spinner />
      </div>

      <div v-else>
        <b-form-fieldset v-show="!chartVisible">
          <vue-bootstrap-typeahead :minMatchingChars="1"
            placeholder="Filter by country"
            v-model="filterQuery"
            :data="countries"
            @hit="selectedCountry = $event"
            @input="onQueryChange" />
        </b-form-fieldset>

        <div v-if="chartVisible">
          <div class="row">
            <div class="col-sm">
              <h2>{{chartData.Country}}</h2>
            </div>
            <div class="col-sm text-sm-right">
              <b-button @click="chartVisible = false"
                variant="outline-secondary"
                size="sm">
                Go back
              </b-button>
            </div>
          </div>

          <Chart :data="chartData" />
        </div>

        <div v-else>
          <b-table responsive
            striped
            bordered
            show-empty
            :items="filteredItems"
            :fields="fields"
            :sort-by="sortBy"
            :sort-desc="sortDesc">
            <template #empty>
              <div class="text-center">
                No data for current filter
              </div>
            </template>

            <template #cell(action)="data">
              <b-button @click="showChart(data.item)"
                size="sm"
                variant="link">
                <b-icon-bar-chart-line />
              </b-button>
            </template>

            <template #custom-foot>
              <b-tr>
                <b-th>Total</b-th>
                <b-th>{{sumBy(filteredItems, 'TotalConfirmed' )}}</b-th>
                <b-th>{{sumBy(filteredItems, 'TotalDeaths' )}}</b-th>
                <b-th>{{sumBy(filteredItems, 'TotalRecovered' )}}</b-th>
                <b-th></b-th>
              </b-tr>
            </template>
          </b-table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "./components/Header.vue";
import Chart from "./components/Chart.vue";
import VueBootstrapTypeahead from "vue-bootstrap-typeahead";
import { sumBy } from "lodash";
import { BIconBarChartLine } from "bootstrap-vue";

export default {
  name: "App",
  components: {
    Header,
    VueBootstrapTypeahead,
    BIconBarChartLine,
    Chart,
  },
  data() {
    return {
      loading: true,
      sortBy: "Country",
      sortDesc: false,
      filterQuery: "",
      selectedCountry: "",
      chartVisible: false,
      chartData: null,
      items: [],
      filteredItems: [],
      fields: [
        {
          key: "Country",
          sortable: true,
        },
        {
          key: "TotalConfirmed",
          sortable: true,
        },
        {
          key: "TotalDeaths",
          sortable: true,
        },
        {
          key: "TotalRecovered",
          sortable: true,
        },
        {
          key: "action",
          label: "Action",
        },
      ],
    };
  },
  computed: {
    // get countries array (for autocomplete)
    countries() {
      return this.items.map((item) => item.Country);
    },
  },
  watch: {
    // on autocomplete select country
    selectedCountry(country) {
      // if empty - reset filtered items
      if (country === "") {
        this.filteredItems = this.items;
        return;
      }

      // filter items by selected country
      this.filteredItems = this.items.filter(
        (item) => item.Country === country
      );
    },
  },

  methods: {
    sumBy,
    /**
     * Fetch summary data from API
     */
    async fetchData() {
      const response = await fetch("https://api.covid19api.com/summary");
      const data = await response.json();
      return data;
    },
    /**
     * On country search input change
     */
    onQueryChange(query) {
      if (query === "" && this.selectedCountry) {
        // if empty query => reset filteredItems and selected country
        this.selectedCountry = "";
        return;
      }

      // fulltext filter items
      this.filteredItems = this.items.filter(
        (item) => item.Country.toLowerCase().indexOf(query.toLowerCase()) !== -1
      );
    },
    /**
     * Visibility of chart
     */
    showChart(item) {
      this.chartData = item;
      this.chartVisible = true;
    },
  },
  async created() {
    this.loading = true;
    const { Countries } = await this.fetchData();
    this.loading = false;
    this.items = Countries;
    this.filteredItems = Countries;
  },
};
</script>

<style>
#app {
  margin-top: 60px;
  margin-bottom: 60px;
}
</style>
