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
