<script>
	import { quintOut } from "svelte/easing";
	import { crossfade } from "svelte/transition";
	import { flip } from "svelte/animate";

	import { data } from "$lib/data.js";

	const [send, receive] = crossfade({
		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === "none" ? "" : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`,
			};
		},
	});

	let de = [];
	let ds = [];
	let dv = [];
	let sre = [];

	let shakeDe = "";
	let shakeDs = "";
	let shakeDv = "";
	let shakeSre = "";
	let paradeDe = "";
	let paradeDs = "";
	let paradeDv = "";
	let paradeSre = "";
	let animatePointDe = "";
	let animatePointDs = "";
	let animatePointDv = "";
	let animatePointSre = "";

	let uid = 0;
	let score = 0;
	let addPoint = "0";

	let colors = [
		"#42145F",
		"#714F87",
		"#8D729F",
		"#A995B7",
		"#C6B8CE",
		"#E3DCE7",
	];
	let randomColor = colors[3];

	let avatars = ["", "1"];
	let randomAvatar = avatars[0];

	function selectRandomQuestion() {
		if (data.length == 0) {
			return false;
		}
		randomColor = colors[Math.floor(Math.random() * colors.length)];
		randomAvatar = avatars[Math.floor(Math.random() * avatars.length)];
		return data.splice(Math.floor(Math.random() * data.length), 1)[0];
	}
	let currentQuestion = selectRandomQuestion();

	function handleAnswer(role) {
		if (currentQuestion == false) {
			return;
		}

		// Check answer
		let correct = currentQuestion.belongs_to.includes(role);

		// Give visual feedback
		if (!correct) {
			shakeButton(role);
			addPoint = "0";
			for (let correctRole of currentQuestion.belongs_to) {
				animateAddPoint(correctRole);
			}
		} else {
			addPoint = "1";
			animateAddPoint(role);
		}
		for (let correctRole of currentQuestion.belongs_to) {
			paradeButton(correctRole);
		}

		// Update score
		if (correct) {
			score++;
		}

		// Add answer(s)
		const answer = {
			id: uid++,
			description: currentQuestion.description,
			correct: correct,
		};

		if (currentQuestion.belongs_to.includes("Data Engineer")) {
			de = [answer, ...de];
		}
		if (currentQuestion.belongs_to.includes("Data Scientist")) {
			ds = [answer, ...ds];
		}
		if (currentQuestion.belongs_to.includes("Datavisualisator")) {
			dv = [answer, ...dv];
		}
		if (
			currentQuestion.belongs_to.includes(
				"Site Reliability Engineer (SRE)",
			)
		) {
			sre = [answer, ...sre];
		}
		currentQuestion = selectRandomQuestion();
	}

	function shakeButton(role) {
		if (role === "Data Engineer") {
			shakeDe = "shaking";
		} else if (role === "Data Scientist") {
			shakeDs = "shaking";
		} else if (role === "Datavisualisator") {
			shakeDv = "shaking";
		} else if (role === "Site Reliability Engineer (SRE)") {
			shakeSre = "shaking";
		}
		setTimeout(() => {
			shakeDe = "";
			shakeDs = "";
			shakeDv = "";
			shakeSre = "";
		}, 250);
	}

	function paradeButton(role) {
		if (role === "Data Engineer") {
			paradeDe = "parading";
		} else if (role === "Data Scientist") {
			paradeDs = "parading";
		} else if (role === "Datavisualisator") {
			paradeDv = "parading";
		} else if (role === "Site Reliability Engineer (SRE)") {
			paradeSre = "parading";
		}
		setTimeout(() => {
			paradeDe = "";
			paradeDs = "";
			paradeDv = "";
			paradeSre = "";
		}, 500);
	}

	function getRandomTranslation() {
		return (
			"transform: translate(" +
			Math.floor(40 + Math.random() * 200) +
			"px, " +
			Math.floor(40 + Math.random() * 50) * -1 +
			"px);opacity:1;"
		);
	}
	function animateAddPoint(role) {
		if (role === "Data Engineer") {
			animatePointDe = getRandomTranslation();
		} else if (role === "Data Scientist") {
			animatePointDs = getRandomTranslation();
		} else if (role === "Datavisualisator") {
			animatePointDv = getRandomTranslation();
		} else if (role === "Site Reliability Engineer (SRE)") {
			animatePointSre = getRandomTranslation();
		}
		setTimeout(() => {
			animatePointDe = "";
			animatePointDs = "";
			animatePointDv = "";
			animatePointSre = "";
		}, 1200);
	}
</script>

<svelte:head>
	<link
		href="https://fonts.googleapis.com/css2?family=Kalam:wght@300;400;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div>
	<div id="prompt-banner">
		<div id="avatar" style="background-color: {randomColor};">
			<img src="./img/avatar{randomAvatar}.png" />
		</div>
		<div id="text-cloud">
			Hoe goed ken jij de verschillende specialisaties binnen het RWS
			Datalab? Welke rol heb ik?
			{#if currentQuestion}
				<div id="prompt">{currentQuestion.description}</div>
			{:else}
				<br /><br />Dit waren alle vragen, dank voor het spelen!
			{/if}
		</div>
		<div id="score">
			Aantal correct {score}<br />Voortgang {de.length +
				ds.length +
				dv.length +
				sre.length} / 120
		</div>
	</div>
	<div id="answer_container">
		<div class="column">
			<button
				class="{shakeDe} {paradeDe}"
				on:click={() => handleAnswer("Data Engineer")}
			>
				<div class="add-point" style={animatePointDe}>+{addPoint}</div>
				<h3>Data Engineer</h3>
			</button>
			{#each de as answer (answer.id)}
				<div
					class="answer {answer.correct ? 'correct' : 'incorrect'}"
					in:receive={{ key: answer.id }}
					out:send={{ key: answer.id }}
					animate:flip
				>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column">
			<button
				class="{shakeDs} {paradeDs}"
				on:click={() => handleAnswer("Data Scientist")}
			>
				<div class="add-point" style={animatePointDs}>+{addPoint}</div>
				<h3>Data Scientist</h3>
			</button>
			{#each ds as answer (answer.id)}
				<div
					class="answer {answer.correct ? 'correct' : 'incorrect'}"
					in:receive={{ key: answer.id }}
					out:send={{ key: answer.id }}
					animate:flip
				>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column">
			<button
				class="{shakeDv} {paradeDv}"
				on:click={() => handleAnswer("Datavisualisator")}
			>
				<div class="add-point" style={animatePointDv}>+{addPoint}</div>
				<h3>Datavisualisator</h3>
			</button>
			{#each dv as answer (answer.id)}
				<div
					class="answer {answer.correct ? 'correct' : 'incorrect'}"
					in:receive={{ key: answer.id }}
					out:send={{ key: answer.id }}
					animate:flip
				>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column">
			<button
				class="{shakeSre} {paradeSre}"
				on:click={() => handleAnswer("Site Reliability Engineer (SRE)")}
			>
				<div class="add-point" style={animatePointSre}>+{addPoint}</div>
				<h3>Site Reliability Engineer (SRE)</h3>
			</button>
			{#each sre as answer (answer.id)}
				<div
					class="answer {answer.correct ? 'correct' : 'incorrect'}"
					in:receive={{ key: answer.id }}
					out:send={{ key: answer.id }}
					animate:flip
				>
					{answer.description}
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	* {
		font-family: Arial, Helvetica, sans-serif;
	}
	#prompt-banner {
		display: flex;
		flex-direction: row;
	}
	#avatar {
		width: 200px;
		height: 200px;
		border-radius: 100px;
		transition: background-color 0.2s;
	}
	#text-cloud {
		font-family: "Kalam", cursive;
		font-weight: 300;
		font-style: normal;
		flex: 1;
		padding-left: 10px;
		padding-top: 15px;
	}
	#prompt {
		font-family: "Kalam", cursive;
		font-style: normal;
		font-weight: 700;
		font-size: 30px;
	}
	#score {
		width: 200px;
		text-align: right;
		padding-right: 50px;
	}

	.column {
		float: left;
		width: 20%;
		padding: 20px;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
	}

	button {
		margin-bottom: 10px;
		height: 44px;
		padding: 0px 16px;
		font-size: small;
		background-color: #42145f;
		color: white;
		border: none;
		position: relative;
	}
	button:hover {
		background-color: #714f87;
		cursor: pointer;
	}
	.add-point {
		font-family: "Kalam", cursive;
		font-weight: 400;
		font-style: normal;
		font-size: 40px;
		color: black;
		position: absolute;
		float: left;
		transform: translate(40px, 0);
		transition: transform 1s;
		opacity: 0;
	}

	.answer {
		top: 0;
		left: 0;
		display: block;
		font-size: 1em;
		line-height: 1;
		padding: 0.5em;
		margin: 0 auto 0.5em auto;
		border-radius: 2px;
		user-select: none;
		color: black;
		transition: transform 0.2s;
	}
	.correct {
		background-color: #e6e6e6;
	}
	.incorrect {
		background-color: #c6b8ce;
	}

	.answer:hover {
		transform: scale(1.1);
	}

	.shaking {
		animation: shake 0.5s infinite;
	}

	@keyframes shake {
		0% {
			transform: translate(1px, 1px) rotate(0deg);
		}
		10% {
			transform: translate(-1px, -2px) rotate(-1deg);
		}
		20% {
			transform: translate(-3px, 0px) rotate(1deg);
		}
		30% {
			transform: translate(3px, 2px) rotate(0deg);
		}
		40% {
			transform: translate(1px, -1px) rotate(1deg);
		}
		50% {
			transform: translate(-1px, 2px) rotate(-1deg);
		}
		60% {
			transform: translate(-3px, 1px) rotate(0deg);
		}
		70% {
			transform: translate(3px, 1px) rotate(-1deg);
		}
		80% {
			transform: translate(-1px, -1px) rotate(1deg);
		}
		90% {
			transform: translate(1px, 2px) rotate(0deg);
		}
		100% {
			transform: translate(1px, -2px) rotate(-1deg);
		}
	}

	.parading {
		animation: parade 0.1s 4 alternate;
	}

	@keyframes parade {
		from {
			box-shadow: 0 0 -10px -10px #c6b8ce;
		}
		to {
			box-shadow: 0 0 10px 10px #c6b8ce;
		}
	}
</style>
