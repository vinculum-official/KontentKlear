<script lang="ts">
    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let img: HTMLImageElement;
    let pickedColour = $state<[number, number, number] | null>(null);
    let tolerance = $state(40);
    let displayWidth = 500;
    let fileInput: HTMLInputElement;
    let isFileSelected = $state(false);

    function handleFileChange(event: Event) {
        isFileSelected = true;
        const input = event.target as HTMLInputElement;
        if (!input.files || input.files.length === 0) return;
        const file = input.files[0];
        const url = URL.createObjectURL(file);
        img = new Image();
        img.onload = () => {
            const scale = Math.min(displayWidth / img.width, 1);
            canvas.width = img.width * scale;
            canvas.height = img.height * scale;
            ctx = canvas.getContext('2d')!;
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        };
        img.src = url;
    }

    function getColourAtClick(event: MouseEvent) {
        if (!ctx) return;
        const rect = canvas.getBoundingClientRect();
        const x = Math.floor(event.clientX - rect.left);
        const y = Math.floor(event.clientY - rect.top);
        const data = ctx.getImageData(x, y, 1, 1).data;
        pickedColour = [data[0], data[1], data[2]];
        resetImage();
        processImage();
    }

    function ColourDistance(
        r1: number, g1: number, b1: number,
        r2: number, g2: number, b2: number
    ) {
        return Math.sqrt(
            (r1 - r2) ** 2 +
            (g1 - g2) ** 2 +
            (b1 - b2) ** 2
        );
    }

    function processImage() {
        if (!ctx || !pickedColour) return;
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const data = imageData.data;
        for (let i = 0; i < data.length; i += 4) {
            const r = data[i];
            const g = data[i + 1];
            const b = data[i + 2];
            const d = ColourDistance(r, g, b, ...pickedColour);
            if (d < tolerance) {
                const alpha = (d / tolerance) * 255;
                data[i + 3] = alpha;
            }
        }
        ctx.putImageData(imageData, 0, 0);
    }

    function handleToleranceChange() {
        resetImage();
        processImage();
    }

    function resetImage() {
        if (!ctx || !img) return;
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
    }

    function resetImageR() {
        if (!ctx || !img) return;
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        pickedColour = null;
    }

    function clearCanvas() {
        isFileSelected = false;
        if (!ctx) return;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        pickedColour = null;
        img = undefined as any;
        if (fileInput) {
            fileInput.value = '';
        }
    }

    function downloadImage() {
        const link = document.createElement('a');
        link.download = 'kontentklear.png';
        link.href = canvas.toDataURL('image/png');
        link.click();
    }

    function rgbToHex(r: number, g: number, b: number): string {
        return "#" + [r, g, b].map(x => {
            const hex = x.toString(16);
            return hex.length === 1 ? "0" + hex : hex;
        }).join('');
    }
</script>


<main class="h-screen bg-[#f2ecce] flex items-center justify-center font-archivo">

<div class="text-center">

{#if !isFileSelected}
<div class="transition-all duration-500 ease-in-out">
    <h1 class="text-5xl mb-4 font-black font-stretch-extra-expanded font-weight-900">KontentKlear</h1>
    <p class="text-lg">Click a colour. Remove it instantly.</p>

    <input 
        bind:this={fileInput} 
        type="file" 
        accept="image/*" 
        onchange={handleFileChange}
        style="display: none;"
    />
    <br />
    <button class="px-8 py-3 bg-white text-zinc-900 font-bold rounded-xl border border-zinc-100 shadow-[5px_5px_0_0_#ff7a00,10px_10px_0_0_#fef3c7] hover:translate-x-1 hover:translate-y-1 hover:shadow-none transition-all duration-300" onclick={() => fileInput.click()}>Browse...</button>
    <br /><br />
    <a href="/faq.html" class="ml-4 text-sm text-zinc-700 hover:underline">How this works and some privacy info</a>
</div>
{/if}



{#if isFileSelected === true}
<div class="transition-all duration-500 ease-in-out opacity-0 animate-fadeIn">
    <h1 class="text-lg mb-4 font-bold font-stretch-extra-expanded font-weight-900">Click a colour in the image to remove it. <br/> Adjust tolerance as needed.</h1>

    <hr class="border-zinc-900 mb-4" />

    <label>
        <span class="cursor-help" title="Controls how similar Colours need to be to get removed. Higher values remove more similar Colours.">Tolerance</span>: {tolerance}
        <input
            type="range"
            min="0"
            max="100"
            bind:value={tolerance}
            oninput={handleToleranceChange}
        />
    </label>

    <br /><br />

    {#if pickedColour !== null}
        <p class="transition-all duration-300 ease-in-out">
            <strong>Picked Colour:</strong>
            <span
                style="
                    display:inline-block;
                    width:50px;
                    height:50px;
                    background: rgb({pickedColour[0]}, {pickedColour[1]}, {pickedColour[2]});
                    border:2px solid black;
                    vertical-align:middle;
                    margin-left:10px;
                "
            ></span>
            <br />
            <span>RGB: {pickedColour[0]}, {pickedColour[1]}, {pickedColour[2]}</span>
            <span class="p-4">HEX: {rgbToHex(pickedColour[0], pickedColour[1], pickedColour[2])}</span>
            <br /> <br/>
        </p>
    {/if}

    <button onclick={resetImageR} class="relative px-8 py-3 bg-zinc-100 text-zinc-900 font-bold border-b-4 border-zinc-300 rounded-lg overflow-hidden group active:border-b-0 active:translate-y-1 transition-all">
      <span class="relative z-10 group-hover:text-white transition-Colours duration-200">Reset</span>
      <div class="absolute inset-0 bg-zinc-900 scale-x-0 group-hover:scale-x-100 transition-transform origin-left duration-300"></div>
    </button>
    <button onclick={clearCanvas} class="ml-2 relative px-8 py-3 bg-zinc-100 text-zinc-900 font-bold border-b-4 border-zinc-300 rounded-lg overflow-hidden group active:border-b-0 active:translate-y-1 transition-all">
      <span class="relative z-10 group-hover:text-white transition-Colours duration-200">Remove Image</span>
      <div class="absolute inset-0 bg-zinc-900 scale-x-0 group-hover:scale-x-100 transition-transform origin-left duration-300"></div>
    </button>

    <br /><br />

    <div style="border: 5px solid #26262a; display: inline-block; min-width: 1px; min-height: 1px;" class="transition-all duration-300 ease-in-out">
        <canvas bind:this={canvas} onclick={getColourAtClick}></canvas>
    </div>

    <br /><br />

    <button onclick={downloadImage} class="ml-2 relative px-8 py-3 bg-zinc-100 text-zinc-900 font-bold border-b-4 border-zinc-300 rounded-lg overflow-hidden group active:border-b-0 active:translate-y-1 transition-all">
      <span class="relative z-10 group-hover:text-white transition-Colours duration-200">Download PNG</span>
      <div class="absolute inset-0 bg-zinc-900 scale-x-0 group-hover:scale-x-100 transition-transform origin-left duration-300"></div>
    </button>
</div>
{/if}

</div>

</main>

<style>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fadeIn {
  animation: fadeIn 0.5s ease-in-out forwards;
}
</style>