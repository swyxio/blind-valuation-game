    // <input type="text" v-model="search" class="form-control" placeholder="Filter field..." />
<template>
  <div id="vue-table">
    <div id="tableanswer" v-if="showanswer & currentstage < 3" v-bind:class="{ alert: true, 'alert-success': Math.abs(Math.round((currentguess / currentprice - 1) * 100)) < 11 , 'alert-danger': Math.abs(Math.round((currentguess / currentprice - 1) * 100)) > 11}">
        Ticker: <a :href="`https://www.google.com/finance?q=${tableticker}`" target="_blank">{{ tableticker.toUpperCase() }}</a> ( {{ currentname }}, ${{ Math.round(market_cap * 100) / 100 }}bn mkt cap ) <br />
        Your guess: {{ currentguess }} vs Current price: {{ currentprice }} (1yr range: {{ range_52 }})<br />
        <span v-if="Math.abs(Math.round((currentguess / currentprice - 1) * 100)) < 11">
          Very close! You were within {{ Math.floor((currentguess / currentprice - 1) * 100) }}%!
        </span>
        <span v-else>
          Oof! You were off by {{ Math.floor((currentguess / currentprice - 1) * 100) }}%!
        </span>
    </div>
    <table class="table table-striped table-hover"> 
      <thead>
        <tr>
          <th>Field</th>
          <th>2010</th>
          <th>2011</th>
          <th>2012</th>
          <th>2013</th>
          <th>2014</th>
          <th>2015</th>
          <th>2016</th>
          <th>2017e</th>
          <th>2018e</th>
          <th>2019e</th>
        </tr>
      </thead>
      <tbody>
          <tr v-for="y in Object.keys(masterFieldList)" v-if="data[y]">
            <th>{{masterFieldList[y]['name']}}</th>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2010)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2010)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2011)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2011)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2012)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2012)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2013)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2013)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2014)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2014)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2015)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2015)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2016)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2016)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2017)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2017)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2018)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2018)}}</span>
            </td>
            <td>{{numFormat(masterFieldList[y]['format'],data[y].data[5],2019)}}<br />
              <span class="yoychg">{{chgFormat(masterFieldList[y]['format'],data[y].data[5],2019)}}</span>
            </td>
          </tr>
      </tbody>
    </table>
    <i>Current share count: {{ shares_count }} million.</i>
  </div>
</template>

<script>
// https://codepen.io/bertog/pen/EjmpXm
// https://vuejs.org/v2/guide/migration.html#Filters
import _ from 'lodash';
import axios from 'axios';
import Vue from 'vue';
import VueAxios from 'vue-axios';

Vue.use(VueAxios, axios);
const REST_API = 'https://app.sentieo.com/api/fetch_value_table/?ticker=';
const APIKEY = '&apikey=37c5d1eb66f3c98d7da2d742743e00bebdf1232abbcffe7decb107c5f8d0b2d6';
const masterFieldList = {
  TotalRevenue: { format: '0dp', name: 'Sales' },
  GrossMargin: { format: 'pct', name: 'Gross Margin' },
  Ebitda: { format: '0dp', name: 'EBITDA' },
  // 'Ebit: { format: '0dp', name: 'EBIT'},
  PurchaseOfPropertyPlantAndEquipment: { format: 'capex', name: 'Capex' },
  NetIncome: { format: '0dp', name: 'Net Income' },
  DilutedEPSTotal: { format: '2dp', name: 'Diluted EPS' },
  DividendsPaidPerShare: { format: '2dp', name: 'DPS' },
  FreeCashFlowPerShare: { format: '2dp', name: 'FCF/Share' },
};
export default {
  name: 'vuetable',
  // el: '#vue-table',
  props: ['tableticker', 'showanswer', 'currentguess', 'currentstage'],
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
      data: {},
      currentprice: 0,
      currentname: '',
      market_cap: 0,
      range_52: '',
      shares_count: 0,
      masterFieldList,
      errors: [],
    };
  },
  // Fetches posts when the component is created.
  created() {
    Object.keys(masterFieldList).forEach((datatype) => {
      axios.get(`${REST_API}${this.tableticker}&type=${datatype}${APIKEY}`)
      .then((response) => {
        this.data[datatype] = response.data.result;
        this.data = JSON.parse(JSON.stringify(this.data));
      })
      .catch((e) => {
        this.errors.push(e);
      });
    });
    axios.get(`https://app.sentieo.com/api/mobile_stock_data/?ticker=${this.tableticker}${APIKEY}`)
    .then((response) => {
      this.currentprice = response.data.result.intraday.current_price;
      this.currentname = response.data.result.intraday.name;
      this.market_cap = Number(response.data.result.intraday.market_cap) / 1000;
      this.range_52 = response.data.result.intraday.range_52;
      this.shares_count = Number(response.data.result.intraday.shares);
    })
    .catch((e) => {
      this.errors.push(e);
    });
  },
  watch: {
    tableticker: function ttfunc(newtt) {
      Object.keys(masterFieldList).forEach((datatype) => {
        axios.get(`${REST_API}${newtt}&type=${datatype}${APIKEY}`)
        .then((response) => {
          this.data[datatype] = response.data.result;
          this.data = JSON.parse(JSON.stringify(this.data));
        })
        .catch((e) => {
          this.errors.push(e);
        });
      });
      axios.get(`https://app.sentieo.com/api/mobile_stock_data/?ticker=${newtt}${APIKEY}`)
      .then((response) => {
        this.currentprice = response.data.result.intraday.current_price;
        this.currentname = response.data.result.intraday.name;
        this.market_cap = Number(response.data.result.intraday.market_cap) / 1000;
        this.range_52 = response.data.result.intraday.range_52;
        this.shares_count = Number(response.data.result.intraday.shares);
      })
      .catch((e) => {
        this.errors.push(e);
      });
    },
  },
  methods: {
    sortBy: function y(sortKey) {
      this.reverse = (this.sortKey === sortKey) ? !this.reverse : false;
      this.sortKey = sortKey;
    },
    chgFormat(format, database, yr) {
      if (!database) return '';
      const yr1 = yr.toString();
      const yr0 = (yr - 1).toString();
      let num = database[yr1];
      let chg = database[yr0];
      if (typeof num === 'string') num = Number(num.split('#')[0]);
      if (typeof chg === 'string') chg = Number(chg.split('#')[0]);
      chg = ((num / chg) - 1) * 100;
      if (Number.isNaN(chg)) {
        chg = '';
      } else {
        chg = `${chg > 0 ? '+' : ''}${(chg).toFixed(1)}%`;
      }

      switch (format) {
        case '0dp':
          return chg;
        case '2dp':
          return chg;
        case 'pct':
          return '';
        case 'capex':
          return chg;
        default:
          return '';
      }
    },
    numFormat(format, database, yr) {
      if (!database) return '';
      const yr1 = yr.toString();
      let num = database[yr1];
      if (typeof num === 'string') num = Number(num.split('#')[0]);
      switch (format) {
        case '0dp':
          return Math.round(num).toLocaleString('en-US', { minimumFractionDigits: 0 });
        case '2dp':
          return num.toFixed(2);
        case 'pct':
          return `${num.toFixed(1)}%`;
        case 'capex':
          return Math.round(-num).toLocaleString('en-US', { minimumFractionDigits: 0 });
        default:
          return num.toLocaleString('en-US', { minimumFractionDigits: 2 });
      }
    },
  },
};


</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
#tableanswer {
  font-size: 1.25em;
  color: red;
  text-align: center;
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
.yoychg {
  font-size: 0.8em;
  font-style: italic;
}
.table-hover tbody tr:hover td, .table-hover tbody tr:hover th {
  background-color: #99f;
}
</style>
