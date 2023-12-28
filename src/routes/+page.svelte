<script lang="ts">
	type State = 'ready' | 'ready.countdown' | 'recording';
	type Optional<T> = T | undefined;

	export let width: Optional<number> = undefined;
	export let height: Optional<number> = undefined;
	export let audio = true;
	export let frameRate = 60;

	let state: State = 'ready';
	let recorder: MediaRecorder;
	let stream: MediaStream;
	let videoChunks = [] as Blob[];
	let timerInterval: NodeJS.Timeout;
	let timer = 5;

	async function startTimer() {
		state = 'ready.countdown';

		return new Promise((resolve) => {
			timerInterval = setInterval(() => {
				timer--;
				if (timer === 0) {
					clearInterval(timerInterval);
					resolve(timer);
				}
			}, 1000);
		});
	}

	function startRecording() {
		state = 'recording';
		recorder = new MediaRecorder(stream, {
			mimeType: 'video/webm;codecs=vp9'
		});
		recorder.start();

		recorder.addEventListener('dataavailable', (event) => {
			videoChunks.push(event.data);
		});

		recorder.addEventListener('stop', () => {
			downloadRecording(videoChunks);
			videoChunks = [];
		});
	}

	function downloadRecording(videoChunks: Blob[]) {
		const date = new Date();
		const blob = new Blob(videoChunks, { type: 'video/webm;codecs=vp9' });
		const a = document.createElement('a');
		a.href = URL.createObjectURL(blob);
		a.download = `Reekordor ${date.toJSON().slice(0, 10)}`;
		a.click();
	}

	async function prepareRecording() {
		try {
			stream = await navigator.mediaDevices.getDisplayMedia({
				video: { width, height, frameRate },
				audio,
				// @ts-ignore
				selfBrowserSurface: 'include'
			});
			stream.addEventListener('inactive', stopRecording);
			await startTimer();
			startRecording();
		} catch (error) {
			console.error(`Error: ${error}`);
		}
	}

	function stopRecording() {
		state = 'ready';
		stream.getTracks().forEach((track) => track.stop());
		recorder && recorder.stop();
		clearInterval(timerInterval);
		timer = 5;
	}

	function handleKeydown(event: KeyboardEvent) {
		switch (event.key) {
			case 'R':
				if (state === 'ready') {
					prepareRecording();
				}
				break;
			case 'S':
				if (state !== 'ready') {
					stopRecording();
				}
				break;
		}
	}
</script>

<svelte:window on:keydown={handleKeydown} />
<div class="container">
	<div class="head">
		<h1 class="heading">Reekordor</h1>
		<p class="sub-heading">
			Free screen recorder to record your entire screen, window or even a browser tab. It downloads
			video in webm format. To convert your video in mp4, visit this site i made. And ofcourse its
			free.
		</p>
	</div>
	{#if state.includes('ready')}
		<div class="recorder" data-state={state}>
			<button class="record" on:click={prepareRecording}>
				<div class="circle">
					{#if state === 'ready.countdown'}
						{timer}
					{/if}
				</div>
			</button>

			<div class="info">
				{#if state === 'ready'}
					<p class="shortcut">Shift + R</p>
					<p class="description">To Start Recording</p>
				{/if}

				{#if state === 'ready.countdown'}
					<p class="shortcut">Shift + S</p>
					<p class="description">To Stop Recording</p>
				{/if}
			</div>
		</div>
	{/if}
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Inter:wght@200;300;400;500;600;700&display=swap');

	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(body) {
		color: hsl(0 0% 98%);
		font-weight: 300;
	}

	.heading {
		font-size: 42px;
	}

	.head {
		padding-top: 200px;
		display: flex;
		flex-direction: column;
		justify-items: center;
		align-items: center;
		text-align: center;
	}

	.sub-heading {
		padding: 1rem;
		max-width: 600px;
	}

	.container {
		height: 100svh;
		width: 100%;
		background-color: #030712;
		font-family: 'Inter', sans-serif;
	}

	.recorder {
		--txt-clr: hsl(0 0% 98%);
		--circle-bg-clr: hsl(0 100% 60%);
		--circle-border-clr: hsl(0 0% 98%);

		position: absolute;
		transform: translateX(-50%);
		left: 50%;
		bottom: 40px;
		color: var(--text-clr);
		text-align: center;
		font-family: 'Inter', sans-serif;
		z-index: 10;
	}

	[data-state='ready.countdown'] {
		& .circle {
			--text-clr: hsl(0 0% 10%);
			--circle-bg-clr: hsl(0 0% 98%);
		}
	}

	.record {
		width: 80px;
		height: 80px;
		padding: 4px;
		border: 4px solid var(--circle-border-clr);
		border-radius: 50%;
	}

	.circle {
		width: 100%;
		height: 100%;
		display: grid;
		place-content: center;
		font-family: 'Inter', sans-serif;
		font-size: 2rem;
		font-weight: 700;
		color: var(--text-clr);
		background: var(--circle-bg-clr);
		border-radius: 50%;
		transition: background-color 0.6s;

		&:hover {
			--circle-bg-clr: hsl(0 100% 64%);
			cursor: pointer;
		}
	}

	.info {
		margin-block-start: 1rem;

		& .shortcut {
			font-weight: 700;
			color: hsl(0 0% 98%);
		}

		& .description {
			margin-block-start: 4px;
			opacity: 0.6;
			color: hsl(0 0% 98%);
		}
	}
</style>
