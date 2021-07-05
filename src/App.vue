<template>
    <div id="app">
        <div v-if="addressInfo.hasOwnProperty('name')">
            <p>Name: <strong>{{ addressInfo.name }}</strong> | Symbol: <strong>{{ addressInfo.symbol }}/USD</strong></p>
        </div>
        <div class="search">
            <input
                @input="search"
                type="text"
                placeholder="Search address/name"
                v-model="searchValue"
            >
            <ul v-if="showSearchChild">
                <li v-for="(item, index) in getSearchAddress" :key="index">
                    <div @click.prevent="changeAddress(item.address)" style="cursor: pointer">
                        <h4>{{ item.name }}</h4>
                        <p>{{ item.address}}</p>
                    </div>
                </li>
            </ul>
        </div>

        <trading-vue
            :data="this.$data"
            title-txt="Coinomic 🔥"
            :width="1200"
            :height="600"
            :toolbar="true"
        ></trading-vue>
    </div>
</template>

<script>
import TradingVue from 'trading-vue-js';

export default {
    name: 'App',
    components: {TradingVue},
    data() {
        return {
            ohlcv: [],
            baseCurrency: "0x2170ed0880ac9a755fd29b2688956bd959f933f8",
            addressInfo: {},
            searchValue: "",
            getSearchAddress: [],
            getAllAddressValue: [],
            showSearchChild: false,
        }
    },
    created() {
        this.getCoinInfo();
        this.getData();
        this.getAllAddress();
    },
    mounted() {
        setInterval(() => {
             this.getData();
        }, 10000)
    },
    methods: {
        changeAddress(address) {
            this.baseCurrency = address;
            this.searchValue = "";
        },
        getAllAddress() {
            const opts = {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                }
            };

            fetch(`https://coinomic-default-rtdb.firebaseio.com/coins.json`, opts)
                .then(res => res.json())
                .then(response => {
                    Object.values(response).forEach(address => {
                        this.getAllAddressValue.push({
                            name: address.tokenName,
                            address: address.tokenBSCAdress,
                            symbol: address.tokenSymbol
                        });
                    })
                })
                .catch(console.error);
        },

        search() {
            let term = this.searchValue;
            let search = new RegExp(term , 'i');

            this.getSearchAddress = this.getAllAddressValue.filter(item => search.test(item.name));

            this.showSearchChild = !!this.getSearchAddress.length;
        },

        getCoinInfo() {
            const query = `
                        {
                          ethereum(network: bsc) {
                            dexTrades(
                              options: {desc: ["block.height", "transaction.index"], limit: 1}
                              exchangeAddress: {is: "0xcA143Ce32Fe78f1f7019d7d551a6402fC5350c73"}
                              baseCurrency: {is: "${this.baseCurrency}"}
                              quoteCurrency: {is: "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c"}
                            )
                            {
                              block {
                                height
                                timestamp {
                                  time(format: "%Y-%m-%d %H:%M:%S")
                                }
                              }
                              transaction {
                                index
                              }
                              baseCurrency {
                                name
                                symbol
                                decimals
                              }
                              quotePrice
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
                    this.addressInfo = response.data.ethereum.dexTrades[0].baseCurrency;
                })
                .catch(console.error);

        },

        getData() {
            const query =  `
                        {
                          ethereum(network: bsc) {
                            dexTrades(
                              options: {asc: "timeInterval.minute"}
                              date: {since: "2021-06-20T07:23:21.000Z", till: "${new Date().toISOString()}"}
                              exchangeAddress: {is: "0xcA143Ce32Fe78f1f7019d7d551a6402fC5350c73"}
                              baseCurrency: {is: "${this.baseCurrency}"},
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
                    let data = response.data.ethereum.dexTrades;
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
    },
    watch: {
        searchValue(value) {
            if (value === '') {
                this.showSearchChild = false;
            }
        },
        baseCurrency() {
            this.getData();
            this.getCoinInfo();
        }
    }
}
</script>

<style scoped>
.search {
    margin-bottom: 20px;
    width: 300px;
}

.search input {
    padding: 10px;
    width: 300px;
    position: relative;
}

.search ul {
    position: absolute;
    background: white;
    width: 322px;
    margin-top: 5px;
    z-index: 999;
    padding: 0;
    height: 200px;
    overflow: scroll;
}

.search ul li {
    list-style: none;
    padding: 10px;
    border-bottom: 1px dotted;
}

.search ul li:hover {
    background: #ddd;
}

.search ul li h4 {
    margin: 0;
    padding: 0;
    text-align: center;
}

.search ul li p {
    margin: 0;
    padding: 0;
}
</style>
