<template>
    <view-box ref="viewBox" body-padding-bottom="55px">
        <div class="orderList bg-gray">
            <sticky scroll-box="vux_view_box_body">
                <tab :line-width='3' active-color='#00ca9d' custom-bar-width="35px">
                    <tab-item selected @on-item-click = 'itemClickHandle'>待确认</tab-item>
                    <tab-item @on-item-click = 'itemClickHandle'>已确认</tab-item>
                </tab>
            </sticky>

            <scroller 
                lock-x scrollbar-y 
                @on-scroll-bottom="onScrollBottom" 
                ref="scrollerBottom" 
                :scroll-bottom-offst="200"
                height="-102"
            >
                <div class="wx-box">
                    <div class="wx-item" v-for = '(item, index) in orderList' :key="index">
                        <div class="tab-swiper vux-center">
                            <a :href="'#/order/confirm?orderNum='+item.order_num">
                                <p class="wx-title">{{item.view_line_content}}</p>
                                <p class="wx-right wx-green">{{['待确认','游客取消','导游取消','确认超时','待付款','支付超时','已付款','已出行完成','有投诉待确认','已退款','待评价','已评价'][item.book_status-1]}}</p>
                                <!-- "book_status": 1,//订单状态:1=待导游确认,2=游客取消,3=导游主动取消,4=导游确认超时,5=待游客付款,6=支付超时,7=已付款,8=已出行完成,9=有投诉待确认,10=已退款,11=待评价,12=已评价 -->
                            </a>
                            <a :href="'#/order/confirm?orderNum='+item.order_num" class="weui-media-box weui-media-box_appmsg">
                                <div class="weui-media-box__hd">
                                    <img :src="item.resource_path ? (item.resource_path.indexOf('http')>-1 ? '' : imgOrigin) + item.resource_path : require('../../assets/images/defaultPhoto.png')" alt="">
                                </div>
                                <div class="weui-media-box__bd">
                                    <h4>浏览时间：{{item.visit_date}}</h4>
                                    <p>游客人数：{{item.person_num}}人</p>
                                    <p>订单金额：￥{{item.amount}}</p>
                                </div>
                            </a>
                        </div>
                    </div>
                    <!-- <load-more :tip="正在加载"></load-more> -->
                </div>
            </scroller>

            <!-- <div class="loadAll" ref="div"></div> -->
            <!-- <load-more :show-loading="false" :tip="暂无数据" background-color="#fbf9fe"></load-more> -->
            <!-- 公用底部 -->
            <wx-footer></wx-footer>
        </div>
    </view-box>
</template>

<script>
import WxFooter from '../../components/WxFooter'
import { 
    Tab, TabItem, Sticky, Divider, XButton, Swiper, SwiperItem, Card,
    Panel, ViewBox, LoadMore, Scroller, Spinner, Group, Cell
 } from 'vux'

export default {
    name: 'orderList',

    data () {
        return {
            loadOnce: false,
            config: vm.config,            // 配置
            onFetching: false,
            imgOrigin: '',                // 图片origin
            pageNo: 1,                    // 分页，第几页
            pageSize: 10,                 // 分页，每页数量
            status: 1,                    // 订单状态，1：待确认 
            orderList: [],                // 列表
            lastPage: false,              // 是否为最后一页
            loadText: '',                 // 
        }
    },

    components: {
        WxFooter,Tab,TabItem,Sticky,Divider,XButton,Swiper,SwiperItem,Card,Panel,
        ViewBox,LoadMore,Scroller,Spinner,Group,Cell
    },

    created () {
        this.config.title('订单')
        this.fetchList()
    },
    
    mounted () {
        this.$nextTick(() => {
          this.$refs.scrollerBottom.reset({top: 0})
        })
    },

    activated () {
        this.$refs.scroller.reset()
    },
    
    methods: {
        itemClickHandle (val){
            this.orderList = []
            this.status = val +1
            this.pageNo = 1
            this.lastPage = false
            this.loadOnce = false
            this.fetchList()
        },

        onScrollBottom () {
            this.pageNo += 1
            this.fetchList()
        },

        fetchList() {
            if(!this.loadOnce && !this.lastPage){
                this.loadOnce = true
                this.$http.get(`/guide/order/list?pageNo=${this.pageNo}&status=${this.status}&pageSize=${this.pageSize}&oid=test1234`)
                .then(rst => {
                    if(rst.body && rst.body.data){
                        this.orderList = this.orderList.concat(rst.body.data.list)
                        this.pageNo = rst.body.data.pageNumber
                        this.imgOrigin = rst.body.prefix
                    }
                    if(rst.body && rst.body.data && rst.body.data.lastPage){
                        this.lastPage = true
                    }else{
                        this.lastPage = false
                    }

                    this.$nextTick(() => {
                        this.$refs.scrollerBottom.reset()
                    })

                    this.loadOnce = false
                }) 
                .catch(err => {
                    this.$vux.toast.show({
                        text: err.body.msg,
                        type: 'text'
                    })
                })
            }
        }
    }
}
</script>

<style scoped lang="sass">
.loadAll
    text-align: center
    position: absolute
    bottom: 60px
    left: 0
    font-size: 12px
    color: #979797
    width: 100%
</style>
<style lang="sass">
.orderList .vux-tab
    height: 50px
.orderList .vux-tab-ink-bar
    bottom: 2px
.orderList .weui-media-box__hd
    position: relative
    overflow: hidden
.orderList .weui-media-box__hd img
    position: absolute
    left: 0
    top: 50%
    transform: translateY(-50%)
</style>