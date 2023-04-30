<script lang="ts">
    import {onMount} from "svelte";
    import Icon from '@iconify/svelte';
    import logoImg from "../../assets/images/logo.png";
    import tippy from 'tippy.js';
    import 'tippy.js/dist/tippy.css'; // optional for styling

    let searchBtnRef: any,
    navbarRef: any;

    function focusSearchInput() {
        const wordInput: HTMLInputElement|null = document.querySelector("#word__input");
        wordInput?.focus();
    }

    onMount(() => {
        initalMount();
    })

    function initalMount() {
        tippy(searchBtnRef, {
            content: 'Search Word',
            placement: 'bottom',
        });
    }
</script>

<div class="w-full bg-stone-800 py-3 px-5 md:px-10 lg:px-20 z-[1001]">
    <div class="w-full max-w-screen-2xl mx-auto">
        <div class="flex items-center justify-between">
            <a href="/">
                <!-- svelte-ignore a11y-img-redundant-alt -->
                <img src="{logoImg}" class="w-[7rem] sm:w-[8rem] h-auto object-contain" alt="Logo Image" />
            </a>

            <div>
                <button bind:this={searchBtnRef} on:click={focusSearchInput} type="button" class="flex items-center justify-center w-[2.5rem] h-[2.5rem] bg-stone-700 rounded-full text-slate-300 hover:bg-stone-900 hover:text-white transition-colors duration-200 ease-linear">
                    <Icon class="text-xl" icon="material-symbols:search-rounded" />
                </button>
            </div>
        </div>
    </div>
</div>