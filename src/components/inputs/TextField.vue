<template>
  <v-text-field
    v-bind="$attrs"
    v-on="$listeners"
  ></v-text-field>
</template>
    <!-- <template
      v-if="defaultValue"
      v-slot:append
    >
      <v-icon @click="handleReset">$refresh</v-icon>
    </template> -->
<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'
import { VTextField } from 'vuetify/lib'

@Component({
  extends: VTextField
})
export default class FluiddTextField extends VTextField {
  @Prop({ type: [String, Number] })
  defaultValue!: string | number

  // handleReset () {
  //   console.log('resetting to', this.defaultValue)
  //   // this.$emit('input', this.defaultValue)
  //   // this.$emit('change', this.defaultValue)
  // }

  // handleChange (e: Event) {
  //   this.$emit('change', e)
  //   console.log('got change', e)
  // }
}
</script>
