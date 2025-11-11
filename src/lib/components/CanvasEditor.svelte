<script lang="ts">
    import { onMount, onDestroy } from "svelte";
    import toast, { Toaster } from 'svelte-french-toast';

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

    // Interaction state
    let isDragging = false;
    let dragStartX = 0;
    let dragStartY = 0;
    let editMode = false; // Toggle between view and edit mode

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
    };

    let options: Options = {
        aspectRatio: "aspect-auto",
        theme: "bg-gradient-to-br from-indigo-400 via-blue-400 to-purple-600",
        customTheme: {
            colorStart: "#ff40ff",
            colorEnd: "#fec700",
        },
        padding: "p-20",
        rounded: "rounded-xl",
        roundedWrapper: "rounded-xl",
        shadow: "shadow-xl",
        noise: false,
        browserBar: "hidden",
    };

    const isValidHexColor = /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/;

    function cls(...parts: Array<string | false | null | undefined | Record<string, boolean>>) {
        return parts
            .flatMap((p) => {
                if (!p) return [];
                if (typeof p === "string") return p;
                return Object.entries(p)
                    .filter(([_, v]) => v)
                    .map(([k]) => k);
            })
            .join(" ");
    }

    function handleResize() {
        if (canvas && imageLoaded) {
            updateCanvasSize();
        }
    }

    onMount(() => {
        if (typeof localStorage !== "undefined") {
            const preset = localStorage.getItem("options");
            if (preset) {
                try {
                    options = JSON.parse(preset);
                } catch (e) {
                    console.warn("Failed to parse options from localStorage", e);
                }
            }
        }

        window.addEventListener("keydown", handleShortcuts);
        window.addEventListener("resize", handleResize);
    });

    onDestroy(() => {
        if (typeof window !== "undefined") {
            window.removeEventListener("keydown", handleShortcuts);
            window.removeEventListener("resize", handleResize);
        }
    });

    $: (function persistOptions() {
        if (typeof localStorage !== "undefined") {
            try {
                localStorage.setItem("options", JSON.stringify(options));
            } catch (e) {
                // ignore
            }
        }
    })();

    function initializeCanvas() {
        if (!canvas) return;
        ctx = canvas.getContext("2d", { willReadFrequently: true });
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
            case "p-0": return 0;
            case "p-10": return 40;  // 2.5rem = 40px
            case "p-20": return 80;  // 5rem = 80px
            case "p-32": return 128; // 8rem = 128px
            default: return 80;
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
            case "aspect-square":
                contentWidth = contentHeight = Math.min(baseWidth, baseHeight);
                break;
            case "aspect-video": // 16:9
                contentWidth = baseWidth;
                contentHeight = Math.round(baseWidth * 9 / 16);
                break;
            case "aspect-[4/3]":
                contentWidth = baseWidth;
                contentHeight = Math.round(baseWidth * 3 / 4);
                break;
            case "aspect-[3/2]":
                contentWidth = baseWidth;
                contentHeight = Math.round(baseWidth * 2 / 3);
                break;
            case "aspect-[9/16]": // Portrait
                contentWidth = Math.round(baseHeight * 9 / 16);
                contentHeight = baseHeight;
                break;
            case "aspect-[21/9]": // Ultrawide
                contentWidth = baseWidth;
                contentHeight = Math.round(baseWidth * 9 / 21);
                break;
            case "aspect-[3/4]":
                contentWidth = Math.round(baseHeight * 3 / 4);
                contentHeight = baseHeight;
                break;
            case "aspect-[2/3]":
                contentWidth = Math.round(baseHeight * 2 / 3);
                contentHeight = baseHeight;
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
        canvasWidth = contentWidth + (padding * 2);
        canvasHeight = contentHeight + (padding * 2);

        if (canvas) {
            canvas.width = canvasWidth;
            canvas.height = canvasHeight;
        }

        if (uploadedImage && imageLoaded) {
            centerImage();
        }
        render();
    }

    function centerImage() {
        if (!uploadedImage) return;

        const padding = getPaddingValue();

        // Calculate available space (canvas minus padding)
        const availableWidth = canvasWidth - (padding * 2);
        const availableHeight = canvasHeight - (padding * 2);

        // Calculate scale to fit image within available space
        const scaleX = availableWidth / uploadedImage.width;
        const scaleY = availableHeight / uploadedImage.height;
        imageScale = Math.min(scaleX, scaleY, 1); // Don't scale up, only down

        imageWidth = uploadedImage.width * imageScale;
        imageHeight = uploadedImage.height * imageScale;

        // Center the image within the canvas
        imageX = (canvasWidth - imageWidth) / 2;
        imageY = (canvasHeight - imageHeight) / 2;
    }

    function render() {
        if (!ctx || !canvas) return;

        // Clear canvas
        ctx.clearRect(0, 0, canvasWidth, canvasHeight);

        // Draw background gradient
        const gradient = ctx.createLinearGradient(0, 0, canvasWidth, canvasHeight);
        if (options.customTheme) {
            gradient.addColorStop(0, options.customTheme.colorStart || "#ff40ff");
            gradient.addColorStop(1, options.customTheme.colorEnd || "#fec700");
        } else {
            gradient.addColorStop(0, "#6366f1");
            gradient.addColorStop(1, "#8b5cf6");
        }
        ctx.fillStyle = gradient;
        ctx.fillRect(0, 0, canvasWidth, canvasHeight);

        // Draw image if loaded
        if (uploadedImage && imageLoaded) {
            ctx.save();
            ctx.drawImage(
                uploadedImage,
                imageX,
                imageY,
                imageWidth,
                imageHeight
            );
            ctx.restore();

            // Draw resize handles in edit mode
            if (editMode) {
                drawResizeHandles();
            }
        }
    }

    function drawResizeHandles() {
        if (!ctx) return;

        const handleSize = 10;
        const handles = [
            { x: imageX, y: imageY }, // Top-left
            { x: imageX + imageWidth, y: imageY }, // Top-right
            { x: imageX, y: imageY + imageHeight }, // Bottom-left
            { x: imageX + imageWidth, y: imageY + imageHeight }, // Bottom-right
        ];

        ctx.fillStyle = "#fff";
        ctx.strokeStyle = "#3b82f6";
        ctx.lineWidth = 2;

        handles.forEach(handle => {
            ctx!.fillRect(handle.x - handleSize / 2, handle.y - handleSize / 2, handleSize, handleSize);
            ctx!.strokeRect(handle.x - handleSize / 2, handle.y - handleSize / 2, handleSize, handleSize);
        });

        // Draw border around image
        ctx.strokeStyle = "#3b82f6";
        ctx.lineWidth = 2;
        ctx.setLineDash([5, 5]);
        ctx.strokeRect(imageX, imageY, imageWidth, imageHeight);
        ctx.setLineDash([]);
    }

    function handleShortcuts(e: KeyboardEvent) {
        if ((e.key === "c" && (e.ctrlKey || e.metaKey))) {
            e.preventDefault();
            copyImage();
        }

        if ((e.key === "s" && (e.ctrlKey || e.metaKey))) {
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

        for (let i = 0; i < items.length; i++) {
            const item = items[i];
            if (!item) continue;
            if (item.kind === "file" || (item.type && item.type.includes("image"))) {
                const file = item.getAsFile ? item.getAsFile() : item;
                loadImage(file);
            } else if (item instanceof File && item.type.includes("image")) {
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

                toast.success("Image loaded!");
            };
            img.src = e.target.result;
        };
        reader.readAsDataURL(file);
    }

    // Canvas interaction handlers
    function handleMouseDown(e: MouseEvent) {
        if (!editMode || !uploadedImage) return;

        const rect = canvas!.getBoundingClientRect();
        const mouseX = e.clientX - rect.left;
        const mouseY = e.clientY - rect.top;

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
        if (!isDragging || !canvas) return;

        const rect = canvas.getBoundingClientRect();
        const mouseX = e.clientX - rect.left;
        const mouseY = e.clientY - rect.top;

        imageX = mouseX - dragStartX;
        imageY = mouseY - dragStartY;

        render();
    }

    function handleMouseUp() {
        isDragging = false;
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

            render();
        }
    }

    function resetImageTransform() {
        centerImage();
        render();
        toast.success("Image reset to center");
    }

    async function saveImage() {
        if (!canvas) {
            toast.error("Nothing to save, make sure to add a screenshot first!");
            return;
        }

        const savingToast = toast.loading("Exporting image...");

        try {
            canvas.toBlob((blob) => {
                if (!blob) {
                    toast.error("Failed to export image");
                    return;
                }

                const url = URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = `pika-${new Date().toISOString()}.png`;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                URL.revokeObjectURL(url);

                toast.dismiss(savingToast);
                toast.success("Image exported!");
            }, 'image/png');
        } catch (err) {
            toast.dismiss(savingToast);
            toast.error("Failed to export image");
            console.error(err);
        }
    }

    async function copyImage() {
        if (!canvas) {
            toast.error("Nothing to copy, make sure to add a screenshot first!");
            return;
        }

        const isSafari = /^((?!chrome|android).)*safari/i.test(navigator?.userAgent || "");
        const isNotFirefox = (navigator?.userAgent || "").indexOf("Firefox") < 0;

        try {
            canvas.toBlob(async (blob) => {
                if (!blob) {
                    toast.error("Failed to copy image");
                    return;
                }

                if (isSafari || isNotFirefox) {
                    try {
                        await navigator.clipboard.write([
                            new ClipboardItem({ "image/png": blob })
                        ]);
                        toast.success("Image copied to clipboard");
                    } catch (err) {
                        console.error(err);
                        toast.error("Failed to copy image");
                    }
                } else {
                    toast.error("Firefox does not support this functionality");
                }
            }, 'image/png');
        } catch (err) {
            console.error(err);
            toast.error("Failed to copy image");
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
            toast.success("First color applied");
        } else {
            toast.error("Invalid Hex color");
        }
    }

    function handleEndColorInput(e: Event) {
        const v = (e.target as HTMLInputElement).value;
        options = { ...options, customTheme: { ...(options.customTheme || {}), colorEnd: v } };
        if (v.match(isValidHexColor)) {
            render();
            toast.success("Second color applied");
        } else {
            toast.error("Invalid Hex color");
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

        toast.success("Canvas cleared!");
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

<style>
    .pika-wrap { min-height: 800px; }
    canvas {
        max-width: 100%;
        height: auto;
        cursor: move;
    }
    canvas.view-mode {
        cursor: default;
    }
</style>

<Toaster position="bottom-center" />

<div
    class="flex flex-col items-start justify-start h-screen px-5 lg:px-10 pika-wrap"
    on:paste|preventDefault={onPaste}
    on:dragover|preventDefault
    on:dragenter|preventDefault
    on:dragleave|preventDefault
    on:drop|preventDefault={(e) => { onPaste(e); }}
    role="application"
    aria-label="Image editor"
>
    <div class="relative flex flex-col-reverse w-full lg:flex-row-reverse max-w-[1600px] mx-auto">
        <div class="w-full lg:w-[350px]">
            <div class={cls(
                "p-6 lg:p-8 h-auto bg-white/90 dark:bg-pink-600/60 rounded-2xl ring-1 ring-pink-300 dark:ring-pink-200/50 ring-offset-1 ring-offset-white dark:ring-offset-red-800 shadow-lg shadow-gray-200 dark:shadow-black lg:max-h-screen w-full relative lg:min-h-[650px] mt-10 lg:mt-0"
            )}>
                <div class="absolute inset-0 w-full lg:h-full bg-gradient-to-br from-pink-400 dark:from-pink-500 dark:to-red-800 to-red-300 blur-xl dark:blur-md lg:scale-y-[1.05] scale-100 lg:scale-x-[1.1] dark:lg:scale-[1.05] lg:max-h-[calc(100vh-60px)] transform-gpu opacity-60"></div>

                <div class="relative flex flex-row flex-wrap items-start justify-start space-y-5 lg:items-start lg:flex-col lg:space-y-4">
                    <div class="flex items-center justify-between w-full">
                        <div class="text-sm font-semibold dark:text-white">Aspect Ratio</div>
                        <div>
                            <select bind:value={options.aspectRatio}
                                class="px-2 py-1 border border-gray-500 rounded-lg shadow-lg appearance-none cursor-pointer opacity-80 hover:opacity-100"
                                on:change={(e) => options = {...options, aspectRatio: (e.target as HTMLSelectElement).value }}>
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

                    {#if imageLoaded}
                        <div class="w-full">
                            <button
                                on:click={() => { editMode = !editMode; render(); }}
                                class="w-full flex items-center justify-center px-3 py-2 text-sm font-semibold text-white rounded-lg shadow duration-200"
                                class:bg-purple-600={!editMode}
                                class:hover:bg-purple-700={!editMode}
                                class:bg-green-600={editMode}
                                class:hover:bg-green-700={editMode}>
                                <span class="w-4 h-4 mr-2" innerHTML={MoveIcon}></span>
                                {editMode ? 'Exit Edit Mode' : 'Edit Position & Scale'}
                            </button>
                        </div>

                        <div class="w-full">
                            <button
                                on:click={resetImageTransform}
                                class="w-full flex items-center justify-center px-3 py-2 text-sm font-semibold text-gray-700 bg-gray-200 rounded-lg shadow hover:bg-gray-300 duration-200">
                                <span class="w-4 h-4 mr-2" innerHTML={ResetIcon}></span>
                                Reset Position
                            </button>
                        </div>

                        {#if editMode}
                            <div class="w-full p-3 bg-blue-100 dark:bg-blue-900/30 rounded-lg">
                                <p class="text-xs text-blue-800 dark:text-blue-200">
                                    <strong>Edit Mode:</strong><br/>
                                    • Drag to move image<br/>
                                    • Scroll to zoom in/out
                                </p>
                            </div>
                        {/if}
                    {/if}

                    <div>
                        <div class="relative flex items-center pb-2 text-sm font-semibold dark:text-white">
                            Background
                            <div class="relative">
                                <button on:click={toggleBG} class="flex items-center px-2 ml-2 border border-gray-400 rounded-lg cursor-pointer bg-white/70 opacity-70 hover:opacity-100 dark:bg-pink-900/80 dark:border-red-600 dark:text-gray-300">
                                    <span class="w-3 h-3 mr-1" innerHTML={ColorPickerIcon}></span>
                                    Pick
                                </button>
                            </div>

                            {#if bgPicker}
                                <div class="fixed inset-0 w-full h-full bg-transparent" on:click={() => bgPicker = false} role="button" tabindex="0" on:keydown={(e) => e.key === 'Escape' && (bgPicker = false)} aria-label="Close color picker"></div>

                                <div class={cls(
                                    "absolute w-auto max-w-[400px] z-10 top-[calc(100%)] left-[-30px] bg-white/80 backdrop-blur shadow-lg py-4 px-5 rounded-xl flex shadow-gray-500/50 dark:shadow-black/80 border border-gray-400 flex-col dark:border-gray-800 dark:bg-black/80 duration-200",
                                    { "opacity-0 pointer-events-none scale-[0.9]": !bgPicker },
                                    { "opacity-100 pointer-events-auto scale-[1]": bgPicker }
                                )}>
                                    <button class="absolute top-[5%] right-[5%] opacity-50 cursor-pointer hover:opacity-100 z-10" on:click={() => bgPicker = false} aria-label="Close">✕</button>

                                    <div class="relative mb-3">
                                        <div class="mb-1">Pick first color</div>
                                        <div class="flex items-center">
                                            <div class="relative group">
                                                <input id="startColorPicker" type="color"
                                                    class="absolute top-0 left-0 w-12 h-12 rounded-full opacity-0 cursor-pointer"
                                                    value={options.customTheme?.colorStart || "#222"}
                                                    on:input={(e) => { options = {...options, customTheme:{...(options.customTheme||{}), colorStart:(e.target as HTMLInputElement).value}}; render(); }} />
                                                <label for="startColorPicker"
                                                    style="background-color: {options?.customTheme?.colorStart || '#222'}"
                                                    class="left-0 flex items-center justify-center w-12 h-12 rounded-full pointer-events-none text-white/50 group-hover:scale-[1.1] duration-100">
                                                    <span class="font-mono text-xs text-white/80 drop-shadow">Pick</span>
                                                </label>
                                            </div>

                                            <span class="px-4 opacity-50">/</span>

                                            <input placeholder="Enter hex value" type="text" value={options.customTheme?.colorStart || "#000000"}
                                                class="px-2 py-1 font-mono text-base text-black border-2 border-gray-500 rounded-lg focus:outline-none focus:border-black"
                                                on:input={handleStartColorInput} />
                                        </div>
                                    </div>

                                    <div>
                                        <div class="mb-1">Pick second color</div>
                                        <div class="flex items-center">
                                            <div class="relative group">
                                                <input id="endColorPicker" type="color"
                                                    class="absolute top-0 left-0 w-12 h-12 rounded-full opacity-0 cursor-pointer"
                                                    value={options.customTheme?.colorEnd || "#222"}
                                                    on:input={(e) => { options = {...options, customTheme:{...(options.customTheme||{}), colorEnd:(e.target as HTMLInputElement).value}}; render(); }} />
                                                <label for="endColorPicker"
                                                    style="background-color: {options?.customTheme?.colorEnd || '#222'}"
                                                    class="left-0 flex items-center justify-center w-12  h-12 rounded-full pointer-events-none text-white/50 group-hover:scale-[1.1] duration-100">
                                                    <span class="font-mono text-xs text-white/80 drop-shadow">Pick</span>
                                                </label>
                                            </div>

                                            <span class="px-4 opacity-50">/</span>

                                            <input placeholder="Enter hex value" type="text" value={options.customTheme?.colorEnd || "#000000"}
                                                class="px-2 py-1 font-mono text-base text-black border-2 border-gray-500 rounded-lg focus:outline-none focus:border-black"
                                                on:input={handleEndColorInput} />
                                        </div>
                                    </div>
                                </div>
                            {/if}
                        </div>

                        <div class="grid flex-wrap grid-cols-6 mt-1 gap-x-4 gap-y-2">
                            {#each [
                                "bg-gradient-to-br from-pink-300 via-orange-200 to-red-300",
                                "bg-gradient-to-br from-green-300 via-yellow-200 to-green-200",
                                "bg-gradient-to-br from-green-200 via-blue-100 to-blue-300",
                                "bg-gradient-to-br from-indigo-300 via-blue-400 to-purple-500",
                                "bg-gradient-to-br from-red-300 via-orange-300 to-yellow-200",
                                "bg-gradient-to-br from-pink-300 via-pink-400 to-red-400",
                                "bg-gradient-to-br from-slate-400 via-gray-500 to-gray-700",
                                "bg-gradient-to-br from-orange-300 via-orange-400 to-red-400",
                                "bg-gradient-to-br from-teal-300 to-cyan-400",
                                "bg-gradient-to-br from-red-300 to-purple-600",
                            ] as theme (theme)}
                                <button class={"cursor-pointer shadow dark:shadow-black/20 shadow-gray-500/20 w-8 h-8 rounded-full " + theme}
                                    on:click={() => { options = {...options, theme, customTheme: false}; render(); }}
                                    aria-label="Select theme"></button>
                            {/each}
                        </div>
                    </div>

                    <div class="flex items-center justify-between w-full">
                        <button class="flex items-center justify-center px-4 py-2 hover:scale-[1.03] duration-200 text-base lg:text-lg font-semibold text-pink-600 bg-pink-200 rounded-lg shadow cursor-pointer border border-pink-600 w-full"
                            on:click={copyImage} title="Use Ctrl/Cmd + C to copy the image">
                            <span class="w-6 h-6 mr-2" innerHTML={ClipboardIcon}></span>
                            Copy
                        </button>

                        <button class="flex items-center justify-center px-4 py-2 hover:scale-[1.03] duration-200 text-base lg:text-lg font-semibold bg-pink-600/90 dark:bg-pink-600/90 text-pink-200 rounded-lg shadow cursor-pointer border border-pink-600 w-full ml-4"
                            title="Use Ctrl/Cmd + S to save the image"
                            on:click={saveImage}>
                            <span class="w-6 h-6 mr-2" innerHTML={SaveIcon}></span>
                            Save
                        </button>
                    </div>

                    <button on:click={clearImage} class="flex items-center justify-center w-full px-3 py-1 mx-auto mt-4 text-sm text-pink-400 rounded-lg cursor-pointer">
                        <span class="w-4 h-4 mr-1" innerHTML={ResetIcon}></span>
                        Reset
                    </button>

                    <div class="hidden mx-auto text-sm text-center opacity-50 dark:text-white lg:block">
                        <div class="mb-1">
                            Use <span class="px-2 py-px font-mono rounded-lg dark:bg-black/40 bg-white/80">Cmd/Ctrl+C</span> to copy or
                        </div>
                        <div>
                            <span class="px-2 py-px font-mono rounded-lg bg-white/80 dark:bg-black/40">Cmd/Ctrl+S</span> to save output image
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="w-full lg:w-[calc(100%-350px)] py-5 lg:p-10 lg:pl-0 flex flex-col items-center justify-center overflow-y-auto">
            {#if imageLoaded}
                <div class="my-5">
                    <canvas
                        bind:this={canvas}
                        class:view-mode={!editMode}
                        on:mousedown={handleMouseDown}
                        on:mousemove={handleMouseMove}
                        on:mouseup={handleMouseUp}
                        on:mouseleave={handleMouseUp}
                        on:wheel={handleWheel}
                        class={cls("relative shadow-xl", options?.rounded)}
                        style="display: block; max-width: 100%; height: auto;"
                    ></canvas>
                </div>
            {:else}
                <div class="flex items-center justify-center min-h-[50vh] lg:min-h-[80vh]">
                    <label class="flex flex-col items-center justify-center text-lg opacity-30 select-none max-w-[550px] rounded-2xl p-10 mt-20 text-center dark:text-white cursor-pointer border-2 border-dashed border-gray-400 hover:opacity-50 duration-300" for="imagesUpload">
                        <input class="hidden" id="imagesUpload" type="file" accept="image/*" on:change={onPaste} />
                        <span class="w-6 h-6 mb-2" innerHTML={PasteIcon}></span>
                        <p>Paste your screenshot (Cmd/Ctrl+V)</p>
                        <p>or drag and drop your screenshot here</p>
                        <p>or click here to add one</p>
                    </label>
                </div>
            {/if}

            <div class="lg:absolute lg:bottom-[20px] flex flex-col items-center justify-center pb-5 text-sm lg:pb-0 lg:pt-20 dark:text-gray-400 lg:flex-row opacity-60">
                <a href="https://twitter.com/thelifeofrishi" target="_blank" class="flex items-center hover:underline">
                    <span class="w-5 h-5 mx-1" innerHTML={TwitterIcon}></span>
                    Created by Rishi Mohan
                </a>
                <span class="hidden px-2 lg:block">-</span>
                <a href="https://github.com/rishimohan/pika" target="_blank" class="flex items-center mt-2 hover:underline lg:mt-0">
                    <span class="w-5 h-5 mx-1" innerHTML={GithubIcon}></span>
                    View Code on Github
                </a>
                <span class="hidden px-2 lg:block">-</span>
                <a href="https://www.buymeacoffee.com/thelifeofrishi" target="_blank" class="flex items-center mt-2 hover:underline lg:mt-0">
                    <span class="w-5 h-5 mx-1" innerHTML={CoffeeIcon}></span>
                    Buy me a coffee
                </a>
            </div>
        </div>
    </div>
</div>
