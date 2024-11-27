<script lang="ts">
  export let showRules: boolean;
  export let onClose: () => void;
</script>

{#if showRules}
  <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
    <div class="bg-white text-gray-900 p-8 rounded-lg w-full max-w-md md:max-w-lg relative flex flex-col items-center md:items-start">
      <div class="w-full flex justify-between items-center mb-8">
        <h2 class="text-3xl font-bold uppercase">Rules</h2>
        <button 
          class="hidden md:block text-gray-400 hover:text-gray-600 transition-colors"
          on:click={onClose}
          aria-label="Close rules"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
      
      <div class="relative w-64 h-64 mx-auto my-8">
        <img src="/images/rules-diagram.svg" alt="Game rules diagram showing Rock beats Scissors, Scissors beats Paper, and Paper beats Rock" class="w-full h-full" />
      </div>

      <button 
        class="md:hidden mt-8 text-gray-400 hover:text-gray-600 transition-colors"
        on:click={onClose}
        aria-label="Close rules"
      >
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <line x1="18" y1="6" x2="6" y2="18"></line>
          <line x1="6" y1="6" x2="18" y2="18"></line>
        </svg>
      </button>
    </div>
  </div>
{/if}