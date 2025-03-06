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
			text: 'Почему кошки',
			answers: [
				{ text: 'мурчат/мурлыкают', points: 4, revealed: false, guessedBy: null },
				{ text: 'мнут лапами', points: 20, revealed: false, guessedBy: null },
				{ text: 'любят валерьянку', points: 18, revealed: false, guessedBy: null },
				{ text: 'боятся воды', points: 15, revealed: false, guessedBy: null },
				{ text: 'и собаки враждуют', points: 10, revealed: false, guessedBy: null },
				{ text: 'кашляют/чихают', points: 10, revealed: false, guessedBy: null },
				{ text: 'любят коробки', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Почему собаки',
			answers: [
				{ text: 'лижут хозяев', points: 30, revealed: false, guessedBy: null },
				{ text: 'воют', points: 25, revealed: false, guessedBy: null },
				{ text: 'слипаются', points: 15, revealed: false, guessedBy: null },
				{ text: 'спят в ногах', points: 12, revealed: false, guessedBy: null },
				{ text: 'едят траву', points: 10, revealed: false, guessedBy: null },
				{ text: 'лижут уши', points: 10, revealed: false, guessedBy: null },
				{ text: 'облизывают ноги', points: 10, revealed: false, guessedBy: null },
				{ text: 'любят палки', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Можно ли выжить',
			answers: [
				{ text: 'в авиакатастрофе', points: 35, revealed: false, guessedBy: null },
				{ text: 'без скальпа', points: 25, revealed: false, guessedBy: null },
				{ text: 'после выстрела в голову', points: 15, revealed: false, guessedBy: null },
				{ text: 'после удара молнии', points: 12, revealed: false, guessedBy: null },
				{ text: 'если оторвался тромб', points: 10, revealed: false, guessedBy: null },
				{ text: 'после инсульта/инфаркта', points: 10, revealed: false, guessedBy: null },
				{ text: 'внутри торнадо', points: 10, revealed: false, guessedBy: null },
				{ text: 'в падающем лифте', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Что будет если съесть',
			answers: [
				{ text: 'еду с плесенью', points: 35, revealed: false, guessedBy: null },
				{ text: 'грифель от карандаша', points: 25, revealed: false, guessedBy: null },
				{ text: 'мухомор красный', points: 15, revealed: false, guessedBy: null },
				{ text: 'много хурмы', points: 12, revealed: false, guessedBy: null },
				{ text: 'много сладкого', points: 10, revealed: false, guessedBy: null },
				{ text: 'просроченный шоколад', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Почему люди',
			answers: [
				{ text: 'храпят', points: 35, revealed: false, guessedBy: null },
				{ text: 'икают', points: 25, revealed: false, guessedBy: null },
				{ text: 'зевают', points: 15, revealed: false, guessedBy: null },
				{ text: 'умирают', points: 12, revealed: false, guessedBy: null },
				{ text: 'изменяют', points: 10, revealed: false, guessedBy: null },
				{ text: 'седеют', points: 10, revealed: false, guessedBy: null },
				{ text: 'пьют алкоголь', points: 10, revealed: false, guessedBy: null },
				{ text: 'плачут', points: 10, revealed: false, guessedBy: null },
				{ text: 'картавят', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Существует ли',
			answers: [
				{ text: 'дед мороз', points: 35, revealed: false, guessedBy: null },
				{ text: 'бог', points: 25, revealed: false, guessedBy: null },
				{ text: 'шаровая молния', points: 15, revealed: false, guessedBy: null },
				{ text: 'девственная плева', points: 12, revealed: false, guessedBy: null },
				{ text: 'материнский инстинкт', points: 10, revealed: false, guessedBy: null },
				{ text: 'жизнь после смерти', points: 10, revealed: false, guessedBy: null },
				{ text: 'сглаз', points: 10, revealed: false, guessedBy: null },
				{ text: 'гипноз', points: 10, revealed: false, guessedBy: null },
				{ text: 'слово нету', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Кто убил',
			answers: [
				{ text: 'лору палмер', points: 35, revealed: false, guessedBy: null },
				{ text: 'пушкина', points: 25, revealed: false, guessedBy: null },
				{ text: 'кеннеди', points: 15, revealed: false, guessedBy: null },
				{ text: 'распутина', points: 12, revealed: false, guessedBy: null },
				{ text: 'лермонтова', points: 12, revealed: false, guessedBy: null },
				{ text: 'цезаря', points: 12, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Что будет если',
			answers: [
				{ text: 'отказаться от сахара', points: 35, revealed: false, guessedBy: null },
				{ text: 'бросить курить', points: 25, revealed: false, guessedBy: null },
				{ text: 'не спать', points: 15, revealed: false, guessedBy: null },
				{ text: 'много дрочить', points: 12, revealed: false, guessedBy: null },
				{ text: 'хрустеть пальцами', points: 10, revealed: false, guessedBy: null },
				{ text: 'не чистить зубы', points: 10, revealed: false, guessedBy: null },
				{ text: 'пить мало воды', points: 10, revealed: false, guessedBy: null },
				{ text: 'много плакать', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Как пахнет',
			answers: [
				{ text: 'газ', points: 35, revealed: false, guessedBy: null },
				{ text: 'аммиак', points: 25, revealed: false, guessedBy: null },
				{ text: 'мускус', points: 15, revealed: false, guessedBy: null },
				{ text: 'пачули', points: 12, revealed: false, guessedBy: null },
				{ text: 'скунс', points: 10, revealed: false, guessedBy: null },
				{ text: 'марихуана', points: 10, revealed: false, guessedBy: null },
				{ text: 'сперма', points: 10, revealed: false, guessedBy: null },
				{ text: 'угарный газ', points: 10, revealed: false, guessedBy: null },
				{ text: 'янтарь', points: 10, revealed: false, guessedBy: null }
			]
		},
		{
			text: 'Почему мои',
			answers: [
				{ text: 'волосы электризуются', points: 35, revealed: false, guessedBy: null },
				{ text: 'сторис смотрят боты', points: 25, revealed: false, guessedBy: null },
				{ text: 'статусы не видят', points: 15, revealed: false, guessedBy: null },
				{ text: 'руки всегда холодные', points: 12, revealed: false, guessedBy: null },
				{ text: 'сны сбываются', points: 10, revealed: false, guessedBy: null },
				{ text: 'волосы быстро жирнеют', points: 10, revealed: false, guessedBy: null }
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
				// Store only the state of questions (revealed/guessedBy), not the questions themselves
				questionStates: questions.map((q) => ({
					answers: q.answers.map((a) => ({
						revealed: a.revealed,
						guessedBy: a.guessedBy
					}))
				})),
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

					// Only load the state of the questions (revealed/guessedBy), not the questions themselves
					if (gameState.questionStates && gameState.questionStates.length === questions.length) {
						// Apply saved states to current questions
						questions = questions.map((sourceQuestion, qIndex) => {
							const savedQuestionState = gameState.questionStates[qIndex];

							// Apply saved answer states if available and arrays match in length
							if (
								savedQuestionState?.answers &&
								savedQuestionState.answers.length === sourceQuestion.answers.length
							) {
								return {
									...sourceQuestion,
									answers: sourceQuestion.answers.map((sourceAnswer, aIndex) => {
										const savedAnswerState = savedQuestionState.answers[aIndex];
										return {
											...sourceAnswer,
											revealed: savedAnswerState?.revealed || false,
											guessedBy: savedAnswerState?.guessedBy || null
										};
									})
								};
							}
							return sourceQuestion;
						});
					} else if (gameState.questions) {
						// Legacy support for old format
						// Keep source question text but use saved state
						questions = questions.map((sourceQuestion, qIndex) => {
							const savedQuestion = gameState.questions[qIndex];
							if (!savedQuestion) return sourceQuestion;

							return {
								...sourceQuestion,
								answers: sourceQuestion.answers.map((sourceAnswer, aIndex) => {
									const savedAnswer = savedQuestion.answers?.[aIndex];
									if (!savedAnswer) return sourceAnswer;

									return {
										...sourceAnswer,
										revealed: savedAnswer.revealed || false,
										guessedBy: savedAnswer.guessedBy || null
									};
								})
							};
						});
					}

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

	// Hard reset - clears localStorage completely and reloads the page
	function hardReset() {
		if (typeof window !== 'undefined') {
			localStorage.removeItem('familyFeudGameState');
			// Show toast and reload after a short delay
			displayToast('Complete reset - reloading with source questions...', 1500);
			setTimeout(() => {
				window.location.reload();
			}, 1500);
		}
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

		// Only end game if all answers in final question are revealed and user clicks "End Game"
		// No longer automatically advancing to next question

		// Display a toast when all answers are revealed
		if (allRevealed) {
			displayToast('All answers revealed! You can move to the next question.');
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

	// Function to check if we're using custom questions
	function hasCustomQuestions(): boolean {
		// Check if first question isn't a default question
		// This is a simple heuristic assuming edits were made to the first question
		return questions[0]?.text !== 'Name something you would find in a kitchen';
	}

	// Generate team names on mount
	onMount(() => {
		// Try to load saved state, if that fails, generate new team names
		if (!loadGameState()) {
			generateTeamNames();
		}

		// If we detect custom questions, show a toast message
		if (hasCustomQuestions()) {
			setTimeout(() => {
				displayToast(
					'Custom questions detected! If game state seems incorrect, use the Hard Reset button.',
					5000
				);
			}, 1000);
		}
	});
</script>

<main class="min-h-screen bg-white p-4 text-gray-800">
	{#if !gameStarted}
		<!-- Game setup screen -->
		<div class="mx-auto my-10 max-w-2xl rounded-lg bg-white p-6 shadow-sm">
			<div class="mb-8 text-center">
				<h1 class="mb-2 text-center text-4xl">
					<span class="text-blue-500">G</span><span class="text-red-500">o</span><span
						class="text-yellow-500">o</span
					><span class="text-blue-500">g</span><span class="text-green-500">l</span><span
						class="text-red-500">e</span
					>
					<span class="font-normal text-gray-700">Autocomplete</span>
				</h1>
				<p class="text-gray-500">Guess what people are searching for</p>
			</div>

			<div class="space-y-6">
				<div class="grid grid-cols-2 gap-6">
					{#each teams as team, i}
						<div
							class="rounded-lg border border-gray-200 bg-white p-4 shadow-sm transition-shadow hover:shadow-md"
						>
							<h2 class="mb-2 text-xl font-medium text-gray-700">Team {i + 1}</h2>
							<div class="mb-2 text-3xl">{team.emoji}</div>
							<button
								class="rounded-md bg-blue-500 px-3 py-1 text-white transition-colors hover:bg-blue-600"
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
					class="w-full rounded-lg bg-blue-500 px-4 py-3 text-xl font-medium text-white shadow-sm transition-colors hover:bg-blue-600"
					on:click={startGame}
				>
					Start Game
				</button>
			</div>
		</div>
	{:else if gameEnded}
		<!-- Winner screen -->
		<div class="mx-auto my-10 max-w-2xl rounded-lg bg-white p-6 text-center shadow-sm">
			<h1 class="mb-6 text-4xl font-medium text-gray-800">Game Over!</h1>

			{#if winner === -1}
				<div class="mb-4 text-6xl">👔 It's a tie! 👔</div>
				<p class="mb-8 text-2xl text-gray-700">Both teams scored {teams[0].score} points</p>
			{:else if winner !== null}
				<div class="mb-4 text-6xl">🎉 Winner! 🎉</div>
				<div class="mb-2 text-5xl">{teams[winner].emoji}</div>
				<p class="mb-8 text-2xl text-gray-700">with {teams[winner].score} points</p>
			{/if}

			<div class="mb-8 grid grid-cols-2 gap-8">
				{#each teams as team, i}
					<div class="rounded-lg border border-gray-200 bg-white p-4 shadow-sm">
						<h2 class="mb-2 text-xl font-medium text-gray-700">Team {i + 1}</h2>
						<div class="mb-2 text-3xl">{team.emoji}</div>
						<p class="text-2xl font-medium text-gray-900">{team.score} points</p>
					</div>
				{/each}
			</div>

			<button
				class="rounded-lg bg-blue-500 px-4 py-2 text-xl font-medium text-white shadow-sm transition-colors hover:bg-blue-600"
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
			<!-- Header with Google-like styling -->
			<div class="mb-6 text-center">
				<h1 class="mb-1 text-center text-3xl">
					<span class="text-blue-500">G</span><span class="text-red-500">o</span><span
						class="text-yellow-500">o</span
					><span class="text-blue-500">g</span><span class="text-green-500">l</span><span
						class="text-red-500">e</span
					>
					<span class="font-normal text-gray-700">Autocomplete</span>
				</h1>
				<p class="mb-4 text-sm text-gray-500">People also ask...</p>
			</div>

			<!-- Teams & Scores -->
			<div class="mb-6 grid grid-cols-2 gap-4">
				{#each teams as team, i}
					<div
						class="flex items-center justify-between rounded-lg border border-gray-200 bg-white p-4 shadow-sm"
					>
						<div>
							<h2 class="text-xl font-medium text-gray-700">{team.name}</h2>
							<div class="text-3xl">{team.emoji}</div>
						</div>
						<div class="text-4xl font-medium text-gray-900">{team.score}</div>
					</div>
				{/each}
			</div>

			<!-- Question navigation -->
			<div class="mb-4 flex items-center justify-between">
				<button
					class="rounded-lg bg-gray-100 px-4 py-2 text-gray-700 transition-colors hover:bg-gray-200 disabled:cursor-not-allowed disabled:opacity-50"
					on:click={prevQuestion}
					disabled={currentQuestionIndex === 0}
				>
					Previous
				</button>

				<div class="text-lg font-medium text-gray-700">
					Question {currentQuestionIndex + 1} of {questions.length}
					{#if questions[currentQuestionIndex].answers.every((answer) => answer.revealed)}
						<span
							class="ml-2 inline-block rounded-full bg-green-500 px-2 py-0.5 text-xs text-white"
						>
							All Revealed
						</span>
					{/if}
				</div>

				<button
					class="rounded-lg {questions[currentQuestionIndex].answers.every(
						(answer) => answer.revealed
					)
						? 'bg-green-500 hover:bg-green-600'
						: 'bg-gray-100 hover:bg-gray-200'} px-4 py-2 text-gray-700 text-white transition-colors disabled:cursor-not-allowed disabled:opacity-50"
					on:click={nextQuestion}
					disabled={currentQuestionIndex === questions.length - 1}
				>
					Next
				</button>
			</div>

			<!-- Current question - Google Search Bar Style -->
			<div class="mb-8 rounded-lg border border-gray-200 bg-white p-4 shadow-sm">
				<div class="mb-4 flex items-center">
					<div class="mr-3 text-gray-500">
						<svg class="h-5 w-5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
							<path
								d="M15.5 14H14.71L14.43 13.73C15.41 12.59 16 11.11 16 9.5C16 5.91 13.09 3 9.5 3C5.91 3 3 5.91 3 9.5C3 13.09 5.91 16 9.5 16C11.11 16 12.59 15.41 13.73 14.43L14 14.71V15.5L19 20.49L20.49 19L15.5 14ZM9.5 14C7.01 14 5 11.99 5 9.5C5 7.01 7.01 5 9.5 5C11.99 5 14 7.01 14 9.5C14 11.99 11.99 14 9.5 14Z"
								fill="currentColor"
							/>
						</svg>
					</div>
					<h2 class="text-xl text-gray-800">{questions[currentQuestionIndex].text}</h2>
				</div>

				<div class="space-y-3">
					{#each questions[currentQuestionIndex].answers as answer, answerIndex}
						<div
							class="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm transition-shadow hover:shadow-md"
						>
							{#if answer.revealed}
								<div class="flex items-center justify-between border-b border-gray-100 p-4">
									<div class="flex items-center">
										<span class="mr-3 text-xl text-gray-500">{answerIndex + 1}.</span>
										<span class="text-xl text-gray-800">{answer.text}</span>
									</div>
									<span class="text-xl font-medium text-gray-700">{answer.points}</span>
								</div>
								<div class="flex flex-wrap justify-end gap-2 bg-gray-50 p-2">
									<span class="mr-2 self-center font-medium text-gray-600">Assign points to:</span>
									{#each teams as team, teamIndex}
										<button
											class="rounded-md px-3 py-1 transition-colors
												{answer.guessedBy === teamIndex
												? 'bg-green-500 text-white hover:bg-green-600'
												: 'bg-blue-500 text-white hover:bg-blue-600'}"
											on:click={() => assignPoints(answerIndex, teamIndex)}
										>
											{team.emoji}
											{answer.guessedBy === teamIndex ? '✓' : ''}
										</button>
									{/each}
									<button
										class="rounded-md px-3 py-1 transition-colors
											{answer.guessedBy === null
											? 'bg-green-500 text-white hover:bg-green-600'
											: 'bg-gray-400 text-white hover:bg-gray-500'}"
										on:click={() => assignPoints(answerIndex, null)}
									>
										None {answer.guessedBy === null ? '✓' : ''}
									</button>
								</div>
							{:else}
								<div class="flex justify-between p-4">
									<div class="flex items-center">
										<span class="mr-3 text-xl text-gray-500">{answerIndex + 1}.</span>
										<span class="text-xl text-gray-400">. . . . . . . . .</span>
									</div>
									<button
										class="rounded-md bg-blue-500 px-3 py-1 font-medium text-white transition-colors hover:bg-blue-600"
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
					class="rounded-lg {currentQuestionIndex === questions.length - 1 &&
					questions[currentQuestionIndex].answers.every((answer) => answer.revealed)
						? 'animate-pulse bg-red-500 text-white hover:bg-red-600'
						: 'bg-red-500 text-white hover:bg-red-600'} px-6 py-2 shadow-sm transition-colors"
					on:click={endGame}
				>
					{currentQuestionIndex === questions.length - 1 &&
					questions[currentQuestionIndex].answers.every((answer) => answer.revealed)
						? 'End Game - All Questions Complete!'
						: 'End Game'}
				</button>
			</div>
		</div>
	{/if}

	<!-- Refresh buttons -->
	<div class="fixed right-4 bottom-4 flex flex-col gap-3">
		<!-- Hard Reset button -->
		<!-- svelte-ignore a11y_consider_explicit_label -->
		<button
			class="flex items-center justify-center rounded-full bg-red-500 p-3 text-white shadow-md transition-colors hover:bg-red-600"
			title="Hard Reset (Reload with Source Questions)"
			on:click={hardReset}
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
					d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
				/>
			</svg>
		</button>

		<!-- Regular Reset button -->
		<!-- svelte-ignore a11y_consider_explicit_label -->
		<button
			class="flex items-center justify-center rounded-full bg-blue-500 p-3 text-white shadow-md transition-colors hover:bg-blue-600"
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
			class="fixed right-4 bottom-20 max-w-xs rounded-md bg-gray-800 p-4 text-white shadow-lg transition-all"
		>
			<div class="flex items-center">
				<svg
					class="mr-2 h-6 w-6 text-green-500"
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
		font-family: 'Arial', 'Helvetica', sans-serif;
	}
</style>
