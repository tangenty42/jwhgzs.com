<style scoped>
    .people_item {
        margin-bottom: 10px;
        
        border: 1px solid #ddd;
        border-radius: 6px;
        padding: 20px;
    }
    .people_boy {
        color: #0080FF;
    }
    .people_girl {
        color: #FBB9DF;
    }
    .people_name {
        font-size: 200%;
        font-weight: bold;
    }
    .people_name_gray {
        color: gray;
    }
    .people_teacher {
        color: yellowgreen;
    }
    .tip {
        margin: 10px;
        padding: 10px;
        text-align: center;
    }
    .describe {
        padding: 15px;
        border-radius: 5px;
        background-color: #f5f7fa;
        margin-bottom: 10px;
        
        white-space: pre-wrap;
    }
</style>

<template>
    <div class="box box_ml hcenter">
        <span class="box_title">新宁空间</span>
        <span class="box_graytitle">纪念册</span>
        <el-divider></el-divider>
        <div v-for="(v, k) in classes">
            <el-check-tag v-for="(v2, k2) in v" :checked="classChecked[0] == k && classChecked[1] == v2" @change="classCheck(k, v2)">
                {{ k }} 秋届 {{ v2 }} 班
            </el-check-tag>
        </div>
        <br/>
        <div class="describe">
            <span style="color: gray; font-weight: bold;">{{ classData.PA_slogan }}</span>
            <span style="color: gray;" v-if="! classData.PA_slogan">还木有标语哦~</span>
            <el-button v-show="isAdmin" @click="editSlogan" style="margin-left: 10px" circle>
                <i class="fas fa-edit"></i>
            </el-button>
        </div>
        <el-carousel v-if="classData && classData.PA_photosName.length > 0" :interval="6000" trigger="click" arrow="always" indicator-position="outside">
            <el-carousel-item v-for="v in classData.PA_photosName">
                <el-button v-show="isAdmin" style="position: absolute; z-index: 26;" size="small" type="danger" @click="delPhotos(v)">
                    <i class="fas fa-trash-alt"></i>&nbsp;删除
                </el-button>
                <el-image style="width: 100%; height: 100%" class="vcenter" fit="contain" :src="buildDefaultImgUrl(v)" :preview-src-list="[buildDefaultImgUrl(v)]" preview-teleported></el-image>
            </el-carousel-item>
        </el-carousel>
        <el-button v-show="isAdmin" size="small" type="primary" @click="uploadPhotos">
            <i class="fas fa-upload"></i>&nbsp;上传照片
        </el-button>
        <br/><br/>
        <el-input v-model="searchTag" type="text" size="large" placeholder="找不到你想要的？请输入关键词搜索">
            <template #prefix>
                <i class="fas fa-search"></i>
            </template>
        </el-input>
        <div class="tip">
            <strong>
                <span style="color: #909399;">灰色信息</span>，<span style="color: #409EFF;">蓝色外号</span>，<span style="color: #67C23A;">绿色梗</span>
            </strong>
        </div>
        <el-skeleton :loading="! peopleList" :throttle="100" :rows="10" animated>
            <template #default>
                <div style="overflow-y: scroll; height: 100vh;">
                    <el-row justify="space-between">
                        <el-col v-for="(v, k) in peopleList" :lg="11" :key="v.id">
                            <div class="people_item">
                                <custom-a class="aAwesomeX" :href="u('local://xnzx/PA/detail') + '/' + v.id">
                                    <el-row>
                                        <el-col style="margin-right: 20px;">
                                            <div class="people_name" :class="{ people_name_gray: parseInt(v.disabled), people_boy: (v.sex == 1), people_girl: (v.sex == 2), people_teacher: (v.type == 1) }">{{ v.name }}</div>
                                            <avatar :src="u('static://user/PA') + '/' + v.id + '/' + paAvatarFile(v.PA_photosName) + '?v=' + v.PA_photosVersion" :size="100" :lazy="true" style="margin: 10px 0;"></avatar>
                                        </el-col>
                                        <el-col>
                                            <el-tag effect="plain" v-if="v.type == 0" type="info" size="small" class="btns2">
                                                {{ v.year }} 秋届 {{ v.class }} 班 {{ v.sid }} 号
                                            </el-tag>
                                            <el-tag effect="plain" v-if="v.type == 1" type="info" size="small" class="btns2">
                                                {{ v.year }} 秋届 
                                            </el-tag>
                                            <el-tag effect="plain" size="small" class="btns2">
                                                <i class="fas fa-thumbs-up"></i>&nbsp;获赞：{{ v.PA_likes }}
                                            </el-tag>
                                            <el-divider></el-divider>
                                            <div>
                                                <template v-for="(v2, k2) in v.PA_properties">
                                                    <el-tag v-if="v2.name" style="font-weight: bold;" :type="getPropColor(v2.type)" class="btns2">
                                                        {{ v2.name }}
                                                    </el-tag>
                                                </template>
                                            </div>
                                        </el-col>
                                    </el-row>
                                </custom-a>
                            </div>
                        </el-col>
                    </el-row>
                </div>
            </template>
        </el-skeleton>
    </div>
</template>

<script setup>
    // -- basics
    let config = useState('config')
    definePageMeta({
        titles: [
            ['local://xnzx', '新宁空间'],
            ['local://xnzx/PA', '纪念册'],
            ['', '主页']
        ]
    })
    
    // -- refs
    let classes = config.value.XNZX_PA_CONFIG.classes,
        yc = getRouteParam('yc')
            ? getRouteParam('yc').split('_')
            : (function() {
                let arr = classes
                // 利用for in的执行顺序特性获取第一项
                for (let k in arr) {
                    let v = arr[k]
                    return [k, arr[k][0]]
                }
            })()
    let udata = useState('udata'),
        classChecked = ref(yc),
        peopleList = ref(false),
        classData = ref(false),
        PAData = ref(false),
        searchTag = ref(''),
        isAdmin = computed(() => {
            return udata.value && inArray(parseInt(udata.value.id), PAData.value.adminUids)
        })
    
    // -- app
    function classCheck(k, k2) {
        j(u('local://xnzx/PA/' + k +'_' + k2))
    }
    function buildDefaultImgUrl(i) {
        return u('static://user/PA') + '/' + classChecked.value[0] + '_' + classChecked.value[1] + '/'
            + paPhotoFile(i) + '?v=' + classData.value.PA_photosVersion
    }
    function uploadPhotos() {
        selectFile(function (data) {
            p({
                name: '上传照片',
                url: u('local://api/xnzx/PA/photosUpload'),
                data: { isClass: true, id: classChecked.value.join('_') },
                file: data,
                ossModule: 'PA',
                ossParams: { isClass: true, id: classChecked.value.join('_'), mode: 0 }
            })
        })
    }
    function delPhotos(name) {
        vaptchaGo(config.value.VAPTCHA_CONFIG.scenes.important, (vaptchaData) => {
            p({
                name: '删除照片',
                url: u('local://api/xnzx/PA/photosDel'),
                data: Object.assign({ isClass: true, id: classChecked.value.join('_'), name: name }, vaptchaData)
            })
        })
    }
    function editSlogan() {
        inputMsg({
            content: '请输入班级标语：',
            inputType: 'textarea',
            callback_ok(txt) {
                p({
                    name: '修改班级标语',
                    url: u('local://api/xnzx/PA/editClassSlogan'),
                    data: { id: classChecked.value, slogan: txt }
                })
            },
            defaultValue: classData.value.PA_slogan
        })
    }
    function getPropColor(type) {
        switch (parseInt(type)) {
            case - 1:
                return 'warning'
            case 0:
                return 'info'
            case 1:
                return ''
            case 2:
                return 'success'
            default:
                return ''
        }
    }
    
    function syncData() {
        const requestedSearchTag = searchTag.value
        return p({
            name: '数据同步',
            url: u('local://api/xnzx/PA'),
            data: { year: classChecked.value[0], 'class': classChecked.value[1], searchTag: requestedSearchTag },
            on_ok(data) {
                if (
                    requestedSearchTag == searchTag.value
                    && ((! requestedSearchTag) || data.data.searchTag == requestedSearchTag)
                ) peopleList.value = data.data.peopleList
                classData.value = data.data.classData
                PAData.value = data.data.PAData
            },
            on_err: () => '即将返回主页~',
            jump_err: () => j(u('local://xnzx')),
            type: 'loop'
        })
    }
    const throttledSyncData = useThrottleFn(syncData, 300, true)
    watch(searchTag, () => {
        peopleList.value = null
        throttledSyncData()
    })
    
    await runThread(syncData)
</script>