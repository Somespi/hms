<script lang="ts">
	import TabButton from '$lib/TabButton.svelte';
	import History from '$lib/History.svelte';

	import { page } from '$app/state';
	import { onMount } from 'svelte';

	import axios from 'axios';
	import { marked } from 'marked';

	const tabs = [
		{ name: 'details', label: 'Details' },
		{ name: 'history', label: 'History' }
	];

	let activeTab = $state('details');

	let wikiName: string = page.params.wiki_name.replaceAll('_', ' ');
	let language: string = $state('en');

	let isUsingBionicReading: boolean = $state(false);

	let summary: string = $state('Generating...');
	let wikiBody: string[] = $state([]);
	let wikiSections: any = $state({});
	let infoBoxImage: string = $state('');
	let infoBoxContent: string = $state('Generating...');

	const isImageAvailable = async (url: string) => {
		console.log(url);
		const img_request = await fetch(url);
		console.log(img_request);
		return img_request.status === 200 && img_request.headers.get('content-type')?.startsWith('image/');
	}

	async function getInfoBox() {
		try {
			const res = await axios.post(`https://12495f02b94ee0.lhr.life/infobox`, {
				lang: language,
				wiki: wikiName
			});
			console.log(res.data);
			let imageIndex = 0;
			while (!await isImageAvailable(res.data.images[Object.keys(res.data.images)[imageIndex]]) && imageIndex < res.data.images.length) {
				imageIndex++;
			}
			infoBoxImage = res.data.images[Object.keys(res.data.images)[imageIndex]];
			const firstKey = Object.keys(res.data.infobox)[0];
			let firstObject = res.data.infobox[firstKey];
			if (
				Object.keys(res.data.infobox).filter((key) => !key.toLowerCase().includes('image'))
					.length != 1
			) {
				firstObject = res.data.infobox;
			}
			console.log(firstObject, res.data);
			let markedDown = '';

			Object.keys(firstObject).forEach((value) => {
				if (firstObject[value] && value !== 'imageGallery') {
					const currentValue = firstObject[value];
					markedDown += `\n**${cleanUpAndTitle(value)}**: `;

					if (
						typeof currentValue === 'string' &&
						new RegExp(`(jpg|jpeg|png|gif|bmp|webp|svg|JPG|JPEG|PNG|GIF|BMP|WEBP|SVG)`).test(
							currentValue
						)
					) {
						//if (!isImageAvailable(currentValue)) return;
						markedDown += ` ![${value}](${currentValue})\n `;
					} else if (Array.isArray(currentValue)) {
						markedDown += '\n';
						markedDown +=
							currentValue.map((item) => `\t- ${cleanUpAndTitle(item)}`).join('\n') + '\n';
					} else if (typeof currentValue === 'string') {
						markedDown += `${cleanUpAndTitle(currentValue)}\n`;
					} else if (typeof currentValue === 'object' && !Array.isArray(currentValue)) {
						Object.keys(currentValue).forEach((objKey) => {
							markedDown += `\n  - **${cleanUpAndTitle(objKey)}**: ${cleanUpAndTitle(currentValue[objKey])}`;
						});
						markedDown += '\n';
					}
				}
				console.log(markedDown);
				infoBoxContent = markedDown;
			});
		} catch (error: any) {
			console.error('Error fetching info box:', error);
			if (error.response.status === 500) {
				getInfoBox();
			}
		}
	}

	async function getWikiSectionSummary(sectionName: string) {
		try {
			const res = await axios.post('https://12495f02b94ee0.lhr.life/summary', {
				lang: language,
				section: sectionName,
				wiki: wikiName
			});
			return res.data.message;
		} catch (error) {
			console.error('Error fetching summary:', error);
		}
	}

	async function getWikiSummary() {
		try {
			const res = await axios.post('https://12495f02b94ee0.lhr.life/summary', {
				lang: language,
				wiki: wikiName
			});
			summary = res.data.message;
			//console.log(summary);
		} catch (error) {
			console.error('Error fetching summary:', error);
		}
	}

	async function getWikiBody() {
		try {
			const res = await axios.get(`https://12495f02b94ee0.lhr.life/wiki/${language}/${wikiName}`);
			wikiBody = [res.data.summary, res.data.bionic_summary.replaceAll('__', '**')];
			wikiSections = res.data.sections;
			for (let section of Object.keys(wikiSections)) {
				wikiSections[section].summary = await getWikiSectionSummary(section);
				console.log(wikiSections[section].summary);
			}
			//console.log(wikiSections);
		} catch (error) {
			console.error('Error fetching wiki body:', error);
		}
	}

	async function loadWikiDataAtOnce() {
		try {
			await Promise.all([getWikiSummary(), getWikiBody(), getInfoBox()]);
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

	function cleanUpAndTitle(string: string) {
		if (!string || typeof string !== 'string') return '';
		let text = string.replaceAll('_', ' ');
		text = text.replaceAll('-', ' ');
		if (new RegExp(`(A-Z)`).test(text)) {
			const index = text.search(new RegExp(`(A-Z)`));
			if (index > 0 && index < text.length - 1) {
				text = text.slice(0, index) + ' ' + text.slice(index);
			}
		}
		text = text.charAt(0).toUpperCase() + text.slice(1);
		return text;
	}
</script>

<svelte:head>
	<title>{wikiName}</title>
</svelte:head>

<div class="flex w-full flex-row justify-between">
	<h1 class="py-6 text-3xl font-semibold">{wikiName}</h1>
	<div class="flex flex-row gap-2 text-sm">
		<span class="font-semibold">Bionic Reading</span>
		<input
			type="checkbox"
			onchange={() => (isUsingBionicReading = !isUsingBionicReading)}
			checked={isUsingBionicReading}
			class="toggle"
		/>
	</div>
</div>
<!-- <div class="inline-block w-auto">
	<div class="bg-base-300 flex space-x-1 rounded-md p-1">
		<TabButton name="details" label="Details" onClick={setActiveTab} bind:activeTab />
		<TabButton name="history" label="History" onClick={setActiveTab} bind:activeTab />
	</div>
</div> -->
{#if activeTab !== 'history'}
	<div class="card bg-base-100 my-10 w-full border">
		<div class="card-body">
			<p class="opacity-80" dir="auto">
				{@html marked(summary)}
			</p>
		</div>
	</div>

	<div class="text-container">
		<div class="ai-card">
			<div class="card bg-base-100 border shadow-sm">
				<figure class="px-10 pt-10">
					<img class="h-34 w-34" src={infoBoxImage} alt={wikiName} />
				</figure>
				<div class="card-body items-center text-center">
					<h2 class="card-title">{wikiName}</h2>
					<ul class="text-left opacity-95" dir="auto">
						{@html marked(infoBoxContent)}
					</ul>
				</div>
			</div>
		</div>

		<div class="space-y-4 font-light" id="wiki_body">
			{@html marked(wikiBody[isUsingBionicReading ? 1 : 0] ?? '')}
			<br /><br /><br />
			{#each Object.keys(wikiSections) as section}
				<details open>
					<summary class="text-2xl font-bold" dir="auto">{section}</summary>

					<div class="card bg-base-100 my-5 w-full border">
						<div class="card-body">
							<p class="opacity-80" dir="auto">
								{@html marked(wikiSections[section]?.summary ?? 'Generating...')}
							</p>
						</div>
					</div>

					<p dir="auto">
						{@html marked(
							(isUsingBionicReading
								? (wikiSections[section]?.body_bionic.replaceAll('__', '**') ?? '')
								: (wikiSections[section]?.body ?? '')
							).replace(new RegExp(`^###\\s*${section}\\s*`), '')
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
{:else}
	<!--History wikiName language -->
{/if}
