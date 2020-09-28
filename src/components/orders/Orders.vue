<template>
    <div>
       <el-button @click="showCompary">冲冲冲</el-button>
    </div>
</template>

<script>
export default {
    data() {
        return {

        }
    },
    created() {

    },
    methods: {
        async showCompary () {
            const {data: res} = await this.$http.get('http://mengxuegu.com:7300/mock/5f6d4b058d90085e5b584c85/data/device/list')
            console.log(res.data)
        }
    }
}
</script>

<style lang="less" scoped>

</style>