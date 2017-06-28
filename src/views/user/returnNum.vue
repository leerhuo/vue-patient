<template>
    <mt-header fixed isgrey title="退款账号">
        <mt-button @click="goBack()" icon="arr-left" slot="left"></mt-button>
    </mt-header>
    <div class="leh-float-box">
        <mt-button type="green" @click="keep()">保存</mt-button>
    </div>
    <mt-content>
        <div class="return-num-list">
            <a class="mint-cell">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">所属银行</span>
                    <div class="return-num-name" @click="showBank = !showBank" v-text="bankName"></div>
                    <ul class="leh-select-drag-box" v-if="showBank">
                        <li v-for="items in bankList" v-text="items.bank" @click="getName(items.bank)"></li>
                    </ul>
                </label>
                <div class="mint-cell-value">
                    <span class="iconfont icon-wx-arr-down"></span>
                </div>
            </a>
            <a class="mint-cell">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">账户名</span>
                    <input type="text" placeholder="请输入持卡人姓名" maxlength="15" v-model="userName"/>
                </label>
                <div class="mint-cell-value"></div>
            </a>
            <a class="mint-cell">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">账号</span>
                    <input type="tel" placeholder="请输入银行卡号" maxlength="25" v-model="cardNum"/>
                </label>
                <div class="mint-cell-value"></div>
            </a>
        </div>
    </mt-content>
    <!--<div class="page-popup">
        <mt-popup v-show="show_popup" position="top" class="mint-popup-2" :modal="false">
            <p v-text="tips"></p>
        </mt-popup>
        <div v-show="show_popup" class="maskbox"></div>
    </div>-->
</template>
<script>
    import MtContent from '../../components/content'
    import MtHeader from '../../components/header.vue'
    import MtButton from '../../components/button.vue'
    /*import MtField from '../../components/field.vue'
    import MtPopup from '../../components/popup.vue'*/
    import MessageBox from 'vue-msgbox'
    import {getJson,postJson} from 'util'
    export default{
        route: {
            data ({to, next}) {
                this.service_type = to.query.servicType
                this.returnMoney = to.query.refundMoney
                this.refund_id = to.query.refundId
                this.refund_status = to.query.refundStatus
                this.cardId = to.query.cardId
                this.listId = to.query.listId
                this.isChange = to.query.isChange
                this.getBank(this.cardId)
                next()
            }
        },
        data () {
            return{
                showBank:false,
                returnMoney:'',  //退款金额
                service_type:'',  //服务类型：1为留言收费，2为电话预约
                refund_id:'',     //退款ID
                refund_status:'',    //退款状态
                cardId:'',          //退款银行账号ID
                listId:'',      //列表主键id
                bankName:'',
                bankList:[],
                userName:'',
                cardNum:'',
                show_popup:false,
                tips:'',
                isChange:'',
                isBankName:'',
                isCardNum:'',
                isUserName:''
            }
        },
        methods:{
            getBank(id){
                let _self = this
                getJson('api/patientRefund/bankList','',(rsp_list)=>{
                    //银行卡
                    _self.bankList = rsp_list
                    getJson('api/patientRefund/bankDetail?pkid=' + id,'',(rsp)=>{
                        //银行卡内容
                        _self.bankName = rsp.bank
                        _self.cardNum = rsp.card_no
                        _self.userName = rsp.card_person
                        //判断是否修改
                        _self.isBankName = rsp.bank
                        _self.isCardNum = rsp.card_no
                        _self.isUserName = rsp.card_person
                    },_self)
                },_self)
            },
            goBack(){
                let _self = this
                _self.$route.router.go({path:'/user/return',query:{'servicType':_self.service_type,'refundId':_self.refund_id ,'refundStatus':_self.refund_status,'refundMoney':_self.returnMoney,'editCardId':_self.cardId,'listId':_self.listId,'txtClear':0,'isChange':_self.isChange}, replace: true})
            },
            getName(name){
                this.bankName = name
                this.showBank = 0
                this.userName = null
                this.cardNum = null
            },
            msgBox (msg) {
                MessageBox({
                    title: '提示',
                    message: msg ,
                    showCancelButton:false
                })
            },
            keep(){
                let _self = this
                if((_self.bankName == '')||(_self.userName == null)||(_self.cardNum == null)){
                    _self.msgBox('请将内容填写完整后再保存！')
                    return
                }else if(isNaN(_self.cardNum)){
                    _self.msgBox('账号填写错误，请在文本框中填入不超25字的数字，不得出现其他内容！')
                    return
                }else if(!_self.userName.match(/^[\u4e00-\u9fa5]+$/)){
                    _self.msgBox('账户名填写错误，请在文本框中填入不超15字的中文，不得出现其他内容！')
                    return
                }else if((_self.bankName == _self.isBankName)&&(_self.userName == _self.isUserName)&&(_self.cardNum == _self.isCardNum)){
                    _self.msgBox('页面内容尚未修改，请修改后再进行操作！')
                    return
                }else{
                    let returnParams = {
                        "pkid": _self.cardId,
                        "bank": _self.bankName,
                        "card_no": _self.cardNum,
                        "card_person": _self.userName
                    }
                    postJson('api/patientRefund/saveRefundAccount', returnParams, (rsp, recode, msg)=>{
                        MessageBox({
                            title: '提示',
                            message: '保存成功!',
                            showCancelButton: false
                            }).then(action => {
                                if(action === 'confirm'){
                                    _self.cardId = rsp
                                    _self.isChange = 1
                                }
                            });
                    },_self)

                }

            }
        },
        components: {
            MtContent,
            MtHeader,
            MtButton,
            /*MtField,
            MtPopup,*/
            MessageBox
        },
        watch:{
            /*show_popup(val) {
                if (val) {
                    setTimeout(() => {
                        this.show_popup = false;
                }, 2000);
                }
            }*/
        }
    }
</script>

<style>
    .return-num-list .mint-cell{padding-bottom: 10px;overflow: visible;}
    .return-num-list .mint-cell:after{border: 0;}
    .return-num-list .mint-cell:before{left: 10px;transform: scaleY(1);}
    .return-num-name,.return-num-list input{width: 100%;margin-top: 15px;font-size: 14px;height: 18px;color: #363636;}
    .return-num-list input{border: 1px solid transparent;overflow-y: hidden;line-height: 20px;color: #363636;height: 28px;padding-left: 0;border-left: 0;}
    .return-num-list .mint-cell .mint-cell-value span{margin-top: 35px;color: #aaa;position: absolute;top: 15px;right: 10px}
    .return-num-tip-box{padding: 10px 10px;background-color: #fffad4;text-align: left;margin: 0 auto;line-height: 22px;font-size: 14px;color: #f77a66;}
</style>
