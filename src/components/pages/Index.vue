<template lang="pug">
    include ../../tools/mixins.pug

    div
        header-component(@headerHeight="headerHeight")
        table-component(:height="headHeight")

</template>

<script>
    import Header from '../pages/content/Header.vue';
    import Table from '../pages/content/Table.vue';

    export default {
        data() {
            return {
                headHeight: null
            }
        },
        methods: {
            headerHeight(data) {
                this.headHeight = data;
            }
        },
        mounted() {

        },
        destroyed() {

        },
        components: {
            'header-component': Header,
            'table-component': Table
        }
    }
</script>