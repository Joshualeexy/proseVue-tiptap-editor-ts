<script setup lang="ts">
import { ref } from 'vue';
import TipTapEditor from './TipTapEditor.vue';
import type { JSONContent } from '@tiptap/core';

const content = ref('<h1>Welcome to Tiptap!</h1><p>Start typing here...</p>');
const jsonContent = ref<JSONContent | null>(null);
const accentColor = ref('#0e2daa');
</script>

<template>
  <div class="min-h-screen sm:p-4 md:p-4 flex flex-col items-center">
    <div class="w-full max-w-4xl bg-white rounded-2xl  shadow-2xl p-6">
      <h1 class="text-2xl font-bold mb-4 text-purple-700 text-center" :style="{ color: accentColor }">Vue3 Tailwind Tiptap 2 Editor</h1>
      
      <div class="flex items-center justify-center gap-4 mb-8">
        <label class="text-gray-600 font-medium">Accent Color:</label>
        <input type="color" v-model="accentColor" class="w-10 h-10 rounded cursor-pointer border-none" />
        <span class="text-sm font-mono text-gray-400">{{ accentColor }}</span>
      </div>

      <TipTapEditor v-model="content" v-model:json="jsonContent" :accent-color="accentColor" placeholder="Type something amazing here..." />
      
      <div class="mt-8">
        <h2 class="text-xl font-semibold mb-4 text-gray-700">HTML Output:</h2>
        <pre class="bg-gray-100 p-4 rounded-lg overflow-x-auto text-xs text-gray-600 border border-gray-200">{{ content }}</pre>
      </div> <div class="mt-8">
        <h2 class="text-xl font-semibold mb-4 text-gray-700">JSON Output:</h2>
        <pre class="bg-gray-100 p-4 rounded-lg overflow-x-auto text-xs text-gray-600 border border-gray-200">{{ jsonContent }}</pre>
      </div>
    </div>
  </div>
</template>
