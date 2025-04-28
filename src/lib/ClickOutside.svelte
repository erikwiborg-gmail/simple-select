<script lang="ts">
	import { type Snippet } from 'svelte';
	let {
		children,
		fullWidth = false,
		exclude = [],
		onOutclick
	}: {
		children: Snippet;
		onOutclick: () => void;
		fullWidth?: boolean;
		/** Exclude any elements passed when determining if there was a click outside.
		 * ie. if an element in this array is clicked, an outclick event will not be fired*/
		exclude?: HTMLElement[];
	} = $props();

	let wrapper: HTMLElement | undefined = $state();

	const didClickOutside = (target: HTMLElement): boolean => {
		if (!wrapper) {
			return false;
		}
		return !wrapper.contains(target) && !exclude.some((el) => el.contains(target));
	};

	const handlePointerup = (e: UIEvent): void => {
		if (didClickOutside(e.target as HTMLElement)) {
			e.stopImmediatePropagation();
			onOutclick();
		}
	};

	const handleKeyUp = (e: KeyboardEvent): void => {
		if (e.key === 'Escape') {
			// close on escape
			e.stopImmediatePropagation();
			onOutclick();
		}
	};
</script>

<!-- Have this to capture the events -->
<svelte:window on:pointerup={handlePointerup} on:keyup={handleKeyUp} />

	<div class:w-full={fullWidth} bind:this={wrapper}>
		{@render children()}
	</div>

