<script>
	import { onMount } from 'svelte';
	import { replaceState } from '$app/navigation';

  let playing = $state(false);
  let buttonClass = $state('silent')
	let audio;
  $inspect(playing)

	let oppgave = $state('');
	let ready = $state(false);
	let tekstfelt;

	const LAGRINGSNOKKEL = 'siri-oppgavetekst';

	onMount(() => {
		audio = new Audio('https://cdn.pixabay.com/download/audio/2021/08/09/audio_9a2f521fc5.mp3');
    audio.loop = true;

		// URL-en vinner over det som ligger lagret, slik at delte lenker viser riktig tekst
		oppgave = new URLSearchParams(location.search).get('q') ?? localStorage.getItem(LAGRINGSNOKKEL) ?? '';
		ready = true;
	});

	// Lagre teksten og speil den i URL-en, men vent til skrivinga har roa seg
	$effect(() => {
		if (!ready) return;
		const tekst = oppgave;
		const timer = setTimeout(() => {
			localStorage.setItem(LAGRINGSNOKKEL, tekst);
			replaceState(tekst ? `?q=${encodeURIComponent(tekst)}` : location.pathname, {});
		}, 400);
		return () => clearTimeout(timer);
	});

	// Textarea-en vokser med innholdet i stedet for å scrolle
	$effect(() => {
		if (!tekstfelt) return;
		oppgave;
		tekstfelt.style.height = 'auto';
		tekstfelt.style.height = `${tekstfelt.scrollHeight}px`;
	});

	const play = async () => {
		if (playing) {
			audio.pause();
			playing = false;
			buttonClass = 'silent';
		} else {
			await audio.play().catch(err => console.error('Play error:', err));;
			playing = true;
			buttonClass = 'playing';
		}
	};
</script>

<svelte:head>
	<title>Siri sier summ</title>
	<meta name="description" content="Trykk på knappen for å høre sirisser." />
</svelte:head>

<main>
	<div id="hoved">
		<h1>Siri sier</h1>
		<textarea
			bind:this={tekstfelt}
			bind:value={oppgave}
			rows="1"
			placeholder="Skriv oppgavetekst …"
			aria-label="Oppgavetekst"
		></textarea>
		<button onclick={play} class={buttonClass}
			>SUMM!</button
		>
	</div>
</main>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Red+Hat+Display:wght@900&display=swap');
	:global(html) {
		background: url('$lib/assets/wp.jpeg');
		background-size: cover;
		background-repeat: no-repeat;
		background-position: center center;
		background-attachment: fixed;
	}

	main {
		min-height: 100vh;
		padding: 2rem 0;
		box-sizing: border-box;
		width: 100vw;
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
		justify-content: center;
	}

	#hoved {
		display: flex;
		flex-direction: column;
		align-items: center;
		max-width: 100vw;
	}

	h1 {
		color: #fff;
		font-family: 'Red Hat Display', Arial;
		font-size: clamp(44pt, 8vw, 72pt);
		margin-bottom: 0.2em;
	}

	textarea {
		width: min(90vw, 800px);
		margin-bottom: 0.6em;
		padding: 8pt 12pt;
		resize: none;
		overflow: hidden;
		text-align: center;
		color: #fff;
		font-family: 'Red Hat Display', Arial;
		font-weight: 800;
		font-size: clamp(18pt, 3.5vw, 32pt);
		line-height: 1.2;
		background: rgba(0, 0, 0, 0.25);
		backdrop-filter: blur(1px);
		-webkit-backdrop-filter: blur(1px);
		border: 2px dashed rgba(255, 255, 255, 0.35);
		border-radius: 6px;
		transition: border-color 0.15s ease-in-out;
	}

	textarea::placeholder {
		color: rgba(255, 255, 255, 0.55);
		font-weight: 400;
	}

	textarea:hover,
	textarea:focus {
		border-color: #ff4742;
		outline: 0;
	}

	button.playing {
		display: inline-block;
		outline: 0;
		cursor: pointer;
		border-radius: 6px;
		border: 4px solid #ff4742;
		background: 0 0;
		backdrop-filter: blur(1px);
		-webkit-backdrop-filter: blur(1px);
		color: #fff;
		/* color: #ff4742; */
		padding: 15pt 30pt 15pt 30pt;
		box-shadow: rgba(0, 0, 0, 0.07) 0px 2px 4px 0px, rgba(0, 0, 0, 0.05) 0px 1px 1.5px 0px;
		font-weight: 800;
		font-size: 36pt;
		font-family: 'Red Hat Display', Arial;
		transition: all 0.1s ease-in-out;
		/* Start the shake animation and make the animation last for 0.5 seconds */
		animation: shake 0.5s;

		/* When the animation is finished, start again */
		animation-iteration-count: infinite;
	}

	@keyframes shake {
		0% {
			transform: scale(1.2) translate(1px, 1px) rotate(0deg);
		}
		10% {
			transform: scale(1.2) translate(-1px, -2px) rotate(-1deg);
		}
		20% {
			transform: scale(1.2) translate(-3px, 0px) rotate(1deg);
		}
		30% {
			transform: scale(1.2) translate(3px, 2px) rotate(0deg);
		}
		40% {
			transform: scale(1.2) translate(1px, -1px) rotate(1deg);
		}
		50% {
			transform: scale(1.2) translate(-1px, 2px) rotate(-1deg);
		}
		60% {
			transform: scale(1.2) translate(-3px, 1px) rotate(0deg);
		}
		70% {
			transform: scale(1.2) translate(3px, 1px) rotate(-1deg);
		}
		80% {
			transform: scale(1.2) translate(-1px, -1px) rotate(1deg);
		}
		90% {
			transform: scale(1.2) translate(1px, 2px) rotate(0deg);
		}
		100% {
			transform: scale(1.2) translate(1px, -2px) rotate(-1deg);
		}
	}
	button.silent {
		display: inline-block;
		outline: 0;
		cursor: pointer;
		border-radius: 6px;
		border: 4px solid #ff4742;
		background: 0 0;
		backdrop-filter: blur(1px);
		-webkit-backdrop-filter: blur(1px);
		color: #fff;
		padding: 15pt 30pt 15pt 30pt;
		box-shadow: rgba(0, 0, 0, 0.07) 0px 2px 4px 0px, rgba(0, 0, 0, 0.05) 0px 1px 1.5px 0px;
		font-weight: 800;
		font-size: 36pt;
		font-family: 'Red Hat Display', Arial;
		transition: all 0.1s ease-in-out;
	}
	button:hover {
		color: #fff;
		background-color: #ff4742;
		transform: scale(1.2);
	}
</style>
