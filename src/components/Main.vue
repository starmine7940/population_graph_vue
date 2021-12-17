<template>
    <div id="main">
        <div id="prefectures_wrapper">
            <h1>都道府県</h1>
            <div id="prefectures">
                <div v-for="prefecture in prefectures" :key="prefecture.id" id="prefecture">
                    <label :for="prefecture.id">
                        <input
                            type="checkbox"
                            :id="prefecture.id"
                            :checked="prefecture.isChecked"
                            @click="check(prefecture.id, prefecture.name, prefecture.isChecked)"
                        />
                        {{ prefecture.name }}
                    </label>
                </div>
            </div>
        </div>
        <div id="graph_wrapper">
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
        return {
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
            draw_counter: 0,                    //再描画のために用意した変数
            prefectures: []
        };
    },
    methods: {
        get_prefectures: async function(){      //都道府県名を取得する関数
            try{
                const res = await fetch(PREFECTURES_URL, {headers: {"x-api-key": API_KEY}});
                const res_json = await res.json();
                const prefectures = res_json.result;
                this.prefectures = prefectures.map(val => {
                    return {
                        id: val['prefCode'],
                        name: val['prefName'],
                        isChecked: false
                    };
                });
            }catch(error){
                console.log(error.message);
            }
        },
        check: function(id, name, isChecked){   //チェックボックスが押されているかを判断する関数
            if(isChecked){
                this.delete_graph(id);
            }else{
                this.add_graph(id, name);
            }
        },
        add_graph: async function(id, name){    //グラフのデータを追加する関数
            try{
                const res = await fetch(POPULATION_URL + id, {headers: {"x-api-key": API_KEY}});
                const res_json = await res.json();
                const population = res_json.result.data[0].data.map(
                    val => val["value"]
                );
                this.options.series.push({
                    id: id,
                    name: name,
                    data: population
                });
                this.prefectures[id - 1].isChecked = true;
                this.draw_counter++;
            }catch(error){
                console.log(error.message);
            }
            
        },
        delete_graph: function(id){             //グラフのデータを削除する関数
            this.options.series = this.options.series.filter(val => val.id !== id);
            this.prefectures[id - 1].isChecked = false;
            this.draw_counter++;
        },
    },
    mounted(){
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

#prefectures {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr;
}

#graph_wrapper {
    width: 100%;
    margin-top: 100px;
}

@media screen and (max-width:480px) {
    #graph_wrapper {
        max-width: 480px;
        margin-top: 50px;
    }
}
</style>