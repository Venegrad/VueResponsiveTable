<template>
  <div id="app">
    <ven-table :data="tdata" :search="true" title="Table Example"></ven-table>
  </div>
</template>

<script>
import venTable from './components/venTable'

function formatDate(date) {
    var d = new Date(date),
        month = '' + (d.getMonth() + 1),
        day = '' + d.getDate(),
        year = d.getFullYear(),
        minutes = d.getMinutes(),
        hour = d.getHours();

    if (month.length < 2) 
        month = '0' + month;
    if (day.length < 2) 
        day = '0' + day;

    return [day, month, year].join('.') + " - " + hour +":"+minutes;
}

export default {
  data() {
    return {
      tdata: null
    }
  },
  name: 'App',
  components: {
    venTable
  },

  mounted() {
    const json = require("@/assets/1.json");
    const headers = [];
    const rows = [];
    const keys = Object.keys(json.transactions[0].fields);

    keys.forEach(h => {
      const getth = json.transactions[0].fields;
      headers.push(getth[h]);
    }) 

    json.transactions.forEach(tr => {
      const result = []
      keys.forEach(td => {

        let cls = "";
        let data = tr[td];
        if(['hash', 'time'].includes(td)) cls += "nowrap ";
        if(['time'].includes(td)) {
          cls += "nowrap ";
          data = formatDate(tr[td])
        }

        if(td === 'hash') data = `<a href='/' class="link-blue">${tr[td]}</a>`;

        result.push({
          data,
          cls
        })
      });
      rows.push(result);      
    })

    this.tdata = {
      headers,
      rows
    }
  }
}
</script>

<style>

body {
  overflow-y: scroll;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  /* max-width: 1000px; */
  margin-left: auto;
  margin-right: auto;
}

.link-blue {
  color: #3498db;
}
</style>
