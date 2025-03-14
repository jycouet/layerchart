<script lang="ts">
  import { cls } from '@layerstack/tailwind';

  import { setRenderContext } from '../Chart.svelte';
  import { chartContext } from '../ChartContext.svelte';
  import { transformContext } from '../TransformContext.svelte';

  /** The layer's outermost `<div>` tag. Useful for bindings. */
  export let element: HTMLDivElement | undefined = undefined;

  /** The layer's z-index. */
  export let zIndex = undefined;

  /** Set this to `false` to set `pointer-events: none;` on the entire layer. */
  export let pointerEvents: boolean | undefined = undefined;

  /** A string passed to the `aria-role` on the `<div>` tag. This is `undefined` by default but will be set by default to `'figure'` if `label`, `labelledby` or `describedby` is set. That default will be overridden by whatever is passed in. */
  export let role: string | undefined = undefined;

  /** A string passed to the `aria-label` property on the `<div>` tag. */
  export let label: string | undefined = undefined;

  /** A string passed to the `aria-labelledby property` on the `<div>` tag. */
  export let labelledBy: string | undefined = undefined;

  /** A string passed to the `aria-describedby` property on the `<div>` tag. */
  export let describedBy: string | undefined = undefined;

  /**
   * Translate children to center (useful for radial layouts)
   */
  export let center: boolean | 'x' | 'y' = false;

  /** Ignore TransformContext.  Useful to add static elements such as legends. */
  export let ignoreTransform = false;

  $: roleVal = role || (label || labelledBy || describedBy ? 'figure' : undefined);

  const { width, height, padding } = chartContext();
  const { mode, scale, translate } = transformContext();

  let transform = '';
  $: if (mode === 'canvas' && !ignoreTransform) {
    transform = `translate(${$translate.x}px,${$translate.y}px) scale(${$scale})`;
  } else if (center) {
    transform = `translate(${center === 'x' || center === true ? $width / 2 : 0}px, ${center === 'y' || center === true ? $height / 2 : 0}px)`;
  }

  setRenderContext('html');
</script>

<div
  bind:this={element}
  class={cls(
    'layercake-layout-html',
    'absolute top-0 left-0',
    pointerEvents === false && 'pointer-events-none',
    $$props.class
  )}
  style:transform
  style:transform-origin="top left"
  style:z-index={zIndex}
  style:pointer-events={pointerEvents === false ? 'none' : null}
  style:top="{$padding.top}px"
  style:bottom="{$padding.bottom}px"
  style:left="{$padding.left}px"
  style:right="{$padding.right}px"
  role={roleVal}
  aria-label={label}
  aria-labelledby={labelledBy}
  aria-describedby={describedBy}
>
  <slot {element}></slot>
</div>
