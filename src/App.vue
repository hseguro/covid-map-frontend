<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <h2>COVID19</h2>
      </div>
    </v-app-bar>
    <v-content class="ma-12">
      <div>
        <p style="font-size: 30px;">
          <strong>DISCLAIMER:</strong>
          this data may be wrong do not take this for granted.
        </p>
      </div>
      <div v-if="global !== null" style="display: flex;">
        <div class="pl-6 pt-6">
          <p>Total confirmed: {{ global.total.confirmed.toLocaleString() }}</p>
          <p>Total recovered: {{ global.total.recovered.toLocaleString() }}</p>
          <p>Total deaths: {{ global.total.deaths.toLocaleString() }}</p>
        </div>
        <div class="pa-6">
          <p>Global mortality rate: {{ ((global.total.deaths * 100) / global.total.confirmed).toFixed(2) }}%</p>
          <p>Global recuperation rate: {{ ((global.total.recovered * 100) / global.total.confirmed).toFixed(2) }}%</p>
        </div>
      </div>
      <div v-if="countries !== null">
        <div class="pt-6 pb-6">
          <v-text-field v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details>
          </v-text-field>
        </div>
        <v-data-table :headers="headers" :items="countries" :search="search" :multi-sort="true">
          <template v-slot:body="{ items }">
            <tbody>
              <tr v-for="item in items" :key="item.name">
                <td>{{ item.slug }}</td>
                <td>{{ item.name }}</td>
                <td>{{ item.population.toLocaleString() }}</td>
                <td>{{ item.population_updated }}</td>
                <td>{{ item.total.confirmed.toLocaleString() }}</td>
                <td>{{ item.total.recovered.toLocaleString() }}</td>
                <td>{{ item.total.deaths.toLocaleString() }}</td>
                <td>{{ item.activeCases.toLocaleString() }}</td>
                <td>{{ item.percentageInfected.toFixed(5) }}%</td>
                <td>{{ item.percentageRecovered.toFixed(2) }}%</td>
                <td>{{ item.mortalityConfirmed.toFixed(2) }}%</td>
                <td>{{ item.mortalityPopulation.toFixed(2) }}%</td>
              </tr>
            </tbody>
          </template>
        </v-data-table>
      </div>
      <v-dialog v-model="dialog" width="500">
        <v-card>
          <v-card-title class="headline grey lighten-2" primary-title>
            Error loading
          </v-card-title>
          <v-card-text>
            :C
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="dialog = false">
              OK
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-content>
  </v-app>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'
import Axios from 'axios'

@Component({
  data() {
    return {
      search: '',
      dialog: false,
      global: null,
      countries: null,
      headers: [
        {
          text: 'Code',
          value: 'slug',
        },
        {
          text: 'Name',
          value: 'name',
        },
        {
          text: 'Population (approximation)',
          value: 'population',
        },
        {
          text: 'Population year',
          value: 'population_updated',
        },
        {
          text: 'Confirmed',
          value: 'total.confirmed',
        },
        {
          text: 'Recovered',
          value: 'total.recovered',
        },
        {
          text: 'Deaths',
          value: 'total.deaths',
        },
        {
          text: 'Active cases "confirmed - (recovered + deaths)"',
          value: 'activeCases',
        },
        {
          text: 'Total population infected',
          value: 'percentageInfected',
        },
        {
          text: 'Cases recovered',
          value: 'percentageRecovered',
        },
        {
          text: 'Mortality rate / cases',
          value: 'mortalityConfirmed',
        },
        {
          text: 'Mortality rate / population',
          value: 'mortalityPopulation',
        },
      ],
    }
  },
  mounted() {
    Axios.get('https://5edb0b2c320116001cbc470a-hsegura.esrever.dev/').then(res => {
      this.$data.global = res.data.global
      // @ts-ignore
      this.$data.countries = this.getStatitics(res.data.countries)
    }).catch(error => {
      this.$data.dialog = true
    })
  },
  methods: {
    getStatitics(countries: any[]) {
      return countries
        .filter((country) => country.population !== undefined && country.population !== null)
        .filter((country) => country.total !== undefined && country.total !== null)
        .filter((country) => country.new !== undefined && country.new !== null)
        .map((country) => ({ ...country, activeCases: (country.total.confirmed - (country.total.recovered + country.total.deaths)) }))
        .map((country) => ({ ...country, percentageInfected: (country.total.confirmed * 100) / country.population }))
        .map((country) => ({ ...country, percentageRecovered: (country.total.recovered * 100) / country.total.confirmed }))
        .map((country) => ({ ...country, mortalityConfirmed: (country.total.deaths * 100) / country.total.confirmed }))
        .map((country) => ({ ...country, mortalityPopulation: (country.total.deaths * 100) / country.population }))
    }
  },
})
export default class App extends Vue {}
</script>
