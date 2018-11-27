```vue
methods: {
    getIssues: function() {
        // code goes in here
    }
}
```

So before we can start defining what's supposed to happen inside this `getIssues()` method, lets import
some of the packages that we installed when we were starting this project:

```vue
import * as d3 from 'd3';
import moment from "moment";
import axios from "axios";
import _ from "lodash";
```

All set, we can start coding the expected behaviour iniside the `getIssues()`.

```vue
axios
.get(`https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=2018-11-18`, {
  params: {
    per_page: 100
  },
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
```

What the above block of code does is make an API request to github to get all issues opened in the past one week that are still open. You can refer to [GitHubs search API](http://) if you need more examples on how to come up with search params when consuming the API.
Inside the params section, we set the results we want to 100(GitHubs max apparently) so as to reduce the number of request's we make to Github in the case the results span more than one page. The default value is 30.

In the case the request was successful, we check the link value of the headers so as to determine if the results span more than one page. If thats the case, we get the value of the last page, the iterate through all the pages while concatinating the results. This is made possibel by the `getIssuesPerPage()` method which we are yet to define.

```vue
methods: {
    [...]
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
```

Note, we are still making the request to the same URL, only difference is this time round, we pass in the page number as the parameter, to get the results on that particular page.

Since we can now get all the results, we define another `then` block that takes in the returned results as the response for processing. Something along these lines:

```vue
axios
.get(`https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=2018-11-18`, {
  params: {
    per_page: 100
  },
})
.then(response => {
  // first response
})
.then(responses => {
  // Get all the issues in one array
  const all_issues = responses.reduce((acc, res) => {
    acc = acc.concat(res.data.items);
    return acc;
  }, []);


  // Generate array containing dates when the issues were created in a nice format
  this.days = all_issues.map(issue => {
    return moment(issue.created_at).format("MMM Do YY");
  });

  var issuesPastWeek = {};

  // Group issue count by day
  var issuesByDay = _.countBy(this.days)

  // Get last 7 days
  var dates = this.getDates(moment().subtract(6, 'days'), moment())


  // Assign each day in the last 7 days to the number of issues created
  for (var i = 0; i < dates.length; i++) {
    if(issuesByDay[dates[i]]) {
      issuesPastWeek[dates[i]] = issuesByDay[dates[i]]
      continue
    }

    issuesPastWeek[dates[i]] = 0
  }

  // Generate array of objects each object containing the day and number of issues
  var sample = []
  _.forOwn(issuesPastWeek, function(value, key) {
        sample.push({day: key, issues: value})
    });
})
```

Too much to process? I hope not. What we are doing above is getting all the issues from the response then generating an array containing the dates when all these issues were created. I used [Moment](link) to help me out with the time and date formatting.
Once we get all the days, we group the days by the number of occurrences in that array. After that, we loop through the last 7 days assigning the count we just generated above to that day. In cases where the date is absent in the `issuesByDay` array, we assume no issues were opened on that day then assign a value of zero to that day. Lastly, we generate an array of objects containing the sample data we want processed. This is the data we want to visualize.

Note, D3 has built-in functionality to load from XMLHttpRequest, .csv files, text files etc. Each of these sources may contain data that D3.js can use, but the only important thing is to construct an array out. What we just did.

Before we get to the drawing bit, let's also handle errors inside a catch block.

```
axios
.get(`https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=2018-11-18`, {
  params: {
    per_page: 100
  },
})
.then(response => {
  // first response
})
.then(responses => {
  // Process data
})
.catch(error => {
  console.error(error);
})
```

We'll add some UX for letting the user aware that something went wrong later. In the meantime, we'll just log the error to the console.

Now to the fun bit. Visualizing this data.


