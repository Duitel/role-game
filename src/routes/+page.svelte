<script>
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';
	import { flip } from 'svelte/animate';

	import { data } from '$lib/data.js';
	
	const [send, receive] = crossfade({
		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	let de = [];
	let ds = [];
	let dv = [];
	let sre = [];

	let shakeDe = "";
	let shakeDs = "";
	let shakeDv = "";
	let shakeSre = "";
	
	let uid = 0;
	let score = 0;
	
	function selectRandomQuestion() {
		return data.splice(Math.floor(Math.random()*data.length), 1)[0];
	}
	let currentQuestion = selectRandomQuestion();

	function handleAnswer(role, button_id) {
		// Check answer
		let correct = currentQuestion.belongs_to.includes(role);
		if(!correct){
			shakeButton(role);
		}

		// Update score
		if(correct){
			score++;
		}
		
		// Add answer(s)
		const answer = {
			id: uid++,
			description: currentQuestion.description,
			correct: correct
		};

		if(currentQuestion.belongs_to.includes("data engineer")){
			de = [answer, ...de];
		}
		if(currentQuestion.belongs_to.includes("data scientist")){
			ds = [answer, ...ds];
		}
		if(currentQuestion.belongs_to.includes("datavisualisator")){
			dv = [answer, ...dv];
		}
		if(currentQuestion.belongs_to.includes("Site Reliability Engineer (SRE)")){
			sre = [answer, ...sre];
		}
		currentQuestion = selectRandomQuestion();
	}

	function shakeButton(role){
		console.log(role);
		if (role === "data engineer"){
				shakeDe = "shaking";
		} else if (role === "data scientist"){
				shakeDs = "shaking";
		} else if (role === "datavisualisator"){
				shakeDv = "shaking";
		} else if (role === "Site Reliability Engineer (SRE)"){
				shakeSre = "shaking";
		}
		setTimeout(() => {
			shakeDe = "";
			shakeDs = "";
			shakeDv = "";
			shakeSre = "";
		}, 250);
	}
</script>

<div>
	{#if de.length + ds.length + dv.length + sre.length > 0}
		<div id="score">Score {score}/{de.length + ds.length + dv.length + sre.length}</div>
	{/if}
	<div id="prompt">bij welke rol hoort...<h2>{currentQuestion.description}</h2><hr></div>
	<div id="answer_container">
		<div class="column"><button class={shakeDe} on:click={() => handleAnswer("data engineer")}><h3>data engineer</h3></button>		
			{#each de as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column"><button class={shakeDs} on:click={() => handleAnswer("data scientist")}><h3>data scientist</h3></button>
			{#each ds as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column"><button class={shakeDv} on:click={() => handleAnswer("datavisualisator")}><h3>datavisualisator</h3></button>
			{#each dv as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column"><button class={shakeSre} on:click={() => handleAnswer("Site Reliability Engineer (SRE)")}><h3>site reliability engineer (SRE)</h3></button>
			{#each sre as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	#score {
		position: absolute;
		right: 100px;
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
	}
	.answer{
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
		transition: transform .2s;
	}
	.correct{
		background-color: #e6e6e6;
	}
	.incorrect{
		background-color: #C6B8CE;
	}

	.answer:hover {
		transform: scale(1.1);
	}

	.shaking {
    /* Start the shake animation and make the animation last for 0.5 seconds */
    animation: shake 0.5s;

    /* When the animation is finished, start again */
    animation-iteration-count: infinite;
	}
	
	@keyframes shake {
	    0% { transform: translate(1px, 1px) rotate(0deg); }
	    10% { transform: translate(-1px, -2px) rotate(-1deg); }
	    20% { transform: translate(-3px, 0px) rotate(1deg); }
	    30% { transform: translate(3px, 2px) rotate(0deg); }
	    40% { transform: translate(1px, -1px) rotate(1deg); }
	    50% { transform: translate(-1px, 2px) rotate(-1deg); }
	    60% { transform: translate(-3px, 1px) rotate(0deg); }
	    70% { transform: translate(3px, 1px) rotate(-1deg); }
	    80% { transform: translate(-1px, -1px) rotate(1deg); }
	    90% { transform: translate(1px, 2px) rotate(0deg); }
	    100% { transform: translate(1px, -2px) rotate(-1deg); }
	}
</style>
