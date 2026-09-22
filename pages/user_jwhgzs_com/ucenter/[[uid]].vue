<style scoped>
    .avatar {
        margin-right: 10px;
    }
    .name {
        margin-left: 25px;
        font-size: 200%;
        font-weight: bold;
        color: gray;
    }
    .main1 {
        margin-right: 25px;
    }
    .ske_avatar {
        width: 120px;
        height: 120px;
        margin-right: 30px;
    }
    .ske_info {
        margin-top: 20px;
        height: 200px;
    }
    .changeAvatar {
        position: absolute;
        width: 120px;
        
        text-align: center;
    }
    .changeAvatar_btn {
        margin-top: -20px;
    }
    .col1 {
        margin-right: calc(100% / 24);
        margin-bottom: 20px;
    }
    .selfintroduce {
        padding: 15px;
        border-radius: 5px;
        background-color: #f5f7fa;
        margin-bottom: 10px;
        
        white-space: pre-wrap;
    }
    .userTag {
        font-size: 120%;
        font-weight: bold;
    }
</style>

<template>
    <el-row>
        <el-col class="col1" :md="11">
            <div class="box">
                <el-skeleton :loading="! ucdata" :rows="4" animated>
                    <template #template>
                        <el-skeleton-item variant="circle" class="ske_avatar"></el-skeleton-item>
                        <el-skeleton-item variant="h3" class="ske_info"></el-skeleton-item>
                    </template>
                    <template #default>
                        <el-row class="main1">
                            <el-col :span="-1" style="margin-bottom: 30px">
                                <user-avatar :udata="ucdata" :size="120"></user-avatar>
                                <div v-if="ucenterOwnerIsMe" class="changeAvatar">
                                    <el-button size="large" class="changeAvatar_btn" @click="changeAvatar" circle>
                                        <i class="fas fa-redo"></i>
                                    </el-button>
                                </div>
                            </el-col>
                            <el-col :span="-1" class="vcenter" style="margin-bottom: 30px">
                                <div :class="{ name: true, authed_uname: (ucdata.userGroup || ucdata.userAuth) }">{{ ucdata.name }}</div>
                            </el-col>
                        </el-row>
                        <div v-if="ucdata.isMe || ucdata.selfIntroduce" class="selfintroduce">
                            <span v-if="ucdata.selfIntroduce">{{ ucdata.selfIntroduce }}</span>
                            <span v-if="! ucdata.selfIntroduce" style="color: gray">
                                {{ $t('no_bio_tip') }}
                            </span>
                            <el-button v-if="ucenterOwnerIsMe" @click="setUdata('selfIntroduce', $t('_bio'), 'textarea', ucdata.selfIntroduce)" style="margin-left: 10px" circle>
                                <i class="fas fa-edit"></i>
                            </el-button>
                        </div>
                        <el-divider></el-divider>
                        <div>
                            <template v-for="(v, k) in ucdata.userTags[0]">
                                <el-tag size="large" type="success" class="btns2 userTag">
                                    {{ v }}
                                </el-tag>
                            </template>
                            <template v-if="ucdata.userAuth" v-for="v in ucdata.userAuth.split('\n')">
                                <el-tag size="large" class="btns2 userTag">{{ v }}</el-tag>
                            </template>
                        </div>
                    </template>
                </el-skeleton>
            </div>
        </el-col>
        <el-col :md="12">
            <div class="box">
                <div class="box_title">{{ $t('personal_info') }}</div>
                <el-skeleton style="margin-top: 20px" :loading="! ucdata" :rows="6" animated>
                    <template #default>
                        <el-descriptions style="margin-top: 20px" :column="1" direction="vertical" border>
                            <el-descriptions-item>
                                <template #label>
                                    <i class="fas fa-id-badge"></i>
                                    &nbsp;
                                    {{ $t('form_user_id') }}
                                </template>
                                {{ ucdata.id }}
                            </el-descriptions-item>
                            <el-descriptions-item v-if="! ucdata.isMe">
                                <template #label>
                                    <i class="fas fa-signal"></i>
                                    &nbsp;
                                    {{ $t('form_online_status') }}
                                </template>
                                <component :is="ucdata.isOnline ? 'el-tag' : 'div'" :type="ucdata.isOnline ? 'success' : 'info'">
                                    {{ ucdata.isOnline ? $t('online') : $t('form_last_online_time') + getTimeDesc(ucdata.lastOnlineTime) }}
                                </component>
                            </el-descriptions-item>
                            <el-descriptions-item>
                                <template #label>
                                    <i class="fas fa-phone"></i>
                                    &nbsp;
                                    {{ $t('form_phone_number') }}
                                </template>
                                {{ ucdata.phone }}
                            </el-descriptions-item>
                            <el-descriptions-item>
                                <template #label>
                                    <i class="fas fa-user-plus"></i>
                                    &nbsp;
                                    {{ $t('form_register_info') }}
                                </template>
                                <div>
                                    <span class="stitle">{{ $t('form_register_time') }}</span>
                                    {{ getTimeDesc(ucdata.signupTime) }}
                                </div>
                                <div>
                                    <span class="stitle">{{ $t('form_register_ip') }}</span>
                                    {{ ucdata.signupIP }}
                                </div>
                            </el-descriptions-item>
                        </el-descriptions>
                    </template>
                </el-skeleton>
            </div>
            <div v-if="ucenterOwnerIsMe" class="box" style="margin-top: 20px">
                <div class="box_title">{{ $t('account_safety') }}</div>
                <div class="box_graytitle">{{ $t('account_safety_settings') }}</div>
                <div class="group">
                    <el-button type="primary" size="large" @click="setUdata('pass', $t('_password'), 'password')" plain>
                        <i class="fas fa-key"></i>&emsp;{{ $t('change_password') }}
                    </el-button>
                </div>
                <div class="box_graytitle">{{ $t('recent_login_records') }}</div>
                <el-table :data="ucdata.loginDetails" :stripe="true" style="margin-top: 20px" max-height="300px">
                    <!-- TNND，被坑两天了，这个el-table-column不能用单标签形式，所以尾部不能用 `/>` 必须用 `></el-table-column>` -->
                    <el-table-column prop="id" label="ID"></el-table-column>
                    <el-table-column prop="" :label="$t('form_login_time')" width="200">
                        <template #default="scope">
                            {{ getTimeDesc(scope.row.loginTime) }}
                        </template>
                    </el-table-column>
                    <el-table-column prop="loginIP" :label="$t('form_login_ip')" width="150"></el-table-column>
                    <el-table-column prop="" :label="$t('form_last_online_time')" width="200">
                        <template #default="scope">
                            {{ scope.row.onlineTime ? getTimeDesc(scope.row.onlineTime) : $t('online') }}
                        </template>
                    </el-table-column>
                    <el-table-column prop="" :label="$t('form_operate_or_info')" min-width="150">
                        <template #default="scope">
                            <el-button v-if="scope.row.tokenActive && (! scope.row.isMe)" type="warning" size="small" @click="dieUserToken(scope.row.id)" plain>
                                {{ $t('force_logout') }}
                            </el-button>
                            <span v-if="! scope.row.tokenActive">{{ $t('token_isnt_active_tip') }}</span>
                            <span v-if="scope.row.isMe">{{ $t('is_this_device_tip') }}</span>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </el-col>
    </el-row>
</template>

<script setup>
    // -- basics
    let config = useState('config')
    definePageMeta({
        titles: [
            ['local://user', 'page_title_ucenter'],
            ['', 'page_title_ucenter_my']
        ],
        isUcenterPage: true,
        mustLogin: true
    })
    
    // -- refs
    let ucenterOwnerIsMe = useState('ucenterOwnerIsMe'),
        ucdata = ref(false),
        pageOwner = ref(getRouteParam('uid') || '')
    
    // -- app
    function changeAvatar() {
        selectFile((data) => {
            vaptchaGo(config.value.VAPTCHA_CONFIG.scenes.important, (vaptchaData) => {
                p({
                    name: $t('api_upload_avatar'),
                    url: u('local://api/user/avatarUpload'),
                    data: vaptchaData,
                    file: data,
                    ossModule: 'avatar',
                    succText: $t('wait_for_1s_tip')
                })
            })
        })
    }
    function setUdata_doit(key, desc, value) {
        p({
            name: $t('api_edit_user_info'),
            url: u('local://api/user/setUdata'),
            data: { key: key, value: value }
        })
    }
    function setUdata(key, desc, inputType, defaultValue) {
        inputMsg({
            content: $t('please_input_new') + ' ' + desc + ' :',
            inputType: inputType,
            callback_ok(value) {
                if (inputType == 'password') {
                    inputMsg({
                        content: $t('please_input_repeat') + ' ' + desc + ' :',
                        inputType: inputType,
                        callback_ok(value2) {
                            if (value !== value2) {
                                errMsg($t('wrongly_repeat_tip'))
                                return
                            }
                            setUdata_doit(key, desc, value)
                        }
                    })
                    return
                }
                if (ucdata.value[key] === value) {
                    infoMsg($t('nothing_changed_tip'))
                    return
                }
                setUdata_doit(key, desc, value)
            },
            defaultValue: defaultValue
        })
    }
    function dieUserToken(id) {
        confMsg({
            content: $t('question_confirm_to_force_logout'),
            type: 'warning',
            callback_ok: () => {
                vaptchaGo(config.value.VAPTCHA_CONFIG.scenes.important, (vaptchaData) => {
                    p({
                        name: $t('api_force_logout'),
                        url: u('local://api/user/userTokenDie'),
                        data: Object.assign({ id: id }, vaptchaData)
                    })
                })
            }
        })
    }
    
    if (import.meta.client) {
        await runThread(async () => await p({
            name: $t('api_sync_data'),
            url: u('local://api/user/data'),
            data: { uid: pageOwner.value },
            on_ok(res) {
                ucdata.value = res.data.userData
                ucenterOwnerIsMe.value = ucdata.value.isMe
            },
            on_err: () => $t('ready2jump_home'),
            jump_err: () => j(u('local://www')),
            type: 'loop'
        }))
    }
</script>