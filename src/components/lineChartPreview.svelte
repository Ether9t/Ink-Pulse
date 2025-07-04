<script lang="ts">
  import * as d3 from "d3";

  export let chartData: any[] = [];
  export let paragraphColor: any[] = [];

  let svgContainer: SVGSVGElement;
  let width = 250;
  let height = 150;
  const margin = { top: 10, right: 0, bottom: 30, left: 0 };

  let xScale: any;
  let yScale = d3
    .scaleLinear()
    .domain([0, 100])
    .range([height - margin.top - margin.bottom, 0]);

  let xAxisG: SVGGElement;
  let yAxisG: SVGGElement;

  $: xScale = d3
    .scaleLinear()
    .domain([0, d3.max(chartData, (d) => d.time) || 1])
    .range([0, width - margin.left - margin.right]);

  $: yScale = d3
    .scaleLinear()
    .domain([0, 100])
    .range([height - margin.top - margin.bottom, 0]);

  $: if (xScale && yScale && xAxisG && yAxisG) {
    const xAxis = d3.axisBottom(xScale).ticks(5);
    const yAxis = d3.axisRight(yScale).ticks(5);
    d3.select(xAxisG).call(xAxis);
    d3.select(xAxisG)
      .selectAll(".tick text")
      .filter((_, i) => i === 0)
      .attr("dx", "1px")
      .attr("text-anchor", "start");
    d3.select(yAxisG).call(yAxis);
  }

  function scaledX(val) {
    return xScale ? xScale(val) : 0;
  }

  function scaledY(val) {
    return yScale ? yScale(val) : 0;
  }
</script>

<svg bind:this={svgContainer} {width} {height} style="vertical-align: top">
  <defs>
    <clipPath id="clip_preview">
      <rect
        x="0"
        y="0"
        width={width - margin.left - margin.right}
        height={height - margin.top - margin.bottom}
      />
    </clipPath>
  </defs>

  <g transform={`translate(${margin.left},${margin.top})`}>
    <g clip-path="url(#clip_preview)">
      <g>
        {#each paragraphColor as d}
          <rect
            x={scaledX(d.xMin)}
            width={scaledX(d.xMax) - scaledX(d.xMin)}
            y={scaledY(d.yMax)}
            height={scaledY(d.yMin) - scaledY(d.yMax)}
            fill={d.backgroundColor}
          />
        {/each}
      </g>

      <g>
        {#each chartData.filter((d) => !d.isSuggestionOpen) as d (d.index)}
          <circle
            cx={scaledX(d.time)}
            cy={scaledY(d.percentage)}
            r={2}
            fill={d.color}
            opacity={d.opacity}
          />
        {/each}

        {#each chartData.filter((d) => d.isSuggestionOpen) as d}
          <path
            d={d3.symbol().type(d3.symbolTriangle).size(40)()}
            fill="#FFBBCC"
            opacity={d.opacity + 0.29}
            transform={`translate(${scaledX(d.time)},${scaledY(d.percentage + 6)}) rotate(180)`}
          />
        {/each}
      </g>
    </g>

    <g
      class="x-axis"
      transform={`translate(0, ${height - margin.top - margin.bottom})`}
      bind:this={xAxisG}
    ></g>
    <text
      x={width / 2}
      y={height - margin.top - 7}
      text-anchor="middle"
      font-size="10px"
      fill="black"
    >
      Time (min)
    </text>
    <g class="y-axis" bind:this={yAxisG} style="display: none"></g>
  </g>
</svg>
