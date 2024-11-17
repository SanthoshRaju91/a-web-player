<script lang="ts">
  import { Button } from "$lib/components/ui/button/index.js";
  import Plyr from "plyr";
  import "plyr/dist/plyr.css";
  import {
    Plus,
    Trash,
    ListPlus,
    Play,
    Pause,
    SkipForward,
    ListRestart,
    Maximize2,
    Minimize2,
    ListMusic,
    FileX,
  } from "lucide-svelte";
  import { onMount } from "svelte";

  const ALLOWED_FILE_TYPES = [
    "audio/mpeg",
    "video/mp4",
    "video/mov",
    "video/quicktime",
  ];

  let filesInMediaArea: File[] = [];
  let filesInQueue: File[] = [];
  let plyr: Plyr | null = null;
  let videoElement: HTMLVideoElement | null = null;
  let currentFileIndex: number | null = null;
  let isPlaying: boolean = false;
  let playedFiles: Set<number> = new Set();

  function formatFileName(name: string, maxLength: number = 10): string {
    const extensionIndex = name.lastIndexOf(".");
    const extension = extensionIndex !== -1 ? name.slice(extensionIndex) : "";
    const baseName =
      extensionIndex !== -1 ? name.slice(0, extensionIndex) : name;

    if (baseName.length > maxLength) {
      return `${baseName.slice(0, maxLength)}...${extension}`;
    }
    return name;
  }

  function handleOnRemoveFileInQueue(fileIndex: number) {
    filesInQueue = filesInQueue.filter((_, i) => i !== fileIndex);
  }

  function handleClearAllMediaContent() {
    filesInMediaArea = [];
  }

  function handleFileInput(event: Event) {
    const target = event.target as HTMLInputElement;
    const files = target.files;

    if (files) {
      let newFiles = Array.from(files).filter((file) => {
        return ALLOWED_FILE_TYPES.includes(file.type);
      });
      filesInMediaArea = [...filesInMediaArea, ...newFiles];
    }
  }

  function handleAddMediaFiles() {
    const fileInput = document.getElementById("fileInput") as HTMLInputElement;
    fileInput.click();
  }

  function handleAddToQueue(file: File) {
    filesInQueue = [...filesInQueue, file];
  }

  function handleOnRemoveFileInMediaArea(fileIndex: number) {
    filesInMediaArea = filesInMediaArea.filter((_, i) => i !== fileIndex);
  }

  function playCurrentFile(index: number) {
    if (filesInQueue.length > index) {
      currentFileIndex = index;
      const file = filesInQueue[currentFileIndex];
      if (plyr) {
        console.log(file)
        const fileURL = URL.createObjectURL(file);
        plyr.source = {
          type: "video",
          title: file.name,
          sources: [
            {
              src: fileURL,
              type: file.type,
            },
          ],
        };
        plyr.play().catch((err: any) => {
            console.error("Error starting playback: ", err)
        })
      }
    }
  }

  function onPlay() {
    if (currentFileIndex === null && filesInQueue.length > 0) {
      console.log("Play file now")
      playCurrentFile(0);
    }
  }

  onMount(() => {
    videoElement = document.getElementById("mediaPlayer") as HTMLVideoElement;

    const controls = `
      <div class="plyr__controls">
          <button type="button" class="plyr__control" aria-label="Play, {title}" data-plyr="play">
              <svg class="icon--pressed" role="presentation"><use xlink:href="#plyr-pause"></use></svg>
              <svg class="icon--not-pressed" role="presentation"><use xlink:href="#plyr-play"></use></svg>
              <span class="label--pressed plyr__tooltip" role="tooltip">Pause</span>
              <span class="label--not-pressed plyr__tooltip" role="tooltip">Play</span>
          </button>

          <button type="button" class="plyr__control" id="next__button">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-skip-forward"><polygon points="5 4 15 12 5 20 5 4"/><line x1="19" x2="19" y1="5" y2="19"/></svg>
            <span class="plyr__tooltip" role="tooltip">Play Next</span> 
          <button>

          <button type="button" class="plyr__control" data-plyr="restart">
              <svg role="presentation"><use xlink:href="#plyr-restart"></use></svg>
              <span class="plyr__tooltip" role="tooltip">Restart</span>
          </button>
          
          <div class="plyr__progress">
              <input data-plyr="seek" type="range" min="0" max="100" step="0.01" value="0" aria-label="Seek">
              <progress class="plyr__progress__buffer" min="0" max="100" value="0">% buffered</progress>
              <span role="tooltip" class="plyr__tooltip">00:00</span>
          </div>
          <div class="plyr__time plyr__time--current" aria-label="Current time">00:00</div>
          <div class="plyr__time plyr__time--duration" aria-label="Duration">00:00</div>            
          <button type="button" class="plyr__control" data-plyr="fullscreen">
              <svg class="icon--pressed" role="presentation"><use xlink:href="#plyr-exit-fullscreen"></use></svg>
              <svg class="icon--not-pressed" role="presentation"><use xlink:href="#plyr-enter-fullscreen"></use></svg>
              <span class="label--pressed plyr__tooltip" role="tooltip">Exit fullscreen</span>
              <span class="label--not-pressed plyr__tooltip" role="tooltip">Enter fullscreen</span>
          </button>
      </div>
    `;

    plyr = new Plyr(videoElement, {
      controls,
      clickToPlay: false,
    });

    const nextButtonElement = document.getElementById("next__button") as HTMLButtonElement;

    if (nextButtonElement) {
      nextButtonElement.addEventListener("click", playNextFile);
    }

    plyr.on("play", onPlay);

    return () => {
      plyr?.destroy();
    };
  });

  function playNextFile() {
    console.log("Play next clicked")
    if (
      currentFileIndex !== null &&
      currentFileIndex < filesInQueue.length - 1
    ) {
      playCurrentFile(currentFileIndex + 1);
    } else {
      console.warn("No more files in the queue.");
    }
  }
</script>

<main class="flex h-screen">
    <div class="w-4/5 h-screen flex flex-col">
      <video id="mediaPlayer" class="w-full h-full">
        <track kind="captions" />
      </video>
    </div>
    <div class="w-1/4 border-l border-primary">
      <div class="flex-1">
        <div class="flex flex-col gap-4">
          <div class="flex flex-row items-center gap-2 bg-primary p-4">
            <Play class="h-4 w-4 text-accent" />
            <h3 class="text-base font-bold text-accent">Queue</h3>
          </div>
  
          <ul class="flex flex-col px-4 min-h-6">
            {#each filesInQueue as file, index}
              <li
                class="flex flex-row justify-between text-primary py-4 border-b border-b-primary last:border-none"
              >
                <div class="flex flex-row gap-2 items-center">
                  <span
                    class="flex items-center justify-center bg-primary border border-primary text-accent rounded-full text-sm w-6 h-6"
                  >
                    {index + 1}
                  </span>
                  <p class="text-accent text-sm">
                    {formatFileName(file.name, 20)}
                  </p>
                </div>
                <Button
                  class="text-sm h-6 w-fit"
                  on:click={() => handleOnRemoveFileInQueue(index)}
                >
                  <Trash class="h-3 w-3" />
                </Button>
              </li>
            {/each}
  
            {#if filesInQueue.length <= 0}
              <div class="flex flex-col items-center p-6 gap-4">
                <FileX class="h-6 w-6 text-accent" />
                <p class="text-accent text-center text-sm">
                  No files in queue to play. Add files from "Media Area"
                </p>
              </div>
            {/if}
          </ul>
        </div>
      </div>
      <div class="flex-1 flex flex-col gap- mt-4">
        <div class="flex flex-col gap-8">
          <div class="flex flex-row gap-2 items-center bg-primary p-4">
            <ListMusic class="h-4 w-4 text-accent" />
            <h3 class="text-base font-bold text-accent">Media Area</h3>
          </div>
          <div class="flex flex-row gap-4 px-4">
            <Button class="text-sm h-8 w-fit" on:click={handleAddMediaFiles}>
              <Plus class="mr-2 h-4 w-4" />Add files
            </Button>
            <Button
              class="text-sm h-8 w-fit bg-secondary"
              on:click={handleClearAllMediaContent}
            >
              <Trash class="mr-2 h-4 w-4" />
              Clear all
            </Button>
  
            <input
              id="fileInput"
              type="file"
              multiple
              class="hidden"
              accept=".mp3, .mp4, .mov"
              on:change={handleFileInput}
            />
          </div>
        </div>
        <div class="flex flex-col my-6 px-4">
          <ul class="flex flex-col">
            {#each filesInMediaArea as file, index}
              <li
                class="flex flex-row justify-between py-4 items-center border-b border-b-primary last:border-none"
              >
                <p class="text-sm text-accent">{formatFileName(file.name, 20)}</p>
                <div class="flex flex-row gap-4">
                  <Button
                    class="text-sm h-8 w-fit"
                    on:click={() => handleAddToQueue(file)}
                  >
                    <ListPlus class="h-4 w-4" />
                  </Button>
                  <Button
                    class="text-sm h-8 w-fit"
                    on:click={() => handleOnRemoveFileInMediaArea(index)}
                  >
                    <Trash class="h-4 w-4" />
                  </Button>
                </div>
              </li>
            {/each}
          </ul>
        </div>
      </div>
    </div>
  </main>
