<template>
    <mt-header fixed isgrey title="订单退款须知">
        <mt-button v-link="'/user/order'" icon="arr-left" slot="left"></mt-button>
    </mt-header>
    <mt-content>
        <div class="retrun-know-box">
            <p>1、订单仅允许在未完成的情况下申请退款，如订单状态为已完成，则不能申请，如有特殊情况须申请退款，可直接拨打400-966-8838联系客服处理。</p>
            <p>2、目前随访家园仅允许订单全额退款，不允许拆分金额。</p>
            <p>3、购买收费留言后，如已经使用，则剩余条数不能申请退款。</p>
            <p>4、退款申请后，随访家园会在3-5个工作日进行审核，审核通过后会将款项退到用户的账户中。</p>
            <p>5、退款过程中请关注订单退款进度，以便及时跟进相关操作。</p>
        </div>
    </mt-content>
</template>
<script>
    import MtContent from '../../components/content'
    import MtHeader from '../../components/header.vue'
    import MtButton from '../../components/button.vue'

    export default{
        data () {
            return{
            }
        },
        components: {
            MtContent,
            MtHeader,
            MtButton
        }
    }
</script>

<style>
    .retrun-know-box{padding: 10px;}
    .retrun-know-box p{font-size: 14px;line-height: 25px;margin-bottom: 8px;color: #363636;}
</style>
