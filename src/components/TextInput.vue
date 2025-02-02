<template>
  <div class="w-full flex items-center">
    <input
      :value="modelValue"
      :disabled="disabled"
      class="w-full text-black border bg-transparent px-4 outline-hidden transition-all h-10 text-sm border-gray-400 disabled:border-gray-400 disabled:text-gray-500 hover:border-black"
      :class="{ 'rounded-l-full': buttonText, 'rounded-full': !buttonText }"
      @input="$emit('update:modelValue', $event.target.value)"
    />
    <button
      v-if="buttonText"
      class="bg-blue-500 hover:bg-blue-700 text-white font-medium py-2 px-4 rounded-r-full disabled:bg-gray-400"
      :disabled="disabled"
      @click="$emit('button-click')"
    >
      {{ buttonText }}
    </button>
  </div>
</template>

<script setup>
const { buttonText, modelValue } = defineProps({
  modelValue: {
    type: String,
    default: "",
  },
  buttonText: {
    type: String,
    default: "",
  },
  disabled: {
    type: Boolean,
    default: false,
  },
});
</script>
