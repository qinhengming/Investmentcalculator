<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  modelValue: {
    type: Object,
    required: true
  }
});

const emit = defineEmits(['update:modelValue']);

const principal = ref(props.modelValue.principal || 100);
const rate = ref(props.modelValue.rate || 5);
const compoundPeriod = ref(props.modelValue.compoundPeriod || 'daily');

watch([principal, rate, compoundPeriod], () => {
  emit('update:modelValue', {
    principal: principal.value,
    rate: rate.value,
    compoundPeriod: compoundPeriod.value
  });
});
</script>

<template>
  <div class="calculator-form">
    <h2>复利计算器</h2>
    <div class="form-group">
      <label for="principal">本金 (元)</label>
      <input 
        type="number" 
        id="principal" 
        v-model.number="principal" 
        min="1"
        step="1000"
      />
    </div>
    
    <div class="form-group">
      <label for="rate">收益率 (%)</label>
      <input 
        type="number" 
        id="rate" 
        v-model.number="rate" 
        min="0.01" 
        max="100" 
        step="0.01"
      />
    </div>
    
    <div class="form-group">
      <label for="compoundPeriod">计息周期</label>
      <select id="compoundPeriod" v-model="compoundPeriod">
        <option value="daily">每日</option>
        <option value="monthly">每月</option>
        <option value="yearly">每年</option>
      </select>
    </div>
  </div>
</template>

<style scoped>
.calculator-form {
  background-color: #f5f5f5;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h2 {
  margin-top: 0;
  color: #333;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #555;
}

input, select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

input:focus, select:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.2);
}
</style>