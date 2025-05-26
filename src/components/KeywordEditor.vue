<template>
<div ref="editorContainer" class="editor-container"></div>
</template>
<script setup>
import { onMounted, ref, onBeforeUnmount } from 'vue'
import { EditorState, Plugin } from 'prosemirror-state'
import { EditorView, Decoration, DecorationSet } from 'prosemirror-view'
import { Schema } from 'prosemirror-model'
import { schema as basicSchema } from 'prosemirror-schema-basic'
import { keymap } from 'prosemirror-keymap'
import { baseKeymap } from 'prosemirror-commands'

const editorContainer = ref()
let view = null

function keywordHighlighter(state) {
  const decorations = []
  const keywords = [
    { word: '건강', class: 'highlight-yellow' },
    { word: '주택', class: 'highlight-red' },
  ]

  state.doc.descendants((node, pos) => {
    if (!node.isText) return true
    const text = node.text
    keywords.forEach(({ word, class: className }) => {
      let fromIndex = 0
      let index = text.indexOf(word, fromIndex)
      while (index !== -1) {
        decorations.push(
          Decoration.inline(pos + index, pos + index + word.length, { class: className })
        )
        fromIndex = index + word.length
        index = text.indexOf(word, fromIndex)
      }
    })
  })

  return DecorationSet.create(state.doc, decorations)
}

onMounted(() => {
  const schema = new Schema({
    nodes: basicSchema.spec.nodes,
    marks: basicSchema.spec.marks,
  })

  const keywordHighlightPlugin = new Plugin({
    props: {
      decorations(state) {
        return keywordHighlighter(state)
      }
    }
  })

  const state = EditorState.create({
    schema,
    plugins: [keymap(baseKeymap), keywordHighlightPlugin],
  })

  view = new EditorView(editorContainer.value, {
    state,
  })
})

onBeforeUnmount(() => {
  if (view) view.destroy()
})

</script>

<style >
.ProseMirror {
  border: 1px solid #ccc;
  padding: 10px;
  width: 100%;
  min-height: 150px;
  font-size: 16px;
  text-align: left;
  line-height: 1.4em;
}

.ProseMirror p {
  margin: 0;
}

.highlight-yellow {
  background-color: yellow;
}

.highlight-red {
  color: red;
  font-weight: bold;
}
</style>