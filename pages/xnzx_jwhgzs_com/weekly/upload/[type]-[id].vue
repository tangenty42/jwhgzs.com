<template>
    <div class="box box_md hcenter">
        <span class="box_title">新宁空间</span>
        <span class="box_graytitle">
            班级周报 - {{ title }}
        </span>
        <div>
            <div v-if="type == 'upload'" class="box_tip2">
                <span class="spanAwesome" style="padding-bottom: 5px;">提示：你输入的内容实时保存哦，不用担心！</span>
                <el-button size="small" plain @click="clear">清空表单</el-button>
            </div>
            <el-divider v-if="type != 'upload'"></el-divider>
            <div v-if="type == 'upload' || type == 'edit'">
                <el-form label-width="8rem" :model="formData" label-position="top">
                    <div v-if="type == 'upload'">
                        <el-form-item label="周报" prop="id">
                            <el-select v-model="formData.id" placeholder=" 点击选择周报" @change="resetPeople(0); saveContent()">
                                <template #prefix>
                                    <i class="fas fa-calendar-alt"></i>
                                </template>
                                <template v-for="(v, k) in weeklyList" :key="k">
                                    <el-option v-if="! v.finished" :label="genWeeklyTitle(v)" :value="v.id"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="打稿人（你）" prop="typist">
                            <el-select v-model="formData.typist" placeholder=" 点击选择打稿人（你）" no-data-text="请先选择周报" @change="resetPeople(0, 1); saveContent()">
                                <template #prefix>
                                    <i class="fas fa-keyboard"></i>
                                </template>
                                <template v-for="(v, k) in peopleList.typists" :key="k">
                                    <el-option :label="v[0] + '号 ' + v[1]" :value="v[0]"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="稿件作者" prop="author">
                            <el-select v-model="formData.author" placeholder=" 点击选择稿件作者" no-data-text="请先选择打稿人" @change="saveContent">
                                <template #prefix>
                                    <i class="fas fa-user-edit"></i>
                                </template>
                                <template v-for="(v, k) in peopleList.authors" :key="k">
                                    <el-option v-if="isTypistMatchAuthor(getWeeklyListById(formData.id), formData.typist, v[0])" :label="v[0] + '号 ' + v[1]" :value="v[0]"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                    </div>
                    <div v-if="type == 'edit'">
                        <el-form-item label="修改稿件ID" prop="id">
                            <el-input type="number" v-model="formData.id" disabled>
                                <template #prefix>
                                    <i class="fas fa-list-ol"></i>
                                </template>
                            </el-input>
                        </el-form-item>
                    </div>
                    <el-divider>稿件内容</el-divider>
                    <el-form-item label="标题" prop="title">
                        <el-input type="text" v-model="formData.title" @input="saveContent">
                            <template #prefix>
                                <i class="fas fa-heading"></i>
                            </template>
                        </el-input>
                    </el-form-item>
                    <el-form-item label="题记（选填，没有就别填！）" prop="tiji">
                        <el-input type="textarea" v-model="formData.tiji" placeholder="没有就别填！" :rows="3" :maxlength="config.XNZX_WEEKLY_CONFIG.tijiMaxLength" show-word-limit @input="saveContent"></el-input>
                    </el-form-item>
                    <el-form-item label="正文" prop="content">
                        <el-input type="textarea" v-model="formData.content" :placeholder="'如有小标题请在小标题前加上英文井号 # 哦！\n注意正文不包括题目和题记、后记！'" :rows="20" :maxlength="config.XNZX_WEEKLY_CONFIG.contentMaxLength" show-word-limit @input="saveContent"></el-input>
                    </el-form-item>
                    <el-form-item label="后记（选填，没有就别填！）" prop="houji">
                        <el-input type="textarea" v-model="formData.houji" placeholder="没有就别填！" :rows="3" :maxlength="config.XNZX_WEEKLY_CONFIG.houjiMaxLength" show-word-limit @input="saveContent"></el-input>
                    </el-form-item>
                    <!--
                    <el-form-item label="图片（选填）" prop="attach">
                        <el-button :type="formData.attach ? 'danger' : 'primary'" @click="formData.attach ? delAttach() : selectAttach()" plain>
                            {{ formData.attach ? '删除图片：' + formData.attach.name : '选择图片' }}
                        </el-button>
                    </el-form-item>
                    -->
                    <el-divider style="margin-top: 60px;"></el-divider>
                    <div style="width: 100%; text-align: center;">
                        <el-button type="primary" size="large" style="width: 50%;" @click="doit(0)" plain round>
                            提交
                        </el-button>
                    </div>
                </el-form>
            </div>
            <div v-if="type == 'adminnew' || type == 'adminedit'">
                <el-form :model="formData" label-position="top">
                    <div v-if="type == 'adminedit'">
                        <el-form-item label="修改周报ID" prop="id">
                            <el-input type="number" v-model="formData.id" disabled>
                                <template #prefix>
                                    <i class="fas fa-list-ol"></i>
                                </template>
                            </el-input>
                        </el-form-item>
                        <el-divider></el-divider>
                    </div>
                    <el-form-item label="届数" prop="year">
                        <el-select v-model="formData.year" placeholder=" 点击选择届数" @change="resetPeople(1)">
                            <template #prefix>
                                <i class="fas fa-school"></i>
                            </template>
                            <template v-for="(v, k) in weeklyData.years" :key="k">
                                <el-option :label="v + '秋届'" :value="v"></el-option>
                            </template>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="班级" prop="class_">
                        <el-select v-model="formData.class_" placeholder=" 点击选择班级" @change="resetPeople(1)" no-data-text="请先选择届数">
                            <template #prefix>
                                <i class="fas fa-graduation-cap"></i>
                            </template>
                            <template v-if="formData.year">
                                <template v-for="(v, k) in weeklyData.classes[formData.year]" :key="k">
                                    <el-option :label="v + '班'" :value="v"></el-option>
                                </template>
                            </template>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="学期" prop="term">
                        <el-select v-model="formData.term" placeholder=" 点击选择学期">
                            <template #prefix>
                                <i class="fas fa-calendar-week"></i>
                            </template>
                            <template v-for="(v, k) in config.XNZX_WEEKLY_CONFIG.terms" :key="k">
                                <el-option :label="v" :value="v"></el-option>
                            </template>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="期数" prop="num">
                        <el-input type="number" v-model="formData.num">
                            <template #prefix>
                                <i class="fas fa-list-ol"></i>
                            </template>
                        </el-input>
                    </el-form-item>
                    <el-form-item label="备注（选填）" prop="note">
                        <el-input type="text" v-model="formData.note">
                            <template #prefix>
                                <i class="fas fa-sticky-note"></i>
                            </template>
                        </el-input>
                    </el-form-item>
                    <el-form-item label="人员安排" prop="">
                        <div class="group" style="margin: 0 !important;">
                            <div>
                                <el-select v-model="tmpData.author" style="margin-right: 10px;" placeholder=" 点击选择稿件作者" no-data-text="请先选择班级">
                                    <template #prefix>
                                        <i class="fas fa-user-edit"></i>
                                    </template>
                                    <el-option v-for="(v, k) in allStudents" :key="k" :label="v[0] + '号 ' + v[1]" :value="v[0]"></el-option>
                                </el-select>
                                <el-select v-model="tmpData.typist" placeholder=" 点击选择打稿人" no-data-text="请先选择班级">
                                    <template #prefix>
                                        <i class="fas fa-keyboard"></i>
                                    </template>
                                    <el-option v-for="(v, k) in allStudents" :key="k" :label="v[0] + '号 ' + v[1]" :value="v[0]"></el-option>
                                </el-select>
                            </div>
                            <!-- margin-bottom: -10px用来消掉btns设置的尾margin（间margin还是有必要的） -->
                            <div style="margin-top: 10px; margin-bottom: -10px;">
                                <el-button class="btns" :disabled="! (tmpData.author && tmpData.typist)" type="primary" plain @click="addPeople()">添加</el-button>
                                <el-button class="btns" plain @click="addPeople2()">批量添加</el-button>
                                <el-button class="btns" type="danger" plain @click="delAllPeople()">清除全部</el-button>
                            </div>
                        </div>
                        <el-table :data="formData.people" max-height="300">
                            <el-table-column prop="" label="稿件作者" min-width="150">
                                <template #default="scope">
                                    {{ scope.row[0] + '号 ' + getSnameBySid(scope.row[0], formData.class_) }}
                                </template>
                            </el-table-column>
                            <el-table-column prop="" label="打稿人" min-width="150">
                                <template #default="scope">
                                    {{ scope.row[1] + '号 ' + getSnameBySid(scope.row[1], formData.class_) }}
                                </template>
                            </el-table-column>
                            <el-table-column prop="" label="操作">
                                <template #default="scope">
                                    <el-button size="small" type="danger" plain @click="deletePeople(scope.$index)">删除</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                    </el-form-item>
                    <el-form-item label="是否隐藏作者名" prop="nameInvisible">
                        <el-select v-model="formData.nameInvisible" placeholder=" 点击选择是否隐藏作者名">
                            <template #prefix>
                                <i class="fas fa-user-secret"></i>
                            </template>
                            <el-option label="不隐藏" :value="0"></el-option>
                            <el-option label="隐藏" :value="1"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-divider style="margin-top: 60px;"></el-divider>
                    <div style="width: 100%; text-align: center;">
                        <el-button type="primary" size="large" style="width: 50%;" @click="doit(1)" plain round>
                            提交
                        </el-button>
                    </div>
                </el-form>
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
            ['local://xnzx/weekly', '班级周报'],
            ['', '上传']
        ],
        mustLogin: true
    })
    
    // -- refs
    let type = ref(getRouteParam('type')),
        weeklyList = ref(false),
        weeklyData = ref(false),
        formData = ref({
            id: isNum(getRouteParam('id')) ? getRouteParam('id') : '',
            typist: '',
            author: '',
            title: '',
            tiji: '',
            content: '',
            attach: null,
            
            year: '',
            class_: '',
            term: '',
            num: '',
            note: '',
            people: [],
            nameInvisible: 0
        }),
        tmpData = ref({
            author: '',
            typist: ''
        }),
        peopleList = ref({
            authors: [],
            typists: []
        })
    let allStudents = computed(() => {
            if (weeklyData.value && formData.value.class_)
                return weeklyData.value.students[parseInt(formData.value.year)][parseInt(formData.value.class_)]
            else
                return []
        }),
        title = computed(() => {
            switch (getRouteParam('type')) {
                case 'upload':
                    return '上传稿件'
                case 'edit':
                    return '修改稿件'
                case 'adminnew':
                    return '发布周报'
                case 'adminedit':
                    return '修改周报'
                default:
                    return ''
            }
        })
    let firstLoop = true
    
    // -- app
    function init() {
        if (type.value == 'edit' || type.value == 'adminedit') {
            let data = { selectAll: true }
            if (type.value == 'edit')
                data.aid = formData.value.id
            else
                data.id = formData.value.id
            p({
                name: '数据初始化',
                url: u('local://api/xnzx/weekly/detail'),
                data: data,
                on_ok(res) {
                    if (type.value == 'edit') {
                        let l = res.data.articleList, d = []
                        for (let i in l) {
                            let v = l[i]
                            if (v.id == formData.value.id) {
                                d = v
                            }
                        }
                        formData.value.title = d.title
                        formData.value.tiji = d.tiji
                        formData.value.content = d.oriContent
                        formData.value.houji = d.houji
                    } else {
                        let d = res.data.weeklyDetail
                        let d2 = res.data.articleList
                        formData.value.year = d.year
                        formData.value.class_ = d.class_
                        formData.value.term = d.term
                        formData.value.num = d.num
                        formData.value.note = d.note
                        formData.value.nameInvisible = parseInt(d.nameInvisible)
                        let people = []
                        for (let i in d2) {
                            let v = d2[i]
                            people.push([parseInt(v.author[0]), parseInt(v.typist[0])])
                        }
                        formData.value.people = people
                    }
                },
                on_err: () => '本错误非致命，你可以继续进行操作~',
                type: 'money'
            })
        } else if (type.value == 'upload') {
            if (localStorage.getItem('xnzx_weekly_form')) {
                let d = JSON.parse(localStorage.getItem('xnzx_weekly_form'))
                formData.value.id = d.id
                formData.value.title = d.title
                formData.value.tiji = d.tiji
                formData.value.content = d.content
                formData.value.houji = d.houji
                resetPeople(0)
                formData.value.author = d.author
                formData.value.typist = d.typist
            }
        }
    }
    function genWeeklyTitle(v) {
        return v.year + '秋届' + v.class + '班 ' + v.term + ' 第' + v.num + '期 ' + v.note
    }
    function doit(isNew) {
        vaptchaGo(config.value.VAPTCHA_CONFIG.scenes.important, (vaptchaData) => {
            let data = Object.assign(formData.value, vaptchaData, { type: type.value })
            p({
                name: '周报上传',
                url: u('local://api/xnzx/weekly/upload'),
                data: data,
                file: formData.value.attach,
                ossModule: 'weekly',
                ossParams: { id: formData.value.id, title: formData.value.title, type: type.value },
                on_ok() {
                    _clear()
                    return '即将返回主页'
                },
                jump: () => j(u('local://xnzx/weekly'))
            })
        })
    }
    function getWeeklyListById(id) {
        for (let k in weeklyList.value) {
            let v = weeklyList.value[k]
            if (v.id == id) {
                return v
            }
        }
        return false
    }
    function getSnameBySid(sid, class_) {
        let d = weeklyData.value.students[formData.value.year][class_]
        for (let k in d) {
            let v = d[k]
            if (v[0] == sid)
                return v[1]
        }
        return false
    }
    function isTypistMatchAuthor(list, typist, author) {
        for (let k in list.typists) {
            let v = list.typists[k]
            if (v[0] == typist && list.authors[k][0] == author) {
                return true
            }
        }
        return false
    }
    function saveContent() {
        if (type.value == 'upload')
            localStorage.setItem('xnzx_weekly_form', JSON.stringify(formData.value))
    }
    function _clear() {
        localStorage.setItem('xnzx_weekly_form', '')
    }
    function clear() {
        confMsg({
            content: '确定要清空已输入的表单内容吗？',
            type: 'warning',
            callback_ok() {
                _clear()
                // j() without url means reload
                j()
            }
        })
    }
    function resetPeople(type_, isTypistChange) {
        if (type_ == 0) {
            if (! isTypistChange) {
                formData.value.typist = formData.value.author = ''
                peopleList.value.typists = getWeeklyListById(formData.value.id).typists
                peopleList.value.authors = getWeeklyListById(formData.value.id).authors
            } else {
                formData.value.author = ''
                peopleList.value.authors = getWeeklyListById(formData.value.id).authors
            }
        } else if (type_ == 1) {
            formData.value.tmpData = {
                author: '',
                typist: ''
            }
        }
    }
    function addPeople() {
        formData.value.people.push([ tmpData.value.author, tmpData.value.typist ])
    }
    function addPeople2() {
        if (! formData.value.class_) {
            errMsg('请先选择班级哦~')
            return
        }
        inputMsg({
            content: '请选择批量添加模式：',
            inputType: 'select',
            callback_ok(value) {
                switch (value) {
                    case 0:
                        inputMsg({
                            content: '格式（A = 打稿人学号，B = 稿件作者学号）：<br/><br/>A B,B,B,...<br/><br/>（注意A、B中间有空格；每个多个稿件作者用英文逗号“,”分割；每行对应一个打稿人学号，即设置完一个打稿人请换行）',
                            inputType: 'textarea',
                            callback_ok(value) {
                                try {
                                    let rows = (value ? value : '').split('\n'), list = []
                                    for (let k in rows) {
                                        let v = rows[k]
                                        if (! v.trim())
                                            continue
                                        let arr = v.split(' ')
                                        let typist = parseInt(arr[0]), authors = arr[1].split(',')
                                        for (let k2 in authors) {
                                            let author = parseInt(authors[k2])
                                            list.push([ author, typist ])
                                        }
                                    }
                                    formData.value.people = list
                                    succMsg('批量添加成功~')
                                }
                                catch {
                                    errMsg('批量添加失败！请检查输入格式哦~~')
                                }
                            }
                        })
                        break
                    case 1:
                        let list = []
                        let students = weeklyData.value.students[parseInt(formData.value.year)][parseInt(formData.value.class_)]
                        for (let k in students) {
                            let v = students[k]
                            list.push([ v[0], v[0] ])
                        }
                        formData.value.people = list
                        succMsg('批量添加成功~')
                        break
                    default:
                        break
                }
            },
            inputOptions: [
                {label: '快速1对1添加', value: 0},
                {label: '全班自打', value: 1}
            ]
        })
    }
    function deletePeople(index) {
        formData.value.people.splice(index, 1)
    }
    function delAllPeople() {
        confMsg({
            content: '确定要清除全部人员安排嘛？',
            type: 'warning',
            callback_ok() {
                formData.value.people = []
                succMsg('清除成功~')
            }
        })
    }
    function selectAttach() {
        selectFile((data) => {
            formData.value.attach = data
        })
    }
    function delAttach() {
        formData.value.attach = null
    }
    
    await runThread(async () => await p({
        name: '数据同步',
        url: u('local://api/xnzx/weekly'),
        on_ok(res) {
            weeklyList.value = res.data.weeklyList
            weeklyData.value = res.data.weeklyData
            
            if (firstLoop)
                init()
            
            firstLoop = false
        },
        on_err: () => '即将返回主页',
        jump_err: () => j(u('local://xnzx/weekly')),
        type: 'loop'
    }))
</script>