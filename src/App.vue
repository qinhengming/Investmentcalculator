<script setup>
import { ref } from 'vue';
import CalculatorForm from './components/CalculatorForm.vue';
import CalendarView from './components/CalendarView.vue';

const calculatorData = ref({
  principal: 100,
  rate: 5,
  compoundPeriod: 'daily'
});
</script>

<template>
  <div class="container">
    <h1>复利计算器</h1>
    <CalculatorForm v-model="calculatorData" />
    <CalendarView :calculatorData="calculatorData" />
  </div>
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
  color: #333;
  background-color: #f9f9f9;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
  color: #42b883;
}
</style>
