<template>
  <template v-if="entity">
  <table :class="entity.category">
    <tr>
      <td>
        <div class="name">
          {{name}}
        </div>
        <div class="machines">
        <span>{{Math.ceil(machines)}}</span> machines <span class="detail">({{(machines).toFixed(2)}})</span>
        </div>
        <div class="total">
          <span>{{totalOutput.toFixed(0)}}</span> units /s
        </div>
        <div class="rate">
          target: <span>{{Math.ceil(targetRate)}}</span>/s <span class="detail">({{(targetRate).toFixed(2)}})</span>
        </div>
        <div class="speed">
          <span>{{effectiveSpeed.toFixed(2)}}</span>s/unit
        </div>
        <div class="arrow belt" v-if="entityType === 'item'">
          <span>{{Math.ceil((totalOutput/beltrate)*2)/2}}</span> belt  <span class="detail">({{(totalOutput/beltrate).toFixed(2)}})</span>
        </div>
        <div class="arrow pipe" v-if="entityType === 'fluid'">
          pipe
        </div>
        <div class="modifiers">
          <div>
          <label>
            SM
          <input type="number" v-model="thisSpeedMultiplier" step="0.05"/>
          </label>
          </div>
          <div>
            <label>
                PM
              <input type="number" v-model="thisProductivityMultiplier" step="0.05"/>
            </label>
          </div>
        </div>
      </td>
        <td>
          <table class="raw">
            <tr v-for="rawIngredient in raw" :key="rawIngredient.name">
              <td>
                {{rawIngredient.name}} @ {{rawIngredient.targetRate}}/s
                <div class="arrow belt">
                  <span>{{Math.ceil(((rawIngredient.targetRate)/beltrate)*2)/2}}</span> belt
                </div>
              </td>
            </tr>
          </table>
          <Ingredient v-for="child in children" :key="child.name" :name="child.name" :targetRate="child.targetRate" :entityType="child.type"/>
        </td>
      </tr>
    </table>
  </template>
</template>

<script setup>
    import { ref, computed ,watch} from "vue";
    import {kv} from "./recipe.js";
    import Ingredient from "./Ingredient.vue";

import { beltrate, hideSmelting, hideFluid, speedMultiplier, productivityMultiplier } from "./Config";

const props = defineProps({ 
  name: {
    type: String,
    required: true
  },
  targetRate:{
    type: Number,
    required: true
  },
  entityType: {
    type: String,
    default: 'item'
  }
});

const thisSpeedMultiplier = ref(speedMultiplier.value)
const thisProductivityMultiplier = ref(productivityMultiplier.value)

watch(speedMultiplier,(n,o)=>{
  console.log(n,thisSpeedMultiplier.value)
  if(o === thisSpeedMultiplier.value){
    thisSpeedMultiplier.value = n
  }
})
watch(productivityMultiplier,(n,o)=>{
  console.log(n,thisProductivityMultiplier.value)
  if(o === thisProductivityMultiplier.value){
    thisProductivityMultiplier.value = n
  }
})


  const name = computed(()=>props.name);
  const entity = computed(()=>{
    return kv[props.name]
  });

  const ingredients = computed(()=>{
    return (entity.value.ingredients||[]).map(x=>({
    ...x,
    entity: kv[x.name],
    targetRate: totalOutput.value * (x.quantity || x.amount)/thisProductivityMultiplier.value
  }))
  })

  const isRaw = (ingredient)=>{
    if(hideFluid.value && ingredient.type === 'fluid'){
      return true;
    }
    if(!kv[ingredient.name]){
      return true;
    }
    if(hideSmelting.value && kv[ingredient.name].category === 'smelting'){
      return true;
    }
    return false;
  }

  const targetRate = computed(()=>props.targetRate)
  const children = computed(()=>ingredients.value.filter(x=>!isRaw(x)))
  const raw = computed(()=>ingredients.value.filter(x=>isRaw(x)))


  const speed = computed(()=>{
    const base_speed = entity.value.energy_required || 0.5;
    const modified_speed = base_speed / thisSpeedMultiplier.value;
    return modified_speed;
  })

  const output = computed(()=>{
    const base_output = entity.value.result_count || 1;
    const modified_output = base_output * thisProductivityMultiplier.value;
    return modified_output;
  })

  const effectiveSpeed = computed(()=>{
    return speed.value / output.value
  })
  const machines = computed(()=>effectiveSpeed.value * targetRate.value)
  const totalOutput = computed(()=>Math.ceil(machines.value) / effectiveSpeed.value)
</script>

<style scoped>
  .detail{
    font-size: 0.8em;
    opacity: 0.5;
    
  }
  table.chemistry{
    border-color: yellow;
    background-color: #ffff0040;
  }
  table.advanced-crafting{
    border-color: blueviolet;
    background-color: #8a2be21f;
  }
  table.crafting-with-fluid{
    border-color: greenyellow;
    background-color: #adff2f24;
  }
  table.smelting{
    border-color: brown;
    background-color: #a52a2a40;
  }

  table{
    border: 1px solid #ddd;
    border-radius: 20px;
    margin: 10px;
    background: rgba(0,0,0,0.05);

    position: relative;
  }
  table.raw{
    background: rgb(151 151 151);
    color: white;
  }
  table.raw .belt{
    --arrow-color: #aaaa;
  }
  table.raw tr{
    position: relative;
  }
  table input{
    width: 50px;
    background: transparent;
    border: none;
  }

  .modifiers{
    font-size: 0.8em;
    border-top: 1px solid #aaa;
    padding-top: 2px;
    margin-top: 2px;
  }
  .name{
    font-weight: bold;

  }
  .arrow{
    --arrow-color:rgba(255,255,255,0.8);
    position: absolute;
    left: -50px;
    top:0;
    bottom: 0;
    height: 1.4em;
    margin: auto;
    background: linear-gradient(90deg, var(--arrow-color), transparent);
    padding: 5px 10px;
    font-size: 0.7em;
  }
  .arrow span:not(.detail){
    font-weight: bold;
    font-size: 1.5em;
  }
  .arrow::before{
    content: '';
    display: block;
    position: absolute;
    background: transparent;
    left: -1.95em;
    bottom: 0;
    top: 0;
    margin: auto;
    border-top: transparent 2em solid;
    border-bottom: transparent 2em solid;
    border-right: var(--arrow-color) 2em solid;
  }
  .arrow.pipe{
    left: -25px;
    --arrow-color: grey;
  }
  .machines{
    font-size: 0.8em;
  }
  
  .machines span:not(.detail){
    font-weight: bold;
    font-size:2em;
  }
  .rate{
    font-size: 0.8em;
  }
  .speed{
    font-size: 0.8em;
  }

  th{
    text-align:left
  }
  th,td{
    padding: 3px 30px;
  }
</style>