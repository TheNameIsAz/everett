<template lang="pug">
.editor
  .editor-input-container
    monaco-editor.editor-input(
      ref="monaco",
      v-model="source",
      @change="sourceChanged"
      :options="monacoOptions",
      :theme="monacoEditorTheme",
      :language="monacoEditorLanguage",
    )
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Watch } from 'vue-property-decorator'
import { store } from '@/store'
import EventBus from '@/event-bus'
import MonacoEditor from 'vue-monaco'
import * as monacoEditor from 'monaco-editor'
import { cursorTo } from 'readline'

type MonacoOptions = monacoEditor.editor.IEditorConstructionOptions | { tabSize: number }

@Component({
  components: {
    MonacoEditor,
  },
})
export default class Editor extends Vue {
  public source = ''
  public sharedState = store.state
  public editor!: monacoEditor.editor.ICodeEditor
  public monacoEditorLanguage = 'everett'
  public monacoEditorTheme = 'everett'

  public get monacoOptions (): MonacoOptions {
    return {
      minimap: { enabled: false },
      fontSize: 15,
      tabSize: 2,
      automaticLayout: true,
      wrappingIndent: 'same',
    }
  }

  public created () {
    EventBus.$on('insert-command-clicked', this.insertCommand)
    EventBus.$on('source-read', this.setSource)
    EventBus.$on('side-panel-changed', this.relayoutEditor)
  }

  public beforeDestroy () {
    EventBus.$off('insert-command-clicked', this.insertCommand)
    EventBus.$off('source-read', this.setSource)
    EventBus.$off('side-panel-changed', this.relayoutEditor)
  }

  public mounted () {
    this.editor = (this.$refs.monaco as MonacoEditor.IMonacoEditorCompornent).getEditor()
  }

  public relayoutEditor () {
    this.editor.layout()
  }

  public setSource (value: string) {
    const model = this.editor.getModel()
    if (!model) {
      return
    }
    const lineCount = model.getLineCount()
    const lastLineLength = model.getLineMaxColumn(lineCount)
    const range = new monacoEditor.Range(1, 1, lineCount, lastLineLength)
    const text = value
    this.editor.executeEdits('', [{ range, text, forceMoveMarkers: true }])
  }

  public sourceChanged (value: string, event: monacoEditor.editor.IModelContentChangedEvent) {
    store.updateSource(value)
    this.sharedState.edited = true
  }

  public insertCommand (command: string) {
    const selection = this.editor.getSelection()
    const position = selection ? selection.getStartPosition() : this.editor.getPosition()
    const model = this.editor.getModel()
    if (!position || !model) {
      return
    }
    const range = new monacoEditor.Range(position.lineNumber, 1, position.lineNumber, 1)
    const text = `${command}\n`
    this.editor.executeEdits('commandPallet', [{ range, text, forceMoveMarkers: true }])
    this.ensureCursorVisible(position)
  }

  private ensureCursorVisible (position: monacoEditor.IPosition) {
    const [visibleRange] = this.editor.getVisibleRanges()
    const startLineNumber = visibleRange.getStartPosition()!.lineNumber
    const endLineNumber = visibleRange.getEndPosition()!.lineNumber
    if (endLineNumber <= position.lineNumber) {
      const numberOfVisibleLines = endLineNumber - startLineNumber - 1
      const top = this.editor.getTopForPosition(position.lineNumber - (numberOfVisibleLines), 1)
      this.editor.setScrollTop(top)
    }
  }
}
</script>

<style lang="sass">
.editor-input
  height: 100%
</style>
<style lang="sass" scoped>

.editor
  flex-grow: 1
  height: 100%
  position: relative
  .editor-input-container
    position: absolute
    top: 0
    bottom: 0
    left: 0
    right: 0
</style>
