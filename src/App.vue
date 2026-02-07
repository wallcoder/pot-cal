<script setup>
import Button from "primevue/button";
import { InputNumber } from "primevue";

import Tabs from "primevue/tabs";
import TabList from "primevue/tablist";
import Tab from "primevue/tab";
import TabPanels from "primevue/tabpanels";
import TabPanel from "primevue/tabpanel";
import { Toast } from "primevue";
import { useToast } from "primevue";
import { watch } from "vue";

import { ref } from "vue";
const toast = useToast();

const data = ref({
  t: null, //take home salary
  w: null, //no of warking days
  h: 8, //working hours
  d: null, // deficit/extra working hour
});



const penaltyResult = ref(null);
const penaltyTouched = ref({});
const displayAmount = ref(0)
const penaltyPercentage = ref(null)


watch(penaltyResult, (newVal) => {
  if (!newVal) return
  console.log("HELLO")

  const duration = 400 // ms (lower = faster)
  const start = performance.now()

  function animate(now) {
    const progress = Math.min((now - start) / duration, 1)
    displayAmount.value = Math.floor(progress * newVal)

    if (progress < 1) {
      requestAnimationFrame(animate)
    }
  }

  displayAmount.value = 0
  requestAnimationFrame(animate)
})

const calculatePenalty = () => {
  Object.keys(data.value).forEach((key) => {
    penaltyTouched.value[key] = true
  })

  if (Object.values(data.value).some(v => v == null || v === '')) {
    toast.add({ severity: 'error', summary: 'Empty fields', detail: 'Enter all fields', life: 3000 });
    return;
  }

  const result =
    2 *
    data.value.d *
    (data.value.t / (data.value.w * data.value.h));

  penaltyResult.value = Math.round(result * 100) / 100;
  penaltyPercentage.value = twoDecimals((penaltyResult.value/data.value.t) * 100)
};

function twoDecimals(n){
  return Math.round(n*100) / 100;
}


function penaltyMessage(perc) {
  if (perc <= 2)  return "Barely a scratch 😌"
  if (perc <= 5)  return "You’ll survive. Promise."
  if (perc <= 8)  return "Mild inconvenience 😐"
  if (perc <= 12) return "Okay… that stings 😬"
  if (perc <= 18) return "Your wallet noticed that."
  if (perc <= 25) return "That one hurt. Not gonna lie."
  if (perc <= 35) return "This is why alarms exist ⏰"
  if (perc <= 50) return "Your wallet wants to talk."
  if (perc <= 65) return "Financial damage confirmed 💥"
  if (perc <= 80) return "This is getting personal 😭"
  if (perc <= 95) return "Wallet status: emotionally damaged 💔"
  return "Moment of silence for your money 🪦"
}

function rupeefy(amount) {
  return `₹${Number(amount).toLocaleString('en-IN')}`
}


</script>

<template>
  <section class="flex justify-center items-center w-full h-[100vh] px-[7%]">
    <Toast position="top-center" />
    <Tabs value="0">
      <TabList>
        <Tab value="0">Penalty/Overtime Pay Calculator</Tab>
      
      </TabList>
      <TabPanels>
        <TabPanel value="0" ">
          <div class=" p-0 md:p-2 rounded-lg flex flex-col gap-4">
          <!-- <h1 class="text-3xl font-semibold">Penalty Calculator</h1> -->
          <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
            <div class="w-full">
              <label class="block mb-1 text-sm" for="take_home_salary">Take Home Salary</label>
              <InputNumber class="w-full" size="small" v-model="data.t" placeholder="Take Home Salary" inputId="take_home_salary"
                mode="currency" currency="INR" :min="0" locale="en-IN" @blur="penaltyTouched.t = true"
                :invalid="data.t === null && penaltyTouched.t" />
            </div>
            <div class="w-full">
              <label class="block mb-1 text-sm" for="no_of_working_days">No. of Working Days</label>
              <InputNumber class="w-full" size="small" v-model="data.w" placeholder="No. of Working Days"
                inputId="no_of_working_days" :min="0" suffix=" day(s)" @blur="penaltyTouched.w = true"
                :invalid="data.w === null && penaltyTouched.w" />
            </div>
            <div class="w-full">
              <label class="block mb-1 text-sm" for="working_hours">Working Hours</label>
              <InputNumber class="w-full" :disabled="true" size="small" :min="0" placeholder="Working Hours" v-model="data.h" inputId="working_hours"
                :defaultValue="8" suffix=" hr(s)" @blur="penaltyTouched.h = true"
                :invalid="data.h === null && penaltyTouched.h" />
            </div>
            <div class="w-full">
              <label class="block mb-1 text-sm" for="deficit_working_hours">Deficit/Extra Working Hours</label>
              <InputNumber class="w-full" size="small" @blur="penaltyTouched.d = true" v-model="data.d" mode="decimal" :min="0"
                :minFractionDigits="0" :maxFractionDigits="2" :useGrouping="false" placeholder="Deficit/Extra Working Hours"
                inputId="deficit_working_hours" suffix=" hr(s)" :invalid="data.d === null && penaltyTouched.d" />
            </div>
          </div>
          <Button size="small" @click="calculatePenalty" label="Calculate" />

          <Transition name="bounce">
            <div v-if="penaltyResult" class="flex flex-col items-center">
              <span class="text-md text-gray-100">Your Penalty/Overtime Pay Amount is</span>
              <span class="text-2xl text-yellow-400">{{ rupeefy(displayAmount)}}</span>
              <!-- <span class="mt-1 text-yellow-400">{{ penaltyMessage(penaltyPercentage) }}</span> -->
              <span class="text-xs mt-1 text-gray-300">That's {{ penaltyPercentage }}% of your Take Home Salary</span>
            </div>
          </Transition>

          </div>
        </TabPanel>

        
      </TabPanels>
    </Tabs>
  </section>
</template>

<style scoped>
.bounce-enter-active {
  animation: bounce-in 0.5s;
}

.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
  }

  50% {
    transform: scale(1.1);
  }

  100% {
    transform: scale(1);
  }
}
</style>
