<svelte:head>
    <link href="https://fonts.googleapis.com/css2?family=Kalam:wght@300;400;700&display=swap" rel="stylesheet">
</svelte:head>


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
    let paradeDe = "";
	let paradeDs = "";
	let paradeDv = "";
	let paradeSre = "";
	
	let uid = 0;
	let score = 0;

    let colors = ["#42145F", "#714F87", "#8D729F", "#A995B7", "#C6B8CE", "#E3DCE7"];
    let randomColor = colors[3];

    let avatars = ["", "1"];
    let randomAvatar = avatars[0];
	
	function selectRandomQuestion() {
        if(data.length == 0){
            return false;
        }
        randomColor = colors[Math.floor(Math.random()*colors.length)]
        randomAvatar = avatars[Math.floor(Math.random()*avatars.length)]
		return data.splice(Math.floor(Math.random()*data.length), 1)[0];
	}
	let currentQuestion = selectRandomQuestion();

	function handleAnswer(role) {
        if(currentQuestion == false){return;}

		// Check answer
		let correct = currentQuestion.belongs_to.includes(role);
		
        // Give visual feedback
        if(!correct){
			shakeButton(role);
		}
        for(let correctRole of currentQuestion.belongs_to){
            paradeButton(correctRole);
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

    function paradeButton(role){
		if (role === "data engineer"){
				paradeDe = "parading";
		} else if (role === "data scientist"){
				paradeDs = "parading";
		} else if (role === "datavisualisator"){
				paradeDv = "parading";
		} else if (role === "Site Reliability Engineer (SRE)"){
				paradeSre = "parading";
		}
		setTimeout(() => {
			paradeDe = "";
			paradeDs = "";
			paradeDv = "";
			paradeSre = "";
		}, 500);
	}
</script>

<div>
    <div id="prompt-banner">
        <div id="avatar" style="background-color: {randomColor};">
            <img src="./img/avatar{randomAvatar}.png"/>
        </div>
        <div id="text-cloud">
            Hoe goed ken jij de verschillende specialisaties binnen het RWS Datalab? Welke rol heb ik?
            {#if currentQuestion}
                <div id="prompt">Ik {currentQuestion.description}</div>
            {:else}
            <br><br>Dit waren alle vragen, dank voor het spelen!
            {/if}
        </div>
        <div id="score">Score {score}/{de.length + ds.length + dv.length + sre.length}</div>
    </div>
	<div id="answer_container">
		<div class="column"><button class="{shakeDe} {paradeDe}" on:click={() => handleAnswer("data engineer")}><h3>data engineer</h3></button>		
			{#each de as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column"><button class="{shakeDs} {paradeDs}" on:click={() => handleAnswer("data scientist")}><h3>data scientist</h3></button>
			{#each ds as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column"><button class="{shakeDv} {paradeDv}" on:click={() => handleAnswer("datavisualisator")}><h3>datavisualisator</h3></button>
			{#each dv as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
		<div class="column"><button class="{shakeSre} {paradeSre}" on:click={() => handleAnswer("Site Reliability Engineer (SRE)")}><h3>site reliability engineer (SRE)</h3></button>
			{#each sre as answer (answer.id)}
				<div class="answer {answer.correct ? 'correct':'incorrect'}" in:receive={{ key: answer.id }} out:send={{ key: answer.id }} animate:flip>
					{answer.description}
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
    #prompt-banner{
        display: flex;
        flex-direction: row
    }
	#avatar{
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
        font-weight: 700;
        font-size: 30px;
    }
    #score {
		width: 200px;
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
        height: 44px;
        padding: 0px 16px;
        font-size: small;
        background-color: #42145f;
        color: white;
        border: none;
	}
    button:hover{
        background-color: #714F87;
        cursor: pointer;
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
        animation: shake 0.5s infinite;
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

    .parading {
        animation: parade 0.1s 4 alternate;
	}
	
	@keyframes parade {
        from {
            box-shadow: 0 0 -10px -10px #C6B8CE;
        }
        to {
            box-shadow: 0 0 10px 10px #C6B8CE;
        }
    }
</style>
