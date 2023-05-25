<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue";
import Chart from "chart.js/auto";

//data
const weights = ref([]);
const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});
const weightChart = shallowRef(null);
const weightChartEl = ref(null);
const weightInput = ref(51);
const input = ref(null),
  editMode = ref(false);

//methods
const addWeight = () => {
  if (weightInput.value > 0 || !weightInput.value === "") {
    weights.value.push({
      weight: weightInput.value,
      date: new Date().getTime(),
    });
    weightInput.value = "";

    editMode.value = true;
    nextTick(() => {
      input.value.focus();
    });

  } else {
    alert("Cant be emtpy");
    return;
  }
};

//watch
watch(weights, (newValue) => {
  const ws = [...newValue];

  if(weightChart.value){
    weightChart.value.data.labels = ws
                .sort((a,b)=>a.date - b.date)
                .map(w => new Date(w.date).toLocaleDateString())
                .slice(-7);
    weightChart.value.data.datasets[0].data = ws
                .sort((a,b)=>a.date - b.date)
                .map(w => w.weight)
                .slice(-7);
                
    weightChart.value.update();
    return;

  }

  nextTick(()=>{
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'),{
      type: 'line',
      data: {
        labels: ws
                .sort((a,b)=>a.date - b.date)
                .map(w => new Date(w.date).toLocaleDateString()),
        datasets: [
          {
            label: "Weight",
              data: ws
                .sort((a, b) => a.date - b.date)
                .map((w) => new Date(w.weight)),
              backgroundColor: "rgba(255,105, 180,0.2)",
              borderColor: "rgb(244,105, 180)",
              borderWidth: 1,
              fill: true
          }
        ],
        options:{
          responsive: true,
          maintainAspectRatio: false
        }
      }
    }
    );
    console.log(weightChart.value);
  })
  console.log(weightChartEl);

}, {deep:true});
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <div class="current">
      <span>{{ currentWeight.weight }} </span>
      <small> current weight (kg)</small>
    </div>
    <form @submit.prevent="addWeight">
      <input
        type="number"
        step="0.1"
        v-model="weightInput"
        ref="input"
        @focusout="editMode = false"
        @keydown.enter="editMode = false"
      />
      <input type="submit" value="Add weight" />
    </form>

    <div class="data" v-if="weights && weights.length > 0">
      <h2>Last 7 days</h2>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}kg</span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.data {
  width: 30rem;
}
.current {
  display: flex;
  flex-direction: column;
  height: 15rem;
  width: 15rem;
  border: solid 5px rgb(255, 156, 214);
  border-radius: 999px;
  justify-content: center;
  margin-bottom: 2rem;
}
.current span {
  font-weight: bolder;
  font-size: 3rem;
}
.current small {
  color: rgb(176, 176, 176);
  font-style: italic;
}
form {
  display: flex;
  border: 1px solid #aaa;
  border-radius: 0.5rem;
  transition: 200ms linear;
  overflow: hidden;
  min-width: 30rem;
}
form:focus-within,
form:hover {
  border-color: hotpink;
  border-width: 2px;
}

form input[type="number"] {
  appearance: none;
  outline: none;
  border: none;
  background-color: white;

  flex: 1 1 0%;
  padding: 1rem;
  font-size: 1.25rem;
  color: black;
}
form input[type="submit"] {
  appearance: none;
  outline: none;
  border: none;
  cursor: pointer;
  background-color: hotpink;
  color: white;

  padding: 0.5rem 1rem;
  font-size: 1.25rem;
  font-weight: 700;
  transition: 200ms linear;
  border-left: 3px solid transparent;
}
form input[type="submit"]:hover {
  background-color: white;
  color: hotpink;
  border-left-color: hotpink;
}
.canvas-box {
  max-width: 720px;
  background-color: white;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}
h2 {
  color: grey;
  text-align: left;
}
ul {
  list-style: none;
  padding: 0;
  margin: 0;
  background-color: #cfcfcf;
  border-radius: 0.5rem;
  overflow: hidden;
  margin-bottom: 2rem;
}
.weight-history ul li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  padding: 0.5rem;
  font-size: 1.2rem;
}
.weight-history ul li:nth-child(even) {
  background-color: #dfdfdf;
}
.weight-history ul li:hover {
  background-color: #f8f8f8;
}
.weight-history ul li:last-of-type {
  border-bottom: none;
}
.weight-history ul li span {
  display: block;
  font-weight: 700;
  margin-right: 1rem;
  color: black;
}
.weight-history ul li small {
  color: #888;
  font-style: italic;
}
</style>
