<template>
  <template v-if="entitydata">
  <table>
    <tr>
      <td>
        <div class="name">
          {{name}}
        </div>
        <div class="machines">
        <span>{{rate * entitydata.speed}}</span> machines
        </div>
        <div class="rate">
          <span>{{rate}}</span>/s
        </div>
        <div class="belt">
          <span>{{Math.ceil((rate/beltrate)*2)/2}}</span> belt
        </div>
      </td>
        <td>
          <table class="raw">
            <tr v-for="rawIngredient in raw">
              <td>
                {{rawIngredient.entity}} @ {{rawIngredient.rate * props.rate}}/s
                <div class="belt">
                  <span>{{Math.ceil(((rawIngredient.rate * props.rate)/beltrate)*2)/2}}</span> belt
                </div>
              </td>
            </tr>
          </table>
          <Ingredient v-for="child in children" :key="child.entity" :entity="child.entity" :rate="child.rate * props.rate"/>
        </td>
      </tr>
    </table>
  </template>
</template>

<script setup>
    import { ref, computed ,watch} from "vue";
    import data from "./data.json";
    import Ingredient from "./Ingredient.vue";

import { beltrate } from "./Config";

const props = defineProps({ entity: {
    type: String,
    required: true
  },rate:{
  type: Number,
  required: true
  }});


  const name = computed(()=>props.entity);
  const entitydata = computed(()=>{
    console.log(props.entity);
    return data[props.entity]
  });
  const rate = computed(()=>props.rate)

  const ingredients = computed(()=>{
    return Object.entries(entitydata.value.ingredients).map(([k,v])=>({entity: k, rate:v}))
  })

  const children = computed(()=>ingredients.value.filter(x=>data[x.entity]))
  const raw = computed(()=>ingredients.value.filter(x=>!data[x.entity]))

</script>

<style scoped>
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
    background-color: #aaaa;
  }
  table.raw .belt::before{
    border-right-color: #aaaa;
  }
  .name{
    font-weight: bold;

  }
  .belt{
    position: absolute;
    left: -50px;
    top:0;
    bottom: 0;
    height: 1.4em;
    margin: auto;
    background-color: rgba(255,255,255,0.8);
    padding: 5px 10px;
    font-size: 0.7em;
  }
  .belt span{
    font-weight: bold;
  }
  .belt::before{
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
    border-right: rgba(255,255,255,0.8) 2em solid;
  }

  .machines{
    font-size: 0.8em;
  }
  
  .machines span{
    font-weight: bold;
    font-size:1.2em;
  }
  .rate{
    font-size: 0.8em;
  }

  th{
    text-align:left
  }
  th,td{
    padding: 3px 30px;
  }
</style>