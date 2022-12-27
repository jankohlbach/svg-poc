<script lang="ts" setup>
import { onMounted, ref } from "vue";
import { v4 as uuidv4 } from "uuid";

const svgs = [
  '<svg viewBox="0 0 32 32" width="128px" height="128px"><path d="M32 18.451l-16-12.42-16 12.42v-5.064l16-12.42 16 12.42zM28 18v12h-8v-8h-8v8h-8v-12l12-9z"></path></svg>',
  '<svg viewBox="0 0 32 32" width="128px" height="128px"><path d="M29.996 4c0.001 0.001 0.003 0.002 0.004 0.004v23.993c-0.001 0.001-0.002 0.003-0.004 0.004h-27.993c-0.001-0.001-0.003-0.002-0.004-0.004v-23.993c0.001-0.001 0.002-0.003 0.004-0.004h27.993zM30 2h-28c-1.1 0-2 0.9-2 2v24c0 1.1 0.9 2 2 2h28c1.1 0 2-0.9 2-2v-24c0-1.1-0.9-2-2-2v0z"></path><path d="M26 9c0 1.657-1.343 3-3 3s-3-1.343-3-3 1.343-3 3-3 3 1.343 3 3z"></path><path d="M28 26h-24v-4l7-12 8 10h2l7-6z"></path></svg>',
  '<svg viewBox="0 0 32 32" width="128px" height="128px"><path d="M20.586 23.414l-6.586-6.586v-8.828h4v7.172l5.414 5.414zM16 0c-8.837 0-16 7.163-16 16s7.163 16 16 16 16-7.163 16-16-7.163-16-16-16zM16 28c-6.627 0-12-5.373-12-12s5.373-12 12-12c6.627 0 12 5.373 12 12s-5.373 12-12 12z"></path></svg>',
];

const svgRefs = ref([]);
const dropZone = ref(null);

const handleDragstart = (e: DragEvent) => {
  console.log("dragstart");

  if (e.dataTransfer) {
    e.dataTransfer.effectAllowed = "copyMove";

    const el = e.target as HTMLElement | null;

    if (el) {
      const rect = el.getBoundingClientRect();

      e.dataTransfer.setData("application/drag-id", el.id);
      e.dataTransfer.setData(
        "application/offset",
        `${e.clientX - rect.left},${e.clientY - rect.top}`
      );
    }
  }
};

const handleDragend = (e: DragEvent) => {
  console.log("dragend");
  e.dataTransfer?.clearData();
};

const handleDragover = (e: DragEvent) => {
  e.preventDefault();
  console.log("dragover");
};

const handleDrop = (e: DragEvent) => {
  e.preventDefault();
  console.log("drop");

  if (e.dataTransfer) {
    const offset = e.dataTransfer.getData("application/offset").split(",");
    const elId = e.dataTransfer.getData("application/drag-id");
    const el = document.getElementById(elId);

    switch (el?.dataset.drag) {
      case "copy":
        // eslint-disable-next-line no-case-declarations
        const clone = el.cloneNode(true) as HTMLElement;

        clone.id = uuidv4();
        clone.classList.add("dropped");
        clone.dataset.drag = "move";
        clone.style.left = e.clientX - parseInt(offset[0]) + "px";
        clone.style.top = e.clientY - parseInt(offset[1]) + "px";
        (e.target as HTMLElement | null)?.appendChild(clone);

        clone.addEventListener("dragstart", handleDragstart);
        clone.addEventListener("dragend", handleDragend);
        break;
      case "move":
        el.style.left = e.clientX - parseInt(offset[0]) + "px";
        el.style.top = e.clientY - parseInt(offset[1]) + "px";
        break;
    }
  }
};

onMounted(() => {
  console.log(svgRefs.value);

  svgRefs.value.forEach((svgWrap: HTMLElement) => {
    console.log(svgWrap);
    svgWrap.addEventListener("dragstart", handleDragstart);
    svgWrap.addEventListener("dragend", handleDragend);
  });

  if (dropZone.value) {
    (dropZone.value as HTMLElement).addEventListener(
      "dragover",
      handleDragover
    );
    (dropZone.value as HTMLElement).addEventListener("drop", handleDrop);
  }
});
</script>

<template>
  <div class="container">
    <div class="sidebar">
      <ul>
        <li v-for="(svg, i) in svgs" :key="i">
          <span
            :id="uuidv4()"
            ref="svgRefs"
            v-html="svg"
            draggable="true"
            data-drag="copy"
          ></span>
        </li>
      </ul>
    </div>
    <div class="target" ref="dropZone"></div>
  </div>
</template>

<style lang="scss" scoped>
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
  min-height: 100vh;
}

.sidebar {
  grid-column: 1 / 2;
  background-color: rgb(138, 138, 138);
}

.target {
  grid-column: 2 / 3;
  background-color: rgb(200, 207, 206);
}

span {
  display: inline-block;
  cursor: move;

  &.dropped {
    position: absolute;
  }
}
</style>
