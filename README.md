ProseVue — Tiptap Editor for Vue 3

A modern, fully controlled Tiptap v2 editor component for Vue 3 built with TypeScript and Tailwind CSS.

ProseVue provides a production-ready rich text editor with a clean architecture, strong typing, and customizable UI — without introducing an additional package layer or framework lock-in.

⸻

Overview

ProseVue is designed as a drop-in editor component that integrates naturally with Vue applications.

Instead of shipping as an opaque dependency, the editor is intended to be copied into your project, giving full ownership over behavior, styling, and extensions.

Goals
	•	Predictable Vue reactivity (v-model)
	•	Strong TypeScript support
	•	Minimal setup
	•	Fully customizable UI
	•	Production-ready defaults

⸻

Demo & Source
	•	Live Demo
https://prose-vue-tiptap-editor.vercel.app/
	•	Repository
https://github.com/Joshualeexy/proseVue-tiptap-editor

⸻

Features

Editing Capabilities
	•	Text formatting (bold, italic, underline, highlight)
	•	Headings & typography enhancements
	•	Task lists
	•	Tables with resizable columns
	•	Image and YouTube embeds
	•	Links with custom popovers
	•	Text alignment
	•	Subscript / superscript
	•	Horizontal rules
	•	Smart typography transformations

UX Features
	•	Sticky toolbar
	•	Floating & bubble menus
	•	Dark mode support
	•	Dynamic accent color theming
	•	Character limit with progress tracking
	•	Placeholder support

Output Formats
	•	HTML output
	•	ProseMirror JSON output

⸻

Installation

Install required Tiptap dependencies:

npm install @tiptap/vue-3 @tiptap/starter-kit \
@tiptap/extension-underline \
@tiptap/extension-bubble-menu \
@tiptap/extension-floating-menu \
@tiptap/extension-task-list \
@tiptap/extension-task-item \
@tiptap/extension-table \
@tiptap/extension-table-row \
@tiptap/extension-table-cell \
@tiptap/extension-table-header \
@tiptap/extension-youtube \
@tiptap/extension-subscript \
@tiptap/extension-superscript \
@tiptap/extension-typography \
@tiptap/extension-highlight \
@tiptap/extension-text-align \
@tiptap/extension-color \
@tiptap/extension-text-style \
@tiptap/extension-link \
@tiptap/extension-image \
@tiptap/extension-placeholder \
@tiptap/extension-character-count \
@tiptap/extension-horizontal-rule \
@tiptap/pm


⸻

FontAwesome Setup

import { library } from "@fortawesome/fontawesome-svg-core";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { fas } from "@fortawesome/free-solid-svg-icons";

library.add(fas);
app.component("font-awesome-icon", FontAwesomeIcon);


⸻

Usage

<script setup lang="ts">
import { ref } from 'vue'
import TipTapEditor from './TipTapEditor.vue'

const content = ref('<h1>Hello world</h1>')
</script>

<template>
  <TipTapEditor v-model="content" />
</template>


⸻

Props

Prop	Type	Description
v-model	string	HTML editor output
v-model:json	JSONContent	ProseMirror structured output
accentColor	string	Editor theme color
limit	number	Character limit
placeholder	string	Placeholder text


⸻

Architecture

ProseVue follows a controlled editor pattern:
	•	Parent component owns state
	•	Editor emits updates
	•	Vue reactivity remains the single source of truth

<TipTapEditor v-model="content" />

This ensures predictable state management and easy persistence.

⸻

Customization

Since the editor is included directly in your project:
	•	Modify extensions freely
	•	Change toolbar layout
	•	Replace UI components
	•	Extend commands
	•	Adjust styling without overrides

No abstraction layer or vendor lock-in is introduced.

⸻

Philosophy

ProseVue prioritizes:
	•	Transparency over abstraction
	•	Control over convenience packages
	•	Production UX with open tooling

⸻

License

MIT — free for personal and commercial use.
