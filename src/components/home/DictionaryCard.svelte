<script lang="ts">
    import Icon from '@iconify/svelte';
    import {onMount} from "svelte";
    import "../../css/home/DictionaryCard.css";
    import axios from 'axios';
    import {API_URL} from "../ts/API";
    import LoadingIcon from '../LoadingIcon.svelte';

    let wordInputRef: any;
    let wordInput: string = "";
    let toggleClearButton: boolean = false;
    let searchResult = {};
    let toggleLoadingIcon: boolean = false;

    function handleWordKeyupChange() {
        if (wordInput.length > 0) {
            toggleClearButton = true;
        } else {
            toggleClearButton = false;
            clearWordInput();
        }
    }

    function clearWordInput() {
        wordInput = "";
        toggleClearButton = false;
        searchResult = {};
        initialMount();
    }

    async function handleSearchWord() {
        if (wordInput.trim() === "") return;

        toggleLoadingIcon = true;
        searchResult = {};

        // Get search result
        await axios.get(`${API_URL}/${wordInput}`, {
            headers: {
                'content-type': 'application/json',
            }
        }).then(res => {
            const result = res.data[0];

            if (result) {
                const meanings = result.meanings;

                const definition = meanings[0]?.definitions[0]?.definition;
                const partOfSpeech = meanings[0]?.partOfSpeech;
                const sourceUrls = result?.sourceUrls;
                let synonyms: string[] = [];
                let antonyms: string[] = [];

                meanings.map((meaning: any) => {
                    if (meaning.synonyms.length > 0) {
                        meaning.synonyms.map((synonym: any) => {
                            synonyms.push(synonym);
                        })
                    }
                    if (meaning.antonyms.length > 0) {
                        meaning.antonyms.map((antonym: any) => {
                            antonyms.push(antonym);
                        })
                    }
                })

                let examples: string[] = [];
                meanings.map((meaning: any) => {
                    if (meaning.definitions.length > 0) {
                        meaning.definitions.map((definition: any) => {
                            if (definition.example) {
                                examples.push(definition.example);
                            }
                        })
                    }
                })

                searchResult = {
                    definition,
                    partOfSpeech,
                    synonyms,
                    antonyms,
                    examples,
                    sourceUrls,
                    resultFound: true
                }
            } else {
                searchResult = {
                    resultFound: false
                };
            }
        }).catch(err => {
            searchResult = {
                resultFound: false
            };
        })

        toggleLoadingIcon = false;
    }

    function textToSpeech() {
        if (wordInput.trim() === "") return;

        // Create an instance of SpeechSynthesisUtterance
        const msg = new SpeechSynthesisUtterance(wordInput);

        // Speak the text
        window.speechSynthesis.speak(msg);
    }

    function changeWordInput(newWord: string) {
        wordInput = newWord;

        handleSearchWord();
    }

    onMount(() => {
        initialMount();
    })

    function initialMount() {
        wordInputRef?.focus();
    }
</script>

<div class="w-full max-w-screen-2xl mx-auto px-5 md:px-10 lg:px-20 my-10">
    <div class="w-auto max-w-[800px] mx-auto bg-stone-800 rounded-md p-5 shadow-md">
        <h1 class="text-center font-semibold text-lg sm:text-xl pb-3 border-b border-stone-600 mb-3">Word of Dictionary</h1>

        <div class="flex flex-col gap-y-5 mb-5">
            <form on:submit|preventDefault={handleSearchWord} class="flex items-center gap-x-3 gap-y-2">
                <div class="flex flex-1 items-center py-2 border-b green__border">
                    <input id="word__input" bind:this={wordInputRef} bind:value={wordInput} on:keyup={handleWordKeyupChange} type="text" class="w-full pr-3 bg-transparent focus:outline-none" placeholder="Search a word..." />
                    {#if toggleClearButton}
                        <button on:click={clearWordInput} type="button" class="hover:bg-stone-700 rounded-full p-0.5">
                            <Icon class="text-xl" icon="ic:round-close" />
                        </button>
                    {/if}
                </div>
                <div class="">
                    <button type="submit" class="text-black font-medium px-3 py-2 rounded-md green__background">Search</button>
                </div>
            </form>
        </div>

        {#if Object.keys(searchResult).length > 0}
            {#if searchResult?.resultFound}
                <div class="flex flex-col gap-y-5">
                    <div class="flex flex-col gap-y-3">
                        <div class="flex items-center justify-between gap-x-3">
                            <h3 class="text-lg sm:text-xl font-semibold">{wordInput?.trim()}</h3>

                            <div>
                                <button on:click={textToSpeech} type="button" class="text-sky-400 hover:text-sky-500 transition-colors duration-200 ease-linear">
                                    <Icon class="text-xl" icon="streamline:entertainment-volume-level-high-speaker-high-volume-control-audio-music" />
                                </button>
                            </div>
                        </div>

                        <h5 class="text-slate-300 font-medium">{searchResult?.partOfSpeech}</h5>
                    </div>

                    <div class="border-l-4 green__border pl-3">
                        <h5>{searchResult?.definition}</h5>
                    </div>

                    <div class="flex flex-col gap-y-3">
                        {#if searchResult?.synonyms && searchResult.synonyms.length > 0}
                            <div class="flex flex-wrap gap-1">
                                <h5 class="font-medium text-slate-300">Synonyms: </h5>
                                <div class="flex flex-wrap gap-y-1 gap-x-2 word__diffMean">
                                    {#each searchResult.synonyms as synonym}
                                        <button type="button" on:click={() => changeWordInput(synonym)}>{synonym}</button>
                                    {/each}
                                </div>
                            </div>
                        {/if}
                        {#if searchResult?.antonyms && searchResult.antonyms.length > 0}
                            <div class="flex flex-wrap gap-1">
                                <h5 class="font-medium text-slate-300">Antonyms: </h5>
                                <div class="flex flex-wrap gap-y-1 gap-x-2 word__diffMean">
                                    {#each searchResult.antonyms as antonym}
                                        <button type="button" on:click={() => changeWordInput(antonym)}>{antonym}</button>
                                    {/each}
                                </div>
                            </div>
                        {/if}
                    </div>

                    {#if searchResult?.examples && searchResult.examples.length > 0}
                        <div class="flex flex-col gap-y-2">
                            <h5 class="font-medium text-slate-300">Examples: </h5>
                            <div class="flex flex-col gap-y-2 pl-3">
                                {#each searchResult.examples as example, index}
                                    <p>{index + 1}) {example}</p>
                                {/each}
                            </div>
                        </div>
                    {/if}
                    
                    {#if searchResult?.sourceUrls && searchResult.sourceUrls.length > 0}
                        <div class="flex flex-col gap-y-2">
                            <h5 class="font-medium text-slate-300">Source URLs: </h5>
                            <div class="flex flex-col gap-y-2 pl-3">
                                {#each searchResult.sourceUrls as sourceURL, index}
                                    <a href="{sourceURL}" target="_blank" class="hover:text-sky-400 transition-colors duration-200 ease-linear">{index + 1}) {sourceURL}</a>
                                {/each}
                            </div>
                        </div>
                    {/if}
                </div>
            {:else}
                <div>
                    <h5 class="text-center font-semibold text-rose-400">Nothing found!</h5>
                </div>
            {/if}
        {/if}

        {#if toggleLoadingIcon}
            <LoadingIcon />
        {/if}
    </div>
</div>