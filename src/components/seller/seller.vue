<template>
    <div class="seller">
		<div class="header" v-if="scrollY > -100">
			<div class="back"><i class="icon-back"></i>返回</div>
			<h1 class="title">{{ValidateAccount.seller_name}}</h1>
		</div>
		<div class="headerW" v-if="scrollY < -100">
			<div class="back"><i class="icon-back"></i>返回</div>
			<h1 class="title">{{ValidateAccount.seller_name}}</h1>
		</div>
		<div class="sellerContain">
			<scroll  @scroll="scroll"
					 :listenScroll="listenScroll"
					 @scrollToEnd="loadMore"
					 :pullup="pullup"
					 :probeType="3"
					 ref="scroll" class="sellerContent" :data="zdList">
				<div>
					<div class="m-header">
					<div class="m-money">
						<div class="keyong">可用余额(元)</div>
						<div class="eye" @click="selectEye()">
							<i class="icon-eye" v-if="eyeOpen"></i>
							<i class="icon-eyeclose" v-if="eyeClose"></i>
						</div>
						<div class="kymony"><i class="icon-hua"></i>
						<span v-if="eyeOpen">{{ValidateAccount.keyongyue}}</span>
						<span v-if="eyeClose">{{shuffleArry[0]}}</span>
						<i class="icon-hua"></i></div>
					</div>
					<div class="verfiy">
						<div @click="selectHX()" class="verfiy-code"><i class="icon-code"></i>扫码核销</div>
						<div @click="selectJL()" class="verfiy-jl"><i class="icon-jl"></i>验证记录</div>
					</div>
				</div>
				<div class="m-zc">
					<div class="zcmony">
						<div>资产总额</div>
						<div class="zcmonynum">{{ValidateAccount.zongjine}}</div>
					</div>
					<div class="djmony">
						<div>冻结资金</div>
						<div class="djmonynum">{{ValidateAccount.dongjieyue}}</div>
					</div>
				</div>
				<div class="zd">账单</div>
				<div class="zdlist">
					<div @click="selectItem(item)" class="zdItem" v-for="item in zdList">
						<div class="itemname">
							<div>{{item.title}}</div>
							<div class="xfnum" v-if="item.types == 'xf'">-{{item.amount}}</div>
							<div class="kfnum" v-if="item.types == 'kf'">-{{item.amount}}</div>
							<div class="num" v-if="item.types == 'cz'">
								+{{item.amount}}
							</div>
							<div class="txnum" v-if="item.types == 'tx'">-{{item.amount}}</div>
						</div>
						<div class="itemtime">
							<div class="">{{item.showtime}}</div>
							<div class="txitemtitle" v-if="item.types == 'tx'" >
								<span>{{item.status}}</span>
								<i class="icon-right"></i>
							</div>
							<div class="itemtitle" v-if="item.types == 'cz'" >{{item.status}}</div>
						</div>
					</div>
					<loading v-show="hasMore" title=""></loading>
				</div>
				
				</div>
			</scroll>
			<div class="foot">
				<div class="tixian" @click="selectTX()">提现</div>
				<div class="chongzhi" @click="selectCZ()">充值</div>
			</div>
			<router-view></router-view>
		</div>
    </div>
</template>

<script>
import Scroll from 'base/scroll/scroll'
import Loading from 'base/loading/loading'
import {getValidateAccount,getConsumptionLogs,getCode} from 'api/seller'
import {shuffle} from 'common/js/util'
import {mapMutations} from 'vuex'
import {Base64} from 'js-base64'
import storage from 'best-storage'
export default {
  name:'Seller',
  data() {
      return {
        ValidateAccount: [],
		zdList: [],
		eyeOpen:true,
		eyeClose:false,
		shuffleArry:['富可敌国','腰缠万贯','富贵荣华','挥金如土','财大气粗'],
		scrollY: -1,
		hasMore:true,
		pullup: true,
		listenScroll:true,
		page:1,
		count:1,
		count2:1
      }
    },
  created() {
	  this.listenScroll = true
      this._getValidateAccount()
	  this._getConsumptionLogs()
	  this.checkOpen()
    },
     mounted(){
		/*原生回调扫码 */
		var that=this
		window.scanCallback=function(data){
			const orderData = JSON.parse(data)
			/*拿到解密之后的base64 code值 */
			const scanCode = Base64.decode(orderData.scanCode)
			const seller_wv = storage.get('seller_wv')
			const token = storage.get('token')
			const scanCode2= JSON.parse(scanCode)
			const code = scanCode2.code
			// alert(code)
			
			getCode(seller_wv,token,code).then((res)=>{
				if(res.flag ==='1'){
					// 跳转到核销页面
					 that.setVerfiy(res.data)
					 that.$router.push({
			  			path:`/verification`
		  		     })
					 
				}else{
					alert(res.msg)
				}
				// alert(res)
			})
		}
		/*原生调用方法,传递潍V号,商户号,token */
		window.getParams=function(data){
				const sid = data.sid
				const seller_wv= data.seller_wv
				const user_account = data.seller_wv
				const token = data.token
				storage.set('sid',sid)
				storage.set('seller_id',sid)
				storage.set('seller_wv',seller_wv)
				storage.set('user_account',user_account)
				storage.set('token',token)
		}
    },
  methods:{
	//   获取商户中心
	  _getValidateAccount(){
		  	const sid='52'
			const seller_wv='54057460'
			const user_account='54057460'
			const token='B26711E76E9CADC91EDAE1F949828BEA'
			// 本地存储相关
			storage.set('sid',sid)
			storage.set('seller_id',sid)
			storage.set('seller_wv',seller_wv)
			storage.set('user_account',user_account)
			storage.set('token',token)
		  getValidateAccount(sid,seller_wv,token).then((res) => {
			if (res.flag === '1') {
            this.ValidateAccount = res.data
          }
        })
	  },
	  checkOpen(){
		  if(storage.get('setHis',[]).length>0){
			  this.eyeOpen = false
			  this.eyeClose = true
			  this.shuffleArry = storage.get('setHis')
		}
	  },
	  _getConsumptionLogs(){
		   	this.page = 1
        	this.hasMore = true
			const sid = storage.get('sid')
		   getConsumptionLogs(sid,this.page).then((res) => {
			if(res.flag ==='1'){
				this.zdList = res.data
				this.count=res.page.count
				// console.log(this.count)
				this._checkMore(res.data)
			}
        })
	  },
	  loadMore(){
		   if (!this.hasMore) {
          		return
			}
			const sid = storage.get('sid')
			this.page++
			this.count2++
			 getConsumptionLogs(sid,this.page).then((res) => {
				if(res.flag ==='1'){
					this.zdList = this.zdList.concat(res.data)
					this._checkMore(res.data,this.count2)
				}
        	})

	  },
	scroll(pos) {
		// console.log(pos)
		this.scrollY = pos.y
		// console.log(this.scrollY)

	},
	  selectEye(){
		  this.shuffleArry = shuffle(this.shuffleArry)
		//   this.Hisshuffle(this.shuffleArry)
		  storage.set('setHis',this.shuffleArry)
		  if(this.eyeOpen){
			  this.eyeOpen = false
			  this.eyeClose = true
		  }else{
			  this.eyeOpen = true
			  this.eyeClose = false
			  storage.remove('setHis')
		  }
	  },
	  selectItem(item){
		if(item.types === 'tx'){
			console.log(item.id)
			this.setSeller(item)
			this.$router.push({
          		path: `/seller/${item.id}`
    		})
			
		}
	  },
	  selectTX(){
		  this.$router.push({
			  path:`/enchashment`
		  })
	  },
	  selectCZ(){
		  this.$router.push({
			  path:`/recharge`
		  })
	  },
	   selectHX(){
			try{
				WVJsFunction.scan('scanCallback')
			}catch(e){
				try{
					window.webkit.messageHandlers.WVJsFunction.postMessage({scan:'scanCallback'})
				}catch(e){
					console.log('请在潍V内打开')
				}
			}
	  },
	  selectJL(){
		  this.$router.push({
			  path:`/record`
		  })
	  },
	  _checkMore(data,count){
		const data2 = data
		// console.log(data2)
        if (count >= this.count || !data2.length) {
          this.hasMore = false
        }
	  },
	  ...mapMutations({
		  setSeller: 'SET_SELLER',
		  setVerfiy:'SET_VERFIY'
	  })
  },
  components: {
      Scroll,
	  Loading
    }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
.seller
	width:100%
	.header
		display:flex
		height:44px
		width:100%
		align-items:center
		color:#fff
		font-size:16px
		position:relative
		background: linear-gradient(to top, #ffb857 0%,#ff5d70 00%)
		.back
			position:relative
			left:10px
		.back .icon-back
			font-size:20px;
		.title
			position: absolute
			left: 20%
			width: 60%
			text-overflow: ellipsis
			overflow: hidden
			white-space: nowrap
			text-align: center
			font-size: 18px
			line-height:44px
	.headerW
		display:flex
		height:44px
		width:100%
		align-items:center
		color:#000
		font-size:16px
		position:relative
		background:#fff
		.back
			position:relative
			left:10px
		.back .icon-back
			font-size:20px;
		.title
			position: absolute
			left: 20%
			width: 60%
			text-overflow: ellipsis
			overflow: hidden
			white-space: nowrap
			text-align: center
			font-size: 18px
			line-height:44px
	.sellerContain
		position:fixed
		top:44px
		bottom:0px
		width:100%
		.sellerContent
			height:100%
			overflow:hidden
			.m-header
				height:160px
				background: linear-gradient(to top, #ffb857 0%,#ff5d70 100%)
				position:relative
				// top:-44px
				// padding-top:44px
				.m-money
					display:flex
					justify-content:center
					flex-direction:column
					align-items:center
					position:relative
					top:20px
					.keyong
						margin-top:15px
						color:#fff6be
						font-size:16px
						letter-spacing:2px
						text-shadow: 3px 2px 2px #e25d4d
					.eye
						width:30px
						height:30px
						position:absolute
						right:20px
						top:13px
						text-align:center
						line-height:30px
						.icon-eye ,.icon-eyeclose
							color:#fff6be
							font-size:25px
							font-weight:lighter
					.kymony
						margin-top:20px
						color:#fff6be
						font-size:30px
						letter-spacing:3px
						span
							font-weight:500
							text-shadow: 3px 2px 2px #e25d4d
				.verfiy
					height:70px
					width:94%
					position:absolute
					bottom:-35px
					left:3%
					background:#fff
					border-radius:3px
					box-shadow: 0 5px 5px rgba(250,0,0,0.1)
					display:flex
					justify-content:center
					.verfiy-code
						flex:1
						display:flex
						align-items:center
						justify-content:center
						font-size:18px
						color:#333
						font-weight:lighter
						position:relative
						.icon-code
							color:#ff7108
							font-size:28px
							margin-right:5px
					.verfiy-jl
						flex:1
						display:flex
						font-size:18px
						align-items:center
						justify-content:center
						color:#333
						font-weight:lighter
						.icon-jl
							color:#ff7108
							font-size:28px
							margin-right:5px
					.verfiy-code::after
						content: " "
						position: absolute
						right: 0
						top: 25%
						width: 1px;
						height:50%
						border-right: 1px solid #d6d6d6
						color: #d6d6d6
						transform-origin: 100% 0
						transform: scaleX(.5)
	.m-zc
		padding-top:40px
		display:flex
		justify-content:center
		height:70px
		.zcmony
			flex:1
			display:flex
			align-items:center
			flex-direction:column
			font-size:16px
			justify-content:center
			.zcmonynum
				margin-top:10px
				font-weight:lighter
				color:#2f8aff
		.zcmony > div
			font-weight:lighter
		.djmony
			flex:1
			display:flex
			align-items:center
			justify-content:center
			flex-direction:column
			font-size:16px
			.djmonynum
				margin-top:10px
				font-weight:lighter
				color:#ff7108
		.djmony >div
			font-weight:lighter
	.zd
		height:30px
		background:#f5f2f2
		line-height:30px
		padding-left:14px
		color:#b3b3b3
	.zdlist
		padding-left:14px
		flex-direction:column
		padding-bottom: 50px
		.zdItem
			height:80px
			position:relative
			display:flex
			flex-direction:column
			justify-content:center
			.itemname
				margin-bottom:10px
				font-weight:lighter
				color:#333
				display:flex
				position:relative
				.num
					position:absolute
					right:14px
					color:#2f8aff
				.txnum,.xfnum,.kfnum
					position:absolute
					right:44px
					color:#ff7108
			.itemname >div 
				font-weight:lighter
				font-size:16px
			.itemtime
				color:#b3b3b3
				font-weight:lighter
				display:flex
				position:relative
				.txitemtitle
					position:absolute
					right:25px
					color:#2f8aff
					.icon-right
						position:relative
						right:-10px
						top:-20px
						color:#666
				.itemtitle
					position:absolute
					right:14px
					color:#06c1ae
			.itemtime >div 
				// font-weight:lighter
				font-size:12px
		.zdItem::after
			content: " "
			position: absolute
			left: 0
			bottom: 0
			width: 100%
			height: 1px
			border-bottom: 1px solid #d6d6d6
			color: #d6d6d6
			transform-origin: 0 100%
			transform: scaleY(.5)
	.foot
		position:fixed
		bottom:0px
		height:50px
		width:100%
		display:flex
		.tixian
			flex:1
			text-align:center
			line-height:60px
			background:#fff
			color:#06c1ae
		.chongzhi
			flex:1
			text-align:center
			line-height:50px
			background:#06c1ae
			color:#fff

</style>