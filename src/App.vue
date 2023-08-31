<template>
  <table class="input">
    <tr>
      <td>
        <label>
          Target Item
        </label>
      </td>
      <td>
        <select v-model="option">
          <option v-for="option in options" :key="option" :value="option">
            {{ option }}
          </option>
        </select>
      </td>
    </tr>
    <tr>
      <td>
        <label>
          Target Rate
        </label>
      </td>
      <td>
        <input type="number" v-model="rate" /> /s
      </td>
    </tr>
    <tr>
      <td>
        <label>
          Belt Rate
        </label>
      </td>
      <td>
        <input type="number" v-model="beltrate" /> /s
      </td>
    </tr>
  </table>
   <div class="app">
    <Ingredient v-if="option" :entity="option" :rate="rate" />
  </div>
</template>

<script setup lang="ts">
console.clear()
import { ref, computed } from "vue";
import data from "./data.json";
import Ingredient from "./Ingredient.vue";

import { beltrate } from "./Config";

const options = Object.keys(data);
const rate = ref(1);
const option = ref("blue_circuit");
</script>

<style scoped>
.app{
  padding-left: 50px;
}
</style>