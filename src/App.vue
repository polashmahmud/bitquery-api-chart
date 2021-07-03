<template>
    <div id="app">
        <trading-vue :data="this.$data"></trading-vue>
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
        }
    },
    created() {
        this.getData();

        setInterval(() => {
            this.getData();
        }, 5000)
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
                    console.log(response.data)
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
                })
                .catch(console.error);
        }
    }
}
</script>

<style scoped>

</style>
