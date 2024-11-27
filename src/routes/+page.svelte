<script lang="ts">
    import { onMount } from 'svelte';
    import { fade, scale } from 'svelte/transition';
    import lizard from '$lib/images/icon-lizard.svg';
    import paper from '$lib/images/icon-paper.svg';
    import rock from '$lib/images/icon-rock.svg';
    import scissors from '$lib/images/icon-scissors.svg';
    import spock from '$lib/images/icon-spock.svg';
    import rule from '$lib/images/image-rules.svg';
    import iconclose from '$lib/images/icon-close.svg';
    import logo from '$lib/images/logo.svg';
    import bgPentagon from '$lib/images/bg-pentagon.svg';
    import bgTriangle from '$lib/images/bg-triangle.svg';
    import bonusrules from '$lib/images/image-rules-bonus.svg';
  
    type Choice = 'rock' | 'paper' | 'scissors' | 'lizard' | 'spock';
    type GameState = 'picking' | 'picked' | 'result';
    type GameMode = 'normal' | 'bonus';
    type RuleAction = 'crushes' | 'covers' | 'disproves' | 'cuts' | 'decapitates' | 'eats' | 'poisons' | 'smashes' | 'vaporizes';
    type Rules = {
      [K in Choice]: {
        [L in Choice]?: RuleAction;
      };
    };
  
    let score = $state(0);
    let playerChoice = $state<Choice | null>(null);
    let houseChoice = $state<Choice | null>(null);
    let gameState = $state<GameState>('picking');
    let result = $state<'win' | 'lose' | 'draw' | null>(null);
    let showRules = $state(false);
    let isMobile = $state(false);
    let gameMode = $state<GameMode>('normal');
  
    const normalChoices: Choice[] = ['rock', 'paper', 'scissors'];
    const bonusChoices: Choice[] = ['rock', 'paper', 'scissors', 'lizard', 'spock'];
    
    const rules: Rules = {
      rock: { scissors: 'crushes', lizard: 'crushes' },
      paper: { rock: 'covers', spock: 'disproves' },
      scissors: { paper: 'cuts', lizard: 'decapitates' },
      lizard: { paper: 'eats', spock: 'poisons' },
      spock: { scissors: 'smashes', rock: 'vaporizes' }
    };
  
    const imageMap = {
      rock,
      paper,
      scissors,
      lizard,
      spock
    };
  
    onMount(() => {
      const savedScore = localStorage.getItem('rps-score');
      if (savedScore) score = parseInt(savedScore);
  
      const checkMobile = () => {
        isMobile = window.innerWidth < 768;
      };
      
      checkMobile();
      window.addEventListener('resize', checkMobile);
      
      return () => {
        window.removeEventListener('resize', checkMobile);
      };
    });
  
    function updateScore(newScore: number) {
      score = newScore;
      localStorage.setItem('rps-score', score.toString());
    }
  
    function makeChoice(choice: Choice) {
      playerChoice = choice;
      gameState = 'picked';
      
      setTimeout(() => {
        const choices = gameMode === 'normal' ? normalChoices : bonusChoices;
        const randomChoice = choices[Math.floor(Math.random() * choices.length)] as Choice;
        houseChoice = randomChoice;
        
        if (playerChoice === houseChoice) {
          result = 'draw';
        } else if (playerChoice && houseChoice && getWinningAction(playerChoice, houseChoice)) {
          result = 'win';
          updateScore(score + 1);
        } else {
          result = 'lose';
          updateScore(Math.max(0, score - 1));
        }
        
        gameState = 'result';
      }, 1000);
    }
  
    function playAgain() {
      playerChoice = null;
      houseChoice = null;
      result = null;
      gameState = 'picking';
      gameMode = Math.random() < 0.2 ? 'bonus' : 'normal'; // 20% chance for bonus round
    }
  
    function getChoiceColor(choice: Choice): string {
      switch (choice) {
        case 'rock': return 'border-red-500';
        case 'paper': return 'border-blue-500';
        case 'scissors': return 'border-yellow-500';
        case 'lizard': return 'border-purple-500';
        case 'spock': return 'border-cyan-500';
        default: return '';
      }
    }
  
    function getWinningAction(attacker: Choice, defender: Choice): RuleAction | undefined {
      return rules[attacker]?.[defender];
    }
  
    let choices = $state<Choice[]>(normalChoices);
    $effect(() => {
      choices = gameMode === 'normal' ? normalChoices : bonusChoices;
    });
  </script>
  
  <div class="min-h-screen bg-[#1f3756] text-white flex flex-col items-center p-4 md:p-8">
    <header class="w-full max-w-2xl flex justify-between items-center border-2 border-gray-600 rounded-xl p-3 md:p-4 mb-8 md:mb-16">
      <div class="flex items-center gap-4">
        <img src={logo} alt="Rock Paper Scissors Logo" class="h-12 md:h-20" />
        <div class="hFlex flex flex-col leading-none">
          <span class="text-lg md:text-2xl font-bold">ROCK</span>
          <span class="text-lg md:text-2xl font-bold">PAPER</span>
          <span class="text-lg md:text-2xl font-bold">SCISSORS</span>
          {#if gameMode === 'bonus'}
            <span class="text-lg md:text-2xl font-bold">LIZARD</span>
            <span class="text-lg md:text-2xl font-bold">SPOCK</span>
          {/if}
        </div>
      </div>
      <div class="bg-white text-gray-900 rounded-lg p-3 md:p-4 flex flex-col items-center min-w-[80px] md:min-w-[120px]">
        <span class="text-xs md:text-sm text-blue-600 font-semibold">SCORE</span>
        <span class="text-3xl md:text-5xl font-bold">{score}</span>
      </div>
    </header>

    <main class="flex-1 w-full max-w-2xl flex flex-col justify-center items-center">
      {#if gameState === 'picking'}
        <div class="relative w-[300px] h-[300px] md:w-[400px] md:h-[400px] mx-auto" in:fade>
          <!-- Pentagon background for bonus mode -->
          {#if gameMode === 'bonus'}
            <img src={bgPentagon} alt="" class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-3/4 h-3/4" />
          {:else}
            <img src={bgTriangle} alt="" class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-3/4 h-3/4" />
          {/if}
          
          {#each choices as choice, i (choice)}
            {@const angle = gameMode === 'bonus' 
              ? (i * (360 / choices.length) - 90) 
              : i === 0 ? -60 : i === 1 ? 60 : 180}
            {@const radius = gameMode === 'bonus' ? '120px' : '100px'}
            {@const top = gameMode === 'bonus'
              ? `${50 + Math.sin(angle * Math.PI / 180) * 40}%`
              : i === 2 ? '75%' : '25%'}
            {@const left = gameMode === 'bonus'
              ? `${50 + Math.cos(angle * Math.PI / 180) * 40}%`
              : i === 0 ? '25%' : i === 1 ? '75%' : '50%'}
            
            <button
              class="absolute w-32 h-32 md:w-40 md:h-40 rounded-full bg-white flex items-center justify-center border-[12px] md:border-[16px] transform hover:scale-110 transition-transform -translate-x-1/2 -translate-y-1/2"
              class:border-red-500={choice === 'rock'}
              class:border-blue-500={choice === 'paper'}
              class:border-yellow-500={choice === 'scissors'}
              class:border-purple-500={choice === 'lizard'}
              class:border-cyan-500={choice === 'spock'}
              onclick={() => makeChoice(choice)}
              aria-label={`Choose ${choice}`}
              style="top: {top}; left: {left};"
            >
              <img
                src={imageMap[choice]}
                alt={String(choice)}
                class="w-12 h-12 md:w-16 md:h-16"
              />
            </button>
          {/each}
        </div>
      {:else}
        <div class="w-full flex flex-col md:flex-row justify-center items-center gap-12 md:gap-16 px-4" in:fade>
          <div class="text-center w-full md:w-auto">
            <p class="mb-4 text-sm md:text-lg uppercase tracking-widest">You Picked</p>
            <div
              class="w-28 h-28 md:w-36 md:h-36 rounded-full bg-white flex items-center justify-center border-[12px] md:border-[16px] mx-auto relative"
              class:border-red-500={playerChoice === 'rock'}
              class:border-blue-500={playerChoice === 'paper'}
              class:border-yellow-500={playerChoice === 'scissors'}
              class:border-purple-500={playerChoice === 'lizard'}
              class:border-cyan-500={playerChoice === 'spock'}
              class:winner={result === 'win'}
              in:scale
            >
              <img
                src={playerChoice ? imageMap[playerChoice] : ''}
                alt={playerChoice || ''}
                class="w-14 h-14 md:w-16 md:h-16"
              />
            </div>
          </div>

          {#if gameState === 'result'}
            <div class="text-center w-full md:w-auto order-first md:order-none" in:fade>
              <h2 class="text-4xl md:text-5xl font-bold mb-4">
                YOU {result === 'win' ? 'WIN' : result === 'lose' ? 'LOSE' : 'DRAW'}
              </h2>
              {#if result === 'win' || result === 'lose'}
                <p class="text-base md:text-xl mb-4">
                  {#if playerChoice && houseChoice && rules[playerChoice]?.[houseChoice]}
                    {playerChoice} {rules[playerChoice][houseChoice]} {houseChoice}
                  {/if}
                </p>
              {/if}
              <button
                class="bg-white text-gray-900 px-8 md:px-12 py-2 md:py-3 rounded-lg hover:bg-gray-100 transition-colors text-sm md:text-base uppercase tracking-wider"
                onclick={playAgain}
              >
                Play Again
              </button>
            </div>
          {/if}

          <div class="text-center w-full md:w-auto">
            <p class="mb-4 text-sm md:text-lg uppercase tracking-widest">The House Picked</p>
            {#if houseChoice}
              <div
                class="w-28 h-28 md:w-36 md:h-36 rounded-full bg-white flex items-center justify-center border-[12px] md:border-[16px] mx-auto"
                class:border-red-500={houseChoice === 'rock'}
                class:border-blue-500={houseChoice === 'paper'}
                class:border-yellow-500={houseChoice === 'scissors'}
                class:border-purple-500={houseChoice === 'lizard'}
                class:border-cyan-500={houseChoice === 'spock'}
              >
                <img
                  src={houseChoice ? imageMap[houseChoice] : ''}
                  alt={houseChoice || ''}
                  class="w-14 h-14 md:w-16 md:h-16"
                />
              </div>
            {:else}
              <div class="w-28 h-28 md:w-36 md:h-36 rounded-full bg-black/20 mx-auto"></div>
            {/if}
          </div>
        </div>
      {/if}
    </main>

    <button
      class="mt-12 md:mt-auto border-2 border-white rounded-lg px-8 py-2 hover:bg-white/10 transition-colors text-sm md:text-base uppercase tracking-wider"
      onclick={() => showRules = true}
    >
      Rules
    </button>

    {#if showRules}
      <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50" in:fade>
        <div class="bg-white text-gray-900 p-6 md:p-8 w-full h-full md:h-max md:w-[70vb] md:max-w-md flex flex-col items-center relative md:rounded-2xl">
          <div class="w-full flex justify-between items-center mb-8 md:mb-10">
            <h2 class="text-2xl md:text-3xl font-bold uppercase">Rules</h2>
            {#if !isMobile}
              <button
                class="p-2 hover:opacity-75 transition-opacity"
                onclick={() => showRules = false}
                aria-label="Close rules"
              >
                <img src={iconclose} alt="Close" class="w-5 h-5 md:w-6 md:h-6" />
              </button>
            {/if}
          </div>
          
          <img src={rule} alt="Game rules diagram" class="w-full h-full object-contain" />
          
          {#if isMobile}
            <button
              class="mt-auto p-2 hover:opacity-75 transition-opacity"
              onclick={() => showRules = false}
              aria-label="Close rules"
            >
              <img src={iconclose} alt="Close" class="w-6 h-6" />
            </button>
          {/if}
        </div>
      </div>
    {/if}
  </div>
  
  <style>
    :global(body) {
      margin: 0;
      font-family: 'Barlow Semi Condensed', sans-serif;
    }

    @media (max-width: 350px) {
      .hFlex {
        display: none;
      }
    }

    .winner {
      position: relative;
    }

    .winner::before,
    .winner::after {
      content: '';
      position: absolute;
      inset: -20px;
      border-radius: 50%;
      background: radial-gradient(circle, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 70%);
      animation: ripple 2s ease-out infinite;
      pointer-events: none;
    }

    .winner::after {
      animation-delay: 1s;
    }

    @keyframes ripple {
      0% {
        transform: scale(1);
        opacity: 1;
      }
      100% {
        transform: scale(3);
        opacity: 0;
      }
    }
  </style>