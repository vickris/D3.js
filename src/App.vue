<template>
  <div id="app">
    <form action="#" @submit.prevent="getIssues">
      <div class="form-group">
        <input type="text" placeholder="Search Repo" v-model="repository" name="skill" class="col-md-2 col-md-offset-5">
      </div>
    </form>

    <div class="my-5">
      <div class="alert alert-info" v-show="loading">
        Loading...
      </div>
      <div class="alert alert-danger" v-show="errored">
        Error occured
      </div>

      <chart :issues="issues"></chart>
    </div>
  </div>
</template>

<script>
// import Chart from "chart.js";

import moment from "moment";
import _ from "lodash";
import axios from "axios";
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";
import Chart from './components/Chart.vue'


export default {
  name: "app",
  components: {
    Chart
  },
  data() {
    return {
      issues: [],
      repository: "",
      days: [],
      startDate: null,
      loading: false,
      errored: false,
    };
  },
  methods: {
    getDates: function(startDate, endDate) {
      var now = startDate.clone(), dates = [];
      while (now.isSameOrBefore(endDate)) {
          dates.push(now.format("MMM Do YY"));
          now.add(1, 'days');
      }
      return dates;
    },
    getIssues: function() {
      this.loading = true;
      this.startDate = moment().subtract(6, 'days').format('YYYY-MM-DD');

      axios
        .get(`https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=${this.startDate}`, {
          params: {
            per_page: 100
          }
        })
        .then(response => {
          this.days = response.data.items.map(issue => {
            return moment(issue.created_at).format("MMM Do YY");
          });

          var issuesPastWeek = {};
          var issuesByDay = _.countBy(this.days)
          var dates = this.getDates(moment().subtract(6, 'days'), moment())

          for (var i = 0; i < dates.length; i++) {
            if(issuesByDay[dates[i]]) {
              issuesPastWeek[dates[i]] = issuesByDay[dates[i]]
              continue
            }

            issuesPastWeek[dates[i]] = 0
          }

          var issues = []
          _.forOwn(issuesPastWeek, function(value, key) {
                issues.push({day: key, issues: value})
            });

          this.issues = issues
        })
        .catch(error => {
          console.error(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

svg {
  min-width: 60%;
  height: 100%;
}
</style>
