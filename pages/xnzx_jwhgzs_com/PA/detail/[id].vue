<style scoped>
    .people_inf {
        margin-left: 20px;
    }
    .people_boy {
        color: #0080FF;
    }
    .people_girl {
        color: #FBB9DF;
    }
    .people_name {
        margin-right: 10px;
        font-size: 200%;
        font-weight: bold;
    }
    .people_property {
        font-size: 135%;
    }
    .people_likes {
        margin-left: 10px;
    }
    .people_jwh {
        margin-bottom: 6px;
        font-size: 80%;
        color: gray;
    }
    .people_teacher {
        color: yellowgreen;
    }
    .tip {
        margin: 10px;
        padding: 10px;
        text-align: center;
        border-top: 1px solid #ddd;
    }
    .tip2 {
        font-size: 80%;
        color: gray;
    }
    .prop_stressed {
        border-width: 3px;
    }
    .prop_stressed li {
        font-weight: bold;
        font-size: 135%;
    }
    .tag {
        padding: 16px !important;
    }
</style>

<template>
    <div class="box box_ml hcenter">
        <span class="box_title">新宁空间</span>
        <span class="box_graytitle">纪念册 - 详情</span>
        <el-divider></el-divider>
        <el-row>
            <el-col :span="-1">
                <avatar :src="u('static://user/PA') + '/' + peopleDetail.id + '/' + paAvatarFile(peopleDetail.PA_photosName) + '?v=' + peopleDetail.PA_photosVersion" :size="100"></avatar>
            </el-col>
            <el-col :span="-1" class="people_inf vcenter">
                <span class="people_name" :class="{ people_name_gray: parseInt(peopleDetail.disabled), people_boy: (peopleDetail.sex == 1), people_girl: (peopleDetail.sex == 2), people_teacher: (peopleDetail.type == 1) }">{{ peopleDetail.name }}</span>
            </el-col>
        </el-row>
        <el-divider><strong>个人信息</strong></el-divider>
        <el-tag v-if="peopleDetail.type == 0" type="info" class="btns2">
            {{ peopleDetail.year }} 秋届 {{ peopleDetail.class }} 班 {{ peopleDetail.sid }} 号
        </el-tag>
        <el-tag v-if="peopleDetail.type == 1" type="info" class="btns2">
            {{ peopleDetail.year }} 秋届 
        </el-tag>
        <el-tag class="btns2">
            <i class="fas fa-thumbs-up"></i>&nbsp;获赞：{{ peopleDetail.PA_likes }}
        </el-tag>
        <el-row>
            <el-col class="vcenter people_jwh">
                <strong>九尾狐账号：</strong>
            </el-col>
            <el-col>
                <span v-if="! udata2" class="people_jwh">ta还没有九尾狐账号哦~</span>
                <user-infobox v-else :udata="udata2" :single="true"></user-infobox>
            </el-col>
        </el-row>
        <el-divider><strong>照片</strong></el-divider>
            <el-carousel v-if="peopleDetail && peopleDetail.PA_photosName.length" :interval="6000" trigger="click" arrow="always" indicator-position="outside">
                <el-carousel-item v-for="v in peopleDetail.PA_photosName">
                    <el-button v-show="isAdmin" style="position: absolute; z-index: 26;" size="small" type="danger" @click="delPhotos(v)">
                        <i class="fas fa-trash-alt"></i>&nbsp;删除
                    </el-button>
                    <el-image style="width: 100%; height: 100%" class="vcenter" fit="contain" :src="buildDefaultImgUrl(v)" :preview-src-list="[buildDefaultImgUrl(v)]" preview-teleported></el-image>
                </el-carousel-item>
            </el-carousel>
        <div class="tip2" v-else>暂无照片~</div>
        <el-button v-show="isAdmin" style="margin-top: 10px;" size="small" type="primary" @click="uploadPhotos">
            <i class="fas fa-upload"></i>&nbsp;上传照片
        </el-button>
        <div class="tip">
            <strong>
                <span style="color: #909399;">灰色信息</span>，<span style="color: #409EFF;">蓝色外号</span>，<span style="color: #67C23A;">绿色梗</span>
            </strong>
        </div>
        <!-- 这里曾经被坑过，按vue的推荐加了:key属性后会导致一直热更新，管他数据变没变就是一个劲的刷新…… -->
        <!-- 以后记住了，除非不加:key有问题，就别加了 -->
        <div>
            <template v-for="(v, k) in peopleDetail.PA_properties">
                <el-tag :type="getPropColor(v.type)" class="tag btns2" size="large"
                        :class="{ prop_stressed: v.type == - 1 }">
                    <strong class="people_property" v-html="v.name"></strong>
                    <div v-if="v.detail && v.detail.length && v.detail[0].length">
                        <li v-for="v in v.detail" v-html="v" class="aAwesomeY"></li>
                    </div>
                    <br/>
                    <client-only>
                        <el-link class="people_likes" :type="v.liked ? 'primary' : 'default'" @click="like(v.id)">
                            <i class="fas fa-thumbs-up"></i>&nbsp;{{ v.likes }}
                        </el-link>
                    </client-only>
                    <el-link v-show="isAdmin" class="people_likes" @click="editProp(v)">
                        <i class="fas fa-edit"></i>&nbsp;编辑
                    </el-link>
                    <el-link v-show="isAdmin" class="people_likes" type="danger" @click="delProp(v.id)">
                        <i class="fas fa-trash-alt"></i>&nbsp;删除
                    </el-link>
                </el-tag>
            </template>
            <div v-show="isAdmin">
                <br/>
                <!-- 注意v-for这样（(v, k) in num）用，v值从1开始，k值从0开始 -->
                <el-button v-for="(v, k) in 4" size="small" :type="getPropColor(- 1 + k) || 'primary'" @click="addProp(- 1 + k)">
                    <i class="fas fa-plus-circle"></i>&nbsp;添加{{ getPropName(- 1 + k) }}
                </el-button>
            </div>
        </div>
    </div>
</template>

<script setup>
    // -- basics
    let config = useState('config')
    definePageMeta({
        titles: [
            ['local://xnzx', '新宁空间'],
            ['local://xnzx/PA', '纪念册'],
            ['', '详情']
        ]
    })
    
    // -- refs
    let udata = useState('udata'),
        peopleDetail = ref(false),
        PAData = ref(false),
        udata2 = ref(false),
        isAdmin = computed(() => {
            return udata.value && inArray(parseInt(udata.value.id), PAData.value.adminUids)
        })
    
    // -- app
    function buildDefaultImgUrl(i) {
        return u('static://user/PA') + '/' + peopleDetail.value.id + '/' + paPhotoFile(i) + '?v=' + peopleDetail.value.PA_photosVersion
    }
    function like(id) {
        p({
            name: '点赞',
            url: u('local://api/xnzx/PA/like'),
            data: { id: id }
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
    function getPropName(type) {
        switch (parseInt(type)) {
            case - 1:
                return '置顶信息'
            case 0:
                return '信息'
            case 1:
                return '外号'
            case 2:
                return '梗'
            default:
                return ''
        }
    }
    globalThis.$aTemplate = `<a href="javascript: j(/* 链接地址表达式 */, '_blank')">链接标题</a>`
    let propDetailNote = `<br/><a class="aAwesomeZ" href="javascript: copy(globalThis.$aTemplate)">复制链接模板代码</a>`
    let tortureList = [
        {
            name: '标题',
            vname: 'name',
            type: 'text'
        },
        {
            name: '内容',
            note: propDetailNote,
            vname: 'detail',
            type: 'textarea'
        }
    ]
    function addProp(type) {
        torture({
            list: tortureList,
            next(v) {
                p({
                    name: '添加词条',
                    url: u('local://api/xnzx/PA/addProp'),
                    data: Object.assign({ pid: peopleDetail.value.id, type: type }, v)
                })
            }
        })
    }
    function editProp(odata) {
        let odata2 = Object.assign({}, odata)
        odata2.detail = odata2.detail.join('\n')
        torture({
            isEdit: true,
            oldData: odata2,
            list: tortureList,
            next(v) {
                p({
                    name: '修改词条',
                    url: u('local://api/xnzx/PA/editProp'),
                    data: Object.assign({ id: odata.id }, v)
                })
            }
        })
    }
    function delProp(id) {
        vaptchaGo(config.value.VAPTCHA_CONFIG.scenes.important, (vaptchaData) => {
            p({
                name: '删除词条',
                url: u('local://api/xnzx/PA/delProp'),
                data: Object.assign({ pid: peopleDetail.value.id, id: id }, vaptchaData)
            })
        })
    }
    function uploadPhotos() {
        inputMsg({
            content: '请选择操作：',
            inputType: 'select',
            callback_ok(mode) {
                selectFile(function (data) {
                    p({
                        name: '上传照片',
                        url: u('local://api/xnzx/PA/photosUpload'),
                        data: { id: peopleDetail.value.id, mode: mode },
                        file: data,
                        ossModule: 'PA',
                        ossParams: { id: peopleDetail.value.id, mode: mode }
                    })
                })
            },
            inputOptions: [
                {
                    label: '上传照片',
                    value: 0
                },
                {
                    label: '更换头像',
                    value: 1
                },
            ]
        })
    }
    function delPhotos(name) {
        vaptchaGo(config.value.VAPTCHA_CONFIG.scenes.important, (vaptchaData) => {
            p({
                name: '删除照片',
                url: u('local://api/xnzx/PA/photosDel'),
                data: Object.assign({ id: peopleDetail.value.id, name: name }, vaptchaData)
            })
        })
    }
    
    await runThread(async () => await p({
        name: '数据同步',
        url: u('local://api/xnzx/PA/detail'),
        data: { id: getRouteParam('id') },
        on_ok: (data) => {
            peopleDetail.value = data.data.peopleDetail
            udata2.value = data.data.udata
            PAData.value = data.data.PAData
        },
        on_err: () => '即将返回主页~',
        jump_err: () => j(u('local://xnzx/PA')),
        type: 'loop'
    }))
</script>