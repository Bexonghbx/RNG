<template>
    <v-container fluid align="center">
        <v-row justify="center">
            <v-col cols="12" align="center">
                <canvas id="myChart"></canvas>
            </v-col>
            <v-col cols="12" align="center">
                <v-btn text="Refresh Graph" variant="outlined" color="secondary" flat @click="updateGraph()"></v-btn>
            </v-col>
        </v-row>
        <v-row>
            <v-divider class="mt-5 mb-5"></v-divider>
        </v-row>
        <v-row justify="center">
            <VCard class="ma-2" subtitle="LED A" width="150" flat border align="center"> 
                <VCardItem class="pa-0">Turned on </VCardItem>     
                <VCardItem class="pa-0">
                    <span class="text-h5 text-primary font-weight-bold">{{led_A}}</span>
                </VCardItem>  
                <VCardItem class="pa-0">times</VCardItem>        
                <VCardItem> 
                    <VBtn text="Update" class="ma-1 text-caption" rounded="pill" flat color="secondary" variant="tonal" @click="updateLEDCount('ledA')"></VBtn> 
                </VCardItem> 
            </VCard> 

            <VCard class="ma-2" subtitle="LED B" width="150" flat border align="center"> 
                <VCardItem class="pa-0">Turned on </VCardItem>     
                <VCardItem class="pa-0">
                    <span class="text-h5 text-primary font-weight-bold">{{led_B}}</span>
                </VCardItem>  
                <VCardItem class="pa-0">times</VCardItem>        
                <VCardItem> 
                    <VBtn text="Update" class="ma-1 text-caption" rounded="pill" flat color="secondary" variant="tonal" @click="updateLEDCount('ledB')"></VBtn> 
                </VCardItem> 
            </VCard> 
        </v-row>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import Chart from 'chart.js/auto'; 
import { useAppStore } from "@/store/appStore";  
 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const led_A       = ref(0); // Store count for LED A 
const led_B       = ref(0); // Store count for LED B
const AppStore    = useAppStore();   

let chart         = null;  // Chart object 
const data        = { labels:  ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'], 
                    datasets: [ 
                        { 
                            label: 'Fully Rounded', 
                            data: [0, 0, 0, 0, 0, 0], 
                            borderColor: '#1ECBE1', 
                            backgroundColor: '#4BD5E7', 
                            borderWidth: 2, 
                            borderRadius: 5, 
                            borderSkipped: false, 
                        }] 
                    }; 

const config      = { type: 'bar', 
                    data: data, 
                    options: { 
                        responsive: true, 
                        plugins: { 
                            legend: { 
                                position: 'top', 
                            }, 
                            title: { 
                                display: true, 
                                text: 'Chart.js Bar Chart' 
                            } 
                        } 
                    }, 
                }; 

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    const ctx = document.querySelector('#myChart'); // Select canvas for rendering chart 
    chart = new Chart(ctx, config ); // create chart 
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
});

// Update graph with labels and new data 
const updateData = ( chart, label, newData) => { 
    chart.data.labels = label; 
    chart.data.datasets[0].data = newData; 
    chart.update(); 
}

// Fetch new data and update graph 
const updateGraph = async () =>{ 
    let result = await AppStore.getFrequencies(); 
    let labels = []; 
    let data   = []; 
    
    if (result.length > 0){ 
        result.forEach(obj => { 
            labels.push(obj["number"]) 
            data.push(obj["frequency"]) 
        }); 
        
        updateData(chart,labels,data); 
    } 
} 
// Fetch new data and update cards 
const updateLEDCount = async(name)=>{ 
    let result =  await AppStore.getOnCount(name); 
    // console.log(result); 
    if (name == "ledA"){ 
        led_A.value = result; 
    } 
    if (name == "ledB"){ 
        led_B.value = result; 
    } 
} 

</script>


<style scoped>
/** CSS STYLE HERE */
.row{
    max-width: 1200px;
}

</style>
  