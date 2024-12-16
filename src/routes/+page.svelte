<!-- +page.svelte -->
<script>
  let coefficients = {
    a: 1,
    b: 0,
    c: 0,
    d: 0,
  };

  // SVG viewport settings
  const width = 800;
  const height = 600;
  const padding = 40;
  const plotWidth = width - 2 * padding;
  const plotHeight = height - 2 * padding;

  // Scale factors
  const xScale = plotWidth / 10; // -5 to 5 = 10 units
  const yScale = plotHeight / 24; // -12 to 12 = 24 units

  // Transform coordinate system
  function transformX(x) {
    return padding + (x + 5) * xScale;
  }

  function transformY(y) {
    return padding + (12 - y) * yScale;
  }

  // Generate x values
  $: xValues = Array.from({ length: 201 }, (_, i) => -5 + i * 0.05);

  // Calculate function values
  $: points = xValues.map((x) => {
    const f =
      coefficients.a * Math.pow(x, 3) +
      coefficients.b * Math.pow(x, 2) +
      coefficients.c * x +
      coefficients.d;

    const f_prime =
      3 * coefficients.a * Math.pow(x, 2) +
      2 * coefficients.b * x +
      coefficients.c;

    return {
      x,
      f,
      f_prime,
    };
  });

  // Generate SVG paths
  $: functionPath = points
    .map(
      (p, i) => `${i === 0 ? "M" : "L"} ${transformX(p.x)} ${transformY(p.f)}`
    )
    .join(" ");

  $: derivativePath = points
    .map(
      (p, i) =>
        `${i === 0 ? "M" : "L"} ${transformX(p.x)} ${transformY(p.f_prime)}`
    )
    .join(" ");

  // Find critical points
  $: criticalPoints = findCriticalPoints(
    coefficients.a,
    coefficients.b,
    coefficients.c
  );

  function findCriticalPoints(a, b, c) {
    const quadA = 3 * a;
    const quadB = 2 * b;
    const quadC = c;

    if (quadA === 0) {
      if (quadB === 0) return [];
      return [-quadC / quadB];
    }

    const discriminant = quadB * quadB - 4 * quadA * quadC;
    if (discriminant < 0) return [];

    const sqrtDisc = Math.sqrt(discriminant);
    const x1 = (-quadB + sqrtDisc) / (2 * quadA);
    const x2 = (-quadB - sqrtDisc) / (2 * quadA);

    return [x1, x2]
      .filter((x) => x >= -5 && x <= 5)
      .sort((a, b) => a - b)
      .map((x) => ({
        x,
        f:
          coefficients.a * Math.pow(x, 3) +
          coefficients.b * Math.pow(x, 2) +
          coefficients.c * x +
          coefficients.d,
      }));
  }

  function handleCoefficientChange(coeff, event) {
    coefficients[coeff] = Number(event.target.value);
    coefficients = coefficients; // trigger reactivity
  }

  // Generate grid lines
  const xGridLines = Array.from({ length: 11 }, (_, i) => -5 + i);
  const yGridLines = Array.from({ length: 25 }, (_, i) => -12 + i);
</script>

<div class="container mx-auto p-4">
  <div class="bg-white rounded-lg shadow-lg p-6">
    <h2 class="text-2xl font-bold mb-4">Cubic Polynomial and Derivative</h2>

    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
      {#each Object.entries(coefficients) as [coeff, value]}
        <div class="flex flex-col space-y-2">
          <label class="text-sm font-medium">
            {#if coeff === "a"}
              a (x³)
            {:else if coeff === "b"}
              b (x²)
            {:else if coeff === "c"}
              c (x)
            {:else}
              d
            {/if}
          </label>
          <input
            type="range"
            min="-5"
            max="5"
            step="0.25"
            {value}
            on:input={(e) => handleCoefficientChange(coeff, e)}
            class="w-full"
          />
          <span class="text-sm">{value}</span>
        </div>
      {/each}
    </div>

    <div class="h-[600px]">
      <svg {width} {height}>
        <!-- Grid -->
        {#each xGridLines as x}
          <line
            x1={transformX(x)}
            y1={padding}
            x2={transformX(x)}
            y2={height - padding}
            stroke="#e5e7eb"
            stroke-dasharray="4,4"
          />
          <text
            x={transformX(x)}
            y={height - padding + 20}
            text-anchor="middle"
            class="text-sm">{x}</text
          >
        {/each}

        {#each yGridLines as y}
          <line
            x1={padding}
            y1={transformY(y)}
            x2={width - padding}
            y2={transformY(y)}
            stroke="#e5e7eb"
            stroke-dasharray="4,4"
          />
          <text
            x={padding - 10}
            y={transformY(y)}
            text-anchor="end"
            alignment-baseline="middle"
            class="text-sm">{y}</text
          >
        {/each}

        <!-- Axes -->
        <line
          x1={padding}
          y1={transformY(0)}
          x2={width - padding}
          y2={transformY(0)}
          stroke="#666"
          stroke-width="2"
        />
        <line
          x1={transformX(0)}
          y1={padding}
          x2={transformX(0)}
          y2={height - padding}
          stroke="#666"
          stroke-width="2"
        />

        <!-- Function curves -->
        <path d={functionPath} stroke="#8884d8" fill="none" stroke-width="2" />
        <path
          d={derivativePath}
          stroke="#82ca9d"
          fill="none"
          stroke-width="2"
        />

        <!-- Critical points -->
        {#each criticalPoints as point}
          <!-- Vertical line -->
          <line
            x1={transformX(point.x)}
            y1={padding}
            x2={transformX(point.x)}
            y2={height - padding}
            stroke="red"
            stroke-dasharray="4,4"
          />

          <!-- Point on function -->
          <circle
            cx={transformX(point.x)}
            cy={transformY(point.f)}
            r="4"
            fill="red"
          />
          <text
            x={transformX(point.x)}
            y={transformY(point.f) - 10}
            text-anchor="middle"
            class="text-sm"
            >{point.f > 0 ? "Max" : "Min"} ({point.x.toFixed(1)}, {point.f.toFixed(
              1
            )})</text
          >

          <!-- Point on x-axis -->
          <circle
            cx={transformX(point.x)}
            cy={transformY(0)}
            r="4"
            fill="#0066cc"
          />
          <text
            x={transformX(point.x)}
            y={transformY(0) + 20}
            text-anchor="middle"
            class="text-sm">f'({point.x.toFixed(1)}) = 0</text
          >
        {/each}

        <!-- Legend -->
        <g transform="translate({width - padding - 150}, {padding + 20})">
          <circle cx="0" cy="0" r="4" fill="#8884d8" />
          <text x="10" y="0" alignment-baseline="middle" class="text-sm"
            >f(x) = ax³ + bx² + cx + d</text
          >
          <circle cx="0" cy="20" r="4" fill="#82ca9d" />
          <text x="10" y="20" alignment-baseline="middle" class="text-sm"
            >f'(x) = 3ax² + 2bx + c</text
          >
        </g>
      </svg>
    </div>
  </div>
</div>

<style>
  :global(.text-sm) {
    font-size: 12px;
  }
</style>
