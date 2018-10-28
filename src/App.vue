<template>
  <div id="app">
    <form action="#" @submit.prevent="getIssues">
      <div class="form-group">
        <input type="text" placeholder="Search Repo" v-model="repository" name="skill" class="col-md-4 col-md-offset-4">
      </div>
    </form>

    <div class="days" v-for="day in days">
      <p>{{ day }}</p>
    </div>
    <div class="my-5">
      <div class="alert alert-info" v-show="loading">
        Loading...
      </div>
      <div class="alert alert-danger" v-show="errored">
        Error occured
      </div>
      <div v-show="chart != null">
        <canvas ref="myChart"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import chart from "chart.js";
import moment from "moment";
import axios from "axios";
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
      loading: false,
      errored: false,
      page: 1
    };
  },
  methods: {
    getIssues: function() {
      this.loading = true;

      axios
        .get(`https://api.github.com/search/issues?q=repo:${this.repository}+is:open+is:issue+created:>=2017-10-17`, {
          params: {
            // q: `repo:${this.repository}+is:open+is:issue+created:>=2017-10-17`,
            page: 1
          },
          auth: {
            username: "vickris",
            password: "a3dcec8a3e719cce058feaf8b6657c620931fcdf"
          }
        })
        .then(response => {
          var days = response.data.items.map(issue => {
            return moment(issue.created_at).format("MMM Do YY");
          });

          console.log(response.headers)
        this.days = Array.from(new Set(days));
        //   let last_page = response.headers.link
        //     .split(";")[1]
        //     .split("page=")[1]
        //     .split("")[0];
        //   const promises = [];

        //   for (let page = 1; page <= last_page; page++) {
        //     promises.push(this.getIssuesPerPage(page));
        //   }

        //   return Promise.all(promises);
        // })
        // .then(responses => {
        //   const all_issues = responses.reduce((acc, res) => {
        //     acc = acc.concat(res.data);
        //     return acc;
        //   }, []);

        //   var days = all_issues.map(issue => {
        //     return moment(issue.updated_at).format("MMM Do YY");
        //   });
        //   this.days = Array.from(new Set(days));
        })
        .catch(error => {
          console.error(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    getIssuesPerPage: function(page) {
      return axios.get(
        `https://api.github.com/repos/${this.repository}/issues`,
        {
          params: {
            state: "all",
            page: page,
            q: "created:>2018-10-16+"
            // since: moment().subtract(7, 'days').toISOString()
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
