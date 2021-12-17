<template>
    <div id="main">
        <div id="prefectures">
            <h1>都道府県</h1>
        </div>
        <div id="graph">
            <Graph :options="options" :key="draw_counter" />
        </div>
    </div>
</template>

<script>
import VueHighcharts from 'vue3-highcharts';

const API_KEY = process.env.VUE_APP_API_KEY;
const PREFECTURES_URL = 'https://opendata.resas-portal.go.jp/api/v1/prefectures';
const POPULATION_URL= 'https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?prefCode=';

export default {
    name: 'Main',
    components:{
        'Graph': VueHighcharts
    },
    data() {
        return{
            options: {
                title: {
                    style: {
                        display: 'none'
                    }
                },
                xAxis: {
                    title:{
                        text: '年度'
                    },
                    categories: [1960, 1965, 1970, 1975, 1980, 1985, 1990, 1995, 2000, 2005, 2010, 2015, 2020, 2025, 2030, 2035, 2040, 2045]
                },
                yAxis: {
                    title: {
                        text: '人口数'
                    }
                },
                series: []
            },
            draw_counter: 0
        }
    },
    methods: {
        get_prefectures: async function(){
            const res = await fetch(PREFECTURES_URL, {headers: {"x-api-key": API_KEY}});
            const res_json = await res.json();
            const prefectures = res_json.result;
            for(let i = 0; i < prefectures.length; i++){
                let checkbox = document.createElement('input');
                checkbox.setAttribute('type', 'checkbox');
                checkbox.setAttribute('prefCode', prefectures[i].prefCode);
                checkbox.setAttribute('prefName', prefectures[i].prefName);
                checkbox.onclick = () => {
                    this.check_checkbox(checkbox, prefectures[i].prefCode, prefectures[i].prefName);
                }
                let label = document.createElement('label');
                label.appendChild(checkbox);
                label.appendChild(document.createTextNode(prefectures[i].prefName));
                document.getElementById('prefectures').appendChild(label);
            }
        },
        check_checkbox: function(checkbox, prefCode, prefName){
            if(checkbox.checked){
                this.add_graph(prefCode, prefName);
            }else{
                this.delete_graph(prefName);
            }
        },
        add_graph: async function(prefCode, prefName){
            const res = await fetch(POPULATION_URL + prefCode, {headers: {"x-api-key": API_KEY}});
            const res_json = await res.json();
            const population = res_json.result;
            let population_data = []
            for(let i = 0; i < population.data[0].data.length; i++){
                population_data.push(population.data[0].data[i].value);
            }
            this.options.series.push({
                name: prefName,
                data: population_data
            });
            this.draw_counter++;
        },
        delete_graph: function(prefName){
            for(let i = 0; i < this.options.series.length; i++){
                if(this.options.series[i].name == prefName){
                    this.options.series.splice(i, 1);
                    this.draw_counter++;
                }
            }
        }
    },
    mounted() {
        this.get_prefectures();
    }
}
</script>

<style>
#main {
    display: flex;
    flex-direction: column;
}

h1 {
    font-size: large;
}

label {
    float: left;
    margin: 0px 10px 0px 0px;
}

#graph {
    width: 100%;
    margin-top: 100px;
}

@media screen and (max-width:480px) {
    #graph {
        max-width: 480px;
        margin-top: 50px;
    }
}
</style>