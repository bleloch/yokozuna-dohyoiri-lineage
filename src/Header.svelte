<script lang="ts">
    import githubLogo from "../public/icons/github.svg?raw";
</script>

<header>
    <div class="p-1 bg-black flex items-center justify-between drop-shadow-2xl">
        <div class="pl-1.5 text-3xl text-blue-200">Yokozuna Dohyō-iri Lineage</div>
        <div class="logo pt-1">
            <button>
                <a href="https://github.com/bleloch/yokozuna-dohyoiri-lineage/" target="_blank ">{@html githubLogo}</a>
            </button>
        </div>
    </div>
</header>

<style>
    .logo:hover,
    .logo:focus {
        filter: brightness(120%);
    }
</style>