<script>
import { onMount } from "svelte";

export let progress = 0;
export let progressRaw = 0;
export let threshold = 0.5;
export let margin = 30;
export let debounce = false;
export let throttle = false;

let container, vizContainer;
let minTop, maxTop;
let pageTop;
let rect = {};
let intersectionObserver, resizeObserver;
let intersectionRatio = 0;

const clamp = (min, value, max) => Math.min(Math.max(min, value), max);
const getProgress = (value, min, max) => 100 * (value - min) / (max - min);
const runImmediately = fn => fn();
const identity = fn => fn;

let throttled;
$: {
	let last = 0;
	throttled = throttle > 0 ? function(fn) {
		return function () {
			let now = performance.now();
			if (now - last >= throttle) {
				fn();
				last = now;
			}
		}
	} : identity;
}

let debounced;
$: {
	let debouncerId;
	debounced = debounce? debounce > 0? function(fn) {
		clearTimeout(debouncerId);
		debouncerId = setTimeout(fn, debounce);
	} : function(fn) {
		cancelAnimationFrame(debouncerId);
		debouncerId = requestAnimationFrame(fn);
	} : runImmediately;
}

onMount(() => {
	function calculateProgress ({top = container.getBoundingClientRect().top} = {}) {
		progressRaw = getProgress(top, minTop, maxTop);
		// console.table({ minTop, maxTop, pageTop, top, progress, progressRaw });
		updateProgress();
	}

	function updateProgress () {
		let clampedProgress = clamp(0, progressRaw, 100);

		if (clampedProgress === 0 || clampedProgress === 100) {
			// 0% and 100% should happen immediately
			progress = clampedProgress;
		}
		else {
			debounced(throttled(() => progress = clampedProgress));
		}
	}

	function calculateBounds () {
		rect = container.getBoundingClientRect();

		// progress = 0 when the top of the container passes threshold% of the viewport
		// OR when we’re scrolled all the way to the top,
		// whichever is greater
		pageTop = window.scrollY + rect.top; // this is always constant
		minTop = Math.min(pageTop, innerHeight * threshold) + margin;

		// progress = 100 when the bottom of the container is at the bottom of the viewport (minus the margin),
		maxTop = innerHeight - rect.height + margin;

		calculateProgress(rect);
		// console.table({margin, minTop, maxTop, pageTop, top: rect.top, height: rect.height, progress, progressRaw });
	}

	intersectionObserver = new IntersectionObserver(entries => {
		let lastEntry = entries.at(-1);
		intersectionRatio = lastEntry.intersectionRatio;

		if (lastEntry.isIntersecting) {
			calculateBounds();
			calculateProgress();
			window.addEventListener('scroll', calculateProgress);
			window.addEventListener('resize', calculateBounds);
			resizeObserver?.observe(container);
		}
		else {
			window.removeEventListener('scroll', calculateProgress);
			window.removeEventListener('resize', calculateBounds);
			resizeObserver?.unobserve(container);
		}
	});

	intersectionObserver.observe(container, {
		rootMargin: (margin ?? 0) + "px",
		threshold: threshold
	});

	resizeObserver = new ResizeObserver(calculateBounds);

	calculateBounds();
});
</script>

<section class="scrolly" bind:this={container} style="--scrolly-margin: { margin }">
	<section class="story">
		<slot />
	</section>
	<section class="viz" bind:this={vizContainer}>
		<slot name="viz" />
	</section>
</section>

<style>
.scrolly {
	position: relative;
	display: grid;
	grid-template-columns: var(--scrolly-story-width, 1fr) var(--scrolly-viz-width, 1fr);
	grid-auto-flow: row dense;
	gap: var(--scrolly-gap, 4rem);
}

.viz,
.story {
	grid-row: 1;
}

.viz {
	position: sticky;
	top: max(var(--scrolly-margin, 0) * 1px, var(--scrolly-viz-top, 2em));
	max-height: 100vh;
}

@container style(--scrolly-layout: viz-first) {
	.scrolly {
		grid-template-columns: var(--scrolly-viz-width, 1fr) var(--scrolly-story-width, 1fr);
	}

	.viz {
		grid-column: 1;
	}

	.story {
		grid-column: 2;
	}
}

@container style(--scrolly-layout: overlap) {
	.scrolly {
		grid-template-columns: 1fr;
	}

	.viz,
	.story {
		grid-column: 1;
	}
}
</style>

