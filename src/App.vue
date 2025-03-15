<script setup>
import { ref } from "vue";
import CalculatorForm from "./components/CalculatorForm.vue";
import CalendarView from "./components/CalendarView.vue";

const calculatorData = ref({
  principal: 100,
  rate: 5,
  compoundPeriod: "daily",
});
</script>

<template>
  <div class="container">
    <h1>复利计算器</h1>
    <div class="calculator-wrapper">
      <CalculatorForm v-model="calculatorData" />
      <CalendarView :calculatorData="calculatorData" />
    </div>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Arial", sans-serif;
  line-height: 1.6;
  color: #333;
  background-color: #f9f9f9;
}

.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}
.calculator-wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
h1 {
  text-align: center;
  margin-bottom: 30px;
  color: #42b883;
}
@media (max-width: 768px) {
  .container {
    padding: 10px;
  }

  .calculator-wrapper {
    gap: 15px;
  }
}

@media (max-width: 480px) {
  .container {
    padding: 5px;
  }

  h1 {
    margin-bottom: 20px;
  }
}
</style>
