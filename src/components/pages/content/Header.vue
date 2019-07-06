<template lang="pug">
    include ../../../tools/mixins.pug

    +b.HEADER.header(ref="header")
        +e.container.container
            +e.search(:class="{'active': search.status}")
                +e.input
                    input(v-model="search.value")
                    i(@click="search.status=!search.status")
            +e.A.logo(href="#")
                img(src="/public/media/logo.svg")
            +e.BUTTON.button--hamburger.button--hamburger(
                type="button"
                @click="sidebar=!sidebar"
                :class="{'active': sidebar}"
                )
                span
                span
                span
            +e.wrap(:class="{'active': sidebar}")
                +e.NAV.nav.nav__
                    +e.list
                        +e.A.item
                            +e.I.img--bitcoin
                            +e.A.link--(href="#")
                            span Coins
                        +e.A.item
                            +e.I.img--reverse
                            +e.A.link(href="#")
                            span Exchanges
                        +e.A.item--charts
                            +e.A.link(href="#")
                            span Charts
                        +e.A.item
                            +e.I.img--api
                            span API
                        +e.A.item--settings
                            +e.I.img--settings
                            span Settings

</template>

<script>

    export default {
        data() {
            return {
                search: {
                    status: false,
                    value: ''
                },
                sidebar: false
            }
        },
        methods: {

        },
        mounted() {
            this.$emit('headerHeight', this.$refs.header.getBoundingClientRect().height)
        },
        destroyed() {

        }
    }
</script>