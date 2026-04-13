<script>

  import { onMount } from 'svelte';
  import logo from '/pwlogo.png'

  let borders = [ 'Alpha', 'Beta', 'Delta', 'Eta', 'Epsilon', 'Gamma', 'Jota', 'Lambda', 'Theta', 'Zeta', 'Omega' ]

  let deres = [ 'Bodere', 'Dandere', 'Deredere', 'Kamidere', 'Kuudere', 'Mayadere',
    'Tsundere', 'Yandere', 'Raito', 'Yami', 'Yato' ]

  let variantsMap = {
  };

  async function fetchVariants() {
    try {
      const response = await fetch('https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Extensions/CardExtension.cs');
      const text = await response.text();
      const startIndex = text.indexOf('public static int GetCardVariantsCount(this Card card)');
      if (startIndex !== -1) {
        const functionText = text.substring(startIndex);
        const endIndex = functionText.indexOf('}');
        if (endIndex !== -1) {
          const functionBody = functionText.substring(0, endIndex + 1);
          const lines = functionBody.split(/\r?\n/);
          for (const line of lines) {
            const match = line.match(/Quality\.(\w+)\s*=>\s*(\d+)/);
            if (match) {
              variantsMap = { ...variantsMap, [match[1]]: parseInt(match[2]) };
            }
          }
        }
      }
      initDefaults();
      updateData();
    } catch (error) {
      console.error('Error fetching variants:', error);
    }
  }

  onMount(fetchVariants);

  function getVariantsCount(border) {
    return variantsMap[border] || 0;
  }

  function getStyleList() {
    switch (selectedBorder) {
      case 'Delta':
        return Array.from({length: getVariantsCount('Delta') + 1}, (_, i) => i.toString());
      case 'Eta':
        return Array.from({length: getVariantsCount('Eta') + 1}, (_, i) => i.toString());
      case 'Lambda':
        return Array.from({length: getVariantsCount('Lambda') + 1}, (_, i) => i.toString());
      default:
        return null;
    }
  }

  let image = "https://sanakan.pl/i/ss/sUwh3io.png";
  let customBorder = "";
  let showStats = false;
  let localImage = false;
  let dragOver = false;

  let selectedBorder = 'Delta';
  let selectedDere = 'Mayadere';
  let selectedStyle = '2'
  let styles = [];

  let fileinput;

  function processFile(imageFile) {
    if (!imageFile.type.startsWith('image/')) {
      alert('Proszę przeciągnąć plik obrazu JPG lub PNG.');
      return;
    }

    if (imageFile) {
      let reader = new FileReader();
      reader.onload = e => {
        image = e.target.result;
        localImage = true;
      };
      reader.readAsDataURL(imageFile);
    }
  }

  function initDefaults() {
    selectedBorder = 'Delta';
    selectedDere = 'Mayadere';
    selectedStyle = '2';
    customBorder = "";
    showStats = false;
    localImage = false;
    image = "https://sanakan.pl/i/ss/sUwh3io.png";
  }

  function onFileSelected(e) {
    let imageFile = e.target.files[0];
    processFile(imageFile);
  }

  function onDragOver(e) {
    e.preventDefault();
    e.stopPropagation();
    dragOver = true;
  }

  function onDragLeave(e) {
    e.preventDefault();
    e.stopPropagation();
    dragOver = false;
  }

  function onDrop(e) {
    e.preventDefault();
    e.stopPropagation();
    dragOver = false;

    const imageFile = e.dataTransfer.files[0];
    if (!imageFile) return;

    const dt = new DataTransfer();
    dt.items.add(imageFile);
    fileinput.files = dt.files;
    processFile(imageFile);
  }

  $: styles = getStyleList();

  function getStyle() {
    let variantsCount = getVariantsCount(selectedBorder);
    if (variantsCount === 0) return "";

    let selectedStyleInt = parseInt(selectedStyle);
    if (selectedStyleInt > variantsCount || selectedStyleInt <= 0)
    {
      selectedStyle = '0';
      return "";
    }

    return selectedStyle;
  }

  function getBorder() {
    switch (selectedBorder) {
      case 'Beta':
      case 'Epsilon':
      case 'Gamma':
      case 'Theta':
        return "";
      case 'Omega':
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Border${styleUri}.webp`;
      default:
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Border${styleUri}.png`;
    }
  }

  function getBackBorder() {
    switch (selectedBorder) {
      case 'Jota':
          return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Border/${selectedDere}.png`;
      case 'Delta':
      case 'Eta':
      case 'Lambda':
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/BorderBack${styleUri}.png`;
      case 'Omega':
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/BorderBack${styleUri}.webp`;
      default:
        return "";
    }
  }

  function getDere() {
    switch (selectedBorder) {
      case 'Beta':
      case 'Epsilon':
      case 'Gamma':
      case 'Theta':
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Border/${selectedDere}.png`
      default:
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Dere/${selectedDere}.png`;
    }
  }

  function getStats() {
    switch (selectedBorder) {
      case 'Lambda':
      case 'Zeta':
        return "";
      case 'Gamma':
      case 'Jota':
      case 'Theta':
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Stats/${selectedDere}.png`;
      case 'Beta':
      case 'Epsilon':
        if (selectedDere === 'Yami' || selectedDere === 'Raito' || selectedDere === 'Yato')
          return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Stats/${selectedDere}.png`
      default:
        return `https://raw.githubusercontent.com/MZKNEK/sanakan/master/src/Pictures/PW/CG/${selectedBorder}/Stats${styleUri}.png`;
    }
  }

  let styleUri = getStyle();
  let borderUri = getBorder();
  let backBorderUri = getBackBorder();
  let statsUri = getStats();
  let dereUri = getDere();

  function updateData() {
    styles = getStyleList();
    styleUri = getStyle();
    borderUri = getBorder();
    backBorderUri = getBackBorder();
    statsUri = getStats();
    dereUri = getDere();
  }

</script>

<main>
  <div>
    <a href="https://sanakan.pl" target="_blank" rel="noreferrer"> <img src={logo} class="logo" alt="Logo" /> </a>
  </div>

  <div class="selector">
    <label><div class="stext">Ramka:</div> <select bind:value={selectedBorder} on:change={() => updateData()} >
      {#each borders as value}<option {value}>{value}</option>{/each}
    </select></label>

    <label><div class="stext">Dere:</div> <select class="nselect" bind:value={selectedDere} on:change={() => updateData()} >
      {#each deres as value}<option {value}>{value}</option>{/each}
    </select></label>

    {#if styles}
      <label><div class="stext">Styl:</div> <select class="nselect" bind:value={selectedStyle} on:change={() => updateData()} >
        {#each styles as value}<option {value}>{value}</option>{/each}
      </select></label>
    {/if}
  </div>
  <div class="selector">
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="dropzone {dragOver ? 'drag-over' : ''}"
      on:dragover={onDragOver}
      on:dragleave={onDragLeave}
      on:drop={onDrop}>
      <div class="dropzone-content">
        <div class="ltext-inline">Lokalny plik:</div>
        <input type="file" class="file-input" accept=".jpg, .jpeg, .png" on:change={(e)=>onFileSelected(e)} bind:this={fileinput} />
      </div>
    </div>
    <br/>
    {#if !localImage}
      <label><div class="ltext">Link do obrazka:</div> <input bind:value={image} /> </label><br/>
    {/if}
    <label><div class="ltext">Link do ramki:</div> <input bind:value={customBorder} /> </label><br/>
    <label><div class="ltext">Pokaż statystyki:</div> <input type="checkbox" bind:checked={showStats} /> </label><br/>
  </div>
  <div class="looks">
    <img src={image} class="scalp" alt="Scalpel" />
      {#if customBorder}
        <img src={customBorder} class="border" alt="Border" />
      {:else}
        {#if backBorderUri}
          <img src="{backBorderUri}" class="back" alt="BorderBack" />
        {/if}

        {#if borderUri}
          <img src="{borderUri}"  class="border" alt="Border" />
        {/if}

        {#if dereUri}
          <img src="{dereUri}" class="dere" alt="Dere" />
        {/if}

        {#if showStats && statsUri}
          <img src="{statsUri}" class="stats" alt="Stats" />
        {/if}
      {/if}
  </div>
</main>

<style>
  .logo {
    height: 6em;
    will-change: filter;
    transition: filter 300ms;
  }
  .ltext {
    display: inline-block;
    width: 130px;
    text-align: left;
  }
  .stext {
    display: inline-block;
    padding-left: 0.5em;
    padding-right: 0.2em;
  }
  .looks {
    position: relative;
    width: 475px;
    height: 667px;
  }
  .scalp {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 475px;
    height: auto;
    pointer-events: none;
    clip-path: xywh(0 0 100% 667px);
  }
  .border {
    position: relative;
    pointer-events: none;
    z-index: 1;
  }
  .dere {
    position: absolute;
    pointer-events: none;
    z-index: 2;
    top: 0px;
    left: 0px;
  }
  .back {
    position: absolute;
    pointer-events: none;
    z-index: -1;
    top: 0px;
  }
  .stats {
    position: absolute;
    pointer-events: none;
    z-index: 3;
    top: 0px;
    left: 0px;
  }
  .dropzone {
    border: 2px dashed #ccc;
    border-radius: 8px;
    padding: 20px;
    text-align: center;
    cursor: pointer;
    transition: all 0.3s ease;
    background-color: #f9f9f9;
    margin-top: 10px;
    margin-bottom: 10px;
  }
  .dropzone:hover {
    border-color: #999;
    background-color: #f0f0f0;
  }
  .dropzone.drag-over {
    border-color: #4CAF50;
    background-color: #e8f5e9;
    box-shadow: 0 0 10px rgba(76, 175, 80, 0.5);
  }
  .dropzone-content {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 10px;
    flex-wrap: wrap;
    justify-content: center;
  }
  .ltext-inline {
    display: inline-block;
    width: auto;
    text-align: left;
  }
  .file-input {
    cursor: pointer;
  }

  @media (prefers-color-scheme: dark) {
    .dropzone {
      border-color: #555;
      background-color: #1e1e1e;
    }
    .dropzone:hover {
      border-color: #777;
      background-color: #2a2a2a;
    }
    .dropzone.drag-over {
      border-color: #66BB6A;
      background-color: #1b5e20;
      box-shadow: 0 0 10px rgba(102, 187, 106, 0.3);
    }
  }
  .selector {
    width: 475px;
    padding-bottom: 1em;
  }
</style>
