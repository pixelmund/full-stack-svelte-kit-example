<script context="module" lang="ts">
	import type { Load } from '@sveltejs/kit';

	export const load: Load = function ({ session }) {
		if (session?.isLoggedIn) {
			return {
				status: 302,
				redirect: '/'
			};
		}

		return {};
	};
</script>

<script lang="ts">
	import { page, navigating } from '$app/stores';
	import { onDestroy } from 'svelte';

	let blurred = false;
	let timeout: NodeJS.Timeout;

	$: signIn = $page.url.pathname === '/auth/login';
	$: if (
		($navigating?.from.pathname === '/auth/register' &&
			$navigating?.to.pathname === '/auth/login') ||
		($navigating?.from.pathname === '/auth/login' && $navigating?.to.pathname === '/auth/register')
	) {
		blurred = true;
	} else {
		timeout = setTimeout(function () {
			blurred = false;
		}, 200);
	}

	onDestroy(() => {
		if (timeout) {
			clearTimeout(timeout);
		}
	});
</script>

<div class="min-h-screen bg-gray-50 flex flex-col justify-center py-12 sm:px-6 lg:px-8">
	<div class="sm:mx-auto sm:w-full sm:max-w-lg">
		<h2 class="mt-6 text-center text-4xl font-extrabold text-gray-900">
			{signIn ? 'Sign in' : 'Sign up'}
		</h2>
		<p class="mt-4 text-center text-base text-gray-600">
			Or
			<a
				href={signIn ? '/auth/register' : '/auth/login'}
				class="font-medium text-indigo-600 hover:text-indigo-500"
			>
				{signIn ? 'create an account' : 'log in to your account.'}
			</a>
		</p>
	</div>
	<div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
		<div
			class:blur-sm={blurred}
			class="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10 transition-all duration-200"
		>
			<slot />
		</div>
	</div>
</div>
