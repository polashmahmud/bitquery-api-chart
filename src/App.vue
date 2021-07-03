<template>
    <div id="app">
        <trading-vue :data="this.$data"></trading-vue>
        <br><br>
        <table id="customers" v-if="lastTen.length">
            <tr>
                <th>minute</th>
                <th>open</th>
                <th>high</th>
                <th>low</th>
                <th>close</th>
                <th>volume</th>
            </tr>
            <tr v-for="(item, index) in lastTen" :key="index">
                <td>{{ item[0] }}</td>
                <td>{{ item[1] }}</td>
                <td>{{ item[2] }}</td>
                <td>{{ item[3] }}</td>
                <td>{{ item[4] }}</td>
                <td>{{ item[5] }}</td>
            </tr>
        </table>
    </div>
</template>

<script>
import TradingVue from 'trading-vue-js';
import {GET_COIN_BARS} from "./Bitquery";

export default {
    name: 'App',
    components: {TradingVue},
    data() {
        return {
            ohlcv: [],
            lastTen: [],
        }
    },
    created() {
        this.getData();

        // setInterval(() => {
        //     this.getData();
        // }, 5000)
    },
    methods: {
        getData() {
            const query = GET_COIN_BARS;
            const url = "https://graphql.bitquery.io/";
            const opts = {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    "X-API-KEY": "BQYyGwlM3uQRKbNrPS01UwFpoDHrTaEq"
                },
                body: JSON.stringify({
                    query
                })
            };
            fetch(url, opts)
                .then(res => res.json())
                .then(response => {
                    let data = response.data.ethereum.dexTrades;
                    console.log(data)
                    const cdata = data.map(d => {
                        return [
                            Date.parse(d.timeInterval.minute),
                            parseFloat(d.open),
                            parseFloat(d.high),
                            parseFloat(d.low),
                            parseFloat(d.close),
                            parseFloat(d.volume)
                        ]
                    });
                    this.ohlcv = cdata;

                    this.lastTen = cdata.slice(-5);

                })
                .catch(console.error);
        }
    }
}
</script>

<style scoped>
#customers {
    font-family: Arial, Helvetica, sans-serif;
    border-collapse: collapse;
    width: 100%;
}

#customers td, #customers th {
    border: 1px solid #ddd;
    padding: 8px;
}

#customers tr:nth-child(even){background-color: #f2f2f2;}

#customers tr:hover {background-color: #ddd;}

#customers th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #04AA6D;
    color: white;
}
</style>
