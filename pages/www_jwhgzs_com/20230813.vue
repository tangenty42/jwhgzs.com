<style scoped>
    .item {
        display: block;
        margin-bottom: 20px;
    }
    .item .time {
        font-size: 80%;
    }
    .item .name {
        width: 100%;
        text-align: center;
    }
</style>

<template>
    <div class="box box_md hcenter">
        <div class="box_title">临时上传页 - 2023/8/13</div>
        <el-divider></el-divider>
        <el-button size="large" type="primary" plain @click="upload">{{ btnText }}</el-button>
        <el-divider>已上传文件列表：</el-divider>
        <div v-for="v in fileList" class="item">
            <div class="time">{{ getTimeDesc(v.time) }}</div>
            <el-link class="name" :href="v.url" target="__blank">{{ v.name }}</el-link>
        </div>
    </div>
</template>

<script setup>
    // -- basics
    let config = useState('config')
    definePageMeta({
        titles: [
            ['local://www', '主站'],
            ['', '临时页']
        ]
    })
    
    // -- refs
    let btnText = ref(/*'傻逼子健赶紧点这里' +*/ '上传'),
        fileList = ref([])
    
    // -- app
    function upload() {
        selectFile(function (data) {
            p({
                name: '文件上传',
                url: u('local://api/tmp/20230813'),
                data: { fname: data.name },
                file: data,
                ossModule: 'tmp20230813',
                ossParams: { fname: data.name }
            })
        })
    }
    
    await runThread(async () => await p({
        name: '数据同步',
        url: u('local://api/tmp/20230813/data'),
        on_ok(res) {
            fileList.value = res.data.fileList
        },
        type: 'loop'
    }))
</script>