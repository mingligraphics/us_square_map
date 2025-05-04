<!-- https://gist.github.com/officeofjane/2c3ed88c4be050d92765de912d71b7c4 -->
<script>
  import data from "./data/states.json";
  import { scaleLinear } from "d3-scale";
  import { tweened } from 'svelte/motion';
  import { cubicOut } from "svelte/easing";
  import { interpolate } from "d3-interpolate";
  console.log(data)

  let width = 600;
  let height = 480;

  $: square_width = Math.min((innerWidth / 11 - 5), (innerHeight / 8 - 5));
  
  const margin = {
      top:20,
      right: 20,
      bottom: 30,
      left: 0
    };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  $: xScale = scaleLinear().domain([0, 11]).range([0, innerWidth]);
  let yScale = scaleLinear().domain([0, 8]).range([innerHeight, 0]);

  let initialData = data.sort((a, b) => a.grade - b.grade);

  let newData_A = initialData.map(item => ({
  ...item,
  case: "A",
  share_china: 0  // add the new column
}));

  let newData_B = initialData.map(item => ({
  ...item,
  case: "B"   // add the new column
}));

const tweenedPoints = tweened(newData_A, {
      delay: 0,
      duration: 3000,
      easing: cubicOut,
      interpolate: (a, b) => interpolate(a, b)
    });

    tweenedPoints.set(newData_B);


</script>

<main>
<section>
<div class="sticky">
  <div class="chart-container" bind:clientWidth={width}>
    <svg {width} {height}>
      <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
        {#each initialData as d}
        <rect
        x={xScale(d.x_pos)}
        y={yScale(d.y_pos)}
        width={square_width}
        height={square_width}
        stroke="none"
        stroke-width={1}
        fill="#E2E3E4"
        opacity={1}
      />
      {#if width > 450}
      <text
      class="text-lables"
      x={xScale(d.x_pos) + (square_width - square_width*(d.code).length/6.5)/2}
      y={yScale(d.y_pos - 0.5)}
      >
      {d.code}
      </text>
      {/if}
      {/each}
      {#each $tweenedPoints as d}
      <rect
        x={xScale(d.x_pos)}
        y={yScale(d.y_pos) + (1 - d.share_china/100) * square_width}
        width={square_width}
        height={d.share_china/100 * square_width}
        stroke="none"
        stroke-width={1}
        fill="#AC8FBE"
        opacity={1}
      />
      {/each}
      </g>
    </svg>
  </div>
</div>
</section>
</main>

<style>
.sticky {
  position: sticky;
  z-index: 1;
  height:90vh;
  top:5vh;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  /* the three lines above is how a parent center its children */
}
.chart-container{
  height:100%;
  width:100%;
  max-width: 700px;
  max-height: 500px;
}

section {
  position: relative;
}

main{
  max-width: 700px;
  margin:0 auto;
}

.text-lables{
    font-family: Retina, sans-serif;
    font-size: 15px;
  }
</style>