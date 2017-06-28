<template>
    <mt-header fixed isgrey title="我的订单">
        <mt-button v-link="'/user/wallet'" icon="arr-left" slot="left"></mt-button>
    </mt-header>
    <mt-content :class="{'leh-bg-grey-body':orderList.length != 0}">
        <p class="return-tip-box" v-if="orderList.length != 0" v-link="'/user/returnText'">订单退款须知<span class="iconfont icon-wx-arr-right fr"></span></p>
        <div class="bill-title">
            <ul>
                <li class="bill-title-list">
                    <p>¥ {{payNum}}</p>
                    <span>支出</span>
                </li>
                <li class="bill-title-list">
                    <p>¥ {{takeNum}}</p>
                    <span>收入</span>
                </li>
            </ul>
        </div>
        <div class="bill-content">
            <div class="leh-null-data" v-if="orderList.length == 0">暂无订单数据</div>
            <ul v-if="!orderIf">
                <li class="bill-content-list" v-for="items in orderList">
                    <div class="page-cell">
                        <a class="mint-cell">
                            <label class="mint-cell-title">
                                <span class="mint-cell-text leh-fs-fourteen" v-if="items.service_type == 2">电话预约</span>
                                <span class="mint-cell-text leh-fs-fourteen" v-if="items.service_type == 1">收费留言</span>
                            </label>
                            <div class="mint-cell-value">
                                <span class="leh-c-orange-tint leh-fs-fourteen" v-if="items.pay_status == 1">支付成功</span>
                                <span class="leh-c-orange-tint leh-fs-fourteen" v-if="items.pay_status == 0">已关闭</span>
                                <span class="leh-c-orange-tint leh-fs-fourteen" v-if="items.pay_status == 2">已完成</span>
                            </div>
                        </a>
                        <a class="mint-cell">
                            <label class="mint-cell-title">
									<span class="mint-cell-text">
										<span v-text="items.docter_name"></span>
										<span class="leh-c-black fr" v-text="'¥ '+items.real_money"></span>
									</span>
									<span class="mint-cell-label" v-if="">
										<span class="fr">优惠券：¥10</span>
										<span class="fr iconfont icon-wx-pack"></span>
									</span>
                            </label>
                            <div class="mint-cell-value">
                                <span></span>
                            </div>
                        </a>
                        <a class="mint-cell">
                            <label class="mint-cell-title">
									<span class="mint-cell-text">
										<p>创建时间：<i v-text="items.create_time"></i></p>
										<p v-if="items.pay_status == 1">支付时间：<i v-text="items.pay_time"></i></p>
									</span>
                            </label>
                            <div class="mint-cell-value" v-if="items.is_refund " @click="goRefund(items.service_type,items.refund_id,items.refund_status,items.real_money,items.pkid)">
                                <!--修改的(退款)-->
                                <span class="bill-content-tip-btn leh-c-green-strong" v-if="items.refund_status == -1">退款</span>
                                <span class="bill-content-tip-btn leh-c-blue-strong" v-if="items.refund_status == 0">退款审核中</span>
                                <span class="bill-content-tip-btn leh-c-orange-strong" v-if="items.refund_status == 1">退款成功</span>
                                <span class="bill-content-tip-btn" v-if="items.refund_status == 2">退款关闭</span>
                                <span class="bill-content-tip-btn leh-c-green-strong" v-if="items.refund_status == 3">退款待修改</span>
                            </div>
                        </a>
                    </div>
                </li>

            </ul>
        </div>
        <div class="page-infinite-loading document-index-load-tap" v-if="orderNum*10 < orderQty">
            <mt-button size="large" type="transparent" icon="load" @click="moreOrderList()" >点击加载更多</mt-button>
        </div>
    </mt-content>
</template>
<script>
    import MtHeader from '../../components/header.vue'
    import MtContent from '../../components/content.vue'
    import MtButton from '../../components/button.vue'
    import MtCell from '../../components/cell.vue'
    import {getJson} from 'util'

    export default{
        route: {
            data ({to, next}) {
                let _self = this
                _self.payNum="0.00"
                _self.takeNum="0.00"
                _self.orderNum = 1
                _self.getOrderList()
                next()
            }
        },
        data () {
            return{
                msg:'hello vue',
                orderList:[],
                orderQty:0,
                orderNum:1,
                orderIf:false,
                payNum:"0.00",
                takeNum:"0.00"
            }
        },
        methods:{
          getOrderList(){
              let _self = this
              getJson('api/patientOrder/patientOrderMoneyTotal','',(rsp_title)=>{
                  //总汇
                  _self.takeNum = rsp_title.income
                  _self.payNum = rsp_title.expenditure
                  getJson('api/patientOrder/patientOrderIndex?pageindex=1&pagesize=10','',(rsp_list)=>{
                      //列表
                      _self.orderList = rsp_list.items
                      _self.orderQty = rsp_list.totalQty
                  },_self)
              },_self)
          },
          moreOrderList(){
              let _self = this
              if(_self.orderNum*10 >= _self.orderQty) {
                  return
              }
              _self.orderNum = _self.orderNum + 1
              getJson('api/patientOrder/patientOrderIndex?pageindex='+ _self.orderNum +'&pageSize=10', '', (rsp_list)=>{
                  // 合并数组
                  _self.orderList = _self.orderList.concat(rsp_list.items)
              },_self)
          },
          goRefund(serviceType,refundId,refundStatus,refundMoney,listId){
              if(refundStatus == 3){
                  this.$route.router.go({path:'/user/return',query:{'servicType':serviceType,'refundId':refundId,'refundStatus':refundStatus,'refundMoney':refundMoney,'listId':listId,'txtClear':1,'isChange':0}, replace: true})
              }else{
                  this.$route.router.go({path:'/user/return',query:{'servicType':serviceType,'refundId':refundId,'refundStatus':refundStatus,'refundMoney':refundMoney,'listId':listId,'txtClear':1,'isChange':1}, replace: true})
              }
          }
        },
        components: {
            MtHeader,
            MtContent,
            MtButton,
            MtCell
        }
    }
</script>

<style>
    .bill-content-list .mint-cell:after,
    .bill-content-list .mint-cell:nth-last-of-type(1):before,
    .bill-title-list:nth-of-type(1){border: 0;}
    .bill-content-list .mint-cell:before{left: 0; -webkit-transform: scaleY(1);transform: scaleY(1);border-color: #e5e5e5}
    .bill-title{padding: 12px 0;overflow: hidden;background-color: #f0f0f0}
    .bill-title-list{float: left;width: 50%;border-left: 1px solid #aaa;text-align: center;}
    .bill-title-list p{font-size: 16px;}
    .bill-content-list i{font-style: normal}
    .bill-title-list span{font-size: 12px;color: #a9a9a9;}
    .bill-content-list{margin-bottom: 10px;}
    .bill-content-list:nth-last-of-type(1){margin-bottom: 0;}
    .bill-content-list .mint-cell-text p{font-size: 12px;}
    .bill-content-list .mint-cell-text p:nth-of-type(even){margin-top: 10px;}
    .bill-content-list .mint-cell-label{margin-top: 8px;color: #363636;}
    .bill-content-list .mint-cell-label .icon-wx-pack{font-size: 20px;line-height: 14px;margin-right: 8px;color: #ffc040;}

    .bill-content-tip-btn{height:25px;line-height:25px;min-width:60px;text-align: center;font-size: 13px;padding: 0 7px;border-radius: 5px;border-width: 1px;border-style: solid;display: inline-block;}
    .return-tip-box{padding: 10px 10px;background-color: #fffad4;text-align: left;margin: 0 auto;line-height: 22px;font-size: 14px;color: #f77a66;overflow: hidden;}
    .return-tip-box .icon-wx-arr-right{font-size: 12px;font-weight: normal;}

    .document-index-load-tap .mint-button--transparent{text-align: center;}
    .document-index-load-tap .icon-wx-load{color: #e5e5e5;}
    .document-index-load-tap .mint-button-text{font-size: 14px;}
</style>
