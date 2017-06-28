<template>
  <div class="container">
    <div id="vue-table">
      <input type="text" v-model="search" class="form-control" placeholder="Search..." />
      <table class="table table-striped"> 
        <thead>
            <tr>
            <th v-for="column in columns">
                <a href="#" 
                v-on:click.prevent="sortBy(column)"
                v-bind:class="{ active: sortKey == column }"
                >
                {{ column.toUpperCase() }}
                </a>
            </th>
            </tr>
        </thead>
        
        <tbody>
            <tr v-for="person in filterpeople">
              <td>{{ person.name }}</td>
              <td>{{ person.age }}</td>
            </tr>
        </tbody>
      </table>  
    </div>
  </div>
</template>

<script>
// https://codepen.io/bertog/pen/EjmpXm
// https://vuejs.org/v2/guide/migration.html#Filters
import _ from 'lodash';
import axios from 'axios';
const REST_API = 'https://app.sentieo.com/api/fetch_value_table/?ticker=amzn';
const APIKEY='&apikey=73e127d36e46d3ef6e13c34da5ac3bb12558f657d4f67b2679a282b545e5f315';
const masterFieldList= {
  'TotalRevenue':{format:"0dp",name:"Sales"},
  'GrossMargin':{format:"pct",name:"Gross Margin"},
  'Ebitda':{format:"0dp",name:"EBITDA"},
  // 'Ebit':{format:"0dp",name:"EBIT"},
  'PurchaseOfPropertyPlantAndEquipment':{format:"capex",name:"Capex"},
  'NetIncome':{format:"0dp",name:"Net Income"},
  'DilutedEPSTotal':{format:"2dp",name:"Diluted EPS"},
  'DividendsPaidPerShare':{format:"2dp",name:"DPS"},
  'FreeCashFlowPerShare':{format:"2dp",name:"FCF/Share"},
};
let numFormat = (format,database,yr) => {
  let yr1 = yr.toString(),
      num = database[yr1]
  if (typeof num === "string") num = Number(num.split('#')[0])
  switch(format) {
    case "0dp":
      return Math.round(num).toLocaleString('en-US', {minimumFractionDigits: 0})
    case "2dp":
      return num.toFixed(2)
    case "pct":
      return num.toFixed(1) + "%"
    case "capex":
      return Math.round(-num).toLocaleString('en-US', {minimumFractionDigits: 0})
    default: 
      return num.toLocaleString('en-US', {minimumFractionDigits: 2});
               }
}
let chgFormat = (format,database,yr) => {
  let yr1 = yr.toString(), 
      yr0 = (yr-1).toString(),
      num = database[yr1],
      chg = database[yr0]
  if (typeof num === "string") num = Number(num.split('#')[0])
  if (typeof chg === "string") chg = Number(chg.split('#')[0])
  chg = (num/chg - 1) * 100
  if (chg != chg) {
    chg = ""
  } else {
    chg = "(" + (chg > 0 ? "+" : "") + (chg).toFixed(1) + "%)"    
  }

  switch(format) {
    case "0dp":
      return chg
    case "2dp":
      return chg
    case "pct":
      return ""
    case "capex":
      return chg
    default: 
      return ""
               }
}



export default {
  name: 'vuetable',
  // el: '#vue-table',
  computed: {
    filterpeople: function x() {
      const self = this;
      const filtered = self.people.filter(user =>
        user.name.toUpperCase().indexOf(self.search.toUpperCase()) !== -1,
      );
      const sorted = _.orderBy(filtered, self.sortKey, self.reverse ? 'asc' : 'desc');
      return sorted;
    },
  },
  data() {
    return {
      sortKey: '',
      search: '',
      reverse: false,
      columns: ['name', 'age'],
      people: [
        { name: 'John', age: 50 },
        { name: 'Jack', age: 35 },
        { name: 'Keith', age: 28 },
        { name: 'Alain', age: 17 },
        { name: 'Neil', age: 1 },
        { name: 'Mark', age: 72 },
        { name: 'Don', age: 47 },
        { name: 'Walter', age: 41 },
        { name: 'Jessy', age: 33 },
        { name: 'Henck', age: 22 },
        { name: 'Sal', age: 9 },
        { name: 'Skyler', age: 42 },
        { name: 'Holly', age: 55 },
      ],
    };
  },

  methods: {
    sortBy: function y(sortKey) {
      this.reverse = (this.sortKey === sortKey) ? !this.reverse : false;
      this.sortKey = sortKey;
    },
  },
};


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
#vue-table {
  margin: 2em 0;
}
a {
  font-weight: bold;
  text-decoration: none;  
}
active {
  font-weight: bold;
  color: black;
  text-decoration: underline;
}
</style>
