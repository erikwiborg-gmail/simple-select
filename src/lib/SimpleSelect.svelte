<script lang="ts" generics="T">
	import ClickOutside from '$lib/ClickOutside.svelte';

	import { computePosition, flip, offset, type Placement, shift } from '@floating-ui/dom';
	import { tick } from 'svelte';

	type Option<T> = { label: string; value: T };

	let {
		options,
		onchange,
		label = null,
		placement = 'bottom-end',
		selected = options[0]
	}: {
		options: Option<T>[];
		onchange: (value: T) => void;
		label?: string | null;
		placement?: Placement;
		selected?: Option<T>;
	} = $props();

	let visible = $state(false);
	let prev = $state({ ...selected });

	$effect(() => {
		if (prev.value !== selected.value) {
			visible = false;
			prev = { ...selected };
			onchange(selected.value);
		}
	});

	let trigger: HTMLElement | undefined = $state();
	let float: HTMLElement | undefined = $state();

	const showSelectItems = async () => {
		visible = !visible;
		await tick(); // wait for DOM to update before computing position of the floating element
		if (visible && trigger !== undefined && float !== undefined) {
			computePosition(trigger, float, {
				placement,
				middleware: [flip(), shift(), offset(2)]
			}).then(({ x, y }) => {
				if (float === undefined) return;
				Object.assign(float.style, {
					left: `${x}px`,
					top: `${y}px`
				});
			});
		}
	};
</script>

{#snippet selectItems()}
	<ul
		role="menu"
		bind:this={float}
		class="list-style-none absolute z-50 max-h-96 overflow-y-auto rounded-xl border p-1 shadow-lg"
		class:hidden={!visible}
	>
		{#each options as option}
			<li
				role="menuitem"
				class="text-md cursor-pointer rounded-lg px-1 py-1.5 whitespace-nowrap"
				class:selected={option.value === selected.value}
				onkeydown={(e) => {
					if (e.key === 'Enter') {
						selected = option;
						visible = false;
					}
				}}
				onclick={(e) => {
					e.stopImmediatePropagation();
					if (option.value === selected.value) {
						visible = false;
						return;
					}
					selected = option;
				}}
			>
				{option.label}
			</li>
		{/each}
	</ul>
{/snippet}

<div class="flex flex-row items-center justify-between" class:gap-10={label !== null}>
	{#if label !== null}
		<label class="text-md whitespace-nowrap">{label}</label>
	{/if}

	<div
		bind:this={trigger}
		role="button"
		tabindex="0"
		class="relative cursor-pointer rounded-md max-w-fit outline text-sm"
		onclick={(e) => {
			e.stopImmediatePropagation();
			showSelectItems();
		}}
		onkeydown={(e) => {
			if (e.key === 'Enter') {
				showSelectItems();
			}
		}}
	>
		<div class="flex flex-row items-center justify-between gap-0.5">
			<span class="text-md whitespace-nowrap">{selected.label}</span>
			<span class="arrow -translate-y-1/4 ml-2"
						>⌄</span
			>
		</div>
		{#if visible}
			<ClickOutside
				exclude={[trigger]}
				onOutclick={() => {
					visible = false;
				}}>{@render selectItems()}</ClickOutside
			>
		{/if}
	</div>
</div>

<style lang="postcss">
	@reference "tailwindcss/theme";
	@reference '../app.css';

	[role='button'] {
		transition: all 150ms ease;
		@apply relative cursor-pointer rounded-md max-w-fit;
		@apply ring-gray-500 ring outline-0 ring-inset;
		@apply py-1.5 pr-1.5 pl-3;
	}

	[role='button']:hover {
		@apply ring-hover;
  }

	[role='button']:focus {
		@apply ring-selected ring ring-inset;
	}

	[role='menu'] {
		@apply border-selected;
	}

	[role='menuitem']:hover {
		background-color: var(--color-hover);
  }

	[role='menuitem']:active {
		background-color: var(--color-active);
  }

	[role='menuitem'].selected,
	[role='menuitem'].selected:hover {
			background-color: var(--color-selected);
	}
</style>
