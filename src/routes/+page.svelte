<script lang="ts">
    import { browser } from '$app/environment';
    import { onMount } from 'svelte';

    let Editor: any;
    let mounted = false;

    // Use new canvas-based editor
    const useCanvasEditor = true;

    onMount(async () => {
        if (browser) {
            if (useCanvasEditor) {
                const module = await import('$lib/components/CanvasEditor.svelte');
                Editor = module.default;
            } else {
                const module = await import('$lib/components/PikaEditor.svelte');
                Editor = module.default;
            }
            mounted = true;
        }
    });
</script>

<svelte:head>
    <title>Pika - Beautiful Screenshots</title>
    <meta name="description" content="Create beautiful, customizable screenshots with Pika. Add backgrounds, shadows, and effects to your images." />
</svelte:head>

{#if mounted && Editor}
    <svelte:component this={Editor} />
{:else}
    <!-- Loading state / SSR fallback -->
    <div class="flex items-center justify-center min-h-screen bg-linear-to-br from-pink-100 via-purple-100 to-indigo-100 dark:from-gray-900 dark:via-pink-900 dark:to-purple-900">
        <div class="text-center">
            <div class="inline-block animate-spin rounded-full h-12 w-12 border-b-2 border-pink-600"></div>
            <p class="mt-4 text-lg text-gray-700 dark:text-gray-300">Loading Pika Editor...</p>
        </div>
    </div>
{/if}
