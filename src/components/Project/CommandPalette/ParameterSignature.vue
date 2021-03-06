<template lang="pug">
span.parameter-signature
  | &ensp;
  template(v-if="signature.type === 'optional'") (
  template(v-if="signature.key")
    span.attribute {{ signature.key }}
    template(v-if="!signature.value || signature.value[0] !== 'true'")
      span.operator :
      | &nbsp;
  template(v-if="type(signature.type) === 'select'")
    select-value(:value="signature.value")
  template(v-else-if="type(signature.type) === 'or'")
    span.value
      | {&nbsp;
      template(v-for="({ key, values}, index) in signature.values")
        template(v-if="index > 0")
          | &nbsp;|&ensp;
        template(v-if="key")
          span.attribute {{ key }}
          span.operator :
          | &nbsp;
        list-value(:values="values")
      | &nbsp;}
  template(v-else-if="type(signature.type) === 'list'")
    list-value(:values="signature.values")
  template(v-else-if="type(signature.type) === 'spriteset'")
    span.value
      value(:value="signature.value[0]") {{ signature | toValue }}
    | :&nbsp;
    span.value
      value(value="number") number
  template(v-else)
    span.value
      value(:value="signature.value[0]") {{ signature | toValue }}
  template(v-if="signature.type === 'optional'") )
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Prop } from 'vue-property-decorator'
import { IParameterSignature } from '@/commands/definitions'
import ListValue from './ListValue.vue'
import SelectValue from './SelectValue.vue'
import Value from './Value.vue'

@Component({
  components: {
    ListValue,
    SelectValue,
    Value,
  },
  filters: {
    toValue (signature: IParameterSignature) {
      switch (signature.type) {
        case 'optional':
          if (signature.value[0] === 'true') {
            return
          }
          return signature.value[0]
        case 'filename':
          return `"${signature.value[1].name} file"`
        case 'spriteset':
          return `"${signature.value[1].name} file"`
        default:
          return signature.value[0]
      }
    },
  },
})
export default class ParameterSignature extends Vue {
  @Prop()
  public signature!: IParameterSignature

  public type (type: string) {
    switch (type) {
      case 'or':
      case 'select':
      case 'optional':
      case 'spriteset':
        return type
      case 'color':
      case 'tone':
      case 'vector':
        return 'list'
      default:
        return 'other'
    }
  }
}
</script>

<style lang="sass", scoped>
.parameter-signature
  .operator
    color: #3f51b5
  .attribute
    color: #2196f3
</style>
