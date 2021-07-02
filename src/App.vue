<template>
    <div id="app">
        <trading-vue :data="this.$data"></trading-vue>
    </div>
</template>

<script>
import TradingVue from 'trading-vue-js';

export default {
    name: 'App',
    components: {TradingVue},
    data() {
        return {
            ohlcv: [
                [1551128400000, 33, 37.1, 14, 14, 196],
                [1551132000000, 13.7, 30, 6.6, 30, 206],
                [1551135600000, 29.9, 33, 21.3, 21.8, 74],
                [1551139200000, 21.7, 25.9, 18, 24, 140],
                [1551142800000, 24.1, 24.1, 24, 24.1, 29],
            ]
        }
    },
    mounted() {
        this.getData();
    },
    methods: {
        getData() {
            const query = `
                 {
                  ethereum(network: bsc) {
                    dexTrades(
                      options: {asc: "timeInterval.minute"}
                      date: {since: "2021-06-20T07:23:21.000Z", till: "2021-06-23T15:23:21.000Z"}
                      exchangeAddress: {is: "0xcA143Ce32Fe78f1f7019d7d551a6402fC5350c73"}
                      baseCurrency: {is: "0x2170ed0880ac9a755fd29b2688956bd959f933f8"},
                      quoteCurrency: {is: "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c"},
                      tradeAmountUsd: {gt: 10}
                    )
                    {
                      timeInterval {
                        minute(count: 15, format: "%Y-%m-%dT%H:%M:%SZ")
                      }
                      volume: quoteAmount
                      high: quotePrice(calculate: maximum)
                      low: quotePrice(calculate: minimum)
                      open: minimum(of: block, get: quote_price)
                      close: maximum(of: block, get: quote_price)
                    }
                  }
                }
              `;
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
                    console.log(response.data.ethereum)
                })
                .catch(console.error);
        }
    }
}
</script>

<style>

</style>
