ProseVue — A Modern Tiptap Editor for Vue 3 (TypeScript) ✨

I wanted a Notion-level editor in Vue…
without installing a giant ecosystem, fighting configs, or writing 1,000 lines of glue code.
So I built one.

ProseVue is a high-performance WYSIWYG editor component powered by Tiptap v2, styled with Tailwind CSS, and written in strict TypeScript — designed to feel like a premium product while remaining a drop-in Vue component.

No packages.
No wrappers.
No magic.

Just copy → install deps → ship.

⸻


👉 Live Demo: https://prose-vue-tiptap-editor.vercel.app/
👉 JavaScript Version: https://github.com/Joshualeexy/proseVue-tiptap-editor

⸻

🧠 The Story

Most Vue editors fall into one of two categories:
	•	❌ Too basic
	•	❌ Too heavy and opinionated

I wanted something different:
	•	Fully controlled by the parent (v-model)
	•	Beautiful by default
	•	Strictly typed
	•	Production-ready
	•	Easy enough to copy into any project

So instead of publishing another dependency, I built a standalone editor component you actually understand.

You own the code.
You customize everything.
No lock-in.

⸻

⚡ What Makes ProseVue Different

🧩 A Component — Not a Package

This is intentional.

You copy the editor into your project instead of depending on a black box.

That means:
	•	Full control
	•	Zero vendor friction
	•	Easy customization
	•	No version conflicts

⸻

🔄 Fully Controlled Editor (Real Vue Pattern)

The parent owns state:

<TipTapEditor v-model="content" />

The editor behaves like a native Vue input:
	•	reactive
	•	predictable
	•	composable
	•	TypeScript-safe

⸻

🎨 Premium UX (Not Just Functional)

Built with product-level polish:
	•	Glassmorphic sticky toolbar
	•	Floating & bubble menus
	•	Micro-interaction feedback
	•	Dark mode support
	•	Dynamic accent theming

Feels closer to Notion / Google Docs than a typical OSS editor.

⸻

🛠 Powerful Editing Features
	•	Tables with resizable columns
	•	Task lists ✅
	•	YouTube embeds
	•	Link & image popovers (no browser prompts)
	•	Typography engine (smart symbols)
	•	Subscript / Superscript
	•	Text alignment & formatting
	•	Character limits + progress tracking
	•	HTML + ProseMirror JSON output

⸻

🚀 Zero-Config Setup

1️⃣ Install dependencies

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

2️⃣ Register FontAwesome Icons

import { library } from "@fortawesome/fontawesome-svg-core";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { fas } from "@fortawesome/free-solid-svg-icons";

library.add(fas);
app.component("font-awesome-icon", FontAwesomeIcon);


⸻

3️⃣ Use It

<script setup lang="ts">
import { ref } from 'vue'
import TipTapEditor from './TipTapEditor.vue'

const content = ref('<h1>Hello world</h1>')
</script>

<template>
  <TipTapEditor v-model="content" />
</template>

Done.

⸻

🎛 Props

Prop	Type	Description
v-model	string	HTML output
v-model:json	JSONContent	Structured editor data
accentColor	string	Theme color
limit	number	Character limit
placeholder	string	Editor hint text


⸻

🧩 Philosophy

ProseVue is built around a simple idea:

Open source should feel premium.

You shouldn’t need enterprise software to get enterprise UX.

⸻

⭐ If You Like It
	•	Star the repo
	•	Fork it
	•	Improve it
	•	Build something beautiful

⸻

📜 License

MIT — free for personal & commercial use.

⸻
