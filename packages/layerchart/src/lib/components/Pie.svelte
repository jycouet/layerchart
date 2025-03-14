<script lang="ts">
  import type { spring as springStore, tweened as tweenedStore } from 'svelte/motion';
  import { pie as d3pie } from 'd3-shape';
  import { min, max } from 'd3-array';

  import Arc from './Arc.svelte';
  import { chartContext } from './ChartContext.svelte';
  import { degreesToRadians } from '$lib/utils/math.js';
  import { motionStore } from '$lib/stores/motionStore.js';
  import type { TooltipContextValue } from './tooltip/TooltipContext.svelte';

  /*
    TODO:
    - [ ] Offset (always, on hover)
    - [ ] Labels
    - [ ] Multiple nested circles (zScale, or take in data to override context data).  See Path/Area/Threshold
    - [ ] Hover events / change innerRadius / outerRadius, etc
  */

  export let data: any[] | undefined = undefined; // TODO: Update Type

  /**
   * Range [min,max] in degrees.  See also startAngle/endAngle
   */
  export let range = [0, 360]; // degrees

  /**
   * Start angle in radians
   */
  export let startAngle: number | undefined = undefined;

  /**
   * End angle in radians
   */
  export let endAngle: number | undefined = undefined;

  /**
   * Define innerRadius.
   *   value >= 1: discrete value
   *   value >  0: percent of `outerRadius`
   *   value <  0: offset of `outerRadius`
   *   default: yRange min
   */
  export let innerRadius: number | undefined = undefined;

  /**
   * Define outerRadius.  Defaults to yRange max/2 (ie. chart height / 2)
   */
  export let outerRadius: number | undefined = undefined;

  export let cornerRadius = 0;
  export let padAngle = 0;

  export let spring: boolean | Parameters<typeof springStore>[1] = undefined;
  export let tweened: boolean | Parameters<typeof tweenedStore>[1] = undefined;

  /**
   * Offset all arcs from center
   */
  export let offset = 0;

  /**
   * Tooltip context to setup pointer events to show tooltip for related data
   */
  export let tooltip: TooltipContextValue | undefined = undefined;

  export let sort: ((a: any, b: any) => number) | null | undefined = undefined;

  const { data: contextData, x, y, xRange, c, cScale, config } = chartContext();

  // @ts-expect-error
  $: resolved_endAngle = endAngle ?? degreesToRadians($config.xRange ? max($xRange) : max(range));
  let tweened_endAngle = motionStore(0, { spring, tweened });
  $: tweened_endAngle.set(resolved_endAngle);

  let pie: ReturnType<typeof d3pie<any>>;
  $: {
    pie = d3pie<any>()
      // @ts-expect-error
      .startAngle(startAngle ?? degreesToRadians($config.xRange ? min($xRange) : min(range)))
      .endAngle($tweened_endAngle)
      .padAngle(padAngle)
      .value($x);

    if (sort === null) {
      pie = pie.sort(null);
    } else if (sort) {
      pie = pie.sort(sort);
    }
  }

  $: arcs = pie(data ?? (Array.isArray($contextData) ? $contextData : []));
</script>

<slot {arcs}>
  {#each arcs as arc}
    <Arc
      startAngle={arc.startAngle}
      endAngle={arc.endAngle}
      padAngle={arc.padAngle}
      {innerRadius}
      {outerRadius}
      {cornerRadius}
      {offset}
      fill={$config.c ? $cScale?.($c(arc.data)) : null}
      data={arc.data}
      {tooltip}
    />
  {/each}
</slot>
