<template>
  <div id="app">
    <form action="#" @submit.prevent="getIssues">
      <div class="form-group">
        <input type="text" placeholder="Search Repo" v-model="repository" name="skill" class="col-md-4 col-md-offset-4">
      </div>
    </form>

    <div class="my-5">
      <div class="alert alert-info" v-show="loading">
        Loading...
      </div>
      <div class="alert alert-danger" v-show="errored">
        Error occured
      </div>

      <div v-show="chart !== null">
        <svg />
      </div>
    </div>
  </div>
</template>
<script src="https://d3js.org/d3.v5.min.js"></script>
<script>
// import Chart from "chart.js";
import * as d3 from 'd3';
import moment from "moment";
import axios from "axios";
import _ from "lodash";
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";

export default {
  name: "app",
  data() {
    return {
      chart: null,
      repository: "",
      days: [],
      issues: [],
      issuesPastWeek: [],
      dates: [],
      loading: false,
      errored: false,
      page: 1
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

      axios
        .get(`https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=2018-11-18`, {
          params: {
            per_page: 100
          },
          auth: {
            username: "vickris",
            password: "a3dcec8a3e719cce058feaf8b6657c620931fcdf"
          }
        })
        .then(response => {
          if(response.headers.link) {
            let last_page = response.headers.link
              .split(";")[1]
              .split("page=")[1]
              .split("")[0];
            const promises = [];

            for (let page = 1; page <= last_page; page++) {
              promises.push(this.getIssuesPerPage(page));
            }

            return Promise.all(promises);
          }
          return [response];
        })
        .then(responses => {
          const all_issues = responses.reduce((acc, res) => {
            acc = acc.concat(res.data.items);
            return acc;
          }, []);

          this.days = all_issues.map(issue => {
            return moment(issue.created_at).format("MMM Do YY");
          });

          if (this.chart != null) {
            this.chart.remove();
          }

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

          var sample = []
          _.forOwn(issuesPastWeek, function(value, key) {
                sample.push({day: key, issues: value})
            });

          console.log(sample)
        })
        .catch(error => {
          console.error(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    getIssuesPerPage: function(page) {
      return axios.get(
        `https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=2018-11-18`,
        {
          params: {
            page: page,
            per_page: 100,
          },
          auth: {
            username: "vickris",
            password: "a3dcec8a3e719cce058feaf8b6657c620931fcdf"
          }
        }
      );
    }
  }
};
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
</style>
