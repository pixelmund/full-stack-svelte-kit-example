<script context="module" lang="ts" ssr>
	import type { Action, Loader } from 'svemix/server';
	import type { Post } from '@prisma/client';
	import db from '$lib/db';

	interface LoaderData {
		posts: Post[];
		pageInfo: {
			totalPages: number;
			currentPage: number;
			totalCount: number;
		};
	}

	const TAKE = 6;

	export const loader: Loader<LoaderData> = async function ({ url }) {
		const page = parseInt(url.searchParams.get('page')) || 1;
		const skip = (page - 1) * TAKE;

		const [totalCount, posts] = await db.$transaction([
			db.post.count(),
			db.post.findMany({ take: TAKE, skip, orderBy: { createdAt: 'desc' } })
		]);

		const totalPages = Math.ceil(totalCount / TAKE);

		return {
			pageInfo: {
				totalPages,
				currentPage: page,
				totalCount
			},
			posts
		};
	};

	export const action: Action<any> = async function ({ locals, request }) {
		const body = await request.formData();

		const _action = body.get('_action');

		if (_action === 'logout') {
			locals.session.destroy();
		}

		return {};
	};
</script>

<script lang="ts">
	import { session } from '$app/stores';
	import { Form, getLoaderData } from 'svemix';

	const data = getLoaderData<LoaderData>();
</script>

<div class="relative bg-gray-50 min-h-screen pt-16 pb-20 px-4 sm:px-6 lg:pt-24 lg:pb-28 lg:px-8">
	<div class="absolute inset-0">
		<div class="bg-white h-1/3 sm:h-2/3" />
	</div>
	<div class="relative max-w-7xl mx-auto">
		<div class="text-center">
			<h2 class="text-3xl tracking-tight font-extrabold text-gray-900 sm:text-5xl">Svemix Blog</h2>
			<p class="mt-3 max-w-2xl mx-auto text-xl text-gray-500 sm:mt-4">
				Lorem ipsum dolor sit amet consectetur, adipisicing elit. Ipsa libero labore natus atque,
				ducimus sed.
			</p>
		</div>
		<div class="mt-4 flex justify-between items-center">
			<h4 class="text-lg font-semibold">
				{$data.pageInfo.totalCount} posts
			</h4>
			<div class="flex items-center">
				{#if $session.isLoggedIn}
					<a
						sveltekit:prefetch
						class="text-indigo-600 uppercase tracking-wide font-semibold mr-6"
						href="/posts/new">Add New Post</a
					>
					<Form>
						<input type="hidden" name="_action" value="logout" />
						<button class="text-indigo-600 uppercase tracking-wide font-semibold" type="submit"
							>Logout</button
						>
					</Form>
				{:else}
					<a class="text-indigo-600 uppercase tracking-wide font-semibold" href="/auth/login">
						Sign in
					</a>
				{/if}
			</div>
		</div>
		<div class="mt-8 max-w-lg mx-auto grid gap-5 lg:grid-cols-3 lg:max-w-none">
			{#if $data.posts && $data.posts.length > 0}
				{#each $data.posts as post (post.slug)}
					<div class="flex flex-col rounded-lg shadow-lg overflow-hidden">
						<div class="flex-shrink-0">
							<img
								class="h-48 w-full object-cover"
								src="https://images.unsplash.com/photo-1496128858413-b36217c2ce36?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1679&q=80"
								alt=""
							/>
						</div>
						<div class="flex-1 bg-white p-6 flex flex-col justify-between">
							<div class="flex-1">
								<a sveltekit:prefetch href="/posts/{post.slug}" class="block mt-2">
									<p class="text-xl font-semibold text-gray-900">{post.title}</p>
									<p class="mt-3 text-base text-gray-500">
										{post.excerpt}
									</p>
								</a>
							</div>
							<div class="mt-6 flex items-center">
								<div>
									<div class="flex space-x-1 text-sm text-gray-500">
										<time datetime={new Date(post.createdAt).toISOString()}>
											{new Date(post.createdAt).toLocaleDateString()}
										</time>
										<span aria-hidden="true">&middot;</span>
										<span>{post.read_time} min read</span>
									</div>
								</div>
							</div>
						</div>
					</div>
				{/each}
			{/if}
		</div>
		<div class="flex justify-center mt-4 items-center">
			{#each { length: $data.pageInfo.totalPages } as _itm, index}
				<a
					class="py-1 px-3 bg-gray-200 mx-1 font-semibold rounded-md {index ===
					$data.pageInfo.currentPage - 1
						? 'text-indigo-600'
						: ''}"
					href="/?page={index + 1}">{index + 1}</a
				>
			{/each}
		</div>
	</div>
</div>
