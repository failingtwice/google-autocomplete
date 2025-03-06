<script lang="ts">
	import { onMount } from 'svelte';

	interface Answer {
		text: string;
		points: number;
		revealed: boolean;
		guessedBy: number | null;
	}

	interface Question {
		text: string;
		answers: Answer[];
	}

	// Team data
	let teams = [
		{ name: '', emoji: '', score: 0, emojiPair: [] as string[] },
		{ name: '', emoji: '', score: 0, emojiPair: [] as string[] }
	];

	// Game state
	let gameStarted = false;
	let gameEnded = false;
	let winner: number | null = null;
	let questions: Question[] = [
		{
			text: 'Name something you would find in a kitchen',
			answers: [
				{ text: 'Refrigerator', points: 25, revealed: false, guessedBy: null },
				{ text: 'Stove/Oven', points: 20, revealed: false, guessedBy: null },
				{ text: 'Microwave', points: 18, revealed: false, guessedBy: null },
				{ text: 'Sink', points: 15, revealed: false, guessedBy: null },
				{ text: 'Dishwasher', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Name a popular vacation destination',
			answers: [
				{ text: 'Beach/Ocean', points: 30, revealed: false, guessedBy: null },
				{ text: 'Disney World/Land', points: 25, revealed: false, guessedBy: null },
				{ text: 'Mountains', points: 15, revealed: false, guessedBy: null },
				{ text: 'Europe', points: 12, revealed: false, guessedBy: null },
				{ text: 'National Park', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Name something people are afraid of',
			answers: [
				{ text: 'Spiders', points: 35, revealed: false, guessedBy: null },
				{ text: 'Heights', points: 25, revealed: false, guessedBy: null },
				{ text: 'Death', points: 15, revealed: false, guessedBy: null },
				{ text: 'Public Speaking', points: 12, revealed: false, guessedBy: null },
				{ text: 'Snakes', points: 10, revealed: false, guessedBy: null }
			]
		}
	];
	let currentQuestionIndex = 0;

	// Instrument and animal emojis for team names
	const instrumentEmojis = ['🎸', '🎹', '🎺', '🎻', '🥁', '🎷', '🪕', '🪗', '🎤', '🪘', '📯'];
	const animalEmojis = [
		'🐶',
		'🐱',
		'🐭',
		'🐹',
		'🐰',
		'🦊',
		'🐻',
		'🐼',
		'🐨',
		'🐯',
		'🦁',
		'🐮',
		'🐷',
		'🐸',
		'🐙',
		'🦄',
		'🦓',
		'🦍',
		'🐘',
		'🦒'
	];

	// Generate random team names
	function generateTeamNames() {
		teams = teams.map((team) => {
			const instrument = instrumentEmojis[Math.floor(Math.random() * instrumentEmojis.length)];
			const animal = animalEmojis[Math.floor(Math.random() * animalEmojis.length)];
			return {
				...team,
				name: `Team ${instrument}${animal}`,
				emoji: `${instrument}${animal}`,
				emojiPair: [instrument, animal]
			};
		});
	}

	// LocalStorage functions
	function saveGameState() {
		if (typeof window !== 'undefined') {
			const gameState = {
				teams,
				questions,
				currentQuestionIndex,
				gameStarted,
				gameEnded,
				winner
			};
			localStorage.setItem('familyFeudGameState', JSON.stringify(gameState));
		}
	}

	function loadGameState(): boolean {
		if (typeof window !== 'undefined') {
			const savedState = localStorage.getItem('familyFeudGameState');
			if (savedState) {
				try {
					const gameState = JSON.parse(savedState);
					teams = gameState.teams;
					questions = gameState.questions;
					currentQuestionIndex = gameState.currentQuestionIndex;
					gameStarted = gameState.gameStarted;
					gameEnded = gameState.gameEnded;
					winner = gameState.winner;
					return true;
				} catch (error) {
					console.error('Error loading game state:', error);
				}
			}
		}
		return false;
	}

	// Toast notification
	let showToast = false;
	let toastMessage = '';

	function displayToast(message: string, duration: number = 3000) {
		toastMessage = message;
		showToast = true;
		setTimeout(() => {
			showToast = false;
		}, duration);
	}

	function resetGameState() {
		// Clear localStorage
		if (typeof window !== 'undefined') {
			localStorage.removeItem('familyFeudGameState');
		}

		// Reset to initial state
		gameStarted = false;
		gameEnded = false;
		winner = null;
		currentQuestionIndex = 0;

		// Reset teams but keep current emojis
		teams = teams.map((team) => ({
			...team,
			score: 0
		}));

		// Reset questions and answers
		questions = questions.map((question) => ({
			...question,
			answers: question.answers.map((answer) => ({
				...answer,
				revealed: false,
				guessedBy: null
			}))
		}));

		// Generate new team names
		generateTeamNames();

		// Show a temporary message
		const originalTitle = document.title;
		document.title = 'Game Reset!';
		setTimeout(() => {
			document.title = originalTitle;
		}, 1500);

		// Display toast notification
		displayToast('Game reset successfully!');
	}

	// Start game
	function startGame() {
		gameStarted = true;
		gameEnded = false;
		winner = null;
		teams.forEach((team) => (team.score = 0));
		questions.forEach((question) => {
			question.answers.forEach((answer) => {
				answer.revealed = false;
				answer.guessedBy = null;
			});
		});
		currentQuestionIndex = 0;
		saveGameState();
	}

	// Reveal answer
	function revealAnswer(answerIndex: number) {
		const answer = questions[currentQuestionIndex].answers[answerIndex];
		if (!answer.revealed) {
			// Create a copy of the questions array to trigger reactivity
			const updatedQuestions = [...questions];
			updatedQuestions[currentQuestionIndex].answers[answerIndex].revealed = true;
			updatedQuestions[currentQuestionIndex].answers[answerIndex].guessedBy = null;
			questions = updatedQuestions;

			// Save game state
			saveGameState();
		}
	}

	// Assign points to team
	function assignPoints(answerIndex: number, teamIndex: number | null) {
		const answer = questions[currentQuestionIndex].answers[answerIndex];

		// Only allow assigning points if the answer is revealed
		if (answer.revealed) {
			// Create a copy to trigger reactivity
			const updatedQuestions = [...questions];
			const updatedTeams = [...teams];

			// If a team was previously assigned, remove points
			if (answer.guessedBy !== null && answer.guessedBy >= 0) {
				updatedTeams[answer.guessedBy].score -= answer.points;
			}

			// Assign to new team
			updatedQuestions[currentQuestionIndex].answers[answerIndex].guessedBy = teamIndex;

			// Add points to team
			if (teamIndex !== null && teamIndex >= 0) {
				updatedTeams[teamIndex].score += answer.points;
			}

			// Update both arrays to trigger reactivity
			questions = updatedQuestions;
			teams = updatedTeams;

			// Save game state
			saveGameState();

			// Check if all answers are revealed
			checkRoundEnd();
		}
	}

	// Check if round should end
	function checkRoundEnd() {
		const allRevealed = questions[currentQuestionIndex].answers.every((answer) => answer.revealed);
		if (allRevealed && currentQuestionIndex < questions.length - 1) {
			// Increment to next question with a slight delay to let the user see the fully revealed answers
			setTimeout(() => {
				currentQuestionIndex++;
			}, 500);
		} else if (allRevealed && currentQuestionIndex === questions.length - 1) {
			// End the game with a slight delay
			setTimeout(() => {
				endGame();
			}, 500);
		}
	}

	// End game and determine winner
	function endGame() {
		gameEnded = true;
		if (teams[0].score > teams[1].score) {
			winner = 0;
		} else if (teams[1].score > teams[0].score) {
			winner = 1;
		} else {
			winner = -1; // Tie
		}
		saveGameState();
	}

	// Next question
	function nextQuestion() {
		if (currentQuestionIndex < questions.length - 1) {
			currentQuestionIndex++;
			saveGameState();
		}
	}

	// Previous question
	function prevQuestion() {
		if (currentQuestionIndex > 0) {
			currentQuestionIndex--;
			saveGameState();
		}
	}

	// Generate team names on mount
	onMount(() => {
		// Try to load saved state, if that fails, generate new team names
		if (!loadGameState()) {
			generateTeamNames();
		}
	});
</script>

<main class="min-h-screen bg-blue-900 p-4 text-white">
	{#if !gameStarted}
		<!-- Game setup screen -->
		<div class="mx-auto my-10 max-w-2xl rounded-lg bg-blue-800 p-6 shadow-xl">
			<h1 class="mb-6 text-center text-4xl font-bold">Family Feud</h1>

			<div class="space-y-6">
				<div class="grid grid-cols-2 gap-6">
					{#each teams as team, i}
						<div class="rounded-lg bg-blue-700 p-4">
							<h2 class="mb-2 text-xl font-bold">Team {i + 1}</h2>
							<div class="mb-2 text-3xl">{team.emoji}</div>
							<button
								class="rounded-md bg-blue-500 px-3 py-1 transition-colors hover:bg-blue-600"
								on:click={() => {
									const newTeams = [...teams];
									const instrument =
										instrumentEmojis[Math.floor(Math.random() * instrumentEmojis.length)];
									const animal = animalEmojis[Math.floor(Math.random() * animalEmojis.length)];
									newTeams[i] = {
										...team,
										name: `Team ${instrument}${animal}`,
										emoji: `${instrument}${animal}`,
										emojiPair: [instrument, animal]
									};
									teams = newTeams;
								}}
							>
								Randomize
							</button>
						</div>
					{/each}
				</div>

				<button
					class="w-full rounded-lg bg-yellow-500 px-4 py-2 text-xl font-bold text-black transition-colors hover:bg-yellow-600"
					on:click={startGame}
				>
					Start Game
				</button>
			</div>
		</div>
	{:else if gameEnded}
		<!-- Winner screen -->
		<div class="mx-auto my-10 max-w-2xl rounded-lg bg-blue-800 p-6 text-center shadow-xl">
			<h1 class="mb-6 text-4xl font-bold">Game Over!</h1>

			{#if winner === -1}
				<div class="mb-4 text-6xl">👔 It's a tie! 👔</div>
				<p class="mb-8 text-2xl">Both teams scored {teams[0].score} points</p>
			{:else if winner !== null}
				<div class="mb-4 text-6xl">🎉 Winner! 🎉</div>
				<div class="mb-2 text-5xl">{teams[winner].emoji}</div>
				<p class="mb-8 text-2xl">with {teams[winner].score} points</p>
			{/if}

			<div class="mb-8 grid grid-cols-2 gap-8">
				{#each teams as team, i}
					<div class="rounded-lg bg-blue-700 p-4">
						<h2 class="mb-2 text-xl font-bold">Team {i + 1}</h2>
						<div class="mb-2 text-3xl">{team.emoji}</div>
						<p class="text-2xl font-bold">{team.score} points</p>
					</div>
				{/each}
			</div>

			<button
				class="rounded-lg bg-yellow-500 px-4 py-2 text-xl font-bold text-black transition-colors hover:bg-yellow-600"
				on:click={() => {
					gameStarted = false;
					generateTeamNames();
				}}
			>
				Play Again
			</button>
		</div>
	{:else}
		<!-- Game screen -->
		<div class="mx-auto max-w-6xl">
			<!-- Teams & Scores -->
			<div class="mb-6 grid grid-cols-2 gap-4">
				{#each teams as team, i}
					<div class="flex items-center justify-between rounded-lg bg-blue-800 p-4">
						<div>
							<h2 class="text-xl font-bold">{team.name}</h2>
							<div class="text-3xl">{team.emoji}</div>
						</div>
						<div class="text-4xl font-bold">{team.score}</div>
					</div>
				{/each}
			</div>

			<!-- Question navigation -->
			<div class="mb-4 flex items-center justify-between">
				<button
					class="rounded-lg bg-blue-700 px-4 py-2 hover:bg-blue-600 disabled:cursor-not-allowed disabled:opacity-50"
					on:click={prevQuestion}
					disabled={currentQuestionIndex === 0}
				>
					Previous
				</button>

				<div class="text-lg font-semibold">
					Question {currentQuestionIndex + 1} of {questions.length}
				</div>

				<button
					class="rounded-lg bg-blue-700 px-4 py-2 hover:bg-blue-600 disabled:cursor-not-allowed disabled:opacity-50"
					on:click={nextQuestion}
					disabled={currentQuestionIndex === questions.length - 1}
				>
					Next
				</button>
			</div>

			<!-- Current question -->
			<div class="mb-6 rounded-lg bg-blue-800 p-6">
				<h2 class="mb-4 text-2xl font-bold">{questions[currentQuestionIndex].text}</h2>

				<div class="space-y-3">
					{#each questions[currentQuestionIndex].answers as answer, answerIndex}
						<div class="overflow-hidden rounded-lg bg-gray-800 shadow-lg">
							{#if answer.revealed}
								<div class="flex items-center justify-between p-4">
									<span class="text-xl">{answer.text}</span>
									<span class="text-xl font-bold">{answer.points}</span>
								</div>
								<div class="flex flex-wrap justify-end gap-2 bg-gray-700 p-2">
									<span class="mr-2 self-center font-medium">Assign points to:</span>
									{#each teams as team, teamIndex}
										<button
											class="rounded-md px-3 py-1 transition-colors
												{answer.guessedBy === teamIndex
												? 'bg-green-600 hover:bg-green-700'
												: 'bg-blue-600 hover:bg-blue-500'}"
											on:click={() => assignPoints(answerIndex, teamIndex)}
										>
											{team.emoji}
											{answer.guessedBy === teamIndex ? '✓' : ''}
										</button>
									{/each}
									<button
										class="rounded-md px-3 py-1 transition-colors
											{answer.guessedBy === null ? 'bg-green-600 hover:bg-green-700' : 'bg-gray-600 hover:bg-gray-500'}"
										on:click={() => assignPoints(answerIndex, null)}
									>
										None {answer.guessedBy === null ? '✓' : ''}
									</button>
								</div>
							{:else}
								<div class="flex justify-between p-4">
									<span class="text-xl">?????</span>
									<button
										class="rounded-md bg-yellow-500 px-3 py-1 font-medium text-black hover:bg-yellow-600"
										on:click={() => revealAnswer(answerIndex)}
									>
										Reveal Answer
									</button>
								</div>
							{/if}
						</div>
					{/each}
				</div>
			</div>

			<!-- End game button -->
			<div class="text-center">
				<button
					class="rounded-lg bg-red-600 px-6 py-2 transition-colors hover:bg-red-700"
					on:click={endGame}
				>
					End Game
				</button>
			</div>
		</div>
	{/if}

	<!-- Refresh button -->
	<div class="fixed right-4 bottom-4">
		<!-- svelte-ignore a11y_consider_explicit_label -->
		<button
			class="flex items-center justify-center rounded-full bg-blue-600 p-3 shadow-lg transition-colors hover:bg-blue-700"
			title="Reset Game State"
			on:click={resetGameState}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-6 w-6"
				fill="none"
				viewBox="0 0 24 24"
				stroke="currentColor"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
				/>
			</svg>
		</button>
	</div>

	<!-- Toast notification -->
	{#if showToast}
		<div
			class="fixed right-4 bottom-20 max-w-xs rounded-md bg-green-600 p-4 shadow-lg transition-all"
		>
			<div class="flex items-center">
				<svg
					class="mr-2 h-6 w-6"
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M5 13l4 4L19 7"
					/>
				</svg>
				<span>{toastMessage}</span>
			</div>
		</div>
	{/if}
</main>

<style>
	main {
		font-family: 'Inter', sans-serif;
	}
</style>
