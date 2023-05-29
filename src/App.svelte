<script>
  // @ts-nocheck
  import "./app.css";
  import { onMount } from "svelte";

  let shapeSize = 150;
  let shapeColor = "#5754E7";
  let shapeOpacity = 0.7;
  let shapeRotation = 0;
  let shapes = [];

  let selectedShapes = [];
  let currentShape = "Circle";
  let svg;
  let aside;
  let isDragging = false;

  function downloadJSON() {
    const json = JSON.stringify(shapes);
    const blob = new Blob([json], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = "shapesJSON";
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }

  function loadJSON(event) {
    const file = event.target.files[0];
    const reader = new FileReader();

    reader.onload = () => {
      const json = JSON.parse(reader.result);
      shapes = [...shapes, ...json];
    };

    reader.readAsText(file);
  }

  const handleRotationChange = (event) => {
    shapeRotation = Number(event.target.value);

    for (let i = 0; i < selectedShapes.length; i++) {
      const index = shapes.findIndex(
        (shape) => shape.id === parseInt(selectedShapes[i].id)
      );
      shapes[index].rotate = shapeRotation;
    }
  };

  const handleSizeChange = (event) => {
    console.log("poggers");
    shapeSize = Number(event.target.value);
    for (let i = 0; i < selectedShapes.length; i++) {
      const index = shapes.findIndex(
        (shape) => shape.id === parseInt(selectedShapes[i].id)
      );
      shapes[index].size = shapeSize;
    }
  };

  const handleColorChange = (event) => {
    shapeColor = event.target.value;
    for (let i = 0; i < selectedShapes.length; i++) {
      const index = shapes.findIndex(
        (shape) => shape.id === parseInt(selectedShapes[i].id)
      );
      shapes[index].color = shapeColor;
    }
  };

  const handleOpacityChange = (event) => {
    shapeOpacity = Number(event.target.value);
    for (let i = 0; i < selectedShapes.length; i++) {
      const index = shapes.findIndex(
        (shape) => shape.id === parseInt(selectedShapes[i].id)
      );
      shapes[index].opacity = shapeOpacity;
    }
  };

  const addShape = () => {
    const shape = {
      type: currentShape,
      id: Date.now(),
      name: "Element " + (shapes.length + 1),
      x: Math.floor(Math.random() * (window.innerWidth - 100 * 2)) + 30,
      y: Math.floor(Math.random() * (window.innerHeight - 100 * 2)) + 10,
      size: shapeSize,
      color: shapeColor,
      opacity: shapeOpacity,
      rotate: shapeRotation,
      isEditing: false,
    };
    shapes = [...shapes, shape];
  };

  const deleteShapes = () => {
    if (selectedShapes) {
      for (let i = 0; i < selectedShapes.length; i++) {
        shapes = shapes.filter((c) => c.id !== parseInt(selectedShapes[i].id));
      }
      selectedShapes = [];
    }
  };

  const startDragging = (event, shape) => {
    event.preventDefault();
    selectedShapes = [];
    const updatedshapes = shapes.map((c) => {
      if (c.id === shape.id) {
        return {
          ...c,
          isDragging: true,
          prevX: event.clientX,
          prevY: event.clientY,
        };
      }
      return c;
    });
    shapes = updatedshapes;
    selectedShapes.push(shape);
  };

  const selectByID = (shape) => {
    selectedShapes = [];
    selectedShapes.push(shape);
  };

  const stopDragging = (event, shape) => {
    event.preventDefault();
    const updatedshapes = shapes.map((c) => {
      if (c.id === shape.id) {
        return { ...c, isDragging: false };
      }
      return c;
    });
    shapes = updatedshapes;
  };

  const handleMouseMove = (event, shape) => {
    event.preventDefault();
    if (shape.isDragging) {
      const deltaX = event.clientX - shape.prevX;
      const deltaY = event.clientY - shape.prevY;
      const updatedshapes = shapes.map((c) => {
        if (c.id === shape.id) {
          return {
            ...c,
            x: c.x + deltaX,
            y: c.y + deltaY,
            prevX: event.clientX,
            prevY: event.clientY,
          };
        }
        return c;
      });
      shapes = updatedshapes;
    }
  };

  function saveSvgAsFile() {
    const svgElement = document.querySelector("svg");

    const svgXml = new XMLSerializer().serializeToString(svgElement);

    const blob = new Blob([svgXml], { type: "image/svg+xml" });

    const url = URL.createObjectURL(blob);

    const link = document.createElement("a");
    link.href = url;
    link.download = "svg_file.svg";

    link.dispatchEvent(new MouseEvent("click"));

    URL.revokeObjectURL(url);
  }

  onMount(() => {
    svg.addEventListener("click", (event) => {
      if (event.target === svg && !isDragging) {
        selectedShapes = [];
      }

      shapes.forEach((shape) => {
        if (shape.isEditing) {
          shape.isEditing = false;
        }
      });
    });

    svg.addEventListener("mousedown", () => {
      isDragging = false;
    });

    window.addEventListener("mousemove", (event) => {
      const isInputClicked = event.target.closest("input");

      if (!isInputClicked) {
        shapes.forEach((shape) => handleMouseMove(event, shape));
        isDragging = true;
      }
    });

    aside.addEventListener("click", (event) => {
      if (event.target === "input") {
        shapes.forEach((shape) => (shape.isEditing = false));
      }
    });

    window.addEventListener("mouseup", (event) => {
      const isInputClicked = event.target.closest("input");
      if (!isInputClicked) {
        shapes.forEach((shape) => stopDragging(event, shape));
      }
    });
  });

  const selection = new SelectionArea({
    // Class for the selection-area itself (the element).
    selectionAreaClass: "selection-area",

    // Class for the selection-area container.
    selectionContainerClass: "selection-area-container",

    // Query selector or dom-node to set up container for the selection-area element.
    container: "body",

    // document object - if you want to use it within an embed document (or iframe).
    // If you're inside of a shadow-dom make sure to specify the shadow root here.
    document: window.document,

    // Query selectors for elements which can be selected.
    selectables: ["circle, rect"],

    // Query selectors for elements from where a selection can be started from.
    startareas: ["html"],

    // Query selectors for elements which will be used as boundaries for the selection.
    // The boundary will also be the scrollable container if this is the case.
    boundaries: ["html"],

    // Behaviour related options.
    behaviour: {
      // Specifies what should be done if already selected elements get selected again.
      //   invert: Invert selection for elements which were already selected
      //   keep: Keep selected elements (use clearSelection() to remove those)
      //   drop: Remove stored elements after they have been touched
      overlap: "keep",

      // On which point an element should be selected.
      // Available modes are cover (cover the entire element), center (touch the center) or
      // the default mode is touch (just touching it).
      intersect: "touch",

      // px, how many pixels the point should move before starting the selection (combined distance).
      // Or specifiy the threshold for each axis by passing an object like {x: <number>, y: <number>}.
      startThreshold: 5,

      // Scroll configuration.
      scrolling: {
        // On scrollable areas the number on px per frame is devided by this amount.
        // Default is 10 to provide a enjoyable scroll experience.
        speedDivider: 10,

        // Browsers handle mouse-wheel events differently, this number will be used as
        // numerator to calculate the mount of px while scrolling manually: manualScrollSpeed / scrollSpeedDivider.
        manualSpeed: 750,

        // This property defines the virtual inset margins from the borders of the container
        // component that, when crossed by the mouse/touch, trigger the scrolling. Useful for
        // fullscreen containers.
        startScrollMargins: { x: 0, y: 0 },
      },
    },

    // Features.
    features: {
      // Enable / disable touch support.
      touch: true,

      // Range selection.
      range: true,

      // Configuration in case a selectable gets just clicked.
      singleTap: {
        // Enable single-click selection (Also disables range-selection via shift + ctrl).
        allow: true,

        // 'native' (element was mouse-event target) or 'touch' (element visually touched).
        intersect: "native",
      },
    },
  });

  selection
    .on("beforestart", ({ event }) => {
      if (event.target.tagName) {
        return event.target.tagName === "svg";
      }
    })
    .on("beforedrag", ({ event }) => {
      // Same as 'beforestart' but before a selection via dragging happens.
      if (event.target.tagName) {
        selectedShapes = [];
        return event.target.tagName === "svg";
      }
    })
    .on("start", ({ store, event }) => {
      // if (!event.ctrlKey && !event.metaKey) {
      //   for (const el of store.stored) {
      //     el.classList.remove("selected");
      //   }
      //   selection.clearSelection();
      // }
    })
    .on("move", (evt) => {
      // Here you can update elements based on their state.
    })
    .on("stop", (evt) => {
      // Do something with the selected elements.
      if (evt.store) {
        for (let i = 0; i < evt.store.selected.length; i++) {
          selectedShapes.push(evt.store.selected[i]);
        }
      }
    });
</script>

<div class="h-screen">
  <nav class="flex flex-col lg:flex-row h-[30%] lg:h-[10%]">
    <div class="wrapper">
      <div>
        <label for="input-rotate">Rotate</label>
        <input
          id="input-rotate"
          type="number"
          min="0"
          max="360"
          step="10"
          bind:value={shapeRotation}
          on:change={handleRotationChange}
        />
      </div>
      <div>
        <label for="input-size">Size</label>
        <input
          id="input-size"
          type="number"
          min="10"
          max="400"
          step="10"
          bind:value={shapeSize}
          on:change={handleSizeChange}
        />
      </div>
      <div>
        <label for="input-opacity">Opacity</label>
        <input
          id="input-opacity"
          type="number"
          min="0.1"
          max="1"
          step="0.1"
          bind:value={shapeOpacity}
          on:change={handleOpacityChange}
        />
      </div>

      <input
        id="input-color"
        type="color"
        bind:value={shapeColor}
        on:input={handleColorChange}
      />
    </div>
    <div class="flex flex-wrap gap-6">
      <select
        class="select bg-[#F7FAFB] text-black shadow-md"
        bind:value={currentShape}
        on:change={(event) => (currentShape = event.target.value)}
      >
        <option>Circle</option>
        <option>Rectangle</option>
        <option>Custom</option>
      </select>
      <button
        on:click={addShape}
        class="text-2xl m-auto text-[#46414D] transition hover:text-green-600 hover:scale-110 active:scale-95"
        ><i class="fa-solid fa-plus" /></button
      >
      <button
        on:click={deleteShapes}
        class="text-2xl text-[#46414D] transition hover:text-red-600 hover:scale-110 active:scale-95"
        ><i class="fa-solid fa-trash" /></button
      >

      <div class="divider lg:divider-horizontal" />

      <button
        on:click={downloadJSON}
        class="text-2xl items-center text-[#46414D] transition hover:text-sky-600 hover:scale-110 active:scale-95"
        ><i class="fa-solid fa-download" /></button
      >
      <label
        for="file"
        class="text-2xl m-auto text-[#46414D] cursor-pointer transition hover:text-fuchsia-600 hover:scale-110 active:scale-95"
        ><i class="fa-solid fa-upload" /></label
      >
      <input type="file" id="file" on:change={loadJSON} style="display:none" />
      <button
        on:click={saveSvgAsFile}
        class="btn bg-[#F45930] text-white rounded-xl"
      >
        <i class="fa-solid fa-share text-white mr-2" />
        Share</button
      >
    </div>
  </nav>
  <div class="flex justify-center h-[70%] lg:h-[90%]">
    <span class="w-[75%] lg:w-[90%]">
      <svg bind:this={svg}>
        {#each shapes as shape}
          {#if shape.type === "Circle"}
            <circle
              id={shape.id}
              name={shape.name}
              isEditing={shape.isEditing}
              cx={shape.x}
              cy={shape.y}
              r={shape.size / 2}
              fill={shape.color}
              opacity={shape.opacity}
              transform={`rotate(${shape.rotate} ${shape.x} ${shape.y})`}
              stroke-width={selectedShapes.some(
                (selected) => selected.id == shape.id
              )
                ? 3
                : 0.5}
              on:mousedown={(event) => startDragging(event, shape)}
              on:mouseup={(event) => stopDragging(event, shape)}
              on:mousemove={(event) => handleMouseMove(event, shape)}
            />
          {:else if shape.type === "Rectangle"}
            <rect
              id={shape.id}
              name={shape.name}
              x={shape.x}
              y={shape.y}
              height={shape.size}
              width={shape.size}
              fill={shape.color}
              opacity={shape.opacity}
              transform={`rotate(${shape.rotate} ${shape.x} ${shape.y})`}
              stroke-width={selectedShapes.some(
                (selected) => selected.id == shape.id
              )
                ? 3
                : 0.5}
              on:mousedown={(event) => startDragging(event, shape)}
              on:mouseup={(event) => stopDragging(event, shape)}
              on:mousemove={(event) => handleMouseMove(event, shape)}
            />
          {:else if shape.type === "Custom"}
            <g
              fill={shape.color}
              opacity={shape.opacity}
              transform={`rotate(${shape.rotate} ${shape.x} ${shape.y})`}
              stroke-width={selectedShapes.some(
                (selected) => selected.id == shape.id
              )
                ? 3
                : 0.5}
              on:mousedown={(event) => startDragging(event, shape)}
              on:mouseup={(event) => stopDragging(event, shape)}
              on:mousemove={(event) => handleMouseMove(event, shape)}
            >
              <circle
                id={shape.id}
                name={shape.name}
                cx={shape.x}
                cy={shape.y}
                r={shape.size / 2}
              />

              <polygon
                points={`${shape.x},${shape.y - shape.size / 2}
                      ${shape.x - shape.size / 4},${shape.y + shape.size / 4}
                      ${shape.x + shape.size / 4},${shape.y + shape.size / 4}`}
                fill="black"
              />
            </g>
          {/if}
        {/each}
      </svg>
    </span>
    <aside
      bind:this={aside}
      class="bg-[#F7FAFB] w-[25%] lg:w-[10%] overflow-y-auto text-left text-black rounded-bl-3xl border-l-2 border-black"
    >
      {#each shapes as shape}
        <div class="border-b-2 border-gray-300">
          {#if !shape.isEditing}
            <button
              class="w-full flex justify-between p-3 text-lg font-semibold"
              on:click={() => {
                selectByID(shape);
              }}
            >
              <div>{shape.name}</div>
              <button
                on:click={() => {
                  shape.isEditing = true;
                }}><i class="fa-solid fa-pen-to-square" /></button
              >
            </button>
          {:else}
            <input
              type="text"
              class="bg-transparent p-3 text-lg font-semibold"
              class:selected={shape.selected}
              autoFocus
              bind:value={shape.name}
              disabled={!shape.isEditing}
              onfocus="this.select()"
              on:click={() => {
                shape.isEditing = true;
                selectByID(shape);
              }}
              on:blur={() => {
                shape.isEditing = false;
              }}
              on:keydown={(e) => {
                if (e.key === "Enter") {
                  shape.isEditing = false;
                }
              }}
            />
          {/if}
        </div>
      {/each}
    </aside>
  </div>
</div>

<style>
  svg {
    @apply w-full h-full;
  }
  circle,
  rect {
    cursor: move;
    stroke: black;
  }

  button {
    @apply border-none;
  }

  nav {
    @apply bg-[#F7FAFB] border-b-2 border-black flex justify-evenly text-black text-lg text-center items-center;
  }

  input[type="color"] {
    @apply h-[3rem] w-[2.8rem] rounded-full cursor-pointer bg-transparent;
  }

  input[type="color"]::-webkit-color-swatch {
    @apply rounded-full border-none;
  }

  input[type="color"]::-moz-color-swatch {
    @apply rounded-full border-none;
  }

  input[type="number"] {
    @apply pl-2 p-1 rounded-lg border border-[#46414D] w-[4.5rem];
  }

  .wrapper {
    @apply flex justify-center text-center items-center align-middle gap-6;
  }

  .wrapper div {
    @apply flex items-center gap-2;
  }

  span {
    @apply cursor-crosshair;
  }

  *:focus {
    @apply outline-0;
  }

  ::-webkit-scrollbar {
    width: 5px;
  }

  ::-webkit-scrollbar-thumb {
    @apply bg-gray-300 rounded-3xl;
  }
</style>
