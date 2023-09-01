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
          Target 
        </label>
      </td>
      <td>
        <select v-model="target">
          <option value='machine'> Machine Running fulltime</option>
          <option value='rate'> Rate</option>
        </select>
      </td>
    </tr>
    <tr v-if="target === 'rate'">
      <td>
        <label>
          Target Rate
        </label>
      </td>
      <td>
        <input type="number" v-model="targetRate" /> /s
      </td>
    </tr>
    <tr v-if="target === 'machine'">
      <td>
        <label>
          Target Machine
        </label>
      </td>
      <td>
        <input type="number" v-model="targetMachine" /> machine running full time
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
    <tr>
      <td>
        Treat as Raw: 
      </td>
      <td>
        <label>
          Smelting
          <input type="checkbox" v-model="hideSmelting"/>
        </label>
        <label>
          Fluid
          <input type="checkbox" v-model="hideFluid"/>
        </label>
      </td>
    </tr>
    <tr>
      <td>
        <label>
          Speed Multiplier
        </label>
      </td>
      <td>
        <input type="number" v-model="speedMultiplier" step="0.05" /> (eg. speed module, factory type)
      </td>
    </tr>
    <tr>
      <td>
        <label>
          Productivity Multiplier
        </label>
      </td>
      <td>
        <input type="number" v-model="productivityMultiplier" step="0.05" /> (eg. productivity module)
      </td>
    </tr>
  </table>
   <div class="app">
    <Ingredient v-if="option" :name="option" :targetRate="effectiveTargetRate" />
  </div>
</template>

<script setup lang="ts">
console.clear()
import { ref, computed, watch } from "vue";
import {recipe,kv} from "./recipe.js";
import Ingredient from "./Ingredient.vue";

import { beltrate, hideSmelting, hideFluid, productivityMultiplier, speedMultiplier } from "./Config";

const options = recipe.map(x=>x.name)


const targetRate = ref(1);
const targetMachine = ref(1);
const target = ref('rate')
const option = ref("processing-unit");


const entity = computed(()=>{
  return kv[option.value]
})

const effectiveTargetRate = computed(()=>{
  if(target.value === 'rate'){
    return targetRate.value
  }
  if(target.value === 'machine'){
    const speed = (entity.value.energy_required || 0.5) / speedMultiplier.value
    const output = (entity.value.result_count || 1) / productivityMultiplier.value
    return targetMachine.value / (speed * output)
  }
})
</script>

<style scoped>
.app{
  padding-left: 50px;
}
</style>