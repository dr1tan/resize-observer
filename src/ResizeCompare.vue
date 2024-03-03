<template>
  <div>
    <h2 :style="{ color: resizeObserverColor }">ResizeObserver</h2>
    <h2 :style="{ color: vueResizeObserverColor }">Vue UseResizeObserver</h2>
    <h2 :style="{ color: mutationObserverColor }">MutationObserver</h2>
    <h2 :style="{ color: windowResizeEventColor }">Window Resize Event</h2>
    <div class="interactive-element" ref="observedElement">
      Ändere meine Größe oder Inhalt
    </div>
    <button @click="addBigText">Viel Text</button>
    <button @click="addSmallText">Wenig Text</button>
    <button @click="toggleClass">Klasse ändern</button>
    <button @click="addChildDiv">Add Div</button>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, Ref, nextTick } from "vue";
import { useResizeObserver } from "@vueuse/core";

const observedElement = ref(null) as Ref<HTMLElement | null>;
const resizeObserverColor = ref("black");
const vueResizeObserverColor = ref("black");
const mutationObserverColor = ref("black");
const windowResizeEventColor = ref("black");

let ro: ResizeObserver;
let mo: MutationObserver;
let initialROTrigger = true; // Flag für ResizeObserver
let initialVueROTrigger = true; // Flag für Vue UseResizeObserver
let resizeTimer: ReturnType<typeof setTimeout>;

const resetColors = () => {
  resizeObserverColor.value = "black";
  vueResizeObserverColor.value = "black";
  mutationObserverColor.value = "black";
  windowResizeEventColor.value = "black";
};

const triggerColorChange = (colorRef: typeof resizeObserverColor) => {
  colorRef.value = "green";
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => resetColors(), 3000);
};

let isClassAdded = false;

const toggleClass = () => {
  if (observedElement.value) {
    if (!isClassAdded) {
      observedElement.value.classList.add("fixed-size");
      isClassAdded = true;

      setTimeout(() => {
        if (observedElement.value && isClassAdded) {
          observedElement.value.classList.remove("fixed-size");
          isClassAdded = false;
        }
      }, 3000);
    }
  }
};

const addBigText = () => {
  if (observedElement.value) {
    const newContent = document.createTextNode(
      "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet."
    );
    observedElement.value.appendChild(newContent);

    // Entferne den neuen Inhalt nach kurzer Zeit, um den Observer erneut zu triggern
    setTimeout(() => {
      if (
        observedElement.value &&
        newContent.parentNode === observedElement.value
      ) {
        observedElement.value.removeChild(newContent);
      }
    }, 1000);
  }
};
const addSmallText = () => {
  if (observedElement.value) {
    const newContent = document.createTextNode("Text");
    observedElement.value.appendChild(newContent);

    // Entferne den neuen Inhalt nach kurzer Zeit, um den Observer erneut zu triggern
    setTimeout(() => {
      if (
        observedElement.value &&
        newContent.parentNode === observedElement.value
      ) {
        observedElement.value.removeChild(newContent);
      }
    }, 1000);
  }
};

const addChildDiv = () => {
  if (observedElement.value) {
    const newElement = document.createElement("div");
    newElement.textContent = "Neues Subtree-Element";
    observedElement.value.appendChild(newElement);

    // Entferne das neue Element nach kurzer Zeit
    setTimeout(() => {
      if (
        observedElement.value &&
        newElement.parentNode === observedElement.value
      ) {
        observedElement.value.removeChild(newElement);
      }
    }, 3000);
  }
};

onMounted(() => {
  nextTick(() => {
    if (observedElement.value) {
      ro = new ResizeObserver(() => {
        if (!initialROTrigger) {
          triggerColorChange(resizeObserverColor);
        }
        initialROTrigger = false;
      });
      ro.observe(observedElement.value);

      // Ähnlich für Vue UseResizeObserver
      useResizeObserver(observedElement, () => {
        if (!initialVueROTrigger) {
          triggerColorChange(vueResizeObserverColor);
        }
        initialVueROTrigger = false;
      });

      mo = new MutationObserver(() =>
        triggerColorChange(mutationObserverColor)
      );
      mo.observe(observedElement.value, {
        attributes: true,
        childList: true,
        subtree: true,
      });

      window.addEventListener("resize", () =>
        triggerColorChange(windowResizeEventColor)
      );
    }
  });
});

onUnmounted(() => {
  if (ro) ro.disconnect();
  if (mo) mo.disconnect();
  window.removeEventListener("resize", () =>
    triggerColorChange(windowResizeEventColor)
  );
  clearTimeout(resizeTimer);
});
</script>

<style>
body {
  background-color: white;
}
.interactive-element {
  width: 300px;
  min-height: 150px;
  border: 1px solid black;
  resize: both;
  overflow: auto;
  color: black;
}

.fixed-size {
  background-color: yellow;
}
</style>
