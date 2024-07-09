<script lang="ts">
import { onMount } from 'svelte';


// CursorInteraction.ts
class CursorInteraction {
    canvas: HTMLCanvasElement;
    ctx: CanvasRenderingContext2D | null;
    dpr: number;
    rect: DOMRect;
    mouseX: number = 0;
    mouseY: number = 0;
    isHovered: boolean = false;

    constructor(canvas: HTMLCanvasElement) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.dpr = window.devicePixelRatio || 1;
        this.rect = canvas.getBoundingClientRect();
    }

    private resizeCanvas(): void {
        this.rect = this.canvas.getBoundingClientRect();
        this.canvas.width = this.rect.width * this.dpr;
        this.canvas.height = this.rect.height * this.dpr;
        if (this.ctx) {
            this.ctx.scale(this.dpr, this.dpr);
        }
    }

    private addEventListeners(): void {
        window.addEventListener('resize', this.handleResize.bind(this));
        document.addEventListener('mousemove', this.handleMouseMove.bind(this));
        this.canvas.addEventListener('mouseenter', () => this.isHovered = true);
        this.canvas.addEventListener('mouseleave', () => this.isHovered = false);
    }

    private handleResize(): void {
        this.resizeCanvas();
        requestAnimationFrame(this.draw.bind(this))
    }

    private handleMouseMove(e: MouseEvent): void {
        this.mouseX = e.clientX - this.rect.left;
        this.mouseY = e.clientY - this.rect.top;
        requestAnimationFrame(this.draw.bind(this));
    }

    draw(): void {
        if (!this.ctx) return;
        // Clear canvas
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        // Draw a simple rectangle following the mouse
        this.ctx.fillStyle = 'rgba(0, 0, 255, 0.5)';
        this.ctx.fillRect(this.mouseX - 25, this.mouseY - 25, 50, 50);
    }

    init(): void {
        this.resizeCanvas();
        this.addEventListeners();
        requestAnimationFrame(this.draw.bind(this));
    }

    getDebugInfo(): { mouseX: number; mouseY: number; isHovered: boolean } {
        return {
            mouseX: this.mouseX,
            mouseY: this.mouseY,
            isHovered: this.isHovered
        };
    }
}

let canvas: HTMLCanvasElement;
let cursorInteraction: CursorInteraction;
let debugInfo = { mouseX: 0, mouseY: 0, isHovered: false };
let showDebug = false;

onMount(() => {
    cursorInteraction = new CursorInteraction(canvas);
    cursorInteraction.init();

    // Update debug info every 100ms
    const intervalId = setInterval(() => {
        if (showDebug) {
            debugInfo = cursorInteraction.getDebugInfo();
        }
    }, 100);

    return () => {
        clearInterval(intervalId);
    };
});

function toggleDebug() {
    showDebug = !showDebug;
}
</script>

<canvas bind:this={canvas}></canvas>

{#if import.meta.env.DEV}
    <button id="toggle-debug" on:click={toggleDebug}>
        {showDebug ? 'Hide' : 'Show'} Debug Info
    </button>
    
    {#if showDebug}
        <div id="debug-info">
            <p>Mouse X: {debugInfo.mouseX.toFixed(2)}</p>
            <p>Mouse Y: {debugInfo.mouseY.toFixed(2)}</p>
            <p>Is Hovered: {debugInfo.isHovered}</p>
        </div>
    {/if}
{/if}

<style>
canvas {
    width: 100%;
    height: 100%;
}

#toggle-debug {
    position: fixed;
    bottom: 10px;
    left: 10px;
    padding: 10px;
    border-radius: 5px;
    background: white;
    color: black;
    cursor: pointer;
}

#debug-info {
    position: fixed;
    bottom: 60px;
    left: 10px;
    background: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 10px;
    border-radius: 5px;
}
</style>