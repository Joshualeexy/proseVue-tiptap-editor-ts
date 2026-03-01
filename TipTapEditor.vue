<script setup lang="ts">
import { useEditor, EditorContent, BubbleMenu, FloatingMenu } from "@tiptap/vue-3";
import { Extension, type JSONContent } from '@tiptap/core';
import StarterKit from "@tiptap/starter-kit";
import Highlight from "@tiptap/extension-highlight";
import TextAlign from "@tiptap/extension-text-align";
import { Color } from "@tiptap/extension-color";
import TextStyle from "@tiptap/extension-text-style";
import Link from "@tiptap/extension-link";
import Image from "@tiptap/extension-image";
import Placeholder from "@tiptap/extension-placeholder";
import CharacterCount from '@tiptap/extension-character-count';
import Underline from '@tiptap/extension-underline';
import TaskList from '@tiptap/extension-task-list';
import TaskItem from '@tiptap/extension-task-item';
import Table from '@tiptap/extension-table';
import TableRow from '@tiptap/extension-table-row';
import TableCell from '@tiptap/extension-table-cell';
import TableHeader from '@tiptap/extension-table-header';
import Youtube from '@tiptap/extension-youtube';
import Subscript from '@tiptap/extension-subscript';
import Superscript from '@tiptap/extension-superscript';
import Typography from '@tiptap/extension-typography';
import HorizontalRule from '@tiptap/extension-horizontal-rule';
import { computed, watch, onBeforeUnmount, ref, nextTick } from 'vue';

// Custom Font Size Extension
interface FontSizeOptions {
  types: string[];
}

declare module '@tiptap/core' {
  interface Commands<ReturnType> {
    fontSize: {
      /**
       * Set the font size
       */
      setFontSize: (fontSize: string) => ReturnType;
      /**
       * Unset the font size
       */
      unsetFontSize: () => ReturnType;
    }
  }
}

const FontSize = Extension.create<FontSizeOptions>({
    name: 'fontSize',
    addOptions() { return { types: ['textStyle'] }; },
    addGlobalAttributes() {
        return [{
            types: this.options.types,
            attributes: {
                fontSize: {
                    default: null,
                    parseHTML: (element: HTMLElement) => element.style.fontSize?.replace(/['"]+/g, ''),
                    renderHTML: attributes => {
                        if (!attributes.fontSize) return {};
                        return { style: `font-size: ${attributes.fontSize}` };
                    },
                },
            },
        }];
    },
    addCommands() {
        return {
            setFontSize: (fontSize: string) => ({ chain }) => chain().setMark('textStyle', { fontSize }).run(),
            unsetFontSize: () => ({ chain }) => chain().setMark('textStyle', { fontSize: null }).removeEmptyTextStyle().run(),
        };
    },
});

const props = withDefaults(defineProps<{
    modelValue?: string;
    json?: JSONContent | null;
    placeholder?: string;
    accentColor?: string;
    limit?: number;
}>(), {
    modelValue: '',
    json: null,
    placeholder: "Write something amazing...",
    accentColor: "#3b82f6",
    limit: 0
});

const emit = defineEmits<{
    (e: 'update:modelValue', value: string): void;
    (e: 'update:json', value: JSONContent): void;
}>();

const activePopover = ref<'link' | 'youtube' | 'image' | null>(null);
const popoverInput = ref('');
const popoverInputEl = ref<HTMLInputElement | null>(null);

const editor = useEditor({
    editorProps: {
        attributes: {
            class: "prose prose-slate dark:prose-invert max-w-none min-h-[450px] w-full outline-none bg-transparent p-8 md:p-12 transition-all selection:bg-blue-100 dark:selection:bg-blue-900",
        },
    },
    content: props.json || props.modelValue,
    onUpdate: ({ editor }) => {
        emit("update:modelValue", editor.getHTML());
        emit("update:json", editor.getJSON());
    },
    extensions: [
        StarterKit.configure({
            codeBlock: { HTMLAttributes: { class: 'rounded-lg bg-slate-900 text-slate-100 p-4 font-mono text-sm' } },
        }),
        Underline, Subscript, Superscript, Typography, FontSize,
        TextAlign.configure({ types: ["heading", "paragraph"] }),
        Highlight.configure({ multicolor: true }),
        TextStyle, Color,
        Link.configure({
            openOnClick: false,
            defaultProtocol: "https",
            HTMLAttributes: { class: 'text-purple-600 underline cursor-pointer font-medium hover:text-purple-800' },
        }),
        Image.configure({
            HTMLAttributes: { class: 'rounded-xl shadow-lg border-4 border-white inline-block max-w-full h-auto mx-auto' },
        }),
        Youtube.configure({
            width: 840,
            HTMLAttributes: { class: 'aspect-video rounded-xl shadow-2xl mx-auto my-8 border-4 border-white' },
        }),
        Table.configure({ resizable: true }),
        TableRow, TableHeader, TableCell, TaskList,
        TaskItem.configure({ nested: true }),
        HorizontalRule.configure({
            HTMLAttributes: { class: 'my-8 border-t-2 border-slate-200 rounded-full' },
        }),
        Placeholder.configure({ placeholder: props.placeholder }),
        CharacterCount.configure({ limit: props.limit || null }),
    ],
});

watch(() => props.modelValue, (value) => {
    if (editor.value && editor.value.getHTML() !== value) {
        editor.value.commands.setContent(value || '', false);
    }
});

watch(() => props.json, (value) => {
    if (editor.value && JSON.stringify(editor.value.getJSON()) !== JSON.stringify(value)) {
        editor.value.commands.setContent(value || '', false);
    }
}, { deep: true });

onBeforeUnmount(() => editor.value?.destroy());

const charCount = computed(() => editor.value?.storage.characterCount.characters() || 0);
const wordCount = computed(() => editor.value?.storage.characterCount.words() || 0);
const percentage = computed(() => {
    if (!props.limit) return 0;
    return Math.min(100, Math.round((100 / props.limit) * charCount.value));
});

const glassColor = computed(() => props.accentColor + '10');
const highlightColor = computed(() => props.accentColor + '30');

// Unified Popover System
async function openPopover(type: 'link' | 'youtube' | 'image') {
    activePopover.value = type;
    if (type === 'link') {
        popoverInput.value = editor.value?.getAttributes("link").href || '';
    } else {
        popoverInput.value = '';
    }
    await nextTick();
    popoverInputEl.value?.focus();
}

function confirmPopover() {
    if (!editor.value) return;
    const val = popoverInput.value.trim();
    if (activePopover.value === 'link') {
        if (val === "") {
            editor.value.chain().focus().extendMarkRange("link").unsetLink().run();
        } else {
            editor.value.chain().focus().extendMarkRange("link").setLink({ href: val }).run();
        }
    } else if (activePopover.value === 'image' && val) {
        editor.value.chain().focus().setImage({ src: val }).run();
    } else if (activePopover.value === 'youtube' && val) {
        editor.value.commands.setYoutubeVideo({ src: val });
    }
    closePopover();
}

function closePopover() {
    activePopover.value = null;
    popoverInput.value = '';
    editor.value?.chain().focus().run();
}

function changeFontSize(delta: number) {
    if (!editor.value) return;
    const currentSize = editor.value.getAttributes('textStyle').fontSize || '16px';
    const size = parseInt(currentSize);
    if (isNaN(size)) return;
    const newSize = Math.max(8, Math.min(100, size + delta));
    editor.value.chain().focus().setFontSize(`${newSize}px`).run();
}
</script>

<template>
    <div 
        class="tiptap-container" 
        :style="{ '--accent': accentColor, '--accent-glass': glassColor, '--accent-highlight': highlightColor }"
    >
        <!-- Main Toolbar -->
        <div v-if="editor" class="main-toolbar">
            <div class="toolbar-fade-left"></div>
            <div class="toolbar-inner">
                <!-- Group: Text Basics -->
                <div class="group">
                    <button @click="editor.chain().focus().toggleBold().run()" :class="{ 'active': editor.isActive('bold') }" title="Bold">
                        <font-awesome-icon icon="bold" />
                    </button>
                    <button @click="editor.chain().focus().toggleItalic().run()" :class="{ 'active': editor.isActive('italic') }" title="Italic">
                        <font-awesome-icon icon="italic" />
                    </button>
                    <button @click="editor.chain().focus().toggleUnderline().run()" :class="{ 'active': editor.isActive('underline') }" title="Underline">
                        <font-awesome-icon icon="underline" />
                    </button>
                    <button @click="editor.chain().focus().toggleStrike().run()" :class="{ 'active': editor.isActive('strike') }" title="Strikethrough">
                        <font-awesome-icon icon="strikethrough" />
                    </button>
                    <button @click="editor.chain().focus().unsetAllMarks().run()" title="Clear Formatting">
                        <font-awesome-icon icon="eraser" />
                    </button>
                </div>

                <div class="divider"></div>

                <!-- Group: Font Size -->
                <div class="group">
                    <button @click="changeFontSize(-2)" title="Decrease Font Size">
                        <font-awesome-icon icon="minus" />
                    </button>
                    <span class="size-display">{{ editor.getAttributes('textStyle').fontSize || '16px' }}</span>
                    <button @click="changeFontSize(2)" title="Increase Font Size">
                        <font-awesome-icon icon="plus" />
                    </button>
                    <button @click="editor.chain().focus().unsetFontSize().run()" title="Reset Font Size">
                         <font-awesome-icon icon="minus" class="rotate-90 scale-x-150" />
                    </button>
                </div>

                <div class="divider"></div>

                <!-- Group: Lists & Blocks -->
                <div class="group">
                    <button @click="editor.chain().focus().toggleBulletList().run()" :class="{ 'active': editor.isActive('bulletList') }" title="Bullets">
                        <font-awesome-icon icon="list-ul" />
                    </button>
                    <button @click="editor.chain().focus().toggleOrderedList().run()" :class="{ 'active': editor.isActive('orderedList') }" title="Numbers">
                         <font-awesome-icon icon="list-ol" />
                    </button>
                    <button @click="editor.chain().focus().toggleTaskList().run()" :class="{ 'active': editor.isActive('taskList') }" title="Tasks">
                        <font-awesome-icon icon="check-square" />
                    </button>
                </div>

                <div class="divider"></div>

                <!-- Group: Alignment -->
                <div class="group">
                    <button @click="editor.chain().focus().setTextAlign('left').run()" :class="{ 'active': editor.isActive({ textAlign: 'left' }) }" title="Align Left">
                        <font-awesome-icon icon="align-left" />
                    </button>
                    <button @click="editor.chain().focus().setTextAlign('center').run()" :class="{ 'active': editor.isActive({ textAlign: 'center' }) }" title="Align Center">
                        <font-awesome-icon icon="align-center" />
                    </button>
                    <button @click="editor.chain().focus().setTextAlign('right').run()" :class="{ 'active': editor.isActive({ textAlign: 'right' }) }" title="Align Right">
                        <font-awesome-icon icon="align-right" />
                    </button>
                    <button @click="editor.chain().focus().setTextAlign('justify').run()" :class="{ 'active': editor.isActive({ textAlign: 'justify' }) }" title="Justify">
                        <font-awesome-icon icon="align-justify" />
                    </button>
                </div>

                <div class="divider"></div>

                <!-- Group: Table Management -->
                <div class="group">
                    <button @click="editor.chain().focus().insertTable({ rows: 3, cols: 3, withHeaderRow: true }).run()" title="Insert Table">
                        <font-awesome-icon icon="table" />
                    </button>
                    <template v-if="editor.isActive('table')">
                        <button @click="editor.chain().focus().addColumnAfter().run()" title="Add Column"><font-awesome-icon icon="columns" class="mr-1" /><font-awesome-icon icon="plus" class="text-[8px]" /></button>
                        <button @click="editor.chain().focus().addRowAfter().run()" title="Add Row"><font-awesome-icon icon="grip-lines" class="mr-1" /><font-awesome-icon icon="plus" class="text-[8px]" /></button>
                        <button @click="editor.chain().focus().deleteColumn().run()" title="Delete Column" class="hover:!text-orange-500"><font-awesome-icon icon="columns" class="mr-1" /><font-awesome-icon icon="minus" class="text-[8px]" /></button>
                        <button @click="editor.chain().focus().deleteRow().run()" title="Delete Row" class="hover:!text-orange-500"><font-awesome-icon icon="grip-lines" class="mr-1" /><font-awesome-icon icon="minus" class="text-[8px]" /></button>
                        <button @click="editor.chain().focus().deleteTable().run()" class="hover:!text-red-500" title="Delete Table"><font-awesome-icon icon="trash-alt" /></button>
                    </template>
                </div>

                <div class="divider"></div>

                <!-- Group: Media (Using Popovers) -->
                <div class="group">
                    <button @click="openPopover('link')" :class="{ 'active': editor.isActive('link') }" title="Insert Link">
                        <font-awesome-icon icon="link" />
                    </button>
                    <button @click="openPopover('image')" title="Insert Image">
                        <font-awesome-icon icon="image" />
                    </button>
                    <button @click="openPopover('youtube')" title="Embed Youtube">
                        <font-awesome-icon icon="video" />
                    </button>
                    <div class="color-picker-btn">
                        <button title="Text Color" @click.prevent><font-awesome-icon icon="palette" /></button>
                        <input 
                            type="color" 
                            @input="(e: Event) => editor.chain().focus().setColor((e.target as HTMLInputElement).value).run()"
                            :value="editor.getAttributes('textStyle').color || '#000000'"
                        >
                    </div>
                </div>

                <div class="spacer"></div>

                <div class="group">
                    <button @click="editor.chain().focus().undo().run()" :disabled="!editor.can().undo()" title="Undo">
                        <font-awesome-icon icon="undo" />
                    </button>
                    <button @click="editor.chain().focus().redo().run()" :disabled="!editor.can().redo()" title="Redo">
                        <font-awesome-icon icon="redo" />
                    </button>
                </div>
            </div>
            <div class="toolbar-fade-right"></div>
        </div>

        <!-- Unified Popover UI -->
        <transition name="popover">
            <div v-if="activePopover" class="popover-overlay">
                <div class="popover-card">
                    <div class="popover-header">
                        <span class="text-sm font-bold uppercase tracking-wider text-slate-500">
                            {{ activePopover === 'link' ? 'Hyperlink' : activePopover === 'image' ? 'Image URL' : 'YouTube Link' }}
                        </span>
                        <button @click="closePopover" class="close-popover">&times;</button>
                    </div>
                    <div class="popover-body">
                        <input 
                            ref="popoverInputEl"
                            v-model="popoverInput" 
                            :placeholder="activePopover === 'link' ? 'https://example.com' : 'Paste URL here...'"
                            @keyup.enter="confirmPopover"
                        />
                        <div class="popover-actions">
                            <button @click="closePopover" class="btn-cancel">Cancel</button>
                            <button @click="confirmPopover" class="btn-confirm" :style="{ background: accentColor }">Apply Change</button>
                        </div>
                    </div>
                </div>
            </div>
        </transition>

        <!-- Floating Menu (Visible on empty lines) -->
        <FloatingMenu v-if="editor" :editor="editor" :tippy-options="{ duration: 100 }" class="floating-menu">
            <button @click="editor.chain().focus().toggleHeading({ level: 1 }).run()" :class="{ 'active': editor.isActive('heading', { level: 1 }) }">
                H1
            </button>
            <button @click="editor.chain().focus().toggleHeading({ level: 2 }).run()" :class="{ 'active': editor.isActive('heading', { level: 2 }) }">
                H2
            </button>
            <button @click="editor.chain().focus().toggleBulletList().run()" :class="{ 'active': editor.isActive('bulletList') }">
                <font-awesome-icon icon="list-ul" />
            </button>
            <button @click="openPopover('image')">
                <font-awesome-icon icon="image" />
            </button>
        </FloatingMenu>

        <!-- Bubble Menu (Visible on text selection) -->
        <BubbleMenu v-if="editor" :editor="editor" :tippy-options="{ duration: 100 }" class="bubble-menu">
            <button @click="editor.chain().focus().toggleBold().run()" :class="{ 'active': editor.isActive('bold') }">
                <font-awesome-icon icon="bold" />
            </button>
            <button @click="editor.chain().focus().toggleItalic().run()" :class="{ 'active': editor.isActive('italic') }">
                <font-awesome-icon icon="italic" />
            </button>
            <button @click="editor.chain().focus().toggleCode().run()" :class="{ 'active': editor.isActive('code') }">
                <font-awesome-icon icon="code" />
            </button>
            <button @click="editor.chain().focus().toggleHighlight({ color: highlightColor }).run()" :class="{ 'active': editor.isActive('highlight') }">
                 <font-awesome-icon icon="highlighter" />
            </button>
            <div class="bubble-divider"></div>
            <button @click="openPopover('link')" :class="{ 'active': editor.isActive('link') }">
                <font-awesome-icon icon="link" />
            </button>
             <button @click="editor.chain().focus().unsetAllMarks().run()" title="Clear Marks">
                <font-awesome-icon icon="eraser" />
            </button>
        </BubbleMenu>

        <!-- Editor Content -->
        <div class="editor-viewport" @click="activePopover = null">
            <EditorContent :editor="editor" />
        </div>

        <!-- Stats Footer -->
        <div class="editor-footer" v-if="(limit ?? 0) > 0 || wordCount > 0">
            <div class="footer-info">
                <span class="pill">{{ wordCount }} words</span>
                <span class="pill">{{ charCount }}<span v-if="(limit ?? 0) > 0"> / {{ limit }}</span> chars</span>
            </div>
            <div v-if="(limit ?? 0) > 0" class="progress-wrapper">
                <div class="progress-track"><div class="progress-fill" :style="{ width: percentage + '%', background: percentage > 90 ? '#ef4444' : 'var(--accent)' }"></div></div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.tiptap-container {
    --bg-card: #ffffff;
    --bg-editor: #ffffff;
    --border-ui: #e5e7eb;
    --text-main: #1f2937;
    --text-dim: #6b7280;
    --toolbar-bg: rgba(255, 255, 255, 0.95);
    --group-bg: #f3f4f6;
    --popover-bg: rgba(255, 255, 255, 0.9);
    --footer-bg: #f9fafb;
    
    position: relative;
    background: var(--bg-card);
    border: 1px solid var(--border-ui);
    border-radius: 1.25rem;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.05);
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@media (prefers-color-scheme: dark) {
    .tiptap-container {
        --bg-card: #111827;
        --bg-editor: #111827;
        --border-ui: #374151;
        --text-main: #f3f4f6;
        --text-dim: #9ca3af;
        --toolbar-bg: rgba(17, 24, 39, 0.95);
        --group-bg: #1f2937;
        --popover-bg: rgba(31, 41, 55, 0.9);
        --footer-bg: #1f2937;
    }
}

.dark .tiptap-container {
    --bg-card: #111827;
    --bg-editor: #111827;
    --border-ui: #374151;
    --text-main: #f3f4f6;
    --text-dim: #9ca3af;
    --toolbar-bg: rgba(17, 24, 39, 0.95);
    --group-bg: #1f2937;
    --popover-bg: rgba(31, 41, 55, 0.9);
    --footer-bg: #1f2937;
}

.tiptap-container:focus-within {
    border-color: var(--accent);
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.2);
}

/* Toolbar Styling */
.main-toolbar {
    background: var(--toolbar-bg);
    backdrop-filter: blur(12px);
    border-bottom: 1px solid var(--border-ui);
    padding: 0.75rem;
    position: sticky;
    top: 0;
    z-index: 50;
    display: flex;
    align-items: center;
}

.toolbar-fade-left { left: 0; background: linear-gradient(to right, var(--bg-card), transparent); }
.toolbar-fade-right { right: 0; background: linear-gradient(to left, var(--bg-card), transparent); }
.toolbar-fade-left, .toolbar-fade-right { position: absolute; top: 0; bottom: 0; width: 1.5rem; z-index: 60; pointer-events: none; }

.toolbar-inner {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    overflow-x: auto;
    scrollbar-width: none;
    padding: 0 0.5rem;
}
.toolbar-inner::-webkit-scrollbar { display: none; }

.group {
    display: flex; gap: 2px; background: var(--group-bg); padding: 3px; border-radius: 0.75rem; align-items: center;
}

.divider { width: 1px; height: 1.5rem; background: var(--border-ui); margin: 0 0.1rem; }
.spacer { flex: 1; }

button {
    width: 2.1rem; height: 2.1rem; display: flex; align-items: center; justify-content: center;
    border: none; background: transparent; color: var(--text-main); border-radius: 0.5rem; cursor: pointer;
    font-size: 0.85rem; font-weight: 700; transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

button:hover:not(:disabled) {
    background: var(--bg-card); color: var(--accent); box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}
button:active:not(:disabled) { transform: scale(0.92); }
button.active { background: var(--accent-highlight); color: var(--accent) !important; }
button:disabled { opacity: 0.3; cursor: not-allowed; }

.size-display { padding: 0 0.4rem; font-size: 0.7rem; font-weight: 800; color: var(--text-dim); min-width: 2rem; text-align: center; }

/* Popover Overlay */
.popover-overlay {
    position: absolute;
    top: 4.5rem; left: 1rem; right: 1rem;
    z-index: 100;
    display: flex;
    justify-content: center;
}

.popover-card {
    background: var(--popover-bg);
    backdrop-filter: blur(20px);
    border: 1px solid var(--border-ui);
    border-radius: 1rem;
    box-shadow: 0 20px 40px -10px rgba(0,0,0,0.4);
    width: 100%;
    max-width: 400px;
    overflow: hidden;
    animation: slideDown 0.3s ease-out;
}

@keyframes slideDown { from { transform: translateY(-10px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }

.popover-header {
    display: flex; justify-content: space-between; align-items: center; padding: 0.75rem 1rem; background: rgba(0,0,0,0.05); border-bottom: 1px solid var(--border-ui);
}

.close-popover { font-size: 1.5rem; color: #94a3b8; width: 1.5rem; height: 1.5rem; display: flex; align-items: center; justify-content: center; border-radius: 50%; }
.close-popover:hover { background: #f1f5f9; color: #ef4444; }

.popover-body { padding: 1rem; display: flex; flex-direction: column; gap: 0.75rem; }

.popover-body input {
    width: 100%; padding: 0.6rem 0.8rem; border: 2px solid var(--border-ui); background: var(--group-bg); color: var(--text-main); border-radius: 0.6rem; outline: none; transition: all 0.2s; font-size: 0.9rem;
}

.popover-body input:focus { border-color: var(--accent); box-shadow: 0 0 0 3px var(--accent-highlight); }

.popover-actions { display: flex; gap: 0.5rem; justify-content: flex-end; margin-top: 0.5rem; }
.btn-cancel, .btn-confirm { 
    width: auto !important; 
    height: auto !important; 
    padding: 0.6rem 1.25rem !important; 
    border-radius: 0.75rem;
    font-size: 0.85rem;
    font-weight: 700;
}
.btn-cancel { color: var(--text-dim); background: var(--group-bg); }
.btn-cancel:hover { background: var(--border-ui); color: var(--text-main); }
.btn-confirm { color: white; box-shadow: 0 4px 12px var(--accent-highlight); }
.btn-confirm:hover { filter: brightness(1.1); transform: translateY(-1px); }

/* Transitions */
.popover-enter-active, .popover-leave-active { transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1); }
.popover-enter-from, .popover-leave-to { opacity: 0; transform: translateY(-10px) scale(0.95); }

/* Menus */
.bubble-menu, .floating-menu { display: flex; background: #1f2937; padding: 0.4rem; border-radius: 0.8rem; gap: 0.25rem; z-index: 100; box-shadow: 0 10px 25px rgba(0,0,0,0.2); }
.bubble-menu button, .floating-menu button { color: #f3f4f6; height: 1.75rem; width: 1.75rem; font-size: 0.75rem; }
.bubble-menu button:hover, .floating-menu button:hover { background: rgba(255,255,255,0.1); }
.bubble-divider { width: 1px; background: rgba(255,255,255,0.1); margin: 0 0.25rem; }

/* Editor Adjustments */
.editor-viewport { background: var(--bg-editor); flex: 1; overflow-y: auto; }

:deep(.ProseMirror) {
    outline: none !important;
    color: var(--text-main);
}

/* Bullet Fixes */
:deep(.ProseMirror ul) {
    list-style-type: disc !important;
    padding-left: 2rem !important;
    margin: 1.25rem 0 !important;
}

:deep(.ProseMirror ol) {
    list-style-type: decimal !important;
    padding-left: 2rem !important;
    margin: 1.25rem 0 !important;
}

:deep(.ProseMirror li) {
    margin-bottom: 0.5rem;
    display: list-item !important; /* Ensure they behave like bullets */
}

/* Table Styling */
:deep(table) { border-collapse: collapse; table-layout: fixed; width: 100%; margin: 2rem 0; border: 1px solid #e5e7eb; border-radius: 0.5rem; }
:deep(table td), :deep(table th) { min-width: 1em; border: 1px solid var(--border-ui); padding: 12px; vertical-align: top; position: relative; }
:deep(table th) { font-weight: bold; background-color: var(--footer-bg); }
:deep(table .selectedCell:after) { z-index: 2; position: absolute; content: ""; inset: 0; background: var(--accent-highlight); pointer-events: none; }

/* Task Lists */
:deep(ul[data-type="taskList"]) { list-style: none; padding: 0; }
:deep(ul[data-type="taskList"] li) { display: flex; align-items: flex-start; gap: 0.75rem; margin-bottom: 0.75rem; }
:deep(ul[data-type="taskList"] input[type="checkbox"]) { margin-top: 0.4rem; cursor: pointer; accent-color: var(--accent); width: 1.25rem; height: 1.25rem; }

/* Color Picker */
.color-picker-btn { position: relative; overflow: hidden; }
.color-picker-btn input { position: absolute; opacity: 0; inset: 0; cursor: pointer; }

/* Footer */
.editor-footer { padding: 0.75rem 1.25rem; background: var(--footer-bg); border-top: 1px solid var(--border-ui); display: flex; flex-direction: column; gap: 0.5rem; }
.footer-info { display: flex; justify-content: space-between; }
.pill { font-size: 0.75rem; font-weight: 600; color: var(--text-dim); background: var(--group-bg); padding: 2px 8px; border-radius: 99px; }
.progress-wrapper { width: 100%; height: 4px; background: var(--group-bg); border-radius: 99px; overflow: hidden; }
.progress-fill { height: 100%; transition: width 0.3s ease; }

@media (max-width: 640px) {
    .tiptap-container { border-radius: 0; border: none; }
    .main-toolbar { padding: 0.4rem; }
}
</style>
