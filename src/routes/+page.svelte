<script module lang="ts">
	import { onMount } from 'svelte';
</script>

<script lang="ts">
	import { Canvas, FabricImage, FabricText } from 'fabric';

	let canvasElement: HTMLCanvasElement;
	let canvas: Canvas;
	let charaName: string = $state('');
	let section: string = $state('');
	let credential: string = $state('');

	const scaleRatio = 1 / 5;

	onMount(async () => {
		const img = await FabricImage.fromURL(
			'/sho.png',
			{},
			{
				selectable: false
			}
		);

		const imgWidth = img.width * scaleRatio;
		const imgHeight = img.height * scaleRatio;

		canvasElement.width = imgWidth;
		canvasElement.height = imgHeight;

		img.scale(scaleRatio); // 画像のスケーリング

		canvas = new Canvas(canvasElement);
		canvas.add(img);
		canvas.renderAll();

		document.addEventListener('keydown', (event) => {
			handleKeyDown(event);
		});
	});

	function addCharName(): void {
		const textObj = new FabricText(charaName, {
			left: 320,
			top: 150,
			fontSize: 22,
			fontWeight: 'bold',
			fill: 'black'
		});

		canvas.add(textObj);
		canvas.setActiveObject(textObj);
		canvas.renderAll();
	}

	function addSection(): void {
		const textObj = new FabricText(section, {
			left: 320,
			top: 222,
			fontSize: 22,
			fontWeight: 'bold',
			fill: 'black'
		});

		canvas.add(textObj);
		canvas.setActiveObject(textObj);
		canvas.renderAll();
	}

	function addCredential(): void {
		const textObj = new FabricText(credential, {
			left: 320,
			top: 297,
			fontSize: 22,
			fontWeight: 'bold',
			fill: 'black'
		});

		canvas.add(textObj);
		canvas.setActiveObject(textObj);
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
				fabricImg.scale(scaleRatio);
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
		const imageSrc = canvas.toDataURL();
		// some download code down here
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
<section class="body-font text-gray-600">
	<div class="container flex">
		<div class="flex flex-col rounded-lg bg-white p-2 shadow-md md:m-auto">
			<h1 class="title-font mb-1 text-lg font-medium text-gray-900">職員証作成サービス</h1>
			<p class="mb-5 leading-relaxed text-gray-600">画像に文字入れられます!</p>
			<div class=" mb-4">
				<canvas
					bind:this={canvasElement}
					onkeydown={(e) => handleKeyDown(e)}
					tabindex={0}
					class="border-1"
				></canvas>
			</div>
			<div class=" mb-4">
				<label for="charaName" class=" text-sm leading-7 text-gray-600">氏名</label>
				<input
					type="text"
					id="charaName"
					name="charaName"
					bind:value={charaName}
					class="w-[50%] rounded border border-gray-300 bg-white px-3 py-1 text-base text-gray-700 transition-colors duration-200 ease-in-out outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200"
				/>
				<button
					onclick={addCharName}
					class="w-[10rem] rounded border-0 bg-indigo-500 px-6 py-1 text-white hover:bg-indigo-600 focus:outline-none"
					>氏名を挿入</button
				>
			</div>
			<div class=" mb-4">
				<label for="section" class=" text-sm leading-7 text-gray-600">部署</label>
				<input
					type="text"
					id="section"
					name="section"
					bind:value={section}
					class="w-[50%] rounded border border-gray-300 bg-white px-3 py-1 text-base text-gray-700 transition-colors duration-200 ease-in-out outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200"
				/>
				<button
					onclick={addSection}
					class="w-[10rem] rounded border-0 bg-indigo-500 px-6 py-1 text-white hover:bg-indigo-600 focus:outline-none"
					>部署を挿入</button
				>
			</div>
			<div class=" mb-4">
				<label for="credential" class=" text-sm leading-7 text-gray-600">資格</label>
				<input
					type="text"
					id="credential"
					name="credential"
					bind:value={credential}
					class="w-[50%] rounded border border-gray-300 bg-white px-3 py-1 text-base text-gray-700 transition-colors duration-200 ease-in-out outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200"
				/>
				<button
					onclick={addCredential}
					class="w-[10rem] rounded border-0 bg-indigo-500 px-6 py-1 text-white hover:bg-indigo-600 focus:outline-none"
					>資格を挿入</button
				>
			</div>
			<div class=" mb-4">
				<label for="gazo" class="text-sm leading-7 text-gray-600">写真</label>
				<input
					type="file"
					name="gazo"
					accept="image/*"
					onchange={handleImageUpload}
					class="w-[24rem] cursor-pointer rounded border border-gray-300 px-3 text-sm leading-8 focus:outline-none"
				/>
				<button
					onclick={deleteImage}
					class="w-[10rem] rounded border-0 bg-indigo-500 px-6 py-1 text-white hover:bg-indigo-600 focus:outline-none"
					>画像を削除</button
				>
			</div>
			<p class="mb-4 text-xs text-gray-500">
				※画面に挿入したものは選んでデリートキーかバックスペースキーで消せます
			</p>
			<div>
				<button
					onclick={downloadImage}
					class="w-[26rem] rounded border-0 bg-indigo-500 px-6 py-1 text-white hover:bg-indigo-600 focus:outline-none"
					>職員証をダウンロード</button
				>
			</div>
		</div>
	</div>
</section>

<style>
	* {
		font-family: 'IBM Plex Sans JP', sans-serif;
		font-weight: 400;
		font-style: normal;
	}
</style>
