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

          const margin = 60;
          const width = 1000 - 2 * margin;
          const height = 600 - 2 * margin;

          const svg = d3.select('svg')
                .attr("width", width)
                .attr("height", height)

          this.chart = svg.append('g')
          .attr('transform', `translate(${margin}, ${margin})`);



          const yScale = d3.scaleLinear()
          .range([height, 0])
          .domain([0, _.maxBy(sample, 'issues').issues]);

          this.chart.append('g')
            .call(d3.axisLeft(yScale));

          const xScale = d3.scaleBand()
            .range([0, width])
            .domain(sample.map((s) => s.day))
            .padding(0.2)

          this.chart.append('g')
              .attr('transform', `translate(0, ${height})`)
              .call(d3.axisBottom(xScale));

          this.chart.append('g')
            .attr('class', 'grid')
            .call(d3.axisLeft()
                .scale(yScale)
                .tickSize(-width, 0, 0)
                .tickFormat(''))

          const barGroups = this.chart.selectAll()
            .data(sample)
            .enter()
            .append('g')

          barGroups
            .append('rect')
            .attr('class', 'bar')
            .attr('x', (g) => xScale(g.day))
            .attr('y', (g) => yScale(g.issues))
            .attr('height', (g) => height - yScale(g.issues))
            .attr('width', xScale.bandwidth())
            .on('mouseenter', function (actual, i) {
              d3.select(this)
                .transition()
                .duration(300)
                .attr('opacity', 0.6)
                .attr('x', (a) => xScale(a.day) - 5)
                .attr('width', xScale.bandwidth() + 10)

                const y = yScale(actual.issues)

               line = chart.append('line')
                .attr('id', 'limit')
                .attr('x1', 0)
                .attr('y1', y)
                .attr('x2', width)
                .attr('y2', y)

            })
            .on('mouseleave', function () {
              d3.selectAll('.issues')
                .attr('opacity', 1)

              d3.select(this)
                .transition()
                .duration(300)
                .attr('opacity', 1)
                .attr('x', (a) => xScale(a.day))
                .attr('width', xScale.bandwidth())

              chart.selectAll('#limit').remove()
              chart.selectAll('.divergence').remove()
            })

          barGroups
            .append('text')
            .attr('class', 'value')
            .attr('x', (a) => xScale(a.day) + xScale.bandwidth() / 2)
            .attr('y', (a) => yScale(a.issues) + 30)
            .attr('text-anchor', 'middle')
            .text((a) => `${a.issues} issues`)

          svg
            .append('text')
            .attr('class', 'label')
            .attr('x', -(height / 2) - margin)
            .attr('y', margin / 2.4)
            .attr('transform', 'rotate(-90)')
            .attr('text-anchor', 'middle')
            .text('Issues created')

          svg.append('text')
            .attr('class', 'label')
            .attr('x', width / 2 + margin)
            .attr('y', height + margin * 1.7)
            .attr('text-anchor', 'middle')
            .text('Days')

          svg.append('text')
            .attr('class', 'title')
            .attr('x', width / 2 + margin)
            .attr('y', 40)
            .attr('text-anchor', 'middle')
            .text('Issues in the past 1 week')
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
