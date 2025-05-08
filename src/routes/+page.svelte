<script module lang="ts">
	import { onMount } from 'svelte';
	const textBoxClasses =
		'rounded border border-gray-300 bg-white px-3 py-1 text-base text-gray-700 transition-colors duration-200 ease-in-out outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200';
	const buttonClasses =
		'rounded border-0 bg-indigo-500 px-6 py-1 text-white hover:bg-indigo-600 focus:outline-none';
	const labelClasses = 'text-sm leading-7 text-gray-600';
</script>

<script lang="ts">
	import { Canvas, FabricImage, FabricText } from 'fabric';
	/** 表示用縮尺倍率 */
	const multiplier = 5;
	let canvasElement: HTMLCanvasElement;
	let canvas: Canvas;
	let charaName: string = $state('');
	let section: string = $state('公安対魔特務六課');
	let credential: string = $state('特別捜査官');

	onMount(async () => {
		const img = await FabricImage.fromURL('/sho.png', {}, { selectable: false });

		const scaleWidth = img.width / multiplier;
		const scaleHeight = img.height / multiplier;
		// 縮小して表示
		img.scaleToWidth(scaleWidth);
		img.scaleToHeight(scaleHeight);
		canvasElement.width = scaleWidth;
		canvasElement.height = scaleHeight;

		canvas = new Canvas(canvasElement);
		canvas.add(img);
		canvas.renderAll();

		document.addEventListener('keydown', (event) => {
			handleKeyDown(event);
		});
	});

	function addCharName(): void {
		const textObj = new FabricText(charaName, {
			left: 325,
			top: 150,
			fontSize: 30,
			fontWeight: 'bold',
			fill: 'black'
		});

		canvas.add(textObj);
		canvas.renderAll();
	}

	function addSection(): void {
		const textObj = new FabricText(section, {
			left: 325,
			top: 224,
			fontSize: 22,
			fontWeight: 'bold',
			fill: 'black'
		});

		canvas.add(textObj);
		canvas.renderAll();
	}

	function addCredential(): void {
		const textObj = new FabricText(credential, {
			left: 325,
			top: 297,
			fontSize: 22,
			fontWeight: 'bold',
			fill: 'black'
		});

		canvas.add(textObj);
		canvas.renderAll();
	}

	function handleImageUpload(event: Event): void {
		const input = event.target as HTMLInputElement;
		const file = input.files?.[0];
		if (!file) return;

		const reader = new FileReader();
		reader.onload = () => {
			const img = new Image();
			img.onload = async () => {
				const MAX_WIDTH = 950;
				const MAX_HEIGHT = 1120;

				const ratio = Math.min(img.width / MAX_WIDTH, img.height / MAX_HEIGHT);
				const targetWidth = MAX_WIDTH * ratio;
				const targetHeight = MAX_HEIGHT * ratio;

				const cropX = (img.width - targetWidth) / 2;
				const cropY = (img.height - targetHeight) / 2;

				const offscreen = document.createElement('canvas');
				offscreen.width = MAX_WIDTH;
				offscreen.height = MAX_HEIGHT;

				const ctx = offscreen.getContext('2d');
				if (!ctx) return;

				ctx.drawImage(
					img,
					cropX,
					cropY,
					targetWidth,
					targetHeight, // source
					0,
					0,
					MAX_WIDTH,
					MAX_HEIGHT // destination
				);

				const resizedDataURL = offscreen.toDataURL();

				const fabricImg = await FabricImage.fromURL(resizedDataURL);
				fabricImg.set({
					left: 43,
					top: 116,
					selectable: true
				});
				fabricImg.scale(1 / multiplier);
				canvas.add(fabricImg);
				canvas.sendObjectToBack(fabricImg);
				canvas.setActiveObject(fabricImg);
				canvas.renderAll();
			};
			img.src = reader.result as string;
		};
		reader.readAsDataURL(file);
	}

	function downloadImage(): void {
		const imageSrc = canvas.toDataURL({ multiplier });

		const a = document.createElement('a');
		a.href = imageSrc;
		a.download = 'image.png';
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
	}

	function handleKeyDown(event: KeyboardEvent) {
		if (event.key === 'Delete' || event.key === 'Backspace') {
			const activeObject = canvas.getActiveObject();
			if (activeObject && (activeObject.type === 'text' || activeObject.type === 'image')) {
				canvas.remove(activeObject);
				canvas.renderAll();
			}
		}
	}

	function deleteImage() {
		const objects = canvas.getObjects();
		objects.map((obj) => {
			if (obj.type === 'image' && obj.height !== 2160) {
				canvas.remove(obj);
				canvas.renderAll();
				return;
			}
		});
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+JP&display=swap"
		rel="stylesheet"
	/>
</svelte:head>
<section class="container max-w-[49rem] p-2">
	<div class="grid grid-cols-1 gap-2 md:grid-cols-[3rem_auto_10rem] md:gap-4">
		<div class="col-span-full">
			<h1 class="title-font mb-1 text-lg font-medium text-gray-900">職員証作成サービス</h1>
			<p class="leading-relaxed text-gray-600">職員証に文字を入れてダウンロードできます!</p>
		</div>
		<div class="col-span-full overflow-auto md:justify-items-center">
			<div class="w-full pb-5 md:pb-0">
				<canvas
					bind:this={canvasElement}
					onkeydown={(e) => handleKeyDown(e)}
					tabindex={0}
					class="border-1"
				></canvas>
			</div>
		</div>
		<label for="charaName" class={`${labelClasses}`}>氏名</label>
		<input
			type="text"
			id="charaName"
			name="charaName"
			bind:value={charaName}
			class={`w-full ${textBoxClasses}`}
		/>
		<button onclick={addCharName} class={`w-full ${buttonClasses}`}>氏名を挿入</button>
		<label for="section" class={`${labelClasses}`}>部署</label>
		<input
			type="text"
			id="section"
			name="section"
			bind:value={section}
			class={`w-full ${textBoxClasses}`}
		/>
		<button onclick={addSection} class={buttonClasses}>部署を挿入</button>
		<label for="credential" class={`${labelClasses}`}>資格</label>
		<input
			type="text"
			id="credential"
			name="credential"
			bind:value={credential}
			class={`w-full ${textBoxClasses}`}
		/>
		<button onclick={addCredential} class={`w-full ${buttonClasses}`}>資格を挿入</button>
		<label for="gazo" class={`${labelClasses}`}>写真</label>
		<input
			type="file"
			id="gazo"
			name="gazo"
			accept="image/*"
			onchange={handleImageUpload}
			class="w-full cursor-pointer rounded border border-gray-300 px-3 text-sm leading-8 focus:outline-none"
		/>
		<button onclick={deleteImage} class={`w-full ${buttonClasses}`}>画像を削除</button>
		<div class="col-span-full">
			<p class="text-xs text-gray-500">
				※画面に挿入したものは選んでデリートキーかバックスペースキーで消せます。挿入した画像は削除ボタンで削除します。
			</p>
		</div>
		<div class="col-span-full md:col-span-2">
			<button onclick={downloadImage} class={`w-full ${buttonClasses}`}>職員証をダウンロード</button
			>
		</div>
		<button
			onclick={() => {
				window.location.reload();
			}}
			class={`w-full ${buttonClasses}`}>リセット</button
		>
	</div>
</section>

<style>
	* {
		font-family: 'IBM Plex Sans JP', sans-serif;
		font-weight: 400;
		font-style: normal;
	}
</style>
