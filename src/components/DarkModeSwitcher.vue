<script setup lang="ts">
import SunIcon from "./icons/Sun.vue";
import { roomStore } from "@/stores/room";
const room = roomStore();

if (
  localStorage.darkMode === "true" ||
  (!("darkMode" in localStorage) &&
    window.matchMedia("(prefers-color-scheme: dark)").matches)
) {
  document.documentElement.classList.add("dark");
  room.isDarkMode = true;
} else {
  document.documentElement.classList.remove("dark");
  room.isDarkMode = false;
}

const toggleDarkMode = () => {
  localStorage.darkMode = room.isDarkMode = !room.isDarkMode;
  room.isDarkMode
    ? document.documentElement.classList.add("dark")
    : document.documentElement.classList.remove("dark");
};
</script>

<template>
  <span @click="toggleDarkMode" class="cursor-pointer">
    <SunIcon v-if="!room.isDarkMode" width="18" height="18" color="#000" />
    <MoonIcon v-else width="18" height="18" color="#e4e4e7" />
  </span>
</template>
