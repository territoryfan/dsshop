<template>
	<view class="app">
		<view class="price-box">
			<text>支付金额</text>
			<text class="price">{{orderInfo.total | 1000}}</text>
		</view>

		<view class="pay-type-list">

			<view class="type-item b-b" @click="changePayType('weixin')">
				<text class="icon yticon icon-weixinzhifu"></text>
				<view class="con">
					<text class="tit">微信支付</text>
					<text>推荐使用微信支付</text>
				</view>
				<label class="radio">
					<radio value="" color="#fa436a" :checked="payType == 'weixin'"/>
					</radio>
				</label>
			</view>
			<!-- <view class="type-item b-b" @click="changePayType('alipay')">
				<text class="icon yticon icon-alipay"></text>
				<view class="con">
					<text class="tit">支付宝支付</text>
				</view>
				<label class="radio">
					<radio value="" color="#fa436a" :checked="payType == 'alipay'" />
					</radio>
				</label>
			</view> -->
			<view class="type-item" @click="changePayType(1)">
				<text class="icon yticon icon-erjiye-yucunkuan"></text>
				<view class="con">
					<text class="tit">预存款支付</text>
					<text>可用余额 ¥{{orderInfo.user.money | 1000}}</text>
				</view>
				<label class="radio">
					<radio value="" color="#fa436a" :checked="payType == 1" />
					</radio>
				</label>
			</view>
		</view>
		
		<text class="mix-btn" @click="confirm">确认支付</text>
	</view>
</template>

<script>
	import Indents from '../../api/indents'
	import Pay from '../../api/pay'
	import {
	  authMsg
	} from '../../utils'
	export default {
		data() {
			return {
				id: '',
				payType: 'weixin',
				orderInfo: {
					total: 0,
					user: {
						money: 0
					}
				}
			};
		},
		computed: {
		
		},
		onLoad(options) {
			if(!options.id){
				this.$api.msg('参数有误')
				return false
			}
			this.id = options.id
			this.getList()
		},

		methods: {
			getList(){
				const that = this
				Indents.getPay(this.id,function(res){
					that.orderInfo = res
				})
			},
			//选择支付方式
			changePayType(type) {
				this.payType = type;
			},
			//确认支付
			confirm: async function() {
				const that = this
				if(this.payType === 1) {
					Pay.balancePay({
						id: this.id
					},function(res){
						authMsg(['4iOC-HyjJeKK5HiYORcOtrKHvu2Ho1ScVF0aqP3KkzQ'])
						uni.redirectTo({
							url: '/pages/money/paySuccess'
						})
					})
				} else {
					Pay.unifiedPayment({
						id: this.id,
						platform: this.payType,
						type: 'goodsIndent'
					},function(res){
						uni.requestPayment({
							timeStamp: res.msg.timestamp,
							nonceStr: res.msg.nonceStr,
							package: res.msg.package,
							signType: res.msg.signType,
							paySign: res.msg.paySign,
							success(res) {
							  // console.log(res)
							  // 订阅消息
							  authMsg(['4iOC-HyjJeKK5HiYORcOtrKHvu2Ho1ScVF0aqP3KkzQ'])
							  uni.redirectTo({
							  	url: '/pages/money/paySuccess'
							  })
							},
							fail(res) {
								that.$api.msg('支付失败，请重新支付')
							}
						})
						
					})
				}
			},
		}
	}
</script>

<style lang='scss'>
	.app {
		width: 100%;
	}

	.price-box {
		background-color: #fff;
		height: 265upx;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		font-size: 28upx;
		color: #909399;

		.price{
			font-size: 50upx;
			color: #303133;
			margin-top: 12upx;
			&:before{
				content: '￥';
				font-size: 40upx;
			}
		}
	}

	.pay-type-list {
		margin-top: 20upx;
		background-color: #fff;
		padding-left: 60upx;
		
		.type-item{
			height: 120upx;
			padding: 20upx 0;
			display: flex;
			justify-content: space-between;
			align-items: center;
			padding-right: 60upx;
			font-size: 30upx;
			position:relative;
		}
		
		.icon{
			width: 100upx;
			font-size: 52upx;
		}
		.icon-erjiye-yucunkuan {
			color: #fe8e2e;
		}
		.icon-weixinzhifu {
			color: #36cb59;
		}
		.icon-alipay {
			color: #01aaef;
		}
		.tit{
			font-size: $font-lg;
			color: $font-color-dark;
			margin-bottom: 4upx;
		}
		.con{
			flex: 1;
			display: flex;
			flex-direction: column;
			font-size: $font-sm;
			color: $font-color-light;
		}
	}
	.mix-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 630upx;
		height: 80upx;
		margin: 80upx auto 30upx;
		font-size: $font-lg;
		color: #fff;
		background-color: $base-color;
		border-radius: 10upx;
		box-shadow: 1px 2px 5px rgba(219, 63, 96, 0.4);
	}

</style>
