<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import domtoimage from 'dom-to-image';
	import toast, { Toaster } from 'svelte-french-toast';

	// Dynamic import for Cropper to avoid SSR issues
	let Cropper: any = null;

	// --------- Icons (simple inline placeholders) ----------
	const ResetIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M12 6v6l4 2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const SaveIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M17 21v-8H7v8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const ClipboardIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><rect x="8" y="2" width="8" height="4" rx="1" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const PasteIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M16 2H8a2 2 0 0 0-2 2v14" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><rect x="8" y="6" width="8" height="14" rx="2" stroke="currentColor" stroke-width="2"/></svg>`;
	const CropIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M6.13 1L6 16a2 2 0 0 0 2 2h15" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M1 6.13L16 6a2 2 0 0 1 2 2v15" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const TwitterIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M23 3a10.9 10.9 0 0 1-3.14 1.53A4.48 4.48 0 0 0 22.43 1s-2.22 1.2-3.9 1.55A4.48 4.48 0 0 0 16 1a4.48 4.48 0 0 0-4.47 4.47c0 .35.04.7.11 1.03A12.8 12.8 0 0 1 3 2s-4 9 5 13a13 13 0 0 1-7 2c9 5 20 0 20-11.5 0-.17 0-.34-.01-.51A7.18 7.18 0 0 0 23 3z" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const GithubIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M9 19c-4.97 1.5-5-2.5-7-3 0 0 1-1 2-1 0 0 1 1 3 1 2 0 3-2 6-2s4 1 7 1c0 0 1 0 1 1 0 0-2 1-6 2" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const CoffeeIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M18 8h1a3 3 0 0 1 0 6h-1" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M2 8h15v7a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const ColorPickerIcon = `<svg width="14" height="14" viewBox="0 0 24 24" fill="none"><path d="M21 11a8 8 0 1 0-8 8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;

	// ---------------- State (Svelte)
	let wrapperRef: HTMLElement | null = null;
	let blob: { src: string | null; w?: number; h?: number } = { src: null, w: 0, h: 0 };
	let bgPicker = false;

	// Cropper state
	let showCropper = false;
	// Store cropper in an object to avoid Svelte reactivity issues
	const cropperWrapper = { instance: null as any };
	let cropperImageElement: HTMLImageElement | null = null;
	let originalImageSrc: string | null = null;

	type Options = {
		aspectRatio?: string;
		customWidth?: string;
		customHeight?: string;
		theme?: any;
		customTheme?: any;
		padding?: string;
		rounded?: string;
		roundedWrapper?: string;
		shadow?: string;
		noise?: boolean;
		browserBar?: string;
		position?: string;
	};

	let options: Options = {
		aspectRatio: 'aspect-auto',
		customWidth: '',
		customHeight: '',
		theme: 'bg-gradient-to-br from-indigo-400 via-blue-400 to-purple-600',
		customTheme: {
			colorStart: '#ff40ff',
			colorEnd: '#fec700'
		},
		padding: 'p-20',
		rounded: 'rounded-xl',
		roundedWrapper: 'rounded-xl',
		shadow: 'shadow-xl',
		noise: false,
		browserBar: 'hidden',
		position: ''
	};

	const isValidHexColor = /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/;

	// helper: join classes (basic)
	function cls(...parts: Array<string | false | null | undefined | Record<string, boolean>>) {
		return parts
			.flatMap((p) => {
				if (!p) return [];
				if (typeof p === 'string') return p;
				return Object.entries(p)
					.filter(([_, v]) => v)
					.map(([k]) => k);
			})
			.join(' ');
	}

	// --------- lifecycle ----------
	onMount(async () => {
		// Load Cropper dynamically
		const cropperModule = await import('cropperjs');
		Cropper = cropperModule.default;

		// Load Cropper CSS from CDN
		if (!document.querySelector('link[href*="cropper.css"]')) {
			const link = document.createElement('link');
			link.rel = 'stylesheet';
			link.href = 'https://cdn.jsdelivr.net/npm/cropperjs@1.6.2/dist/cropper.min.css';
			document.head.appendChild(link);
		}

		// load options
		if (typeof localStorage !== 'undefined') {
			const preset = localStorage.getItem('options');
			if (preset) {
				try {
					options = JSON.parse(preset);
				} catch (e) {
					console.warn('Failed to parse options from localStorage', e);
				}
			}
		}

		window.addEventListener('keydown', handleShortcuts);
	});

	onDestroy(() => {
		if (typeof window !== 'undefined') {
			window.removeEventListener('keydown', handleShortcuts);
		}
	});

	// save options whenever they change (reactive Svelte)
	$: (function persistOptions() {
		if (typeof localStorage !== 'undefined') {
			try {
				localStorage.setItem('options', JSON.stringify(options));
			} catch (e) {
				// ignore
			}
		}
	})();

	// --------- functions (converted) ----------
	function handleShortcuts(e: KeyboardEvent) {
		if (e.key === 'c' && (e.ctrlKey || e.metaKey)) {
			e.preventDefault();
			copyImage();
		}

		if (e.key === 's' && (e.ctrlKey || e.metaKey)) {
			e.preventDefault();
			saveImage();
		}
	}

	function snapshotCreator(): Promise<Blob> {
		return new Promise((resolve, reject) => {
			try {
				const scale = window.devicePixelRatio || 1;
				const element = wrapperRef;
				if (!element) return reject(new Error('No element'));
				domtoimage
					.toBlob(element, {
						height: element.offsetHeight * scale,
						width: element.offsetWidth * scale,
						style: {
							transform: 'scale(' + scale + ')',
							transformOrigin: 'top left',
							width: element.offsetWidth + 'px',
							height: element.offsetHeight + 'px'
						}
					})
					.then((b: Blob) => resolve(b))
					.catch((err: any) => reject(err));
			} catch (e) {
				reject(e);
			}
		});
	}

	async function saveImage() {
		if (!blob?.src) {
			toast.error('Nothing to save, make sure to add a screenshot first!');
			return;
		}

		const savingToast = toast.loading('Exporting image...');
		const scale = window.devicePixelRatio || 1;

		try {
			const data = await domtoimage.toPng(wrapperRef!, {
				height: wrapperRef!.offsetHeight * scale,
				width: wrapperRef!.offsetWidth * scale,
				style: {
					transform: 'scale(' + scale + ')',
					transformOrigin: 'top left',
					width: wrapperRef!.offsetWidth + 'px',
					height: wrapperRef!.offsetHeight + 'px'
				}
			});

			const a = document.createElement('a');
			a.href = data;
			a.download = `pika-${new Date().toISOString()}.png`;
			document.body.appendChild(a);
			a.click();
			document.body.removeChild(a);

			toast.dismiss(savingToast);
			toast.success('Image exported!');
		} catch (err) {
			toast.dismiss(savingToast);
			toast.error('Failed to export image');
			console.error(err);
		}
	}

	async function copyImage() {
		if (!blob?.src) {
			toast.error('Nothing to copy, make sure to add a screenshot first!');
			return;
		}
		const isSafari = /^((?!chrome|android).)*safari/i.test(navigator?.userAgent || '');
		const isNotFirefox = (navigator?.userAgent || '').indexOf('Firefox') < 0;

		if (isSafari) {
			navigator.clipboard
				.write([
					new ClipboardItem({
						'image/png': new Promise(async (resolve, reject) => {
							try {
								const b = await snapshotCreator();
								resolve(new Blob([b], { type: 'image/png' }));
							} catch (err) {
								reject(err);
							}
						})
					})
				])
				.then(() => toast.success('Image copied to clipboard'))
				.catch((err) => {
					console.error(err);
					toast.error('Failed to copy image');
				});
		} else if (isNotFirefox) {
			try {
				const permission = await (navigator as any).permissions?.query?.({
					name: 'clipboard-write'
				});
				if (!permission || permission.state === 'granted' || permission.state === 'prompt') {
					const b = await snapshotCreator();
					const type = 'image/png';
					await navigator.clipboard.write([new ClipboardItem({ [type]: b as Blob })]);
					toast.success('Image copied to clipboard');
				} else {
					toast.error('Clipboard permission not granted');
				}
			} catch (err) {
				console.error('Copy failed', err);
				toast.error('Copy failed');
			}
		} else {
			toast.error('Firefox does not support this functionality');
		}
	}

	function onPaste(event: Event) {
		const ev: any = event;
		let items: any[] | FileList | null = null;

		if (ev.clipboardData?.items) items = ev.clipboardData.items;
		else if (ev.target?.files) items = ev.target.files;
		else if (ev.dataTransfer?.files) items = ev.dataTransfer.files;
		else items = [];

		for (let i = 0; i < items.length; i++) {
			const item = items[i];
			if (!item) continue;
			if (item.kind === 'file' || (item.type && item.type.includes('image'))) {
				const file = item.getAsFile ? item.getAsFile() : item;
				const reader = new FileReader();
				reader.onload = function (e: any) {
					// Load image directly without showing crop modal
					const img = new Image();
					img.onload = function () {
						blob = {
							src: e.target.result,
							w: img.naturalWidth,
							h: img.naturalHeight
						};
						originalImageSrc = e.target.result;
						toast.success('Image loaded!');
					};
					img.src = e.target.result;
				};
				reader.readAsDataURL(file);
			} else if (item instanceof File && item.type.includes('image')) {
				const reader = new FileReader();
				reader.onload = function (e: any) {
					// Load image directly without showing crop modal
					const img = new Image();
					img.onload = function () {
						blob = {
							src: e.target.result,
							w: img.naturalWidth,
							h: img.naturalHeight
						};
						originalImageSrc = e.target.result;
						toast.success('Image loaded!');
					};
					img.src = e.target.result;
				};
				reader.readAsDataURL(item);
			}
		}
	}

	// Cropper functions
	function initializeCropper() {
		if (!Cropper) {
			toast.error('Cropper is still loading, please wait...');
			return;
		}
		setTimeout(() => {
			if (cropperImageElement && originalImageSrc && Cropper) {
				if (cropperWrapper.instance) {
					try {
						cropperWrapper.instance.destroy();
					} catch (e) {
						// Silent cleanup
					}
				}
				const aspectRatio = getCropAspectRatioValue();
				try {
					cropperWrapper.instance = new Cropper(cropperImageElement, {
						aspectRatio: aspectRatio,
						viewMode: 1,
						autoCropArea: 1,
						responsive: true,
						ready() {
							updatePreview();
						},
						crop() {
							updatePreview();
						}
					});
				} catch (error) {
					toast.error('Failed to initialize cropper');
				}
			}
		}, 100);
	}

	function getCropAspectRatioValue(): number | undefined {
		// Always return undefined for free cropping
		return undefined;
	}

	function updatePreview() {
		if (!cropperWrapper.instance) return;

		try {
			const canvas = cropperWrapper.instance.getCroppedCanvas({
				width: options.customWidth ? parseInt(options.customWidth) : undefined,
				height: options.customHeight ? parseInt(options.customHeight) : undefined
			});

			if (!canvas) return;

			canvas.toBlob((blobData: Blob | null) => {
				if (blobData) {
					const url = URL.createObjectURL(blobData);
					blob = { src: url, w: canvas.width, h: canvas.height };
				}
			});
		} catch (error) {
			toast.error('Failed to update preview');
		}
	}

	function applyCrop() {
		if (!cropperWrapper.instance) {
			toast.error('Cropper not initialized');
			return;
		}
		updatePreview();
		showCropper = false;
		if (cropperWrapper.instance) {
			cropperWrapper.instance.destroy();
			cropperWrapper.instance = null;
		}
		toast.success('Image ready!');
	}

	function cancelCrop() {
		if (cropperWrapper.instance) {
			try {
				cropperWrapper.instance.destroy();
			} catch (e) {
				// Silent cleanup
			}
			cropperWrapper.instance = null;
		}
		showCropper = false;
		originalImageSrc = null;
		blob = { src: null, w: 0, h: 0 };
	}

	function resetCrop() {
		if (!cropperWrapper.instance) {
			toast.error('Cropper not initialized');
			return;
		}
		try {
			cropperWrapper.instance.reset();
			updatePreview();
		} catch (error) {
			toast.error('Failed to reset crop');
		}
	}

	// background picker helpers
	function toggleBG() {
		bgPicker = !bgPicker;
	}

	// handle color text input change with toast feedback
	function handleStartColorInput(e: Event) {
		const v = (e.target as HTMLInputElement).value;
		options = { ...options, customTheme: { ...(options.customTheme || {}), colorStart: v } };
		if (v.match(isValidHexColor)) {
			toast.success('First color applied');
		} else {
			toast.error('Invalid Hex color');
		}
	}

	function handleEndColorInput(e: Event) {
		const v = (e.target as HTMLInputElement).value;
		options = { ...options, customTheme: { ...(options.customTheme || {}), colorEnd: v } };
		if (v.match(isValidHexColor)) {
			toast.success('Second color applied');
		} else {
			toast.error('Invalid Hex color');
		}
	}

	function getImageRadius() {
		if (options?.padding == 'p-0') {
			return '';
		}

		switch (options?.position) {
			case 'pl-0 pt-0':
				return 'rounded-l-none rounded-tr-none';
			case 'pt-0 pr-0':
				return 'rounded-r-none rounded-tl-none';
			case 'pb-0 pl-0':
				return 'rounded-l-none rounded-br-none';
			case 'pb-0 pr-0':
				return 'rounded-r-none rounded-bl-none';
			default:
				return '';
		}
	}

	function renderBrowserBar() {
		switch (options?.browserBar) {
			case 'hidden':
				return '';
			case 'light':
				return `<div class="flex items-center w-full px-4 py-[10px] rounded-t-lg bg-white/80">
                    <div class="flex items-center space-x-2">
                        <div class="w-3 h-3 bg-red-400 rounded-full"></div>
                        <div class="w-3 h-3 bg-yellow-300 rounded-full"></div>
                        <div class="w-3 h-3 bg-green-500 rounded-full"></div>
                    </div>
                </div>`;
			case 'dark':
				return `<div class="flex items-center w-full px-4 py-[10px] rounded-t-lg bg-black/40">
                    <div class="flex items-center space-x-2">
                        <div class="w-3 h-3 bg-red-400 rounded-full"></div>
                        <div class="w-3 h-3 bg-yellow-300 rounded-full"></div>
                        <div class="w-3 h-3 bg-green-500 rounded-full"></div>
                    </div>
                </div>`;
			default:
				return '';
		}
	}

	function clearBlob() {
		blob = { src: null, w: 0, h: 0 };
	}

	function onImageLoad(e: Event) {
		const img = e.target as HTMLImageElement;
		blob = { ...blob, w: img.naturalWidth, h: img.naturalHeight };
	}
</script>

<!-- Toast container (svelte-french-toast) -->
<Toaster position="bottom-center" />

<!-- top-level wrapper listens for paste & drag/drop -->
<div
	class="pika-wrap flex h-screen flex-col items-start justify-start px-5 lg:px-10"
	on:paste|preventDefault={onPaste}
	on:dragover|preventDefault
	on:dragenter|preventDefault
	on:dragleave|preventDefault
	on:drop|preventDefault={(e) => {
		onPaste(e);
	}}
>
	<div class="relative mx-auto flex w-full max-w-[1600px] flex-col-reverse lg:flex-row-reverse">
		<div class="w-full lg:w-[350px]">
			<!-- options panel -->
			<div
				class={cls(
					'relative mt-10 h-auto w-full rounded-2xl bg-white/90 p-6 shadow-lg ring-1 shadow-gray-200 ring-pink-300 ring-offset-1 ring-offset-white lg:mt-0 lg:max-h-screen lg:min-h-[650px] lg:p-8 dark:bg-pink-600/60 dark:shadow-black dark:ring-pink-200/50 dark:ring-offset-red-800'
				)}
			>
				<div
					class="absolute inset-0 w-full scale-100 transform-gpu bg-gradient-to-br from-pink-400 to-red-300 opacity-60 blur-xl lg:h-full lg:max-h-[calc(100vh-60px)] lg:scale-x-[1.1] lg:scale-y-[1.05] dark:from-pink-500 dark:to-red-800 dark:blur-md dark:lg:scale-[1.05]"
				></div>

				<div
					class="relative flex flex-row flex-wrap items-start justify-start space-y-5 lg:flex-col lg:items-start lg:space-y-4"
				>
					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Aspect Ratio</div>
						<div>
							<select
								bind:value={options.aspectRatio}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, aspectRatio: (e.target as HTMLSelectElement).value })}
							>
								<option value="aspect-auto">Auto</option>
								<option value="aspect-square">1:1 (Square)</option>
								<option value="aspect-video">16:9 (Video)</option>
								<option value="aspect-[4/3]">4:3</option>
								<option value="aspect-[3/2]">3:2</option>
								<option value="aspect-[9/16]">9:16 (Portrait)</option>
								<option value="aspect-[21/9]">21:9 (Ultrawide)</option>
								<option value="aspect-[3/4]">3:4</option>
								<option value="aspect-[2/3]">2:3</option>
							</select>
						</div>
					</div>

					{#if blob?.src && !showCropper}
						<div class="w-full">
							<button
								on:click={() => {
									showCropper = true;
									initializeCropper();
								}}
								class="flex w-full items-center justify-center rounded-lg bg-purple-600 px-3 py-2 text-sm font-semibold text-white shadow duration-200 hover:bg-purple-700"
							>
								<span class="mr-2 h-4 w-4" innerHTML={CropIcon}></span>
								Crop Image
							</button>
						</div>
					{/if}

					<div class="w-full space-y-2">
						<div class="text-sm font-semibold dark:text-white">Custom Dimensions (optional)</div>
						<div class="flex items-center gap-2">
							<input
								type="number"
								bind:value={options.customWidth}
								placeholder="Width"
								class="w-full rounded-lg border border-gray-500 px-2 py-1 text-sm focus:ring-2 focus:ring-pink-400 focus:outline-none"
							/>
							<span class="text-sm dark:text-white">×</span>
							<input
								type="number"
								bind:value={options.customHeight}
								placeholder="Height"
								class="w-full rounded-lg border border-gray-500 px-2 py-1 text-sm focus:ring-2 focus:ring-pink-400 focus:outline-none"
							/>
						</div>
					</div>

					{#if showCropper}
						<div class="w-full space-y-2">
							<div class="flex items-center gap-2">
								<button
									on:click={applyCrop}
									class="flex-1 rounded-lg bg-pink-600 px-3 py-2 text-sm font-semibold text-white shadow duration-200 hover:bg-pink-700"
								>
									Done
								</button>
								<button
									on:click={resetCrop}
									class="rounded-lg bg-pink-100 px-3 py-2 text-sm font-semibold text-pink-600 shadow duration-200 hover:bg-pink-200"
								>
									Reset
								</button>
								<button
									on:click={cancelCrop}
									class="rounded-lg bg-gray-200 px-3 py-2 text-sm font-semibold text-gray-600 shadow duration-200 hover:bg-gray-300"
								>
									Cancel
								</button>
							</div>
						</div>
					{/if}

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Browser Wrapper</div>
						<div>
							<select
								bind:value={options.browserBar}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, browserBar: (e.target as HTMLSelectElement).value })}
							>
								<option value="hidden">None</option>
								<option value="light">Light</option>
								<option value="dark">Dark</option>
							</select>
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Padding</div>
						<div>
							<select
								bind:value={options.padding}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, padding: (e.target as HTMLSelectElement).value })}
							>
								<option value="p-0">None</option>
								<option value="p-10">Small</option>
								<option value="p-20">Medium</option>
								<option value="p-32">Large</option>
							</select>
						</div>
					</div>

					<div>
						<div class="relative flex items-center pb-2 text-sm font-semibold dark:text-white">
							Background
							<div class="relative">
								<div
									on:click={toggleBG}
									class="ml-2 flex cursor-pointer items-center rounded-lg border border-gray-400 bg-white/70 px-2 opacity-70 hover:opacity-100 dark:border-red-600 dark:bg-pink-900/80 dark:text-gray-300"
								>
									<span class="mr-1 h-3 w-3" innerHTML={ColorPickerIcon}></span>
									Pick
								</div>
							</div>

							{#if bgPicker}
								<!-- backdrop to close -->
								<div
									class="fixed inset-0 h-full w-full bg-transparent"
									on:click={() => (bgPicker = false)}
								></div>

								<div
									class={cls(
										'absolute top-[calc(100%)] left-[-30px] z-10 flex w-auto max-w-[400px] flex-col rounded-xl border border-gray-400 bg-white/80 px-5 py-4 shadow-lg shadow-gray-500/50 backdrop-blur duration-200 dark:border-gray-800 dark:bg-black/80 dark:shadow-black/80',
										{ 'pointer-events-none scale-[0.9] opacity-0': !bgPicker },
										{ 'pointer-events-auto scale-[1] opacity-100': bgPicker }
									)}
								>
									<div
										class="absolute top-[5%] right-[5%] z-10 cursor-pointer opacity-50 hover:opacity-100"
										on:click={() => (bgPicker = false)}
									>
										✕
									</div>

									<div class="relative mb-3">
										<div class="mb-1">Pick first color</div>
										<div class="flex items-center">
											<div class="group relative">
												<input
													id="startColorPicker"
													type="color"
													class="absolute top-0 left-0 h-12 w-12 cursor-pointer rounded-full opacity-0"
													value={options.customTheme?.colorStart || '#222'}
													on:input={(e) =>
														(options = {
															...options,
															customTheme: {
																...(options.customTheme || {}),
																colorStart: (e.target as HTMLInputElement).value
															}
														})}
												/>
												<label
													for="startColorPicker"
													style="background-color: {options?.customTheme?.colorStart || '#222'}"
													class="pointer-events-none left-0 flex h-12 w-12 items-center justify-center rounded-full text-white/50 duration-100 group-hover:scale-[1.1]"
												>
													<span class="font-mono text-xs text-white/80 drop-shadow">Pick</span>
												</label>
											</div>

											<span class="px-4 opacity-50">/</span>

											<input
												placeholder="Enter hex value"
												type="text"
												value={options.customTheme?.colorStart || '#000000'}
												class="rounded-lg border-2 border-gray-500 px-2 py-1 font-mono text-base text-black focus:border-black focus:outline-none"
												on:input={handleStartColorInput}
											/>
										</div>
									</div>

									<div>
										<div class="mb-1">Pick second color</div>
										<div class="flex items-center">
											<div class="group relative">
												<input
													id="endColorPicker"
													type="color"
													class="absolute top-0 left-0 h-12 w-12 cursor-pointer rounded-full opacity-0"
													value={options.customTheme?.colorEnd || '#222'}
													on:input={(e) =>
														(options = {
															...options,
															customTheme: {
																...(options.customTheme || {}),
																colorEnd: (e.target as HTMLInputElement).value
															}
														})}
												/>
												<label
													for="endColorPicker"
													style="background-color: {options?.customTheme?.colorEnd || '#222'}"
													class="pointer-events-none left-0 flex h-12 w-12 items-center justify-center rounded-full text-white/50 duration-100 group-hover:scale-[1.1]"
												>
													<span class="font-mono text-xs text-white/80 drop-shadow">Pick</span>
												</label>
											</div>

											<span class="px-4 opacity-50">/</span>

											<input
												placeholder="Enter hex value"
												type="text"
												value={options.customTheme?.colorEnd || '#000000'}
												class="rounded-lg border-2 border-gray-500 px-2 py-1 font-mono text-base text-black focus:border-black focus:outline-none"
												on:input={handleEndColorInput}
											/>
										</div>
									</div>
								</div>
							{/if}
						</div>

						<div class="mt-1 grid grid-cols-6 flex-wrap gap-x-4 gap-y-2">
							{#each ['bg-gradient-to-br from-pink-300 via-orange-200 to-red-300', 'bg-gradient-to-br from-green-300 via-yellow-200 to-green-200', 'bg-gradient-to-br from-green-200 via-blue-100 to-blue-300', 'bg-gradient-to-br from-indigo-300 via-blue-400 to-purple-500', 'bg-gradient-to-br from-red-300 via-orange-300 to-yellow-200', 'bg-gradient-to-br from-pink-300 via-pink-400 to-red-400', 'bg-gradient-to-br from-slate-400 via-gray-500 to-gray-700', 'bg-gradient-to-br from-orange-300 via-orange-400 to-red-400', 'bg-gradient-to-br from-teal-300 to-cyan-400', 'bg-gradient-to-br from-red-300 to-purple-600', 'bg-white', 'bg-black'] as theme (theme)}
								<div
									class={'h-8 w-8 cursor-pointer rounded-full shadow shadow-gray-500/20 dark:shadow-black/20 ' +
										theme}
									on:click={() => (options = { ...options, theme, customTheme: false })}
								/>
							{/each}
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Wrapper Rounded Corners</div>
						<div>
							<select
								bind:value={options.roundedWrapper}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, roundedWrapper: (e.target as HTMLSelectElement).value })}
							>
								<option value="rounded-none">None</option>
								<option value="rounded-lg">Small</option>
								<option value="rounded-xl">Medium</option>
								<option value="rounded-3xl">Large</option>
							</select>
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Screenshot Rounded Corners</div>
						<div>
							<select
								bind:value={options.rounded}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, rounded: (e.target as HTMLSelectElement).value })}
							>
								<option value="rounded-none">None</option>
								<option value="rounded-lg">Small</option>
								<option value="rounded-xl">Medium</option>
								<option value="rounded-3xl">Large</option>
							</select>
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Screenshot Position</div>
						<div>
							<select
								bind:value={options.position}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, position: (e.target as HTMLSelectElement).value })}
							>
								<option value="">Center</option>
								<option value="pl-0 pt-0">Top left</option>
								<option value="pt-0 pr-0">Top right</option>
								<option value="pb-0 pl-0">Bottom left</option>
								<option value="pb-0 pr-0">Bottom right</option>
							</select>
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Shadow</div>
						<div>
							<select
								bind:value={options.shadow}
								class="cursor-pointer appearance-none rounded-lg border border-gray-500 px-2 py-1 opacity-80 shadow-lg hover:opacity-100"
								on:change={(e) =>
									(options = { ...options, shadow: (e.target as HTMLSelectElement).value })}
							>
								<option value="shadow-none">None</option>
								<option value="shadow-lg">Small</option>
								<option value="shadow-xl">Medium</option>
								<option value="shadow-2xl">Large</option>
							</select>
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div class="text-sm font-semibold dark:text-white">Noise</div>
						<div>
							<input
								type="checkbox"
								checked={options?.noise || false}
								class="text-xl"
								on:change={() => (options = { ...options, noise: !options?.noise })}
							/>
						</div>
					</div>

					<div class="flex w-full items-center justify-between">
						<div
							class="flex w-full cursor-pointer items-center justify-center rounded-lg border border-pink-600 bg-pink-200 px-4 py-2 text-base font-semibold text-pink-600 shadow duration-200 hover:scale-[1.03] lg:text-lg"
							on:click={copyImage}
							title="Use Ctrl/Cmd + C to copy the image"
						>
							<span class="mr-2 h-6 w-6" innerHTML={ClipboardIcon}></span>
							Copy
						</div>

						<div
							class="ml-4 flex w-full cursor-pointer items-center justify-center rounded-lg border border-pink-600 bg-pink-600/90 px-4 py-2 text-base font-semibold text-pink-200 shadow duration-200 hover:scale-[1.03] lg:text-lg dark:bg-pink-600/90"
							title="Use Ctrl/Cmd + S to save the image"
							on:click={saveImage}
						>
							<span class="mr-2 h-6 w-6" innerHTML={SaveIcon}></span>
							Save
						</div>
					</div>

					<div
						on:click={clearBlob}
						class="mx-auto mt-4 flex w-full cursor-pointer items-center justify-center rounded-lg px-3 py-1 text-sm text-pink-400"
					>
						<span class="mr-1 h-4 w-4" innerHTML={ResetIcon}></span>
						Reset
					</div>

					<div class="mx-auto hidden text-center text-sm opacity-50 lg:block dark:text-white">
						<div class="mb-1">
							Use <span class="rounded-lg bg-white/80 px-2 py-px font-mono dark:bg-black/40"
								>Cmd/Ctrl+C</span
							> to copy or
						</div>
						<div>
							<span class="rounded-lg bg-white/80 px-2 py-px font-mono dark:bg-black/40"
								>Cmd/Ctrl+S</span
							> to save output image
						</div>
					</div>
				</div>
			</div>
		</div>

		<div
			class="flex w-full flex-col-reverse items-center justify-center overflow-y-auto py-5 lg:w-[calc(100%-350px)] lg:flex-col lg:p-10 lg:pl-0"
		>
			{#if showCropper}
				<!-- Inline Cropper View -->
				<div class="my-5 w-full max-w-4xl">
					<div class="overflow-hidden rounded-2xl bg-white shadow-xl dark:bg-gray-900">
						<div class="border-b border-gray-200 p-4 dark:border-gray-700">
							<h3 class="text-lg font-semibold text-gray-900 dark:text-white">Crop Your Image</h3>
							<p class="mt-1 text-sm text-gray-600 dark:text-gray-400">
								Drag to select the area you want to keep
							</p>
						</div>
						<div class="p-6">
							<div
								class="flex items-center justify-center overflow-hidden rounded-lg bg-gray-100 dark:bg-gray-800"
							>
								{#if originalImageSrc}
									<img
										bind:this={cropperImageElement}
										src={originalImageSrc}
										alt="Crop preview"
										class="max-h-[60vh] max-w-full"
									/>
								{/if}
							</div>
						</div>
					</div>
				</div>
			{:else if blob?.src}
				<!-- Preview Display -->
				<div
					class={cls(
						options?.roundedWrapper,
						'relative my-5 overflow-hidden shadow-xl duration-200 ease-in-out'
					)}
				>
					<div
						bind:this={wrapperRef}
						style={options?.customTheme
							? `background: linear-gradient(45deg, ${options.customTheme?.colorStart || 'transparent'}, ${options.customTheme?.colorEnd || 'transparent'})`
							: ''}
						class={cls(
							'relative flex max-w-[80vw] flex-col items-center justify-center overflow-hidden transition-all duration-200 ease-in-out',
							options?.aspectRatio,
							options?.padding,
							options?.position,
							options?.roundedWrapper,
							!options.customTheme ? options?.theme : ''
						)}
					>
						{@html renderBrowserBar()}

						{#if options?.noise}
							<div
								style="background-image: url('/noise.svg')"
								class={cls(
									'absolute inset-0 h-full w-full bg-repeat opacity-[0.15]',
									options?.rounded,
									options.browserBar !== 'hidden' ? 'rounded-t-none' : ''
								)}
							></div>
						{/if}

						<img
							src={blob.src}
							on:load={onImageLoad}
							class={cls(
								'relative z-10 transition-all duration-200 ease-in-out',
								options?.shadow,
								options?.rounded,
								getImageRadius(),
								options?.browserBar == 'hidden' ? '' : 'rounded-t-none'
							)}
							style={blob?.w
								? `width: ${blob.w / (typeof window !== 'undefined' ? window.devicePixelRatio || 1 : 1)}px`
								: ''}
							alt="Preview"
						/>
					</div>
				</div>
			{:else}
				<!-- Empty State -->
				<div class="flex min-h-[50vh] items-center justify-center lg:min-h-[80vh]">
					<label
						class="mt-20 flex max-w-[550px] cursor-pointer flex-col items-center justify-center rounded-2xl border-2 border-dashed border-gray-400 p-10 text-center text-lg opacity-30 duration-300 select-none hover:opacity-50 dark:text-white"
						for="imagesUpload"
					>
						<input
							class="hidden"
							id="imagesUpload"
							type="file"
							accept="image/*"
							on:change={onPaste}
						/>
						<span class="mb-2 h-6 w-6" innerHTML={PasteIcon}></span>
						<p>Paste your screenshot (Cmd/Ctrl+V)</p>
						<p>or drag and drop your screenshot here</p>
						<p>or click here to add one</p>
					</label>
				</div>
			{/if}
		</div>
	</div>
</div>

<style>
	.pika-wrap {
		min-height: 800px;
	}
</style>
