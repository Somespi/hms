<script lang="ts">
	import { page } from '$app/state';
	import TabButton from '$lib/TabButton.svelte';
	import axios from 'axios';
	import { onMount } from 'svelte';
	import { marked } from 'marked';

	const tabs = [
		{ name: 'details', label: 'Details' },
		{ name: 'history', label: 'History' }
	];

	let activeTab = $state('details');

	let wikiName: string = page.params.wiki_name;
	let language: string = 'en';

	let summary: string = $state('Generating...');
	let wikiBody: string = $state('Generating...');
	let wikiSections: any = $state({});

	async function getWikiSummary() {
		try {
			const res = await axios.post('http://9.141.41.77:8080/summary', {
				lang: language,
				wiki: wikiName
			});
			summary = res.data.message;
			console.log(summary);
		} catch (error) {
			console.error('Error fetching summary:', error);
		}
	}

	async function getWikiBody() {
		try {
			const res = await axios.get(`http://9.141.41.77:8080/wiki/${language}/${wikiName}`);
			wikiBody = res.data.summary;
			wikiSections = res.data.sections;
			console.log(wikiSections);
		} catch (error) {
			console.error('Error fetching wiki body:', error);
		}
	}

	async function loadWikiDataAtOnce() {
		try {
			await Promise.all([getWikiSummary(), getWikiBody()]);
			console.log('Wiki data loaded successfully!');
		} catch (error) {
			console.error('Error loading wiki data:', error);
		}
	}

	function setActiveTab(tabName: string) {
		activeTab = tabName;
	}

	onMount(() => {
		const urlParams = new URLSearchParams(window.location.search);
		language = urlParams.get('lang') ?? 'en';

		loadWikiDataAtOnce();
	});
</script>

<h1 class="py-6 text-3xl font-semibold">{wikiName}</h1>
<div class="inline-block w-auto">
	<div class="bg-base-300 flex space-x-1 rounded-md p-1">
		<TabButton name="details" label="Details" onClick={setActiveTab} bind:activeTab />
		<TabButton name="history" label="History" onClick={setActiveTab} bind:activeTab />
	</div>
</div>
{#if activeTab === 'history'}
	<h1>History</h1>
{:else}
	<div class="card bg-base-100 my-10 w-full border">
		<div class="card-body">
			<p class="opacity-80">
				{summary}
			</p>
		</div>
	</div>

	<div class="text-container">
		<div class="ai-card">
			<div class="card bg-base-100 border shadow-sm">
				<figure class="px-10 pt-10">
					<img
						class="h-34 w-34"
						src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a6/Coat_of_arms_of_Egypt_%28Official%29.svg/113px-Coat_of_arms_of_Egypt_%28Official%29.svg.png"
						alt="AI"
					/>
				</figure>
				<div class="card-body items-center text-center">
					<h2 class="card-title">Arab Republic of Egypt</h2>
					<ul class="space-y-2 text-left font-light">
						<li>
							<strong>Anthem:</strong> Bilady, Bilady, Bilady ("My country, my country, my country")
						</li>
						<li><strong>Capital and Largest City:</strong> Cairo</li>
						<li><strong>Official Language:</strong> Arabic</li>
						<li><strong>National Language:</strong> Egyptian Arabic</li>
						<li>
							<strong>Government:</strong> Unitary semi-presidential republic (described by some as an
							authoritarian dictatorship)
						</li>
						<li><strong>President:</strong> Abdel Fattah el-Sisi</li>
						<li><strong>Prime Minister:</strong> Mostafa Madbouly</li>
						<li><strong>Legislature:</strong> Parliament (Senate and House of Representatives)</li>
						<li>
							<strong>Historical Highlights:</strong>
							<ul>
								<li>Unification of Upper and Lower Egypt (c. 3150 BC)</li>
								<li>Arab conquest (639–642)</li>
								<li>Independence from the United Kingdom (1922)</li>
								<li>Republic declared (1953)</li>
							</ul>
						</li>
						<li><strong>Area:</strong> 1,010,408 km²</li>
						<li><strong>Population:</strong> Approximately 107.3 million (2024 estimate)</li>
						<li><strong>Currency:</strong> Egyptian pound (EGP)</li>
					</ul>
				</div>
			</div>
		</div>

		<div class="space-y-4 font-light" id="wiki_body">
			{@html marked(wikiBody)}
			<br><br><br>
			{#each Object.keys(wikiSections) as section}
				<details open>
					<summary class="text-2xl font-bold">{section}</summary>

					<p>
						{@html marked(
							(wikiSections[section]?.body ?? '').replace(new RegExp(`^###\\s*${section}\\s*`), '')
						)}
					</p>
				</details>
			{/each}

			<div class="clearfix"></div>
		</div>
	</div>

	<style>
		.ai-card {
			float: right;
			width: 250px;
			margin-left: 15px;
			shape-outside: margin-box;
		}

		.clearfix::after {
			content: '';
			display: block;
			clear: both;
		}

		a {
			cursor: pointer;
			text-decoration-line: underline;
			color: var(--color-primary) /* var(--color-primary) */;
		}

		a::focus-visible {
			outline: 2px solid currentColor;
			outline-offset: 2px;
		}

		a:focus {
			--tw-outline-style: none;
			outline-style: none;
		}

		@media (forced-colors: active) {
			a:focus {
				outline: 2px solid transparent;
				outline-offset: 2px;
			}
		}

		@media (hover: hover) {
			.a:hover {
				color: color-mix(in oklab, var(--color-primary) 80%, #000);
			}
		}

		a:hover {
			@apply link-hover !important;
		}

		strong {
			--tw-font-weight: var(--font-weight-semibold) /* 600 */;
			font-weight: var(--font-weight-semibold) /* 600 */;
		}
		@property --tw-font-weight {
			syntax: '*';
			inherits: false;
		}

		#wiki_body img {
			float: right;
			margin-left: 15px;
			shape-outside: margin-box;
			max-width: 100%;
			height: auto;
			display: block;
		}
	</style>
{/if}
