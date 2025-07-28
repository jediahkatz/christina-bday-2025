<script>
  import QuestionSystem from "./lib/QuestionSystem.svelte";
  import { store, loadData } from "./assets/store.js";
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";
  import { isTouchEnabled, getLanguage } from "./assets/utils.js";
  import { transition_in } from "svelte/internal";

  $store.numQuestions = 10;
  let musicBg = null;
  let loaded = loadData();
  let isPlaying = false;
  let canStart = false;
  let finishedTransition = false;
  let isSpanish = getLanguage() === "es";

  onMount(async () => {
    musicBg.volume = 0.1;
    setTimeout(() => {
      window.scrollTo({ top: document.body.scrollHeight, behavior: "smooth" });
    }, 100);
    

    // Add global interpret function for debugging
    const { decodeResponses, getResponseSummary } = await import("./assets/store.js");
    const { getLanguage } = await import("./assets/utils.js");
    
    window.interpretCode = async (code) => {
      try {
        console.log("Interpreting code:", code);
        const responses = decodeResponses(code);
        console.log("Decoded responses:", responses);
        
        // Load questions
        const lang = getLanguage();
        const baseURL = import.meta.env.BASE_URL;
        const questionsResponse = await fetch(`${baseURL}lang/${lang}/christina-questions.json`);
        const questions = await questionsResponse.json();
        console.log("Questions loaded:", questions.length);
        
        const summary = getResponseSummary(responses, questions);
        console.log("Response summary:", summary);
        
        return {
          code,
          responses,
          summary,
          questions
        };
      } catch (error) {
        console.error("Error interpreting code:", error);
        return { error: error.message };
      }
    };
    
    console.log("Global interpretCode function added to window");
    
    // Add test functions
    window.generateTestCode = async () => {
      const { encodeResponses } = await import("./assets/store.js");
      const testResponses = [0, 1, 0, 1, 2, 0, 1, 0, 2, 1]; // Sample responses
      const code = encodeResponses(testResponses);
      console.log("Test code generated:", code, "for responses:", testResponses);
      return code;
    };
    
    window.testInterpret = async () => {
      const testCode = await window.generateTestCode();
      return await window.interpretCode(testCode);
    };
  });

  let pokemons = [
    "bulbasaur",
    "charmander",
    "chikorita",
    "chimchar",
    "cyndaquil",
    "eevee",
    "mudkip",
    "phanpy",
    "pikachu",
    "piplup",
    "riolu",
    "shinx",
    "skitty",
    "squirtle",
    "torchic",
    "totodile",
    "treecko",
    "turtwig",
    "vulpix",
  ];
</script>

<audio
  bind:this="{musicBg}"
  src="audio/quiz-music.ogg"
  type="audio/ogg"
  controls="{false}"
  loop
  preload="auto"></audio>

<section class="hidden">
  {#each pokemons as pokemon}
    <img src="img/pokemonicons/{pokemon}.png" alt="" />
  {/each}
  <img src="img/others/pmdtextbox" alt="" />
</section>

<section>
  <div
    class="fixed h-screen w-screen justify-center bg-cover brightness-75 infinite-scroll-right">
  </div>
  <div
    class="fixed h-screen w-screen justify-center bg-cover brightness-75 opacity-50 infinite-scroll-left">
  </div>
</section>

{#await loaded}
    <div
      class="flex flex-col justify-center items-center text-center h-screen w-screen bg-black/50 z-10">
      <h1 class="text-8xl text-white animate-pulse">
        {isSpanish ? "Cargando..." : "Loading..."}
      </h1>
    </div>
  {:then data}
    <main class="relative" transition:fade>
      <section>
        {#if canStart}
          {#if finishedTransition}
            <div
              class="flex flex-col flex-wrap h-screen justify-end bg-none"
              transition:fade>
              <QuestionSystem dataExt="{data}" />
            </div>
          {/if}
        {:else}
          <div
            class="flex flex-col flex-wrap h-screen justify-center items-center bg-none"
            transition:fade
            on:outroend="{() => {
              if (!isPlaying) {
                musicBg.play();
                isPlaying = true;
              }
              finishedTransition = true;
            }}">
            <div
              class="flex flex-col select-none justify-center items-center text-center w-screen bg-black/30 hover:bg-black/40 transition-all duration-300 z-10 p-8">
              <h1 class="text-6xl lg:text-8xl text-white mb-4 font-bold">
                Christina Lu
              </h1>
              <h2 class="text-3xl lg:text-5xl text-white/75 mb-8">
                August 12, 2000
              </h2>
              <button
                class="bg-white/20 hover:bg-white/30 text-white text-2xl lg:text-4xl px-8 py-4 rounded-lg border-2 border-white/50 hover:border-white transition-all duration-300"
                on:click="{() => {
                  canStart = true;
                  $store.numQuestions = 10;
                }}">
                Take The Quiz
              </button>
            </div>
          </div>
        {/if}
      </section>
    </main>
  {:catch error}
    <div
      class="flex flex-col justify-center items-center text-center h-screen w-screen bg-black/50 z-10"
      transition:fade>
      <h1 class="text-8xl text-white animate-pulse">
        {isSpanish ? "Error al cargar :(" : "Error promise rejected :("}
        {error.message}
      </h1>
    </div>
{/await}
