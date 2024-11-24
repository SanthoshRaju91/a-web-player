<script lang="ts">
  import { Button } from "$lib/components/ui/button/index.js";
  import {
    Plus,
    Trash,
    ListPlus,
    Play,
    ListMusic,
    FileX,
    Pause,
    SkipForward,
    Maximize,
    Minimize,
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
  let videoElement: HTMLVideoElement | null = null;
  let currentFileIndex: number = 0;
  let isPlaying: boolean = false;
  let playerContainer: any;
  let currentTime = 0;
  let duration = 0;
  let isFullscreen = false;

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

  function toggleFullscreen() {
    if (document.fullscreenElement) {
      document.exitFullscreen();
      isFullscreen = false;
    } else {
      if (playerContainer) {
        if (playerContainer.requestFullscreen) {
          playerContainer.requestFullscreen();
        } else if (playerContainer.webkitRequestFullscreen) {
          playerContainer.webkitRequestFullscreen();
        } else if (playerContainer.msRequestFullscreen) {
          playerContainer.msRequestFullscreen();
        }
        isFullscreen = true;
      }
    }
  }

  function togglePlayPause() {
    if (isPlaying) {
      if (videoElement) {
        videoElement.pause();
      }
    } else {
      if (videoElement) {
        videoElement.play();
      }
    }
    isPlaying = !isPlaying;
  }

  function playNext() {
    console.log("Play next file clicked");
    if (
      currentFileIndex !== null &&
      currentFileIndex < filesInQueue.length - 1
    ) {
      currentFileIndex += 1;
      playCurrentFile(currentFileIndex);
    } else {
      console.log("No more files in the queue.");
    }
  }

  function playCurrentFile(index: number = 0) {
    if (filesInQueue.length > index) {
      currentFileIndex = index;
      console.log(filesInQueue);
      const file = filesInQueue[currentFileIndex];
      console.log(file);

      if (videoElement) {
        const fileURL = URL.createObjectURL(file);
        videoElement.src = fileURL;
        videoElement.play();
        isPlaying = true;
      }
    }
  }

  function onPlay() {
    if (filesInQueue.length > 0 && filesInQueue.length > currentFileIndex) {
      const file = filesInQueue[currentFileIndex];
      if (videoElement) {
        const fileURL = URL.createObjectURL(file);
        videoElement.src = fileURL;
        videoElement.play();
        isPlaying = true;
      }
    } else {
      console.warn("No files to play");
    }
  }

  function handleKeydown(event: any) {
    switch (event.key) {
      case " ":
        togglePlayPause();
        break;
      case "N":
      case "n":
        playNext();
        break;
      case "F":
      case "f":
        toggleFullscreen();
        break;
    }
  }

  function handleTimeUpdate() {
    if (videoElement) {
      currentTime = videoElement.currentTime;
    }
  }

  function handleLoadedMetadata() {
    if (videoElement) {
      duration = videoElement.duration;
    }
  }

  function handleSeek(event: Event) {
    const target = event.target as HTMLInputElement;
    if (videoElement) {
      videoElement.currentTime = parseFloat(target.value);
      currentTime = videoElement.currentTime;
    }
  }

  const handleFullscreenChange = () => {
      isFullscreen = Boolean(document.fullscreenElement);
    };

  onMount(() => {
    document.addEventListener("keydown", handleKeydown);
    document.addEventListener("fullscreenchange", handleFullscreenChange);

    if (videoElement) {
      videoElement.addEventListener("timeupdate", handleTimeUpdate);
      videoElement.addEventListener("loadedmetadata", handleLoadedMetadata);
    }
    

    return () => {
      document.removeEventListener("keydown", handleKeydown);
      document.removeEventListener("fullscreenchange", handleFullscreenChange);

      if (videoElement) {
        videoElement.removeEventListener("timeupdate", handleTimeUpdate);
        videoElement.removeEventListener(
          "loadedmetadata",
          handleLoadedMetadata,
        );
      }
    };
  });


  function formatTime(time: number): string {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time % 60);
    return `${minutes}:${seconds < 10 ? "0" + seconds : seconds}`;
  }
</script>

<main class="flex h-screen">
  <div
    class="relative w-full h-full mx-auto bg-black"
    bind:this={playerContainer}
  >
    <video bind:this={videoElement}>
      <track kind="captions" />
    </video>
    <div
      class="absolute bottom-0 left-0 right-0 bg-black/70 flex items-center justify-between p-4 text-white rounded-t-lg gap-4"
      class:opacity-0={isFullscreen}
      class:hover:opacity-100={isFullscreen}
    >
      <button
        on:click={onPlay}
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
      >
        {#if isPlaying}
          <Pause />
        {:else}
          <Play />
        {/if}
      </button>
      <span class="text-sm">{formatTime(currentTime)}</span>
      <input
        type="range"
        min="0"
        max={duration}
        step="0.1"
        value={currentTime}
        on:input={handleSeek}
        class="flex-1 mx-4 appearance-none bg-gray-700 h-1 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <span class="text-sm">{formatTime(duration)}</span>
      <button
        on:click={playNext}
        disabled={currentFileIndex === filesInQueue.length - 1}
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
      >
        <SkipForward />
      </button>
      <button
        on:click={toggleFullscreen}
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
      >
        {#if isFullscreen}
          <Minimize />
        {:else}
          <Maximize />
        {/if}
      </button>
    </div>
  </div>
  <div class="w-1/4 border-l border-primary">
    <div class="flex-1">
      <div class="flex flex-col gap-4">
        <div class="flex flex-row items-center gap-2 bg-primary p-4">
          <Play class="h-5 w-5 text-accent" />
          <h3 class="text-lg font-bold text-accent">Queue</h3>
        </div>

        <ul class="flex flex-col px-4 min-h-6">
          {#each filesInQueue as file, index}
            <li
              class="flex flex-row justify-between text-primary py-4 border-b border-b-primary last:border-none"
            >
              <div class="flex flex-row gap-2 items-center">
                <span
                  class="flex items-center justify-center bg-secondary rounded rounded-full text-sm font-bold w-6 h-6"
                >
                  {index + 1}
                </span>
                <p class="text-accent text-sm">
                  {formatFileName(file.name, 20)}
                </p>
              </div>
              <Button
                class="text-sm h-8 w-fit bg-secondary rounded hover:bg-secondary text-gray-900"
                on:click={() => handleOnRemoveFileInQueue(index)}
              >
                <Trash class="h-4 w-4" />
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
    <div class="flex-1 flex flex-col mt-4">
      <div class="flex flex-col gap-8">
        <div class="flex flex-col gap-1">
          <div class="flex flex-row gap-2 items-center bg-primary p-4">
            <ListMusic class="h-5 w-5 text-accent" />
            <h3 class="text-lg font-bold text-accent">Media Area</h3> 
          </div>
          <p class="px-4 py-2 text-accent text-sm">First download the files in your computer and import it to the "Media Area", before adding them to "Play Queue". This ensure the files are added in the proper order.</p>
        </div>
        
        
        <div class="flex flex-row gap-4 px-4">
          <Button class="text-sm h-8 w-fit bg-primary text-secondary rounded" on:click={handleAddMediaFiles}>
            <Plus class="mr-2 h-4 w-4 text-secondary" />Add files
          </Button>
          <Button
            class="text-sm h-8 w-fit bg-secondary rounded"
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
                  class="text-sm h-8 w-fit bg-secondary rounded hover:bg-secondary"
                  on:click={() => handleAddToQueue(file)}
                >
                  <ListPlus class="h-4 w-4" />
                </Button>
                <Button
                  class="text-sm h-8 w-fit bg-secondary rounded hover:bg-secondary"
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
