<script setup>
import { computed } from 'vue'

const props = defineProps({
    problems: { type: Array, required: true },
    modelValue: { type: Number, required: true },
    disabled: { type: Boolean, default: false }
})

const emit = defineEmits(['update:modelValue'])

const selectedProblem = computed({
    get: () => props.modelValue,
    set: (val) => emit('update:modelValue', val)
})
</script>

<template>
  <div class="problems-container">
    <div class="problem-selector">
      <label 
        v-for="(prob, index) in problems" 
        :key="prob.id" 
        class="radio-card"
        :class="{ 'disabled-card': disabled }"
      >
        <input 
          type="radio" 
          name="problem" 
          :value="index" 
          v-model="selectedProblem" 
          :disabled="disabled"
        />
        <span class="radio-custom"></span>

        <span class="label-text">
          {{ prob.label }}
        </span>
      </label>
    </div>
  </div>
</template>

<style scoped src="../styles/Promblems.css"></style>
