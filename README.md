# svelte-scrolly

A simple Svelte `<Scrolly>` component to facilitate narrative visualizations

There is an official Svelte package for this purpose: [`@sveltejs/svelte-scroller`](https://www.npmjs.com/package/@sveltejs/svelte-scroller).
However it assumes the two layers (narrative and visualization) are overlaid on top of each other,
which is not always what we want.

This is meant to support visualizations that simply enhance the regular scroll behavior by using it to develop a narrative.
Scrollies that actively interfere with the normal user experience of scrolling are out of scope.

Originally created for [this lab](https://vis-society.github.io/labs/9/) of [MIT’s Interactive Visualization & Society course](https://vis-society.github.io/) 2024,
so that can also serve as a tutorial (see after Step 4).

## Installation

```bash
npm install svelte-scrolly
```

## Usage

It’s used like this:

```html
<Scrolly bind:progress={ myProgressVariable }>
	<!-- Story here -->
	<svelte:fragment slot="viz">
		<!-- Visualizations here -->
	</svelte:fragment>
</Scrolly>
```

This will create a scrollytelling narrative with the story on the left and the visualization(s) on the right.
If you instead want the story on the right and the visualizations on the left, you can pass `--layout="viz story"`.
You can also specify `--viz-width` and/or `--story-width` to any number of `fr`s or any CSS length to change the proportion between the two.
E.g. `--viz-width="2fr"` will change the proportion of viz to story to 2:1 give the visualization more space.

```html
<Scrolly bind:progress={raceProgress} --layout="viz story" --viz-width="2fr">
```

## Customization

### Props

| Prop | Description | Type | Default |
| `progress` | The progress of the scrolly. | `number` (0 - 100) | `0` |
| `progressRaw` | The progress of the scrolly, unclamped (i.e. can be negative or > 100). | `number` | `0` |
| `threshold` | What % of the viewport does the scrolly need to reach for it to start? | `number` (0 - 1) | `0.5` |
| `margin` | How much margin to add to the threshold (in pixels) | `number` (0 - 1) | `20` |
| `debounce` | Whether to debounce updating the `progress` variable and optionally by how many milliseconds. If just specified with no value, it debounces by the browser’s framerate. | `number` &#124; `boolean` | `false` |
| `throttle` | Whether to throttle updating the `progress` variable and optionally by how many milliseconds | `number` &#124; `boolean` | `false` |

### CSS Properties

All CSS properties supported:

| Property | Description | Type | Default |
| `--scrolly-layout` | Layout of the scrolly. | `story viz` &#124; `viz story` &#124; `overlap` | `story viz` |
| `--scrolly-gap` | Gap between the story and the visualization. Ignored when `--layout: overlap`. | `<length>` | `1em` |
| `--scrolly-viz-width`, `--scrolly-story-width` | Width of the visualization or the story respectively. Ignored when `--layout: overlap`. | `<length>` | `1fr` |
