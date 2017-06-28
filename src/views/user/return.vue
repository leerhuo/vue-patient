<template>
    <mt-header fixed isgrey title="退款申请">
        <mt-button v-link="'/user/order'" icon="arr-left" slot="left" @click="showReason = false"></mt-button>
    </mt-header>
    <div class="leh-float-box">
        <mt-button type="green" @click="commitApply()" v-if="isRouter">提交申请</mt-button>
    </div>
    <mt-content>
        <p class="return-tip-box" v-if="showTitle" v-text="titleText"></p>
        <div class="return-list">
            <a class="mint-cell">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">退款原因*</span>
                    <div class="return-name" @click="isReason()" v-text="reasonName"></div>
                    <ul class="leh-select-drag-box" v-if="showReason">
                        <li @click="getName(items)" v-for="items in reasonList" v-text="items"></li>
                    </ul>
                </label>
                <div class="mint-cell-value">
                    <span class="iconfont icon-wx-arr-down"></span>
                </div>
            </a>
            <a class="mint-cell">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">退款金额<i class="leh-c-grey leh-fs-twelve">(不可修改)</i></span>
                    <div class="return-name" v-text="returnMoney"></div>
                </label>
                <div class="mint-cell-value"></div>
            </a>
            <a class="mint-cell">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">退款说明<i class="leh-c-grey leh-fs-twelve">(可不填)</i></span>
                    <textarea :readonly='!isEdit' class="msg-val" maxlength="100" placeholder="请输入退款说明" @focus="showReason = false" v-model="msg_val"></textarea>
                </label>
                <div class="mint-cell-value"></div>
            </a>
            <a class="mint-cell" @click="goReturnNum()">
                <label class="mint-cell-title">
                    <span class="mint-cell-text leh-c-green">退款账号* </span>
                </label>
                <div class="mint-cell-value">
                    <i class="leh-fs-fourteen" v-text="numInput"></i>
                    <b class="iconfont icon-wx-arr-right leh-c-grey"></b>
                </div>
            </a>
        </div>
        <div class="return-text-box">
            <p>温馨提示：</p>
            <p>1、带星号内容必须填写。</p>
            <p>2、请确保退款账号的真实性和准确性，否则将会影响退款进度。</p>
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
    import {getJson,postJson} from 'util'

    export default{
        route: {
            data ({to, next}) {
                this.service_type = to.query.servicType
                this.returnMoney = to.query.refundMoney
                this.refund_id = to.query.refundId
                this.refund_status = to.query.refundStatus
                this.editCardId = to.query.editCardId
                this.listId = to.query.listId
                this.txtClear = to.query.txtClear
                this.isChange = to.query.isChange
                if(this.txtClear == 1){
                    this.reasonName = null
                    this.msg_val = null
                }
                this.showTitleTip(this.refund_status)
                if(this.refund_id > 0){
                    this.getReturnList(this.refund_id)
                }
                this.getReturn()

                next()
            }
        },
        data () {
            return{
                showReason:false,
                reasonName:'', //退款原因
                reasonList:[],  //退款原因列表
                returnMoney:'',  //退款金额
                listId:'',      //列表主键id
                service_type:'',  //服务类型：1为留言收费，2为电话预约
                refund_id:'',     //退款ID
                refund_status:'',    //退款状态
                cardId:'',          //退款银行账号ID
                editCardId:0,      //编辑后的退款账号Id
                numInput:'未填写',  //退款账号状态,默认未填写
                show_popup: false,
                tips:'',
                msg_val:'',
                titleText:'',
                showTitle:false,
                isEdit:true,
                isRouter:true,
                isChange:'',
                txtClear:false,
                hasCard:false
            }
        },
        methods:{
            getReturn(){
                let _self = this
                getJson('api/patientRefund/refundReason','',(rsp_reason)=>{
                    //退款原因&退款账号状态
                    if(_self.service_type == 1){
                        _self.reasonList = rsp_reason.msgvisitReason
                    }else{
                        _self.reasonList = rsp_reason.previsitReason
                    }
                    if(rsp_reason.hasAccount){
                        _self.numInput = '已填写'
                        _self.hasCard = rsp_reason.hasAccount
                    }
                    if(_self.editCardId > 0){
                        _self.cardId = _self.editCardId
                    }else{
                        _self.cardId = rsp_reason.card_id
                    }
                },_self)
            },
            getReturnList(id){
                let _self = this
                getJson('api/patientRefund/refundDetail?pkid=' + id,'',(rsp_list)=>{
                    //退款详情
                    _self.reasonName = rsp_list.reason
                    _self.msg_val = rsp_list.remark
                },_self)
            },
            goReturnNum(){
                let _self = this
                if(_self.isRouter){
                    _self.$route.router.go({path:'/user/returnNum',query:{'servicType':_self.service_type,'refundId':_self.refund_id ,'refundStatus':_self.refund_status,'refundMoney':_self.returnMoney, 'cardId':_self.cardId,'listId':_self.listId,'isChange':_self.isChange}, replace: true})
                }
            },
            isReason(){
                if(this.isEdit == true){
                    this.showReason = !this.showReason
                }
            },
            getName(name){
                this.reasonName = name
                this.showReason = 0
            },
            showTitleTip(status){
               // -1:退款    0:退款审核中    1:退款成功    2:退款关闭    3:退款待修改
                if(status == 0){
                    this.showTitle = true
                    this.isEdit = false
                    this.isRouter = false
                    this.titleText = '退款审核中，请耐心等候！'
                }else if(status == 1){
                    this.showTitle = true
                    this.isEdit = false
                    this.isRouter = false
                    this.titleText = '退款成功，请及时查看退款账户！'
                }else if(status == 2){
                    this.showTitle = true
                    this.isEdit = false
                    this.isRouter = false
                    this.titleText = '退款已关闭，此订单不可退款，如有疑问，请及时联系400-966-8838咨询详情。'
                }else if(status == 3){
                    this.showTitle = true
                    this.isEdit = false
                    this.isRouter = true
                    this.titleText = '退款账号有误，请核实账号并修改后重新提交！'
                }else{
                    this.isEdit = true
                    this.isRouter = true
                    this.showTitle = false
                }
            },
            commitApply(){
                let _self = this
                /*if(_self.isChange == 0){
                    _self.show_popup = true
                    _self.tips = '退款账号尚未修改，请修改后再进行操作！'
                    return
                }else*/
                if(_self.reasonName == null){
                    _self.show_popup = true
                    _self.tips = '请先选择退款原因！'
                    return
                }else if(_self.hasCard == false){
                    _self.show_popup = true
                    _self.tips = '请填写退款账号！'
                    return
                }else{
                    let returnParams = {
                        "pkid": _self.refund_id,
                        "order_id": _self.listId,
                        "order_type": _self.service_type,
                        "refund_money": _self.returnMoney,
                        "reason": _self.reasonName,
                        "card_id": _self.cardId,
                        "remark": _self.msg_val || '',
                        "op_status": _self.refund_status
                    }
                    postJson('api/patientRefund/apply', returnParams, (rsp, recode, msg)=>{
                        _self.show_popup = true
                        _self.tips = '已提交申请，请耐心等候！'
                        setTimeout(function(){
                            _self.$route.router.go({path:'/user/order', replace: true})
                        },3000)
                    },_self)
                }

            }
        },
        components: {
            MtContent,
            MtHeader,
            MtButton,
            MtField,
            MtPopup
        },
        watch:{
            'msg_val' (newVal){
                let msgTest = $('.msg-val')
                let scrollH = msgTest[0].scrollHeight
                msgTest.height(scrollH)
                if(!newVal){
                    msgTest.height(24)
                }
            },
            show_popup(val) {
                if (val) {
                    setTimeout(() => {
                        this.show_popup = false
                    }, 2000);
                }
            }
        }
    }
</script>

<style>
    .return-tip-box{padding: 10px 10px;background-color: #fffad4;text-align: left;margin: 0 auto;line-height: 22px;font-size: 14px;color: #f77a66;}
    .return-list .mint-cell{padding-bottom: 10px;overflow: visible;}
    .return-list .mint-cell:after,.return-list .mint-cell:nth-last-of-type(1):before{border: 0;}
    .return-list .mint-cell:before{left: 10px;transform: scaleY(1);}
    .return-name,.return-list textarea{width: 100%;margin-top: 15px;font-size: 14px;height: 18px;color: #363636;}
    .return-list textarea{border: 1px solid transparent;overflow-y: hidden;line-height: 20px;color: #363636;height: 24px;padding-left: 0;border-left: 0;}
    .return-list .mint-cell .mint-cell-value span{margin-top: 35px;color: #aaa;position: absolute;top: 15px;right: 10px}
    .return-list i,.return-list b{font-style: normal;margin-left: 5px;font-weight: normal;}
    .return-tip-box{padding: 10px 10px;background-color: #fffad4;text-align: left;margin: 0 auto;line-height: 22px;font-size: 14px;color: #f77a66;}
    .return-text-box{padding: 30px 10px 20px;}
    .return-text-box p{font-size: 14px;color: #939393;line-height: 24px;}

</style>
