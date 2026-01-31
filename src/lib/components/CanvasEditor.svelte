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
		{ class: 'bg-gradient-to-br from-violet-500 via-purple-500 to-fuchsia-500', stops: ['#8b5cf6', '#a855f7', '#d946ef'] },
		{ class: 'bg-gradient-to-br from-blue-500 via-cyan-400 to-teal-400', stops: ['#3b82f6', '#22d3ee', '#2dd4bf'] },
		{ class: 'bg-gradient-to-br from-rose-400 via-pink-500 to-purple-500', stops: ['#fb7185', '#ec4899', '#a855f7'] },
		{ class: 'bg-gradient-to-br from-orange-400 via-amber-400 to-yellow-300', stops: ['#fb923c', '#fbbf24', '#fde047'] },
		{ class: 'bg-gradient-to-br from-emerald-400 via-teal-400 to-cyan-400', stops: ['#34d399', '#2dd4bf', '#22d3ee'] },
		{ class: 'bg-gradient-to-br from-slate-800 via-slate-700 to-slate-600', stops: ['#1e293b', '#334155', '#475569'] },
		{ class: 'bg-gradient-to-br from-indigo-600 via-purple-600 to-pink-500', stops: ['#4f46e5', '#9333ea', '#ec4899'] },
		{ class: 'bg-gradient-to-br from-sky-400 via-blue-500 to-indigo-600', stops: ['#38bdf8', '#3b82f6', '#4f46e5'] },
		{ class: 'bg-gradient-to-br from-lime-400 via-green-400 to-emerald-500', stops: ['#a3e635', '#4ade80', '#10b981'] },
		{ class: 'bg-gradient-to-br from-red-500 via-orange-500 to-amber-500', stops: ['#ef4444', '#f97316', '#f59e0b'] },
		{ class: 'bg-gradient-to-br from-fuchsia-500 via-pink-500 to-rose-400', stops: ['#d946ef', '#ec4899', '#fb7185'] },
		{ class: 'bg-gradient-to-br from-cyan-400 via-sky-500 to-blue-600', stops: ['#22d3ee', '#0ea5e9', '#2563eb'] },
		{ class: 'bg-gradient-to-br from-zinc-900 via-neutral-800 to-stone-700', stops: ['#18181b', '#262626', '#44403c'] },
		{ class: 'bg-gradient-to-br from-amber-200 via-orange-300 to-rose-300', stops: ['#fde68a', '#fdba74', '#fda4af'] },
		{ class: 'bg-gradient-to-br from-teal-200 via-cyan-200 to-sky-300', stops: ['#99f6e4', '#a5f3fc', '#7dd3fc'] }
	];

	const FONTS = [
		{ name: 'Inter', value: 'Inter, sans-serif' },
		{ name: 'System', value: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif' },
		{ name: 'Georgia', value: 'Georgia, serif' },
		{ name: 'Helvetica', value: 'Helvetica Neue, Helvetica, Arial, sans-serif' },
		{ name: 'Times', value: 'Times New Roman, Times, serif' },
		{ name: 'Courier', value: 'Courier New, Courier, monospace' },
		{ name: 'Monaco', value: 'Monaco, Consolas, monospace' },
		{ name: 'Verdana', value: 'Verdana, Geneva, sans-serif' },
		{ name: 'Trebuchet', value: 'Trebuchet MS, sans-serif' },
		{ name: 'Impact', value: 'Impact, Haettenschweiler, sans-serif' }
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

	// Alignment guides
	type AlignmentGuide = {
		type: 'horizontal' | 'vertical';
		position: number;
		label?: string;
	};
	let alignmentGuides: AlignmentGuide[] = [];
	const SNAP_THRESHOLD = 8;

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
		aspectRatio: 'aspect-video',
		theme: GRADIENTS[0].class,
		gradient: GRADIENTS[0].stops,
		customTheme: false,
		customSize: { width: 1600, height: 900 },
		position: 'center',
		padding: 'p-20',
		rounded: 'rounded-xl',
		roundedWrapper: 'rounded-xl',
		shadow: 'shadow-xl',
		noise: false,
		browserBar: 'hidden',
		scale: 0.9,
		roundness: 16,
		rotate: 0,
		tilt: { active: false, angleX: 0, angleY: 0 },
		inset: { active: true, color: '#000000', padding: 12 },
		shadowBlur: 40,
		shadowOpacity: 0.4,
		shadowY: 20,
		frame: 'None'
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
			case '1600x900':
				canvasWidth = 1600;
				canvasHeight = 900;
				contentWidth = canvasWidth - padding * 2;
				contentHeight = canvasHeight - padding * 2;
				break;
			case '1200x1200':
				canvasWidth = 1200;
				canvasHeight = 1200;
				contentWidth = canvasWidth - padding * 2;
				contentHeight = canvasHeight - padding * 2;
				break;
			case 'custom':
				if (options.customSize) {
					// For custom size, use the exact dimensions as final canvas size
					// (not as content area, to match user expectations)
					canvasWidth = options.customSize.width;
					canvasHeight = options.customSize.height;
					contentWidth = canvasWidth - padding * 2;
					contentHeight = canvasHeight - padding * 2;
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

		// Add padding to canvas size (except for custom, which is already final)
		if (options.aspectRatio !== 'custom') {
			canvasWidth = contentWidth + padding * 2;
			canvasHeight = contentHeight + padding * 2;
		}

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
				// Draw inset background (e.g. black wrapper with roundness)
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

				// Reset shadow for the image itself so it doesn't double up
				ctx.shadowColor = 'transparent';
			}

			// Apply roundness clipping to the image
			// Calculate inner roundness (slightly smaller for inset to look natural)
			const innerRoundness = options.inset?.active
				? Math.max(0, (options.roundness || 0) - options.inset.padding / 2)
				: options.roundness || 0;

			// Clip the image to rounded rectangle
			ctx.save();
			roundRect(ctx, x, y, w, h, innerRoundness);
			ctx.clip();

			ctx.drawImage(uploadedImage, x, y, w, h);

			ctx.restore();
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

		// Draw alignment guides last (on top of everything)
		drawAlignmentGuides();
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
	}

	function calculateAlignmentGuides(elementX: number, elementY: number, elementW: number, elementH: number): { guides: AlignmentGuide[], snappedX: number, snappedY: number } {
		const guides: AlignmentGuide[] = [];
		let snappedX = elementX;
		let snappedY = elementY;
		const padding = getPaddingValue();

		const centerX = canvasWidth / 2;
		const centerY = canvasHeight / 2;

		const elementCenterX = elementX + elementW / 2;
		const elementCenterY = elementY + elementH / 2;
		const elementRight = elementX + elementW;
		const elementBottom = elementY + elementH;

		// Vertical center line
		if (Math.abs(elementCenterX - centerX) < SNAP_THRESHOLD) {
			guides.push({ type: 'vertical', position: centerX, label: 'center' });
			snappedX = centerX - elementW / 2;
		}

		// Horizontal center line
		if (Math.abs(elementCenterY - centerY) < SNAP_THRESHOLD) {
			guides.push({ type: 'horizontal', position: centerY, label: 'center' });
			snappedY = centerY - elementH / 2;
		}

		// Left edge to padding
		if (Math.abs(elementX - padding) < SNAP_THRESHOLD) {
			guides.push({ type: 'vertical', position: padding });
			snappedX = padding;
		}

		// Right edge to padding
		if (Math.abs(elementRight - (canvasWidth - padding)) < SNAP_THRESHOLD) {
			guides.push({ type: 'vertical', position: canvasWidth - padding });
			snappedX = canvasWidth - padding - elementW;
		}

		// Top edge to padding
		if (Math.abs(elementY - padding) < SNAP_THRESHOLD) {
			guides.push({ type: 'horizontal', position: padding });
			snappedY = padding;
		}

		// Bottom edge to padding
		if (Math.abs(elementBottom - (canvasHeight - padding)) < SNAP_THRESHOLD) {
			guides.push({ type: 'horizontal', position: canvasHeight - padding });
			snappedY = canvasHeight - padding - elementH;
		}

		return { guides, snappedX, snappedY };
	}

	function drawAlignmentGuides() {
		if (!ctx || alignmentGuides.length === 0) return;

		ctx.save();
		ctx.strokeStyle = '#3b82f6';
		ctx.lineWidth = 1;
		ctx.setLineDash([5, 5]);

		alignmentGuides.forEach(guide => {
			ctx!.beginPath();
			if (guide.type === 'vertical') {
				ctx!.moveTo(guide.position, 0);
				ctx!.lineTo(guide.position, canvasHeight);
			} else {
				ctx!.moveTo(0, guide.position);
				ctx!.lineTo(canvasWidth, guide.position);
			}
			ctx!.stroke();
		});

		ctx.restore();
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
			const overlay = overlayImages.find(o => o.id === selectedOverlayId);
			if (overlay) {
				const dx = mouseX - dragOverlayStart.x;
				const dy = mouseY - dragOverlayStart.y;
				const newX = overlay.x + dx;
				const newY = overlay.y + dy;
				const w = overlay.image.width * overlay.scale;
				const h = overlay.image.height * overlay.scale;

				const { guides, snappedX, snappedY } = calculateAlignmentGuides(
					newX - w / 2, newY - h / 2, w, h
				);
				alignmentGuides = guides;

				overlayImages = overlayImages.map((o) =>
					o.id === selectedOverlayId
						? { ...o, x: snappedX + w / 2, y: snappedY + h / 2 }
						: o
				);
			}
			dragOverlayStart = { x: mouseX, y: mouseY };
			render();
			return;
		}

		if (isDraggingText && selectedTextId) {
			const text = textElements.find(t => t.id === selectedTextId);
			if (text && ctx) {
				const dx = mouseX - dragTextStart.x;
				const dy = mouseY - dragTextStart.y;
				const newX = text.x + dx;
				const newY = text.y + dy;

				ctx.save();
				ctx.font = `${text.fontSize}px ${text.fontFamily}`;
				const metrics = ctx.measureText(text.text);
				const w = metrics.width;
				const h = text.fontSize;
				ctx.restore();

				const { guides, snappedX, snappedY } = calculateAlignmentGuides(
					newX - w / 2, newY - h / 2, w, h
				);
				alignmentGuides = guides;

				textElements = textElements.map((t) =>
					t.id === selectedTextId
						? { ...t, x: snappedX + w / 2, y: snappedY + h / 2 }
						: t
				);
			}
			dragTextStart = { x: mouseX, y: mouseY };
			render();
			return;
		}

		if (!isDragging) return;

		const newX = mouseX - dragStartX;
		const newY = mouseY - dragStartY;

		const { guides, snappedX, snappedY } = calculateAlignmentGuides(
			newX, newY, imageWidth, imageHeight
		);
		alignmentGuides = guides;

		imageX = snappedX;
		imageY = snappedY;
		options = { ...options, position: 'custom' };

		render();
	}

	function handleMouseUp() {
		isDragging = false;
		isDraggingText = false;
		isDraggingOverlay = false;
		alignmentGuides = [];
		render();
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

				// Focus the input after render
				setTimeout(() => {
					const input = document.getElementById(`text-edit-${text.id}`) as HTMLInputElement;
					if (input) {
						input.focus();
						input.select();
					}
				}, 50);
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
		const padding = getPaddingValue();
		const newText: TextElement = {
			id,
			text: 'Your text here',
			x: canvasWidth / 2,
			y: padding / 2 + 20,
			fontSize: 48,
			fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
			color: '#ffffff',
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

	function resetAll() {
		uploadedImage = null;
		imageLoaded = false;
		editMode = true;
		imageX = 0;
		imageY = 0;
		imageScale = 1;
		imageWidth = 0;
		imageHeight = 0;
		textElements = [];
		selectedTextId = null;
		editingTextId = null;
		overlayImages = [];
		selectedOverlayId = null;
		alignmentGuides = [];

		if (ctx && canvas) {
			ctx.clearRect(0, 0, canvasWidth, canvasHeight);
		}

		toast.success('Canvas cleared! Settings preserved.');
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
				<div class="flex h-full flex-col">
					<!-- Header -->
					<div class="flex items-center justify-between border-b border-gray-100 px-5 py-4 dark:border-gray-800">
						<h2 class="text-base font-semibold text-gray-900 dark:text-white">Editor</h2>
						<div class="flex items-center gap-2">
							<button
								class="rounded-lg p-2 text-gray-400 transition-colors hover:bg-gray-100 hover:text-gray-600 dark:hover:bg-gray-800"
								onclick={resetAll}
								title="Reset everything"
							>
								<svg class="h-4 w-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
									<path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
									<path d="M3 3v5h5" />
								</svg>
							</button>
							<button class="rounded-lg p-2 text-gray-400 transition-colors hover:bg-gray-100 hover:text-gray-600 lg:hidden dark:hover:bg-gray-800" onclick={() => (isSidebarOpen = false)}>
								<svg width="16" height="16" viewBox="0 0 15 15" fill="none"><path d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.50005L3.21846 10.9685C2.99391 11.193 2.99391 11.5571 3.21846 11.7816C3.44301 12.0062 3.80708 12.0062 4.03164 11.7816L7.50005 8.31322L10.9685 11.7816C11.193 12.0062 11.5571 12.0062 11.7816 11.7816C12.0062 11.5571 12.0062 11.193 11.7816 10.9685L8.31322 7.50005L11.7816 4.03157Z" fill="currentColor" fill-rule="evenodd" clip-rule="evenodd"></path></svg>
							</button>
						</div>
					</div>

					<!-- Scrollable Content -->
					<div class="flex-1 overflow-y-auto">
						<div class="space-y-1 p-3">
							<!-- Quick Actions - Add Elements -->
							<div class="rounded-xl bg-gradient-to-r from-indigo-50 to-purple-50 p-4 dark:from-indigo-950/30 dark:to-purple-950/30">
								<p class="mb-3 text-xs font-medium uppercase tracking-wide text-gray-500 dark:text-gray-400">Add Elements</p>
								<div class="grid grid-cols-2 gap-2">
									<button
										class="flex items-center justify-center gap-2 rounded-lg border border-gray-200 bg-white px-3 py-2.5 text-sm font-medium text-gray-700 shadow-sm transition-all hover:border-gray-300 hover:shadow dark:border-gray-700 dark:bg-gray-800 dark:text-gray-200"
										onclick={addText}
									>
										<svg class="h-4 w-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
											<polyline points="4 7 4 4 20 4 20 7"></polyline>
											<line x1="9" y1="20" x2="15" y2="20"></line>
											<line x1="12" y1="4" x2="12" y2="20"></line>
										</svg>
										Text
									</button>
									<label class="flex cursor-pointer items-center justify-center gap-2 rounded-lg border border-gray-200 bg-white px-3 py-2.5 text-sm font-medium text-gray-700 shadow-sm transition-all hover:border-gray-300 hover:shadow dark:border-gray-700 dark:bg-gray-800 dark:text-gray-200">
										<svg class="h-4 w-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
											<rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
											<circle cx="8.5" cy="8.5" r="1.5"></circle>
											<polyline points="21 15 16 10 5 21"></polyline>
										</svg>
										Image
										<input
											type="file"
											accept="image/*"
											class="hidden"
											onchange={(e) => {
												const file = (e.target as HTMLInputElement).files?.[0];
												if (file) {
													addOverlayImage(file);
													(e.target as HTMLInputElement).value = '';
												}
											}}
										/>
									</label>
								</div>
							</div>

							<!-- Selected Text Controls -->
							{#if selectedTextId}
								{@const selectedText = textElements.find(t => t.id === selectedTextId)}
								{#if selectedText}
									<div class="rounded-xl border border-blue-200 bg-blue-50 p-4 dark:border-blue-800 dark:bg-blue-950/30">
										<div class="mb-3 flex items-center justify-between">
											<p class="text-xs font-medium uppercase tracking-wide text-blue-600 dark:text-blue-400">Text Properties</p>
											<button
												class="text-xs text-red-500 hover:text-red-600"
												onclick={() => {
													textElements = textElements.filter(t => t.id !== selectedTextId);
													selectedTextId = null;
													render();
												}}
											>Delete</button>
										</div>
										<div class="space-y-3">
											<div class="flex gap-2">
												<select
													bind:value={selectedText.fontFamily}
													onchange={render}
													class="flex-1 rounded-lg border-none bg-white px-2 py-1.5 text-sm dark:bg-gray-800"
												>
													{#each FONTS as font}
														<option value={font.value}>{font.name}</option>
													{/each}
												</select>
												<input
													type="number"
													bind:value={selectedText.fontSize}
													oninput={render}
													class="w-16 rounded-lg border-none bg-white px-2 py-1.5 text-sm dark:bg-gray-800"
													min="10"
													max="200"
												/>
												<input
													type="color"
													bind:value={selectedText.color}
													oninput={render}
													class="h-8 w-8 cursor-pointer rounded-lg border-none p-0"
												/>
											</div>
											<div class="flex gap-1">
												{#each ['left', 'center', 'right'] as align}
													<button
														class={cls(
															'flex-1 rounded-lg py-1.5 text-xs font-medium transition-colors',
															selectedText.align === align
																? 'bg-blue-500 text-white'
																: 'bg-white text-gray-600 hover:bg-gray-100 dark:bg-gray-800 dark:text-gray-300'
														)}
														onclick={() => {
															selectedText.align = align as 'left' | 'center' | 'right';
															render();
														}}
													>
														{align.charAt(0).toUpperCase() + align.slice(1)}
													</button>
												{/each}
											</div>
										</div>
									</div>
								{/if}
							{/if}

							<!-- Selected Overlay Controls -->
							{#if selectedOverlayId}
								{@const selectedOverlay = overlayImages.find(o => o.id === selectedOverlayId)}
								{#if selectedOverlay}
									<div class="rounded-xl border border-green-200 bg-green-50 p-4 dark:border-green-800 dark:bg-green-950/30">
										<div class="mb-3 flex items-center justify-between">
											<p class="text-xs font-medium uppercase tracking-wide text-green-600 dark:text-green-400">Overlay Image</p>
											<button
												class="text-xs text-red-500 hover:text-red-600"
												onclick={() => {
													overlayImages = overlayImages.filter(o => o.id !== selectedOverlayId);
													selectedOverlayId = null;
													render();
												}}
											>Delete</button>
										</div>
										<div class="space-y-3">
											<div class="space-y-1">
												<div class="flex justify-between text-xs text-gray-600 dark:text-gray-400">
													<span>Scale</span>
													<span>{(selectedOverlay.scale * 100).toFixed(0)}%</span>
												</div>
												<input type="range" min="0.1" max="2" step="0.05" bind:value={selectedOverlay.scale} oninput={render} class="slider-input" />
											</div>
											<div class="space-y-1">
												<div class="flex justify-between text-xs text-gray-600 dark:text-gray-400">
													<span>Rotation</span>
													<span>{selectedOverlay.rotation}°</span>
												</div>
												<input type="range" min="-180" max="180" bind:value={selectedOverlay.rotation} oninput={render} class="slider-input" />
											</div>
										</div>
									</div>
								{/if}
							{/if}

							<!-- Image Controls -->
							<details class="group rounded-xl bg-white shadow-sm dark:bg-gray-800/50" open>
								<summary class="flex cursor-pointer items-center justify-between px-4 py-3 text-sm font-medium text-gray-900 dark:text-white">
									<span class="flex items-center gap-2">
										<svg class="h-4 w-4 text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><path d="m21 15-5-5L5 21"/></svg>
										Image
									</span>
									<svg class="h-4 w-4 text-gray-400 transition-transform group-open:rotate-180" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m6 9 6 6 6-6"/></svg>
								</summary>
								<div class="space-y-4 border-t border-gray-100 px-4 py-4 dark:border-gray-700">
									<div class="space-y-2">
										<div class="flex items-center justify-between text-xs text-gray-500">
											<span>Size</span>
											<span>{Math.round((options.scale || 1) * 100)}%</span>
										</div>
										<input type="range" min="0.5" max="1.5" step="0.01" bind:value={options.scale} oninput={render} class="slider-input" />
									</div>
									<div class="space-y-2">
										<div class="flex items-center justify-between text-xs text-gray-500">
											<span>Roundness</span>
											<span>{options.roundness}px</span>
										</div>
										<input type="range" min="0" max="50" step="1" bind:value={options.roundness} oninput={render} class="slider-input" />
									</div>
									<div class="space-y-2">
										<div class="flex items-center justify-between text-xs text-gray-500">
											<span>Shadow</span>
											<span>{Math.round((options.shadowOpacity || 0) * 100)}%</span>
										</div>
										<input type="range" min="0" max="1" step="0.05" bind:value={options.shadowOpacity} oninput={render} class="slider-input" />
									</div>
									<div class="space-y-2">
										<div class="flex items-center justify-between">
											<span class="text-xs text-gray-500">Inset Border</span>
											<div class="flex items-center gap-2">
												<input type="color" bind:value={options.inset!.color} oninput={render} class="h-6 w-6 cursor-pointer rounded border-none p-0" />
												<input type="checkbox" bind:checked={options.inset!.active} onchange={render} class="rounded text-indigo-600" />
											</div>
										</div>
										{#if options.inset?.active}
											<input type="range" min="0" max="100" step="1" bind:value={options.inset!.padding} oninput={render} class="slider-input" />
										{/if}
									</div>
								</div>
							</details>

							<!-- Position & Transform -->
							<details class="group rounded-xl bg-white shadow-sm dark:bg-gray-800/50" open>
								<summary class="flex cursor-pointer items-center justify-between px-4 py-3 text-sm font-medium text-gray-900 dark:text-white">
									<span class="flex items-center gap-2">
										<svg class="h-4 w-4 text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 9l-3 3 3 3M9 5l3-3 3 3M15 19l-3 3-3-3M19 9l3 3-3 3M2 12h20M12 2v20"/></svg>
										Position
									</span>
									<svg class="h-4 w-4 text-gray-400 transition-transform group-open:rotate-180" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m6 9 6 6 6-6"/></svg>
								</summary>
								<div class="space-y-4 border-t border-gray-100 px-4 py-4 dark:border-gray-700">
									<div class="flex items-center gap-4">
										<div class="grid w-[72px] grid-cols-3 gap-1">
											{#each ['top-left', 'top-center', 'top-right', 'center-left', 'center', 'center-right', 'bottom-left', 'bottom-center', 'bottom-right'] as pos}
												<button
													class={cls(
														'flex h-5 w-5 items-center justify-center rounded transition-all',
														options.position === pos
															? 'bg-indigo-500 shadow-sm'
															: 'bg-gray-100 hover:bg-gray-200 dark:bg-gray-700 dark:hover:bg-gray-600'
													)}
													onclick={() => { applyPosition(pos); render(); }}
													title={pos}
												>
													<div class={cls('h-1.5 w-1.5 rounded-full', options.position === pos ? 'bg-white' : 'bg-gray-400')}></div>
												</button>
											{/each}
										</div>
										<div class="flex-1 space-y-1">
											<div class="flex justify-between text-xs text-gray-500">
												<span>Rotate</span>
												<span>{options.rotate}°</span>
											</div>
											<input type="range" min="-45" max="45" step="1" bind:value={options.rotate} oninput={render} class="slider-input" />
										</div>
									</div>
								</div>
							</details>

							<!-- Canvas Settings -->
							<details class="group rounded-xl bg-white shadow-sm dark:bg-gray-800/50">
								<summary class="flex cursor-pointer items-center justify-between px-4 py-3 text-sm font-medium text-gray-900 dark:text-white">
									<span class="flex items-center gap-2">
										<svg class="h-4 w-4 text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2"/></svg>
										Canvas
									</span>
									<svg class="h-4 w-4 text-gray-400 transition-transform group-open:rotate-180" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m6 9 6 6 6-6"/></svg>
								</summary>
								<div class="space-y-4 border-t border-gray-100 px-4 py-4 dark:border-gray-700">
									<div class="flex items-center justify-between">
										<span class="text-xs text-gray-500">Size</span>
										<select bind:value={options.aspectRatio} onchange={updateCanvasSize} class="rounded-lg border-none bg-gray-100 px-2 py-1 text-xs dark:bg-gray-700">
											<option value="aspect-auto">Auto</option>
											<option value="aspect-square">1:1</option>
											<option value="aspect-video">16:9</option>
											<option value="aspect-[4/3]">4:3</option>
											<option value="aspect-[3/2]">3:2</option>
											<option value="aspect-[9/16]">9:16</option>
											<option value="1600x900">1600×900</option>
											<option value="1200x1200">1200×1200</option>
											<option value="custom">Custom</option>
										</select>
									</div>
									{#if options.aspectRatio === 'custom' && options.customSize}
										<div class="flex gap-2">
											<div class="flex-1">
												<label class="text-xs text-gray-400">W</label>
												<input type="number" bind:value={options.customSize.width} oninput={updateCanvasSize} class="w-full rounded-lg border-none bg-gray-100 px-2 py-1 text-sm dark:bg-gray-700" />
											</div>
											<div class="flex-1">
												<label class="text-xs text-gray-400">H</label>
												<input type="number" bind:value={options.customSize.height} oninput={updateCanvasSize} class="w-full rounded-lg border-none bg-gray-100 px-2 py-1 text-sm dark:bg-gray-700" />
											</div>
										</div>
									{/if}
									<div class="flex items-center justify-between">
										<span class="text-xs text-gray-500">Frame</span>
										<select bind:value={options.frame} class="rounded-lg border-none bg-gray-100 px-2 py-1 text-xs dark:bg-gray-700">
											<option value="Shortboard">Shortboard</option>
											<option value="None">None</option>
											<option value="Browser">Browser</option>
										</select>
									</div>
									<div class="flex items-center justify-between">
										<span class="text-xs text-gray-500">Noise</span>
										<input type="checkbox" bind:checked={options.noise} onchange={render} class="rounded text-indigo-600" />
									</div>
								</div>
							</details>

							<!-- Background -->
							<details class="group rounded-xl bg-white shadow-sm dark:bg-gray-800/50" open>
								<summary class="flex cursor-pointer items-center justify-between px-4 py-3 text-sm font-medium text-gray-900 dark:text-white">
									<span class="flex items-center gap-2">
										<svg class="h-4 w-4 text-gray-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10"/></svg>
										Background
									</span>
									<svg class="h-4 w-4 text-gray-400 transition-transform group-open:rotate-180" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m6 9 6 6 6-6"/></svg>
								</summary>
								<div class="border-t border-gray-100 px-4 py-4 dark:border-gray-700">
									<div class="grid grid-cols-5 gap-2">
										{#each GRADIENTS as gradient (gradient.class)}
											<button
												class={cls(
													'h-8 w-full cursor-pointer rounded-lg shadow-sm ring-2 ring-offset-2 transition-all ' + gradient.class,
													options.theme === gradient.class ? 'ring-indigo-500' : 'ring-transparent hover:ring-gray-300'
												)}
												onclick={() => {
													options = { ...options, theme: gradient.class, gradient: gradient.stops, customTheme: false };
													render();
												}}
											></button>
										{/each}
									</div>
								</div>
							</details>
						</div>
					</div>

					<!-- Footer Actions -->
					<div class="border-t border-gray-100 p-4 dark:border-gray-800">
						<div class="flex gap-2">
							<button
								class="flex flex-1 items-center justify-center gap-2 rounded-xl border border-gray-200 bg-white py-3 text-sm font-medium text-gray-700 transition-all hover:bg-gray-50 dark:border-gray-700 dark:bg-gray-800 dark:text-gray-200"
								onclick={copyImage}
							>
								<span class="h-4 w-4">{@html ClipboardIcon}</span>
								Copy
							</button>
							<button
								class="flex flex-1 items-center justify-center gap-2 rounded-xl bg-gradient-to-r from-indigo-500 to-purple-500 py-3 text-sm font-medium text-white shadow-lg shadow-indigo-500/25 transition-all hover:shadow-xl hover:shadow-indigo-500/30"
								onclick={saveImage}
							>
								<span class="h-4 w-4">{@html SaveIcon}</span>
								Export
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
				<div class="relative my-5">
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
								<input
									type="text"
									id="text-edit-{text.id}"
									value={text.text}
									oninput={(e) => {
										text.text = e.currentTarget.value;
										textElements = textElements;
										render();
									}}
									onblur={() => {
										editingTextId = null;
										render();
									}}
									onkeydown={(e) => {
										if (e.key === 'Enter') {
											e.preventDefault();
											editingTextId = null;
											render();
										}
										if (e.key === 'Escape') {
											editingTextId = null;
											render();
										}
									}}
									class="absolute z-50 border-2 border-dashed border-blue-500 bg-black/20 px-2 backdrop-blur-sm"
									style="
										left: {text.x * scaleX}px;
										top: {text.y * scaleY}px;
										transform: translate(-50%, -50%);
										font-size: {text.fontSize * scaleX}px;
										font-family: {text.fontFamily};
										color: {text.color};
										text-align: {text.align};
										min-width: 100px;
									"
								/>
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
	.slider-input {
		height: 6px;
		width: 100%;
		cursor: pointer;
		appearance: none;
		border-radius: 9999px;
		background-color: #e5e7eb;
	}
	:global(.dark) .slider-input {
		background-color: #374151;
	}
	.slider-input::-webkit-slider-thumb {
		height: 16px;
		width: 16px;
		appearance: none;
		border-radius: 9999px;
		background-color: #6366f1;
		box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
		transition: transform 0.15s;
	}
	.slider-input::-webkit-slider-thumb:hover {
		transform: scale(1.1);
	}
	.slider-input::-moz-range-thumb {
		height: 16px;
		width: 16px;
		appearance: none;
		border-radius: 9999px;
		border: none;
		background-color: #6366f1;
		box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
		transition: transform 0.15s;
	}
	.slider-input::-moz-range-thumb:hover {
		transform: scale(1.1);
	}
	details summary::-webkit-details-marker {
		display: none;
	}
	details summary {
		list-style: none;
	}
</style>
