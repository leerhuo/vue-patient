<template>
    <mt-header fixed isgrey title="新增血制品">
        <mt-button @click="msgBox()" icon="arr-left" slot="left"></mt-button>
    </mt-header>
    <div class="leh-float-box">
        <mt-button type="green" @click="saveBloodAdd()">保存</mt-button>
    </div>
    <mt-content>
        <div class="page-part blood-add-box">
            <div class="mint-field">
                <a class="mint-cell mint-field-cell">
                    <label class="mint-cell-title">
                        <span class="mint-cell-text">种类</span>
                    </label>
                    <div class="mint-cell-value">
                        <p class="leh-text-ellipsis"  @click="show_blood = !show_blood" v-text="bloodName"></p>
                        <span class="iconfont icon-wx-arr-down"></span>
                    </div>
                    <ul class="leh-select-drag-box" v-if="show_blood">
                        <li v-for="items in bloodList" @click="getName(items)">{{items}}</li>
                    </ul>
                </a>
            </div>
            <div class="mint-field">
                <a class="mint-cell mint-field-cell">
                    <label class="mint-cell-title">
                        <span class="mint-cell-text">最近时间</span>
                    </label>
                    <div class="mint-cell-value">
                        <input class="mint-field-core leh-ipt-tap" type="date" v-model="blood_near">
                        <div class="mint-field-clear" style="display: none;">
                            <i class="mintui mintui-field-error"></i>
                        </div>
							<span class="mint-field-state is-default">
								<i class="mintui mintui-field-default"></i>
							</span>
                        <span class="iconfont icon-wx-arr-down"></span>
                    </div>
                </a>
            </div>
            <div class="mint-field">
                <a class="mint-cell mint-field-cell">
                    <label class="mint-cell-title">
                        <span class="mint-cell-text">使用量</span>
                    </label>
                    <div class="mint-cell-value">
                        <input class="mint-field-core" placeholder="请输入数字" type="text" maxlength="5" v-model="blood_num" onkeyup="this.value=this.value.replace(/\D/g,'')">
                        <div class="mint-field-clear" style="display: none;">
                            <i class="mintui mintui-field-error"></i>
                        </div>
							<span class="mint-field-state is-default">
								<i class="mintui mintui-field-default"></i>
							</span>
                        <span class="leh-fs-twelve leh-c-grey">ml</span>
                    </div>
                </a>
            </div>
            <div class="mint-field">
                <a class="mint-cell mint-field-cell">
                    <label class="mint-cell-title">
                        <span class="mint-cell-text">备注</span>
                    </label>
                    <div class="mint-cell-value">
                        <textarea class="msg-val" placeholder="请输入备注信息" maxlength="45" v-model="msg_val"></textarea>
                    </div>
                </a>
            </div>
        </div>
    </mt-content>
    <div class="page-popup">
        <mt-popup v-show="show_popup" position="top" class="mint-popup-2" :modal="false">
            <p v-text="tips"></p>
        </mt-popup>
        <div v-show="show_popup" class="maskbox"></div>
    </div>
</template>
<script>
    import MtContent from '../../components/content'
    import MtHeader from '../../components/header.vue'
    import MtButton from '../../components/button.vue'
    import MtField from '../../components/field.vue'
    import MtPopup from '../../components/popup.vue'
    import MessageBox from 'vue-msgbox'
    import {optionData, postJson} from 'util'
    import $ from 'zepto'
    export default{
        route: {
            data ({ next}) {
                //初始化
                this.bloodInit()
                next()
            }
        },
        data(){
            return{
                bloodList: optionData().bloodProductTypes,
                show_blood: 0,
                bloodName:'',
                tips:'',
                show_popup: false,
                msg_val:'',
                blood_num:'',
                blood_near:''
            }
        },
        methods:{
            getName(name){
                let _self = this
                _self.bloodName = name
                _self.show_blood = 0
            },
            saveBloodAdd(){
                let _self = this
                let bloodAdd = {
                    "bloodType" : _self.bloodName,       //血制品种类
                    "useDate" : _self.blood_near || '',    //最近时间
                    "useCount" : _self.blood_num || '',    //使用量
                    "remark" : _self.msg_val || ''    //备注
                }
                if(bloodAdd.bloodType == ''){
                    _self.show_popup = true
                    _self.tips = '种类尚未填写完整，请填写完全后再保存！'
                    return
                }else if(bloodAdd.useDate == ''){
                    _self.show_popup = true
                    _self.tips = '最近时间尚未填写完整，请填写完全后再保存！'
                    return
                }else if(bloodAdd.useCount == ''){
                    _self.show_popup = true
                    _self.tips = '使用量尚未填写完整，请填写完全后再保存！'
                    return
                }

                postJson('api/BloodProduct',bloodAdd, (rsp, recode, msg)=>{
                    if(recode == "1"){
                        alert(msg)
                        return
                    }
                    _self.tips = '保存成功'
                    _self.show_popup = true
                    setTimeout(() => {
                        _self.$route.router.go({path: '/user/blood', replace: true})
                     },2000)
                },_self)


            },
            bloodInit(){
                let _self = this
                _self.bloodName = ''
               _self.blood_near = ''
                _self.blood_num = ''
                _self.msg_val = ''
            },
            msgBox () {
                let _self = this
                if((_self.bloodName != '')||(_self.blood_near != '')||(_self.blood_num != '')||(_self.msg_val != '')){
                    MessageBox({
                        title: '提示',
                        message: '编辑内容未保存，是否退出?',
                        showCancelButton: true
                    }).then(action => {
                        if(action === 'confirm'){
                        _self.$route.router.go({path: '/user/blood',replace:true})
                        }
                    });
                }else {
                    _self.$route.router.go({path: '/user/blood', replace: true})
                }
            }
        },
        watch:{
            'msg_val' (newVal){
                let msgTest = $('.msg-val')
                let scrollH = msgTest[0].scrollHeight
                msgTest.height(scrollH)
                if(!newVal){
                    msgTest.height(25)
                }
            },
            show_popup(val) {
                if (val) {
                    setTimeout(() => {
                        this.show_popup = false;
                }, 2000);
                }
            }
        },
        components:{
            MtContent,
            MtHeader,
            MtButton,
            MtField,
            MtPopup
        }
    }
</script>

<style>
    .blood-add-box {overflow: hidden;}
    .blood-add-box .mint-cell{padding: 15px 10px;overflow: visible;}
    .blood-add-box .mint-cell:after{border: 0;}
    .blood-add-box input,.blood-add-box textarea{border:0;background-color: transparent;font-size: 14px;}
    .blood-add-box p{width: 100%;height: 24px;line-height: 28px;font-size: 14px;}
    .blood-add-box .icon-wx-arr-down{color: #aaa;position: absolute;right:10px;top:20px;}
    .blood-add-box textarea{min-height: 25px;height: 25px;line-height: 22px;font-size:14px;width: 100%;border: 1px solid transparent;overflow-y: hidden;padding: 0}
    .blood-add-box .mint-cell-value input,.blood-add-box .mint-cell-value textarea{display: block}
</style>
