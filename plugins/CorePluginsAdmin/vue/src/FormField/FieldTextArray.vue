<!--
  Matomo - free/libre analytics platform
  @link https://matomo.org
  @license http://www.gnu.org/licenses/gpl-3.0.html GPL v3 or later
-->

<template>
  <!-- note: @change is used in case the change event is programmatically triggered -->
  <div>
    <label
      :for="name"
      v-html="$sanitize(title)"
    />
    <input
      :class="`control_${ uiControl }`"
      :type="uiControl"
      :name="name"
      @keydown="onKeydown($event)"
      @change="onKeydown($event)"
      :value="concattedValues"
      v-bind="uiControlAttributes"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, nextTick } from 'vue';
import { debounce } from 'CoreHome';

export default defineComponent({
  props: {
    name: String,
    title: String,
    uiControl: String,
    modelValue: Array,
    uiControlAttributes: Object,
  },
  inheritAttrs: false,
  computed: {
    concattedValues() {
      if (typeof this.modelValue === 'string') {
        return this.modelValue;
      }

      return (this.modelValue || []).join(', ');
    },
  },
  emits: ['update:modelValue'],
  created() {
    // debounce because puppeteer types reeaally fast
    this.onKeydown = debounce(this.onKeydown.bind(this), 50);
  },
  methods: {
    onKeydown(event: Event) {
      const values = (event.target as HTMLInputElement).value.split(',').map((v) => v.trim());
      if (values.join(', ') !== this.concattedValues) {
        this.$emit('update:modelValue', values);

        nextTick(() => {
          if ((event.target as HTMLInputElement).value !== this.concattedValues) {
            // change to previous value if the parent component did not update the model value
            // (done manually because Vue will not notice if a value does NOT change)
            (event.target as HTMLInputElement).value = this.concattedValues;
          }
        });
      }
    },
  },
});
</script>
