<template lang="pug">
    include ../../../tools/mixins.pug

    +b.table
        +e.container.container
            +e.wrapper
                +e.head(ref="head")
                    +e.line--head
                        +e.col--1
                            p Name
                        +e.col--2
                            p Price
                        +e.col--3
                            p Market Cap
                        +e.col--4
                            p Volume (24Hr)
                vue-scroll(:ops="ops" :style="{height: tableHeight + 'px'}")
                    +e.body()
                        +e.line--body(v-for="(item,index) in list" :class="{'green': change.green.indexOf(index) >= 0,'red': change.red.indexOf(index) >= 0}")
                            +e.col--1
                                img(:src="`https://static.coincap.io/assets/icons/${item.symbol.toLowerCase()}@2x.png`")
                                div
                                    p {{ item.name }}
                                    span {{ item.symbol }}
                            +e.col--2
                                p ${{ moneyFormat(item.priceUsd,'price') }}
                            +e.col--3
                                p ${{ moneyFormat(item.marketCapUsd, 'volume') }}
                            +e.col--4
                                p ${{ moneyFormat(item.volumeUsd24Hr, 'volume') }}

</template>

<script>
    import axios from 'axios';
    import vuescroll from 'vuescroll';

    export default {
        props: ['height'],
        data() {
            return {
                list: [],
                tableHeight: null,
                change: {
                    green: [],
                    red: []
                },
                ops: {
                    vuescroll: {
                        mode: 'native',
                        sizeStrategy: 'percent',
                        detectResize: true
                    },
                    bar: {
                        hoverStyle: true,
                        onlyShowBarOnScroll: false
                    }
                }
            }
        },
        methods: {
            getData() {
                axios({
                    method: 'GET',
                    url: 'https://api.coincap.io/v2/assets'
                })
                    .then((response) => {
                        let data = response.data.data;
                        //sort down
                        data.sort((a,b) => {
                            return b.marketCapUsd - a.marketCapUsd;
                        });

                        data.map( (item,index) => {
                            if (index < 15) {
                                item.marketCapUsd = parseInt(item.marketCapUsd);
                                item.volumeUsd24Hr = parseInt(item.volumeUsd24Hr);
                                item.priceUsd = Number(item.priceUsd);
                                this.list.push(item);
                            }
                        });

                        this.includeSocket();
                    })
                    .catch(function (error) {
                        // console.log(error)
                    });
            },
            moneyFormat(labelValue,method) {
                switch(method) {
                    case 'volume':
                        return Math.abs(Number(labelValue)) >= 1.0e+9

                            ? (Math.abs(Number(labelValue)) / 1.0e+9).toFixed(2) + "B"
                            // Six Zeroes for Millions
                            : Math.abs(Number(labelValue)) >= 1.0e+6

                                ? (Math.abs(Number(labelValue)) / 1.0e+6).toFixed(2) + "M"
                                // Three Zeroes for Thousands
                                : Math.abs(Number(labelValue)) >= 1.0e+3

                                    ? (Math.abs(Number(labelValue)) / 1.0e+3).toFixed(2) + "K"

                                    : Math.abs(Number(labelValue));
                        break;

                    case 'price':
                        return labelValue > 1 ? labelValue.toFixed(2) : labelValue.toFixed(8);
                        break;
                }
            },
            tableSize() {
                this.tableHeight = window.innerHeight - this.height - this.$refs.head.clientHeight;
            },
            includeSocket() {
                const WebSocket = require('isomorphic-ws');
                let getName = '';

                this.list.map((item) => {
                    getName === '' ? getName = item.name.toLowerCase() : getName = getName + ',' + item.name.toLowerCase();
                });

                let tradeWs = new WebSocket('wss://ws.coincap.io/trades/binance'),
                    priceWs = new WebSocket(`wss://ws.coincap.io/prices?assets=${getName}`);

                tradeWs.onmessage = (msg) => {
                    let data = JSON.parse(msg.data);
                    this.list.map( (item,index) => {
                        if (data['quote'] === item.name.toLowerCase() ) {
                            if (data['direction'] === 'sell') {
                                item.volumeUsd24Hr = item.volumeUsd24Hr - data['volume'];
//                                this.change.push(index);
//
//                                setTimeout(() => {
//                                    this.change.splice(this.change.indexOf(index),1);
//                                },500)
                            } else {
                                item.volumeUsd24Hr = item.volumeUsd24Hr + data['volume'];
                            }
                        }
                    })
                }

                priceWs.onmessage = (msg) => {
                    let data = JSON.parse(msg.data);

                    this.list.map( (item,index) => {
                        if (typeof data[item.name.toLowerCase()] !== 'undefined') {
                            let newPrice = Number(data[item.name.toLowerCase()]);

                            if ((newPrice > 1 && item.priceUsd.toFixed(2) !== newPrice.toFixed(2)) || (newPrice < 1 && item.priceUsd.toFixed(8) !== newPrice.toFixed(8))) {


                                if (newPrice > item.priceUsd) {
                                    this.change.green.push(index);


                                    setTimeout(() => {
                                        this.change.green.splice(this.change.green.indexOf(index),1);
                                    },500)
                                } else {
                                    this.change.red.push(index);

                                    setTimeout(() => {
                                        this.change.red.splice(this.change.red.indexOf(index),1);
                                    },500)
                                }

                                item.priceUsd = newPrice;
                            }
                        }
                    })
                }
            }
        },
        created() {
            this.getData();
        },
        mounted() {
            setTimeout(() => {
                this.tableSize();
            });

            window.addEventListener('resize', this.tableSize);

        },
        destroyed() {
            window.removeEventListener('resize', this.tableSize);
        },
        components: {
            'vue-scroll': vuescroll
        }
    }
</script>