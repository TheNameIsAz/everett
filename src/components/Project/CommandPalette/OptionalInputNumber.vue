<template lang="pug">
.optional-input-number
  .el-input-group
    .el-input-group__prepend
      el-switch(v-model="input[0]")
    el-input-number(
      v-model="input[1]",
      size="small",
      :disabled="!input[0]",
      controls-position="right",
    )
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Prop, Watch } from 'vue-property-decorator'
import { store } from '@/store'

@Component
export default class OptionalInputNumber extends Vue {
  public input: [boolean, number | null] = [false, null]

  @Prop()
  public value!: [boolean, number]

  public get files () {
    return store.state.files
  }

  public created () {
    this.input = this.value
  }

  @Watch('input')
  public updateInput () {
    this.$emit('input', this.input)
  }
}
</script>

<style lang="sass">
.optional-input-number
  .el-input-number
    width: 214px
  .el-input-group
    .el-input-group__prepend + .el-input-number /deep/ .el-input__inner
      border-top-left-radius: 0
      border-bottom-left-radius: 0
</style>
