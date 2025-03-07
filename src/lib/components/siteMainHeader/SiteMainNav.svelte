<script context="module" lang="ts">
	import { writable } from 'svelte/store';
	import { buttonVariants } from '$components/ui/button';
	export const MAIN_NAV_ID: string = 'main_nav';
	export const isSiteNavMenuOpen = writable(false);
</script>

<script lang="ts">
	import { page } from '$app/stores';
	import type { HTMLAttributes } from 'svelte/elements';
	import { mainNavLinks } from '$lib/utils/navLinks';
	import { cn } from '$lib/utils/styleTransitionUtils';
	import Button from '$components/ui/button/button.svelte';
	import SubmitButton from '$components/form/SubmitButton.svelte';

	let className: HTMLAttributes<HTMLElement>['class'] = undefined;
	export { className as class };

	// computed property for dynamic classes
	$: dynamicClasses = {
		'-translate-x-full': !$isSiteNavMenuOpen
	};

	// static classes
	const staticClasses =
		'text-white absolute left-0 z-50 w-full pt-10 transition-transform duration-300 top-full h-svh sm:h-full sm:relative sm:w-fit sm:-translate-x-0 sm:p-0 bg-secondary sm:bg-transparent hover:bg-none';

	const sessionLinks = {
		show_trip: mainNavLinks.show_trip,
		list: mainNavLinks.list_ticket,
	};

	const noneSessionLinks = {
		show_trip: mainNavLinks.show_trip,
		list: mainNavLinks.list_ticket,
		login: mainNavLinks.login,
		register: mainNavLinks.register,
	};

	$: isSession = $page.data.session;
</script>

<nav
	id={MAIN_NAV_ID}
	aria-label="Main Navigation"
	class={cn(staticClasses, dynamicClasses, className)}
>
	<ul class="grid gap-5 *:*:w-full sm:flex sm:gap-2">
		{#each Object.values(isSession ? sessionLinks : noneSessionLinks) as link}
			{@const isCurrentPage = $page.url.pathname === link.href ? 'page' : undefined}
			<li>
				<Button
					href={link.href}
					variant="outline"
					aria-label={link.ariaLabel}
					aria-current={isCurrentPage}
					class={isCurrentPage ? '' : 'text-muted-foreground'}
				>
					{link.title}
				</Button>
			</li>
		{/each}
		{#if isSession}
			<li>
				<form action="/logout" method="POST">
					<SubmitButton class="text-sm font-bold">ออกจากระบบ</SubmitButton>
				</form>
			</li>
		{/if}
	</ul>
</nav>
