<script lang="ts">
  import { openPath,openUrl } from '@tauri-apps/plugin-opener';
  import { readTextFile,exists,BaseDirectory,readFile} from '@tauri-apps/plugin-fs';
  import * as path from '@tauri-apps/api/path';
  import { message } from '@tauri-apps/plugin-dialog';

  import { onMount } from 'svelte';
  import { get } from 'svelte/store';

  let command:string = $state('');
  let response = $state('');
  let isLoading = $state(false);
  let folder = sessionStorage.getItem("folder") ?? "";
  let selecting = $state("1");
  let selectinggame = $derived(() => getGame(selecting));

  
  interface Game {
    id: string;
    name: string;
    gamefile: string;
    caution?: string;
  }

  let games = $state<Game[]>([]);

  function getGame(id:string){
    const game = games.find((game) => game.id == id);
    if (game) {
      return game;
    } else {
      return {
        id: "",
        name: "",
        gamefile: "",
      };
    }
  }

  async function runCommand() {
    await openUrl(command);
  }

  onMount(async () => {

    const folderexists = await exists(await path.join(folder,"config.json"));
    if (!folderexists) {
      await message("config.jsonが見つかりません．\n 最初の画面に戻ります．", {
        title: "エラー",
        kind: "error",
      });
      window.location.href = "/";
    }else{
      const config = await readTextFile(await path.join(folder,"config.json"));
      games = JSON.parse(config);
      games.forEach((game) => {
        if(game.id == undefined || game.name == undefined || game.gamefile == undefined){
          message("config.jsonのフォーマットが正しくありません．\n 最初の画面に戻ります．", {
            title: "エラー",
            kind: "error",
          });
          window.location.href = "/";
        }
      });
      selecting = games[0].id;
    }
  })

  async function getDescription(game: Game) {
    const gamefile = await path.join(folder, game.name, "description.txt");
    const fileexists = await exists(gamefile);
    const description = fileexists ? await readTextFile(gamefile) : "説明がありません";
    console.log(description);
    return description != "" ? description : "説明がありません";
  }

  async function getthumbnail(game: Game) {
    const thumbnailpng = await path.join(folder, game.name, "thumbnail.png");
    const fileexistspng = await exists(thumbnailpng);
    const thumbnailjpg = await path.join(folder, game.name, "thumbnail.jpg");
    const fileexistsjpg = await exists(thumbnailjpg);
    if (fileexistspng || fileexistsjpg) {
      const thumbnail = fileexistspng ? thumbnailpng : thumbnailjpg;
      const content = await readFile(thumbnail);
      const len = content.byteLength;
      const base64 = btoa(
        new Uint8Array(content).reduce((data, byte) => data + String.fromCharCode(byte), "")
      );
      return `data:image/png;base64,${base64}`;
    } else {
      return "";
    }
  }

  async function openGame(game: Game) {
    const gamefile = await path.join(folder, game.name, "game", game.gamefile);
    const fileexists = await exists(gamefile);
    if (fileexists) {
      await openPath(gamefile);
    } else {
      message("ゲームファイルが見つかりません．", {
        title: "エラー",
        kind: "error",
      });
    }
  }


  function echo() {
    response = command;
  }
</script>


<main class="font-line w-screen h-screen bg-[#010015] text-white flex flex-col items-center text-center transition-all duration-200">
  <div class="flex p-8 justify-between w-screen border-b-2 text-center items-center">
    <p class="text-3xl select-none cursor-default">物理部展 2025 ゲーム展示</p>
    <div class="p-4 hover:bg-blue-600/10 rounded-xl transition duration-200 flex items-center gap-4">
      <button onclick={() => {window.location.href = "/"}} class="text-white/0 hover:text-blue-600 transition duration-300 cursor-pointer">Go Home</button>
      <button onclick={() => {window.location.reload()}} class="text-white/0 hover:text-blue-600 transition duration-300 cursor-pointer">Reload</button>
    </div>
  </div>
  <div class="flex w-screen flex-1 justify-center">
    <div class="w-2/5 flex flex-col items-center justify-center p-16 ">
      {#each games as game}
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <div class={`border-2 w-full text-left p-4 m-2 rounded-l-3xl flex flex-col transition-all duration-200  cursor-pointer ${selecting == game.id ? "translate-x-12 scale-110" : "hover:translate-x-4 hover:scale-105"}`} onclick={() => {
          selecting = game.id;}}>
          <p class="text-3xl mb-2">{game.name}</p>
          <p class="text-base overflow-hidden text-ellipsis whitespace-nowrap">
            {#await getDescription(game)}
              読み込み中...
            {:then description}
              {description}
            {:catch error}
              説明を取得できませんでした: {error}
            {/await}
  </p>
        </div>
      {/each}
    </div>
    <div class="w-3/5 bg-[#010015] flex flex-col justify-center items-center p-16">
      <div class="h-2/3 w-full flex flex-col gap-6 p-16">
        {#await getthumbnail(selectinggame())}
          <div class="h-64 border-2 w-128 mx-auto flex text-center">
            <p class="text-3xl">サムネイルを取得中...</p>
          </div>
        {:then thumbnail}
          {#if thumbnail != ""}
            <img src={thumbnail} alt="" class="h-64 mx-auto rounded-xl object-cover" />
          {:else}
          <div class="h-64 bg-blue-700 w-128 mx-auto flex text-center">
            <p class="text-3xl">サムネイルが存在しません</p>
          </div>
          {/if}
        {:catch error}
        <div class="h-64 bg-blue-700 w-128 mx-auto flex text-center">
          <p class="text-3xl">サムネイルを取得できませんでした</p>
        </div>
        {/await}
        <p class="text-left text-6xl">{selectinggame().name}</p>
        <p class="text-left text-3xl h-68 overflow-x-scroll whitespace-pre-line hidden-scrollbar">
          {#await getDescription(selectinggame())}
            読み込み中...
          {:then description}
            {description}
          {:catch error}
            説明を取得できませんでした: {error}
          {/await}
        </p>  
      </div>
      <div class="flex flex-col gap-8 items-center">
        <div class={`border-2 border-amber-300 p-4 rounded-md ${selectinggame().caution ? "opacity-100" : "opacity-0"}`}>このゲームには激しい点滅が含まれます．</div>
        <button class="text-3xl p-4 border-2 rounded-md px-8 transition hover:scale-120 duration-200 w-fit" onclick={() => {
          openGame(selectinggame());
        }}>起動</button>
      </div>
    </div>
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