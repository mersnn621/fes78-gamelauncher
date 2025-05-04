<script>
    import logo from "../lib/assets/general.svg";
    import { open } from '@tauri-apps/plugin-dialog';
    let folder = $state('');
    const openmes = async () => {
        folder = await open({
            directory: true,
            multiple: false,
            title: "フォルダを選択",
        }) ?? "";
    }
    const launch = () => {
        if (folder == "") {
            return;
        }
        window.location.href = "/launcher";
    }
</script>

<main class="font-line w-screen h-screen bg-[#010015] text-white flex flex-col items-center justify-center text-center transition-all duration-200">
    <img src={logo} alt="" class="">
    <p class="text-4xl font-[300]">物理部展2025 ゲームランチャー </p>
    <button class="px-5 py-3 border-2 rounded-md hover:scale-110 transition mt-6" onclick={openmes}>フォルダを選択</button>
    <div class="flex h-32 justify-center flex-col items-center gap-6">
        {#if folder != ""}
        <button class={`px-5 py-3 border-2 rounded-md hover:scale-110 transition  bg-blue-200/20 ${folder=="" ? "border-red-400" :"border-blue-400"}`} onclick={launch}>{folder=="" ? "フォルダーを選択してください" : "ランチャーを起動"}</button>
    <div class="flex justify-center items-center gap-3 h-6">
        <p>{folder}</p>
        <button class="px-3 py-1 border-red-400 border-2 text-sm rounded-sm transition hover:bg-red-400/20" onclick={
() => {folder = ""}
        }>解除</button>
    </div>
    {/if}
    </div>
</main>

<style lang="postcss">
    :global(body){
        @apply select-none cursor-default;
        button{
            @apply cursor-pointer;
        }
    }
</style>