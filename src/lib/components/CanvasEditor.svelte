<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import toast, { Toaster } from 'svelte-french-toast';
	import { Button } from '$lib/components/ui/button/index.js';

	// --------- Icons ----------
	const ResetIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M12 6v6l4 2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const SaveIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M17 21v-8H7v8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const ClipboardIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><rect x="8" y="2" width="8" height="4" rx="1" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const PasteIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M16 2H8a2 2 0 0 0-2 2v14" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><rect x="8" y="6" width="8" height="14" rx="2" stroke="currentColor" stroke-width="2"/></svg>`;
	const MoveIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M5 9l-3 3 3 3M9 5l3-3 3 3M15 19l-3 3-3-3M19 9l3 3-3 3M2 12h20M12 2v20" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const TwitterIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M23 3a10.9 10.9 0 0 1-3.14 1.53A4.48 4.48 0 0 0 22.43 1s-2.22 1.2-3.9 1.55A4.48 4.48 0 0 0 16 1a4.48 4.48 0 0 0-4.47 4.47c0 .35.04.7.11 1.03A12.8 12.8 0 0 1 3 2s-4 9 5 13a13 13 0 0 1-7 2c9 5 20 0 20-11.5 0-.17 0-.34-.01-.51A7.18 7.18 0 0 0 23 3z" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const GithubIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M9 19c-4.97 1.5-5-2.5-7-3 0 0 1-1 2-1 0 0 1 1 3 1 2 0 3-2 6-2s4 1 7 1c0 0 1 0 1 1 0 0-2 1-6 2" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const CoffeeIcon = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none"><path d="M18 8h1a3 3 0 0 1 0 6h-1" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M2 8h15v7a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
	const ColorPickerIcon = `<svg width="14" height="14" viewBox="0 0 24 24" fill="none"><path d="M21 11a8 8 0 1 0-8 8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`;

	const GRADIENTS = [
		{
			class: 'bg-gradient-to-br from-pink-300 via-orange-200 to-red-300',
			stops: ['#f9a8d4', '#fed7aa', '#fca5a5']
		},
		{
			class: 'bg-gradient-to-br from-green-300 via-yellow-200 to-green-200',
			stops: ['#86efac', '#fef08a', '#bbf7d0']
		},
		{
			class: 'bg-gradient-to-br from-green-200 via-blue-100 to-blue-300',
			stops: ['#bbf7d0', '#dbeafe', '#93c5fd']
		},
		{
			class: 'bg-gradient-to-br from-indigo-300 via-blue-400 to-purple-500',
			stops: ['#a5b4fc', '#60a5fa', '#a855f7']
		},
		{
			class: 'bg-gradient-to-br from-red-300 via-orange-300 to-yellow-200',
			stops: ['#fca5a5', '#fdba74', '#fef08a']
		},
		{
			class: 'bg-gradient-to-br from-pink-300 via-pink-400 to-red-400',
			stops: ['#f9a8d4', '#f472b6', '#f87171']
		},
		{
			class: 'bg-gradient-to-br from-slate-400 via-gray-500 to-gray-700',
			stops: ['#94a3b8', '#6b7280', '#374151']
		},
		{
			class: 'bg-gradient-to-br from-orange-300 via-orange-400 to-red-400',
			stops: ['#fdba74', '#fb923c', '#f87171']
		},
		{ class: 'bg-gradient-to-br from-teal-300 to-cyan-400', stops: ['#5eead4', '#22d3ee'] },
		{ class: 'bg-gradient-to-br from-red-300 to-purple-600', stops: ['#fca5a5', '#9333ea'] }
	];

	const FONTS = [
		{ name: 'Inter', value: 'Inter, sans-serif' },
		{ name: 'Serif', value: 'serif' },
		{ name: 'Mono', value: 'monospace' },
		{ name: 'Cursive', value: 'cursive' },
		{ name: 'Fantasy', value: 'fantasy' }
	];

	// Canvas references
	let canvas: HTMLCanvasElement | null = null;
	let ctx: CanvasRenderingContext2D | null = null;
	let previewCanvas: HTMLCanvasElement | null = null;

	// Image state
	let uploadedImage: HTMLImageElement | null = null;
	let imageLoaded = false;

	// Canvas state
	let canvasWidth = 1200;
	let canvasHeight = 800;

	// Image transform state
	let imageX = 0;
	let imageY = 0;
	let imageScale = 1;
	let imageWidth = 0;
	let imageHeight = 0;

	// Overlay images (additional images)
	type OverlayImage = {
		id: string;
		image: HTMLImageElement;
		x: number;
		y: number;
		scale: number;
		rotation: number;
	};

	let overlayImages: OverlayImage[] = [];
	let selectedOverlayId: string | null = null;

	// Interaction state
	let isDragging = false;
	let isDraggingOverlay = false;
	let dragStartX = 0;
	let dragStartY = 0;
	let dragOverlayStart = { x: 0, y: 0 };
	let editMode = true;
	let isSidebarOpen = true;

	// Text state
	type TextElement = {
		id: string;
		text: string;
		x: number;
		y: number;
		fontSize: number;
		fontFamily: string;
		color: string;
		align: 'left' | 'center' | 'right';
		rotation: number;
	};

	let textElements: TextElement[] = [];
	let selectedTextId: string | null = null;
	let editingTextId: string | null = null;
	let isDraggingText = false;
	let dragTextStart = { x: 0, y: 0 };

	let bgPicker = false;

	type Options = {
		aspectRatio?: string;
		theme?: any;
		customTheme?: any;
		padding?: string;
		rounded?: string;
		roundedWrapper?: string;
		shadow?: string;
		noise?: boolean;
		browserBar?: string;
		// New options
		scale?: number;
		roundness?: number;
		rotate?: number;
		tilt?: { active: boolean; angleX: number; angleY: number };
		inset?: { active: boolean; color: string; padding: number };
		shadowBlur?: number;
		shadowOpacity?: number;
		shadowY?: number;
		frame?: string;
		gradient?: string[];
		customSize?: { width: number; height: number };
		position?: string;
	};

	let options: Options = {
		aspectRatio: 'aspect-auto',
		theme: GRADIENTS[3].class,
		gradient: GRADIENTS[3].stops,
		customTheme: {
			colorStart: '#ff40ff',
			colorEnd: '#fec700'
		},
		customSize: { width: 1200, height: 800 },
		position: 'center',
		padding: 'p-20',
		rounded: 'rounded-xl',
		roundedWrapper: 'rounded-xl',
		shadow: 'shadow-xl',
		noise: false,
		browserBar: 'hidden',
		// New defaults
		scale: 0.85,
		roundness: 20,
		rotate: 0,
		tilt: { active: false, angleX: 0, angleY: 0 },
		inset: { active: true, color: '#000000', padding: 40 },
		shadowBlur: 20,
		shadowOpacity: 0.3,
		shadowY: 10,
		frame: 'Shortboard'
	};

	const isValidHexColor = /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/;

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

	function handleResize() {
		if (canvas && imageLoaded) {
			updateCanvasSize();
		}
	}

	onMount(() => {
		if (typeof localStorage !== 'undefined') {
			const preset = localStorage.getItem('options');
			if (preset) {
				try {
					const parsed = JSON.parse(preset);
					// Merge with defaults to ensure new options exist
					options = { ...options, ...parsed };
				} catch (e) {
					console.warn('Failed to parse options from localStorage', e);
				}
			}
		}

		window.addEventListener('keydown', handleShortcuts);
		window.addEventListener('resize', handleResize);
	});

	onDestroy(() => {
		if (typeof window !== 'undefined') {
			window.removeEventListener('keydown', handleShortcuts);
			window.removeEventListener('resize', handleResize);
		}
	});

	$: (function persistOptions() {
		if (typeof localStorage !== 'undefined') {
			try {
				localStorage.setItem('options', JSON.stringify(options));
			} catch (e) {
				// ignore
			}
		}
	})();

	function initializeCanvas() {
		if (!canvas) return;
		ctx = canvas.getContext('2d', { willReadFrequently: true });
		if (ctx) {
			updateCanvasSize();
		}
	}

	// Initialize canvas when it becomes available
	$: if (canvas && !ctx) {
		initializeCanvas();
	}

	// Re-render when image is loaded
	$: if (imageLoaded && ctx) {
		render();
	}

	function getPaddingValue(): number {
		// Convert Tailwind padding classes to pixels
		switch (options.padding) {
			case 'p-0':
				return 0;
			case 'p-10':
				return 40; // 2.5rem = 40px
			case 'p-20':
				return 80; // 5rem = 80px
			case 'p-32':
				return 128; // 8rem = 128px
			default:
				return 80;
		}
	}

	function updateCanvasSize() {
		// Update canvas size based on aspect ratio
		// Use 80% of viewport width as base
		const viewportWidth = typeof window !== 'undefined' ? window.innerWidth : 1200;
		const baseWidth = Math.min(Math.floor(viewportWidth * 0.8), 1200); // 80% of viewport, max 1200px
		const baseHeight = 800;
		const padding = getPaddingValue();

		let contentWidth = baseWidth;
		let contentHeight = baseHeight;

		switch (options.aspectRatio) {
			case 'aspect-square':
				contentWidth = contentHeight = Math.min(baseWidth, baseHeight);
				break;
			case 'aspect-video': // 16:9
				contentWidth = baseWidth;
				contentHeight = Math.round((baseWidth * 9) / 16);
				break;
			case 'aspect-[4/3]':
				contentWidth = baseWidth;
				contentHeight = Math.round((baseWidth * 3) / 4);
				break;
			case 'aspect-[3/2]':
				contentWidth = baseWidth;
				contentHeight = Math.round((baseWidth * 2) / 3);
				break;
			case 'aspect-[9/16]': // Portrait
				contentWidth = Math.round((baseHeight * 9) / 16);
				contentHeight = baseHeight;
				break;
			case 'aspect-[21/9]': // Ultrawide
				contentWidth = baseWidth;
				contentHeight = Math.round((baseWidth * 9) / 21);
				break;
			case 'aspect-[3/4]':
				contentWidth = Math.round((baseHeight * 3) / 4);
				contentHeight = baseHeight;
				break;
			case 'aspect-[2/3]':
				contentWidth = Math.round((baseHeight * 2) / 3);
				contentHeight = baseHeight;
				break;
			case 'aspect-[2/3]':
				contentWidth = Math.round((baseHeight * 2) / 3);
				contentHeight = baseHeight;
				break;
			case 'custom':
				if (options.customSize) {
					contentWidth = options.customSize.width;
					contentHeight = options.customSize.height;
				}
				break;
			default: // aspect-auto
				if (uploadedImage) {
					contentWidth = uploadedImage.width;
					contentHeight = uploadedImage.height;
				} else {
					contentWidth = baseWidth;
					contentHeight = baseHeight;
				}
		}

		// Add padding to canvas size
		canvasWidth = contentWidth + padding * 2;
		canvasHeight = contentHeight + padding * 2;

		if (canvas) {
			canvas.width = canvasWidth;
			canvas.height = canvasHeight;
		}

		if (uploadedImage && imageLoaded) {
			fitImage();
			applyPosition(options.position || 'center');
		}
		render();
	}

	function fitImage() {
		if (!uploadedImage) return;
		const padding = getPaddingValue();
		const availableWidth = canvasWidth - padding * 2;
		const availableHeight = canvasHeight - padding * 2;
		const scaleX = availableWidth / uploadedImage.width;
		const scaleY = availableHeight / uploadedImage.height;
		imageScale = Math.min(scaleX, scaleY, 1);
		imageWidth = uploadedImage.width * imageScale;
		imageHeight = uploadedImage.height * imageScale;
	}

	function applyPosition(position: string = 'center') {
		if (!uploadedImage) return;

		// Effective dimensions including zoom
		const zoom = options.scale || 1;
		const effectiveW = uploadedImage.width * imageScale * zoom;
		const effectiveH = uploadedImage.height * imageScale * zoom;

		// Update state vars to match effective dims
		imageWidth = effectiveW;
		imageHeight = effectiveH;

		const padding = getPaddingValue();
		const w = effectiveW;
		const h = effectiveH;
		let cx = canvasWidth / 2;
		let cy = canvasHeight / 2;

		switch (position) {
			case 'top-left':
				cx = padding + w / 2;
				cy = padding + h / 2;
				break;
			case 'top-center':
				cx = canvasWidth / 2;
				cy = padding + h / 2;
				break;
			case 'top-right':
				cx = canvasWidth - padding - w / 2;
				cy = padding + h / 2;
				break;
			case 'center-left':
				cx = padding + w / 2;
				cy = canvasHeight / 2;
				break;
			case 'center':
				cx = canvasWidth / 2;
				cy = canvasHeight / 2;
				break;
			case 'center-right':
				cx = canvasWidth - padding - w / 2;
				cy = canvasHeight / 2;
				break;
			case 'bottom-left':
				cx = padding + w / 2;
				cy = canvasHeight - padding - h / 2;
				break;
			case 'bottom-center':
				cx = canvasWidth / 2;
				cy = canvasHeight - padding - h / 2;
				break;
			case 'bottom-right':
				cx = canvasWidth - padding - w / 2;
				cy = canvasHeight - padding - h / 2;
				break;
		}

		// Center the image within the canvas
		if (position !== 'custom') {
			imageX = cx - w / 2;
			imageY = cy - h / 2;
		}
		options = { ...options, position };
	}

	function render() {
		if (!ctx || !canvas) return;

		// Clear canvas
		ctx.clearRect(0, 0, canvasWidth, canvasHeight);

		// Draw background gradient
		const gradient = ctx.createLinearGradient(0, 0, canvasWidth, canvasHeight);
		if (options.customTheme) {
			gradient.addColorStop(0, options.customTheme.colorStart || '#ff40ff');
			gradient.addColorStop(1, options.customTheme.colorEnd || '#fec700');
		} else if (options.gradient && options.gradient.length > 0) {
			// Use mapped gradient stops
			options.gradient.forEach((stop, index) => {
				gradient.addColorStop(index / (options.gradient!.length - 1), stop);
			});
		} else {
			// Fallback
			gradient.addColorStop(0, '#6366f1');
			gradient.addColorStop(1, '#8b5cf6');
		}
		ctx.fillStyle = gradient;
		ctx.fillRect(0, 0, canvasWidth, canvasHeight);

		// Draw noise if enabled
		if (options.noise) {
			drawNoise();
		}

		// Draw image if loaded
		if (uploadedImage && imageLoaded) {
			ctx.save();

			// Center of canvas
			// Calculate center based on position
			// Use imageX and imageY as source of truth
			const w = uploadedImage.width * imageScale;
			const h = uploadedImage.height * imageScale;
			const zoom = options.scale || 1;
			const effectiveW = w * zoom;
			const effectiveH = h * zoom;

			const cx = imageX + effectiveW / 2;
			const cy = imageY + effectiveH / 2;

			ctx.translate(cx, cy);

			// Rotate
			if (options.rotate) {
				ctx.rotate((options.rotate * Math.PI) / 180);
			}

			// Scale (using the slider value)
			const scale = options.scale || 1;
			ctx.scale(scale, scale);

			// Draw Shadow
			if (options.shadowOpacity && options.shadowOpacity > 0) {
				ctx.shadowColor = `rgba(0, 0, 0, ${options.shadowOpacity})`;
				ctx.shadowBlur = options.shadowBlur || 0;
				ctx.shadowOffsetY = options.shadowY || 0;
			}

			// Draw Image with Roundness (Clipping)
			// w and h are already calculated above
			const x = -w / 2;
			const y = -h / 2;

			// Inset logic
			if (options.inset?.active) {
				// Draw inset background (e.g. black wrapper)
				ctx.fillStyle = options.inset.color;
				roundRect(
					ctx,
					x - options.inset.padding,
					y - options.inset.padding,
					w + options.inset.padding * 2,
					h + options.inset.padding * 2,
					options.roundness || 0
				);
				ctx.fill();

				// Reset shadow for the image itself so it doesn't double up?
				// Usually the shadow is on the container.
				ctx.shadowColor = 'transparent';
			}

			// Clip for image roundness
			// If inset is active, the image inside might not need roundness or maybe small roundness?
			// Let's assume the roundness applies to the wrapper if inset is active, or the image if not.
			// If inset is active, we already drew the rounded wrapper. Now we draw the image on top.
			// Let's just draw the image normally on top of the inset.

			if (!options.inset?.active) {
				// If no inset, we round the image itself
				roundRect(ctx, x, y, w, h, options.roundness || 0);
				ctx.clip();
			}

			ctx.drawImage(uploadedImage, x, y, w, h);

			ctx.restore();

			ctx.restore();
		}

		// Draw text elements
		const context = ctx!;
		textElements.forEach((text) => {
			// Skip drawing if currently editing
			if (editingTextId === text.id) return;

			context.save();

			// Translate to text position
			context.translate(text.x, text.y);

			// Apply rotation
			if (text.rotation) {
				context.rotate((text.rotation * Math.PI) / 180);
			}

			context.font = `${text.fontSize}px ${text.fontFamily}`;
			context.fillStyle = text.color;
			context.textAlign = text.align;
			context.textBaseline = 'middle';

			// Draw text at origin (0, 0) since we've translated
			context.fillText(text.text, 0, 0);

			// Draw selection box if selected
			if (selectedTextId === text.id) {
				const metrics = context.measureText(text.text);
				const width = metrics.width;
				const height = text.fontSize;
				const padding = 10;

				context.strokeStyle = '#3b82f6';
				context.lineWidth = 2;
				context.strokeRect(
					-width / 2 - padding,
					-height / 2 - padding,
					width + padding * 2,
					height + padding * 2
				);
			}
			context.restore();
		});

		// Draw overlay images
		overlayImages.forEach((overlay) => {
			context.save();

			// Translate to overlay position
			context.translate(overlay.x, overlay.y);

			// Apply rotation
			if (overlay.rotation) {
				context.rotate((overlay.rotation * Math.PI) / 180);
			}

			// Apply scale
			context.scale(overlay.scale, overlay.scale);

			// Draw image centered at origin
			const w = overlay.image.width;
			const h = overlay.image.height;
			context.drawImage(overlay.image, -w / 2, -h / 2, w, h);

			// Draw selection box if selected
			if (selectedOverlayId === overlay.id) {
				const padding = 10;
				context.strokeStyle = '#3b82f6';
				context.lineWidth = 2;
				context.strokeRect(-w / 2 - padding, -h / 2 - padding, w + padding * 2, h + padding * 2);
			}

			context.restore();
		});
	}

	function roundRect(
		ctx: CanvasRenderingContext2D,
		x: number,
		y: number,
		width: number,
		height: number,
		radius: number
	) {
		if (width < 2 * radius) radius = width / 2;
		if (height < 2 * radius) radius = height / 2;
		ctx.beginPath();
		ctx.moveTo(x + radius, y);
		ctx.arcTo(x + width, y, x + width, y + height, radius);
		ctx.arcTo(x + width, y + height, x, y + height, radius);
		ctx.arcTo(x, y + height, x, y, radius);
		ctx.arcTo(x, y, x + width, y, radius);
		ctx.closePath();
	}

	function drawNoise() {
		if (!ctx) return;
		const w = canvasWidth;
		const h = canvasHeight;
		const idata = ctx.getImageData(0, 0, w, h);
		const buffer32 = new Uint32Array(idata.data.buffer);
		const len = buffer32.length;

		for (let i = 0; i < len; i++) {
			if (Math.random() < 0.05) {
				buffer32[i] = 0xff000000 | (Math.random() * 0xffffff);
			}
		}
		// This noise is too harsh, let's use a subtle overlay instead
		// Actually, let's just draw a noise pattern image or simple random dots with low opacity
	}

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

	function onPaste(event: Event) {
		const ev: any = event;
		let items: any[] | FileList | null = null;

		if (ev.clipboardData?.items) items = ev.clipboardData.items;
		else if (ev.target?.files) items = ev.target.files;
		else if (ev.dataTransfer?.files) items = ev.dataTransfer.files;
		else items = [];

		if (!items) return;

		for (let i = 0; i < items.length; i++) {
			const item = items[i];
			if (!item) continue;
			if (item.kind === 'file' || (item.type && item.type.includes('image'))) {
				const file = item.getAsFile ? item.getAsFile() : item;
				loadImage(file);
			} else if (item instanceof File && item.type.includes('image')) {
				loadImage(item);
			}
		}
	}

	function loadImage(file: File) {
		const reader = new FileReader();
		reader.onload = function (e: any) {
			const img = new Image();
			img.onload = function () {
				uploadedImage = img;
				imageLoaded = true;

				// Wait for canvas to be ready
				setTimeout(() => {
					if (canvas && ctx) {
						updateCanvasSize();
					}
				}, 50);

				toast.success('Image loaded!');
			};
			img.src = e.target.result;
		};
		reader.readAsDataURL(file);
	}

	// Canvas interaction handlers
	function handleMouseDown(e: MouseEvent) {
		if (!editMode || !uploadedImage || !canvas) return;

		const rect = canvas.getBoundingClientRect();
		const scaleX = canvas.width / rect.width;
		const scaleY = canvas.height / rect.height;
		const mouseX = (e.clientX - rect.left) * scaleX;
		const mouseY = (e.clientY - rect.top) * scaleY;

		// Check overlay images first (on top)
		let clickedOverlayId = null;
		for (let i = overlayImages.length - 1; i >= 0; i--) {
			const overlay = overlayImages[i];
			const w = overlay.image.width * overlay.scale;
			const h = overlay.image.height * overlay.scale;
			const padding = 10;

			// Simple bounding box check (rotation not accounted for simplicity)
			if (
				mouseX >= overlay.x - w / 2 - padding &&
				mouseX <= overlay.x + w / 2 + padding &&
				mouseY >= overlay.y - h / 2 - padding &&
				mouseY <= overlay.y + h / 2 + padding
			) {
				clickedOverlayId = overlay.id;
				break;
			}
		}

		if (clickedOverlayId) {
			selectedOverlayId = clickedOverlayId;
			isDraggingOverlay = true;
			dragOverlayStart = { x: mouseX, y: mouseY };
			selectedTextId = null; // Deselect text
			render();
			return;
		}

		// Check text selection (on top of image, below overlays)
		let clickedTextId = null;
		// Use the non-null context for measuring
		const context = ctx!;

		for (let i = textElements.length - 1; i >= 0; i--) {
			const text = textElements[i];
			context.save();
			context.font = `${text.fontSize}px ${text.fontFamily}`;
			const metrics = context.measureText(text.text);
			const width = metrics.width;
			const height = text.fontSize;
			const padding = 10;
			context.restore();

			if (
				mouseX >= text.x - width / 2 - padding &&
				mouseX <= text.x + width / 2 + padding &&
				mouseY >= text.y - height / 2 - padding &&
				mouseY <= text.y + height / 2 + padding
			) {
				clickedTextId = text.id;
				break;
			}
		}

		if (clickedTextId) {
			selectedTextId = clickedTextId;
			isDraggingText = true;
			dragTextStart = { x: mouseX, y: mouseY };
			selectedOverlayId = null; // Deselect overlay
			render();
			return;
		} else {
			if (selectedTextId) {
				selectedTextId = null;
				render();
			}
			if (selectedOverlayId) {
				selectedOverlayId = null;
				render();
			}
		}

		// Check if clicking on image
		if (
			mouseX >= imageX &&
			mouseX <= imageX + imageWidth &&
			mouseY >= imageY &&
			mouseY <= imageY + imageHeight
		) {
			isDragging = true;
			dragStartX = mouseX - imageX;
			dragStartY = mouseY - imageY;
		}
	}

	function handleMouseMove(e: MouseEvent) {
		if (!canvas) return;

		const rect = canvas.getBoundingClientRect();
		const scaleX = canvas.width / rect.width;
		const scaleY = canvas.height / rect.height;
		const mouseX = (e.clientX - rect.left) * scaleX;
		const mouseY = (e.clientY - rect.top) * scaleY;

		if (isDraggingOverlay && selectedOverlayId) {
			const dx = mouseX - dragOverlayStart.x;
			const dy = mouseY - dragOverlayStart.y;

			overlayImages = overlayImages.map((overlay) =>
				overlay.id === selectedOverlayId
					? { ...overlay, x: overlay.x + dx, y: overlay.y + dy }
					: overlay
			);
			dragOverlayStart = { x: mouseX, y: mouseY };
			render();
			return;
		}

		if (isDraggingText && selectedTextId) {
			const dx = mouseX - dragTextStart.x;
			const dy = mouseY - dragTextStart.y;

			textElements = textElements.map((text) =>
				text.id === selectedTextId ? { ...text, x: text.x + dx, y: text.y + dy } : text
			);
			dragTextStart = { x: mouseX, y: mouseY };
			render();
			return;
		}

		if (!isDragging) return;

		imageX = mouseX - dragStartX;
		imageY = mouseY - dragStartY;
		options = { ...options, position: 'custom' };

		render();
	}

	function handleMouseUp() {
		isDragging = false;
		isDraggingText = false;
		isDraggingOverlay = false;
	}

	function handleDoubleClick(e: MouseEvent) {
		if (!canvas) return;

		const rect = canvas.getBoundingClientRect();
		const scaleX = canvas.width / rect.width;
		const scaleY = canvas.height / rect.height;
		const mouseX = (e.clientX - rect.left) * scaleX;
		const mouseY = (e.clientY - rect.top) * scaleY;

		// Check text selection
		const context = ctx!;
		for (let i = textElements.length - 1; i >= 0; i--) {
			const text = textElements[i];
			context.save();
			context.font = `${text.fontSize}px ${text.fontFamily}`;
			const metrics = context.measureText(text.text);
			const width = metrics.width;
			const height = text.fontSize;
			const padding = 10;
			context.restore();

			if (
				mouseX >= text.x - width / 2 - padding &&
				mouseX <= text.x + width / 2 + padding &&
				mouseY >= text.y - height / 2 - padding &&
				mouseY <= text.y + height / 2 + padding
			) {
				editingTextId = text.id;
				selectedTextId = text.id;
				render();

				// Focus the textarea after render
				setTimeout(() => {
					const textarea = document.getElementById(`text-edit-${text.id}`);
					if (textarea) {
						(textarea as HTMLTextAreaElement).focus();
						(textarea as HTMLTextAreaElement).select();
					}
				}, 0);
				return;
			}
		}
	}

	function handleWheel(e: WheelEvent) {
		if (!editMode || !uploadedImage) return;
		e.preventDefault();

		const delta = e.deltaY > 0 ? 0.95 : 1.05;
		const newScale = imageScale * delta;

		// Limit scale between 0.1 and 5
		if (newScale >= 0.1 && newScale <= 5) {
			const rect = canvas!.getBoundingClientRect();
			const mouseX = e.clientX - rect.left;
			const mouseY = e.clientY - rect.top;

			// Scale around mouse position
			const relativeX = (mouseX - imageX) / imageWidth;
			const relativeY = (mouseY - imageY) / imageHeight;

			imageScale = newScale;
			imageWidth = uploadedImage.width * imageScale;
			imageHeight = uploadedImage.height * imageScale;

			imageX = mouseX - relativeX * imageWidth;
			imageY = mouseY - relativeY * imageHeight;
			options = { ...options, position: 'custom' };

			render();
		}
	}

	function resetImageTransform() {
		fitImage();
		applyPosition('center');
		render();
		toast.success('Image reset to center');
	}

	function addText() {
		const id = Math.random().toString(36).substr(2, 9);
		const newText: TextElement = {
			id,
			text: 'New Text',
			x: canvasWidth / 2,
			y: canvasHeight / 2,
			fontSize: 40,
			fontFamily: 'Inter, sans-serif',
			color: '#000000',
			align: 'center',
			rotation: 0
		};
		textElements = [...textElements, newText];
		selectedTextId = id;
		render();
	}

	function addOverlayImage(file: File) {
		const reader = new FileReader();
		reader.onload = function (e: any) {
			const img = new Image();
			img.onload = function () {
				const id = Math.random().toString(36).substr(2, 9);
				const newOverlay: OverlayImage = {
					id,
					image: img,
					x: canvasWidth / 2,
					y: canvasHeight / 2,
					scale: Math.min(canvasWidth / img.width, canvasHeight / img.height) * 0.3, // Start at 30% of canvas
					rotation: 0
				};
				overlayImages = [...overlayImages, newOverlay];
				selectedOverlayId = id;
				render();
				toast.success('Overlay image added!');
			};
			img.src = e.target.result;
		};
		reader.readAsDataURL(file);
	}
	async function saveImage() {
		if (!canvas) {
			toast.error('Nothing to save, make sure to add a screenshot first!');
			return;
		}

		const savingToast = toast.loading('Exporting image...');

		try {
			canvas.toBlob((blob) => {
				if (!blob) {
					toast.error('Failed to export image');
					return;
				}

				const url = URL.createObjectURL(blob);
				const a = document.createElement('a');
				a.href = url;
				a.download = `pika-${new Date().toISOString()}.png`;
				document.body.appendChild(a);
				a.click();
				document.body.removeChild(a);
				URL.revokeObjectURL(url);

				toast.dismiss(savingToast);
				toast.success('Image exported!');
			}, 'image/png');
		} catch (err) {
			toast.dismiss(savingToast);
			toast.error('Failed to export image');
			console.error(err);
		}
	}

	async function copyImage() {
		if (!canvas) {
			toast.error('Nothing to copy, make sure to add a screenshot first!');
			return;
		}

		const isSafari = /^((?!chrome|android).)*safari/i.test(navigator?.userAgent || '');
		const isNotFirefox = (navigator?.userAgent || '').indexOf('Firefox') < 0;

		try {
			canvas.toBlob(async (blob) => {
				if (!blob) {
					toast.error('Failed to copy image');
					return;
				}

				if (isSafari || isNotFirefox) {
					try {
						await navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })]);
						toast.success('Image copied to clipboard');
					} catch (err) {
						console.error(err);
						toast.error('Failed to copy image');
					}
				} else {
					toast.error('Firefox does not support this functionality');
				}
			}, 'image/png');
		} catch (err) {
			console.error(err);
			toast.error('Failed to copy image');
		}
	}

	function toggleBG() {
		bgPicker = !bgPicker;
	}

	function handleStartColorInput(e: Event) {
		const v = (e.target as HTMLInputElement).value;
		options = { ...options, customTheme: { ...(options.customTheme || {}), colorStart: v } };
		if (v.match(isValidHexColor)) {
			render();
			toast.success('First color applied');
		} else {
			toast.error('Invalid Hex color');
		}
	}

	function handleEndColorInput(e: Event) {
		const v = (e.target as HTMLInputElement).value;
		options = { ...options, customTheme: { ...(options.customTheme || {}), colorEnd: v } };
		if (v.match(isValidHexColor)) {
			render();
			toast.success('Second color applied');
		} else {
			toast.error('Invalid Hex color');
		}
	}

	function clearImage() {
		uploadedImage = null;
		imageLoaded = false;
		editMode = false;
		imageX = 0;
		imageY = 0;
		imageScale = 1;
		imageWidth = 0;
		imageHeight = 0;

		// Clear canvas
		if (ctx && canvas) {
			ctx.clearRect(0, 0, canvasWidth, canvasHeight);
		}

		toast.success('Canvas cleared!');
	}

	// Reactive: Re-render when aspect ratio changes
	$: if (options.aspectRatio && canvas) {
		updateCanvasSize();
	}

	// Reactive: Re-render when padding changes
	$: if (options.padding && canvas && imageLoaded) {
		updateCanvasSize();
	}

	// Reactive: Re-render when theme changes
	$: if ((options.theme || options.customTheme) && ctx) {
		render();
	}
</script>

<Toaster position="bottom-center" />

<div
	class="pika-wrap flex h-screen flex-col items-start justify-start overflow-hidden px-5 lg:px-10"
	onpaste={(e) => {
		e.preventDefault();
		onPaste(e);
	}}
	ondragover={(e) => e.preventDefault()}
	ondragenter={(e) => e.preventDefault()}
	ondragleave={(e) => e.preventDefault()}
	ondrop={(e) => {
		e.preventDefault();
		onPaste(e);
	}}
	role="application"
	aria-label="Image editor"
>
	<div
		class="relative mx-auto flex h-full w-full max-w-[1600px] flex-col-reverse overflow-hidden lg:flex-row-reverse"
	>
		<!-- Sidebar Trigger (Mobile only or toggle) -->
		{#if !isSidebarOpen}
			<div class="fixed top-4 right-4 z-50 lg:hidden">
				<Button variant="outline" onclick={() => (isSidebarOpen = true)}>Open Settings</Button>
			</div>
		{/if}

		<!-- Static Sidebar -->
		{#if isSidebarOpen}
			<div
				class="fixed top-0 right-0 z-40 h-screen w-full overflow-y-auto border-l border-gray-200 bg-white/90 backdrop-blur-xl transition-all duration-300 lg:relative lg:z-0 lg:h-auto lg:w-[350px] dark:border-gray-800 dark:bg-gray-900/90"
			>
				<div class="p-4 lg:p-6">
					<div class="mb-4 flex items-center justify-between lg:hidden">
						<h2 class="font-semibold">Settings</h2>
						<Button variant="ghost" size="icon" onclick={() => (isSidebarOpen = false)}>
							<span class="sr-only">Close</span>
							<svg
								width="15"
								height="15"
								viewBox="0 0 15 15"
								fill="none"
								xmlns="http://www.w3.org/2000/svg"
								><path
									d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.50005L3.21846 10.9685C2.99391 11.193 2.99391 11.5571 3.21846 11.7816C3.44301 12.0062 3.80708 12.0062 4.03164 11.7816L7.50005 8.31322L10.9685 11.7816C11.193 12.0062 11.5571 12.0062 11.7816 11.7816C12.0062 11.5571 12.0062 11.193 11.7816 10.9685L8.31322 7.50005L11.7816 4.03157Z"
									fill="currentColor"
									fill-rule="evenodd"
									clip-rule="evenodd"
								></path></svg
							>
						</Button>
					</div>
					<div class="flex flex-col space-y-6">
						<!-- Frame -->
						<div class="flex items-center justify-between">
							<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Frame</span>
							<select
								bind:value={options.frame}
								class="cursor-pointer rounded border-none bg-gray-100 px-2 py-1 text-sm dark:bg-gray-800"
							>
								<option value="Shortboard">Shortboard</option>
								<option value="None">None</option>
								<option value="Browser">Browser</option>
							</select>
						</div>

						<!-- Size -->
						<div class="space-y-2">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Size</span>
								<span class="text-xs text-gray-500">{Math.round((options.scale || 1) * 100)}</span>
							</div>
							<input
								type="range"
								min="0.5"
								max="1.5"
								step="0.01"
								bind:value={options.scale}
								oninput={render}
								class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
							/>
						</div>

						<!-- Roundness -->
						<div class="space-y-2">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Roundness</span>
								<span class="text-xs text-gray-500">{options.roundness}</span>
							</div>
							<input
								type="range"
								min="0"
								max="50"
								step="1"
								bind:value={options.roundness}
								oninput={render}
								class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
							/>
						</div>

						<!-- Shadow -->
						<div class="space-y-2">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Shadow</span>
								<span class="text-xs text-gray-500">{options.shadowOpacity}</span>
							</div>
							<div class="flex items-center space-x-2">
								<input
									type="range"
									min="0"
									max="1"
									step="0.05"
									bind:value={options.shadowOpacity}
									oninput={render}
									class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
								/>
							</div>
						</div>

						<!-- Inset -->
						<div class="space-y-2">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Inset</span>
								<div class="flex items-center space-x-2">
									<input
										type="checkbox"
										bind:checked={options.inset!.active}
										onchange={render}
										class="rounded text-pink-600 focus:ring-pink-500"
									/>
									<input
										type="color"
										bind:value={options.inset!.color}
										oninput={render}
										class="h-6 w-6 cursor-pointer rounded border-none p-0"
									/>
								</div>
							</div>
							{#if options.inset?.active}
								<input
									type="range"
									min="0"
									max="100"
									step="1"
									bind:value={options.inset!.padding}
									oninput={render}
									class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
								/>
							{/if}
						</div>

						<!-- Rotate -->
						<div class="space-y-2">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Rotate</span>
								<span class="text-xs text-gray-500">{options.rotate}°</span>
							</div>
							<input
								type="range"
								min="-45"
								max="45"
								step="1"
								bind:value={options.rotate}
								oninput={render}
								class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
							/>
						</div>

						<!-- Position -->
						<div class="space-y-2">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Position</span>
							</div>
							<div class="mx-auto grid w-24 grid-cols-3 gap-1">
								{#each ['top-left', 'top-center', 'top-right', 'center-left', 'center', 'center-right', 'bottom-left', 'bottom-center', 'bottom-right'] as pos}
									<button
										class={cls(
											'flex h-6 w-6 items-center justify-center rounded border border-gray-200 hover:bg-gray-100',
											options.position === pos
												? 'bg-black text-white hover:bg-black'
												: 'bg-white text-gray-500'
										)}
										onclick={() => {
											applyPosition(pos);
											render();
										}}
										title={pos}
									>
										<div class="h-1.5 w-1.5 rounded-full bg-current"></div>
									</button>
								{/each}
							</div>
						</div>

						<!-- Tilt -->
						<div class="flex items-center justify-between">
							<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Tilt</span>
							<div class="flex items-center space-x-2">
								<!-- Placeholder for tilt position grid -->
								<div class="grid h-8 w-8 grid-cols-3 gap-0.5 rounded bg-gray-100 p-0.5">
									{#each Array(9) as _}
										<div class="rounded-[1px] bg-gray-300"></div>
									{/each}
								</div>
							</div>
						</div>

						<hr class="border-gray-200 dark:border-gray-700" />

						<!-- Text -->
						<div class="space-y-4">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Text</span>
								<Button variant="outline" size="sm" onclick={addText}>Add Text</Button>
							</div>

							{#if selectedTextId}
								{#each textElements as text}
									{#if text.id === selectedTextId}
										<div class="space-y-3 rounded-lg bg-gray-50 p-3 dark:bg-gray-800">
											<textarea
												bind:value={text.text}
												oninput={render}
												class="w-full rounded border border-gray-200 bg-white p-2 text-sm dark:border-gray-700 dark:bg-gray-900"
												rows="2"
											></textarea>

											<div class="flex space-x-2">
												<select
													bind:value={text.fontFamily}
													onchange={render}
													class="flex-1 rounded border border-gray-200 bg-white p-1 text-sm dark:border-gray-700 dark:bg-gray-900"
												>
													{#each FONTS as font}
														<option value={font.value}>{font.name}</option>
													{/each}
												</select>
												<input
													type="color"
													bind:value={text.color}
													oninput={render}
													class="h-8 w-8 cursor-pointer rounded border-none p-0"
												/>
											</div>

											<div class="space-y-1">
												<div class="flex justify-between text-xs text-gray-500">
													<span>Size</span>
													<span>{text.fontSize}px</span>
												</div>
												<input
													type="range"
													min="12"
													max="200"
													bind:value={text.fontSize}
													oninput={render}
													class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
												/>
											</div>

											<div class="space-y-1">
												<div class="flex justify-between text-xs text-gray-500">
													<span>Rotation</span>
													<span>{text.rotation}°</span>
												</div>
												<input
													type="range"
													min="-180"
													max="180"
													bind:value={text.rotation}
													oninput={render}
													class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
												/>
											</div>

											<div class="flex justify-end">
												<Button
													variant="destructive"
													size="sm"
													onclick={() => {
														textElements = textElements.filter((t) => t.id !== selectedTextId);
														selectedTextId = null;
														render();
													}}
												>
													Delete
												</Button>
											</div>
										</div>
									{/if}
								{/each}
							{/if}
						</div>

						<hr class="border-gray-200 dark:border-gray-700" />

						<!-- Overlay Images -->
						<div class="space-y-4">
							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300"
									>Overlay Images</span
								>
								<label>
									<Button
										variant="outline"
										size="sm"
										onclick={() => document.getElementById('overlay-image-input')?.click()}
									>
										Add Image
									</Button>
									<input
										id="overlay-image-input"
										type="file"
										accept="image/*"
										class="hidden"
										onchange={(e) => {
											const file = (e.target as HTMLInputElement).files?.[0];
											if (file) {
												addOverlayImage(file);
												// Reset input so same file can be selected again
												(e.target as HTMLInputElement).value = '';
											}
										}}
									/>
								</label>
							</div>

							{#if selectedOverlayId}
								{#each overlayImages as overlay}
									{#if overlay.id === selectedOverlayId}
										<div class="space-y-3 rounded-lg bg-gray-50 p-3 dark:bg-gray-800">
											<div class="space-y-1">
												<div class="flex justify-between text-xs text-gray-500">
													<span>Scale</span>
													<span>{(overlay.scale * 100).toFixed(0)}%</span>
												</div>
												<input
													type="range"
													min="0.1"
													max="2"
													step="0.05"
													bind:value={overlay.scale}
													oninput={render}
													class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
												/>
											</div>

											<div class="space-y-1">
												<div class="flex justify-between text-xs text-gray-500">
													<span>Rotation</span>
													<span>{overlay.rotation}°</span>
												</div>
												<input
													type="range"
													min="-180"
													max="180"
													bind:value={overlay.rotation}
													oninput={render}
													class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
												/>
											</div>

											<div class="flex justify-end">
												<Button
													variant="destructive"
													size="sm"
													onclick={() => {
														overlayImages = overlayImages.filter((o) => o.id !== selectedOverlayId);
														selectedOverlayId = null;
														render();
													}}
												>
													Delete
												</Button>
											</div>
										</div>
									{/if}
								{/each}
							{/if}
						</div>

						<hr class="border-gray-200 dark:border-gray-700" />

						<!-- Canvas Options -->
						<div class="space-y-4">
							<div class="space-y-2">
								<div class="flex items-center justify-between">
									<span class="text-sm font-medium text-gray-700 dark:text-gray-300"
										>Canvas Size</span
									>
									<select
										bind:value={options.aspectRatio}
										onchange={updateCanvasSize}
										class="cursor-pointer rounded border-none bg-gray-100 px-2 py-1 text-sm dark:bg-gray-800"
									>
										<option value="aspect-auto">Auto</option>
										<option value="aspect-square">Square (1:1)</option>
										<option value="aspect-video">Video (16:9)</option>
										<option value="aspect-[4/3]">4:3</option>
										<option value="aspect-[3/2]">3:2</option>
										<option value="aspect-[9/16]">9:16</option>
										<option value="custom">Custom</option>
									</select>
								</div>
								{#if options.aspectRatio === 'custom' && options.customSize}
									<div class="flex space-x-2">
										<div class="flex-1 space-y-1">
											<label class="text-xs text-gray-500">Width</label>
											<input
												type="number"
												bind:value={options.customSize.width}
												oninput={updateCanvasSize}
												class="w-full rounded border-none bg-gray-100 px-2 py-1 text-sm dark:bg-gray-800"
											/>
										</div>
										<div class="flex-1 space-y-1">
											<label class="text-xs text-gray-500">Height</label>
											<input
												type="number"
												bind:value={options.customSize.height}
												oninput={updateCanvasSize}
												class="w-full rounded border-none bg-gray-100 px-2 py-1 text-sm dark:bg-gray-800"
											/>
										</div>
									</div>
								{/if}
							</div>

							<div class="flex items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Background</span>
								<div class="flex items-center space-x-2">
									<span class="text-xs text-gray-500">Gradient</span>
									<div
										class="h-6 w-6 rounded-full border border-gray-200 bg-gradient-to-br from-blue-400 to-purple-500"
									></div>
								</div>
							</div>

							<!-- Background Picker -->
							<div class="grid grid-cols-6 gap-2">
								{#each GRADIENTS as gradient (gradient.class)}
									<button
										class={'h-6 w-6 cursor-pointer rounded-full shadow-sm ' + gradient.class}
										onclick={() => {
											options = {
												...options,
												theme: gradient.class,
												gradient: gradient.stops,
												customTheme: false
											};
											render();
										}}
										aria-label="Select theme"
									></button>
								{/each}
							</div>

							<div class="flex cursor-pointer items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Pattern</span>
								<span class="text-xs text-gray-500">None</span>
							</div>
						</div>

						<hr class="border-gray-200 dark:border-gray-700" />

						<!-- Extras -->
						<div class="space-y-4">
							<div class="flex cursor-pointer items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Add Text</span>
								<span class="text-gray-400">›</span>
							</div>
							<div class="flex cursor-pointer items-center justify-between">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Watermark</span>
								<span class="text-gray-400">›</span>
							</div>
						</div>

						<hr class="border-gray-200 dark:border-gray-700" />

						<!-- Noise -->
						<div class="flex items-center justify-between">
							<div class="flex items-center space-x-2">
								<span class="text-sm font-medium text-gray-700 dark:text-gray-300">Noise</span>
							</div>
							<input
								type="checkbox"
								bind:checked={options.noise}
								onchange={render}
								class="rounded text-pink-600 focus:ring-pink-500"
							/>
						</div>

						<!-- Actions -->
						<div class="flex space-x-2 pt-4">
							<button
								class="flex flex-1 items-center justify-center rounded-lg bg-gray-100 px-4 py-2 text-sm font-semibold text-gray-700 duration-200 hover:bg-gray-200"
								onclick={copyImage}
								title="Use Ctrl/Cmd + C to copy the image"
							>
								<span class="mr-2 h-4 w-4">{@html ClipboardIcon}</span>
								Copy
							</button>
							<button
								class="flex flex-1 items-center justify-center rounded-lg bg-black px-4 py-2 text-sm font-semibold text-white duration-200 hover:bg-gray-800"
								title="Use Ctrl/Cmd + S to save the image"
								onclick={saveImage}
							>
								<span class="mr-2 h-4 w-4">{@html SaveIcon}</span>
								Save
							</button>
						</div>
					</div>
				</div>
			</div>
		{/if}

		<div
			class="flex h-full w-full flex-col items-center justify-center overflow-hidden bg-gray-50 py-5 lg:w-full lg:p-10 lg:pl-0 dark:bg-black"
		>
			{#if imageLoaded}
				<div class="my-5">
					<canvas
						bind:this={canvas}
						class:view-mode={!editMode}
						onmousedown={handleMouseDown}
						onmousemove={handleMouseMove}
						onmouseup={handleMouseUp}
						onmouseleave={handleMouseUp}
						ondblclick={handleDoubleClick}
						class={cls('relative shadow-xl', options?.rounded)}
						style="display: block; max-width: 100%; height: auto;"
					></canvas>

					<!-- Text Editing Overlays -->
					{#if canvas}
						{#each textElements as text}
							{#if editingTextId === text.id}
								{@const rect = canvas.getBoundingClientRect()}
								{@const scaleX = rect.width / canvas.width}
								{@const scaleY = rect.height / canvas.height}
								<textarea
									id="text-edit-{text.id}"
									value={text.text}
									oninput={(e) => {
										text.text = e.currentTarget.value;
										render(); // Re-render to update metrics if needed (though hidden)
									}}
									onblur={() => {
										editingTextId = null;
										render();
									}}
									onkeydown={(e) => {
										if (e.key === 'Enter' && !e.shiftKey) {
											e.preventDefault();
											editingTextId = null;
											render();
										}
									}}
									style="
                                        position: absolute;
                                        left: {text.x * scaleX}px;
                                        top: {text.y * scaleY}px;
                                        transform: translate(-50%, -50%);
                                        font-size: {text.fontSize * scaleX}px;
                                        font-family: {text.fontFamily};
                                        color: {text.color};
                                        background: transparent;
                                        border: 1px dashed #3b82f6;
                                        outline: none;
                                        text-align: {text.align};
                                        resize: none;
                                        overflow: hidden;
                                        white-space: nowrap;
                                        z-index: 50;
                                        padding: 0;
                                        margin: 0;
                                    "
								></textarea>
							{/if}
						{/each}
					{/if}
				</div>
			{:else}
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
							onchange={onPaste}
						/>
						<span class="mb-2 h-6 w-6">{@html PasteIcon}</span>
						<p>Paste your screenshot (Cmd/Ctrl+V)</p>
						<p>or drag and drop your screenshot here</p>
						<p>or click here to add one</p>
					</label>
				</div>
			{/if}

			<div
				class="flex flex-col items-center justify-center pb-5 text-sm opacity-60 lg:absolute lg:bottom-[20px] lg:flex-row lg:pt-20 lg:pb-0 dark:text-gray-400"
			>
				<a
					href="https://twitter.com/thelifeofrishi"
					target="_blank"
					class="flex items-center hover:underline"
				>
					<span class="mx-1 h-5 w-5">{@html TwitterIcon}</span>
					Created by Rishi Mohan
				</a>
				<span class="hidden px-2 lg:block">-</span>
				<a
					href="https://github.com/rishimohan/pika"
					target="_blank"
					class="mt-2 flex items-center hover:underline lg:mt-0"
				>
					<span class="mx-1 h-5 w-5">{@html GithubIcon}</span>
					View Code on Github
				</a>
				<span class="hidden px-2 lg:block">-</span>
				<a
					href="https://www.buymeacoffee.com/thelifeofrishi"
					target="_blank"
					class="mt-2 flex items-center hover:underline lg:mt-0"
				>
					<span class="mx-1 h-5 w-5">{@html CoffeeIcon}</span>
					Buy me a coffee
				</a>
			</div>
		</div>
	</div>
</div>

<style>
	.pika-wrap {
		min-height: 800px;
	}
	canvas {
		max-width: 100%;
		height: auto;
		cursor: move;
	}
	canvas.view-mode {
		cursor: default;
	}
</style>
