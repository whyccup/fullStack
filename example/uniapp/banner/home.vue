<template>
	<view class="home">
		<!-- #ifndef H5 -->
		<button :class="loginBtnClass" type="primary" open-type="getUserInfo" @getuserinfo.stop="goWxLogin"> 微信登录</button>
		<!-- #endif -->
		<!-- #ifdef H5 -->
		<button :class="loginBtnClass" @click.stop="goLogin">游客登录</button>
		<!-- #endif -->
		<view class="title" :style="titleMarginTop">
			<image :src="portrait" @click="goMine" class="playerHead"></image>
			<span class="playerNick">{{nickname}}</span>
		</view>
		<view class="banner" v-if="banners.length > 0">
			<swiper class="swiper" circularn :autoplay="bannerAutoplay" :interval="bannerInterval" :current="bannerCurrent" @change="onBannerChange">
				<swiper-item v-for="(banner, i) of banners" :key="i" v-bind:id="i" @click="openBanner">
					<image class="bannerItem" :src="banner.bannerUrl"></image>
				</swiper-item>
			</swiper>
		</view>
		<view class="bannerIconsView">
			<view class="bannerIconClick" v-for="(banner, i) in banners" v-bind:id="i" :key="i" @click="bannerJump">
				<view class="bannerIcon" :class="i === bannerIndex ? 'bannerIconActive' : ''"></view>
			</view>
		</view>
		<view class="evaList">
			<evaItem class="evaItem" v-for="eva of evas" :key="eva._id" :evaData="eva"></evaItem>
		</view>
	</view>
</template>

<style scoped>
	.bigLoginBtn{
		position: fixed;
		width: 100vw;
		height: 100vh;
		opacity: 0;
		z-index: 1;
	}
	
	.smallLoginBtn{
		width: 0rpx;
		height: 0rpx;
		opacity: 0;
	}
	
	.home {
		width: 100vw;
		padding: 0px 41.66rpx 0 41.66rpx;
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-items: center;
		box-sizing:border-box;
	}
	
	/* title为了适配右侧按钮高度,特异使用px */
	.title {
		width: 100%;
		margin-bottom: 22px;
		display: flex;
		justify-content: flex-start;
		align-items: center;
	}

	.playerHead {
		width:32px;
		height:32px;
		border-radius: 50%;
		margin-right: 8px;
		overflow: hidden;
	}

	.playerNick {
		width:192px;
		height:18px;
		font-size:12px;
		font-family:SourceHanSansSC-Regular,SourceHanSansSC;
		font-weight:400;
		color:rgba(51,51,51,1);
		line-height:18px;
		overflow: hidden;
		text-overflow: ellipsis; 
		white-space:nowrap;
	}

	.banner{
		width: 100%;
		height: 266.66rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	
	.banner .swiper{
		height: 100%;
		width: 100%;
		max-width: 666.66rpx;
	}
	
	.banner .bannerItem {
		width: 100%;
		height: 100%;
		border-radius:20.83rpx;
	}
	
	.bannerIconsView {
		width: 100%;
		height: 12.5rpx;
		margin-top: 16.66rpx;
		margin-bottom: 58.33rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.bannerIconClick {
		width: 31.25rpx;
		height: 12.5rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	},
	.bannerIcon {
		width: 8.33rpx;
		height: 8.33rpx;
		border-radius: 50%;
		background-color:rgba(204,204,204,1);
	}
	
	.bannerIconActive {
		width: 12.5rpx;
		height: 12.5rpx;
		background-color:rgba(0,94,216,1);
	}
	
	.evaList{
		width: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	
	.evaItem{
		width: 100%;
		height: 100%;
	}

</style>

<script>
	import wxLogin from '../../common/wxLogin.js'
	import evaItem from '../../components/evaItem.vue'
	
	export default {
		components: {
			evaItem
		},
		data() {
			return {
				loginStatus: 0, // 请求状态，避免多次请求
				loginBtnClass: 'bigLoginBtn', // 登录按钮class
				platId: 0,
				getLastStatus: false, // 是否去请求了下一页
				portrait: '',
				nickname: '',
				banners: [],
				bannerAutoplay: true,
				bannerInterval: 5000,
				bannerCurrent: 0, // 用于修改swiper播放第几个banner
				bannerIndex: 0, // 用于记录现在播到哪个banner了
				evas: [],
				evaLastId: 0
			}
		},
		// 计算属性
		computed: {
			// 获取title与顶部的距离
			titleMarginTop: function () {
				let top = 0
				// #ifndef H5
				// 获取wx右侧悬浮框高度
				const menuButtonInfo = uni.getMenuButtonBoundingClientRect()
				top = menuButtonInfo.top + 2
				// #endif
				// top + 2px = title与顶部的距离
				return `margin-top:${top}px`
			},
			// // 当swiper滑动到对应的点上
			// bannerActiveIcon: function () {
			// 	return function (index) {
			// 		if (this.bannerIndex === index) return `width:12.5rpx;height:12.5rpx;background-color:rgba(0,94,216,1);`
			// 		return ''
			// 	}
			// }
		 },
		onShareAppMessage(option){
			return{
				title:"快来看专业的剧本评测，寻找高质量剧本",
				imageUrl:"https://image.xijing01.com/reviewApplets/shareLogo.png"
			}
		},
		methods: {
			// // 检查是否有用户数据，没有回去登录
			// checkUserData: function() {
			// 	if (!this.userData || !this.userData.uid) {
			// 		this.goIndex()
			// 		return false
			// 	}
			// 	return true
			// },
			
			// // 回登录页
			// goIndex: function () {
			// 	uni.navigateTo({
			// 		url: '/pages/index/index'
			// 	})
			// },
			
			// 去我的页
			goMine: function(){
				uni.navigateTo({
					url: '/pages/playerinfo/playerinfo'
				})
			},
			
			// 打开banner
			openBanner: function(e){
				const banner = this.banners[e.currentTarget.id]
				getApp().globalData.bannerInfo = banner
				switch (banner.type){
					case 1: // 1-跳转到大图
						uni.navigateTo({
							url: `/pages/home/bannerInfo`
						})
						break;
					case 2: //  2-跳转到指定ID的文章页
						this.getEvaInfo(banner.bigUrl)
						break;
					case 3: //  3-跳转到H5
						uni.navigateTo({
							url: `/pages/home/bannerInfo`
						})
						break;
				}
			},
			
			// 刷新页面
			refresher: function() {
				this.evaLastId = 0
				this.getBanner()
				this.getEva()
			},
			
			// 请求测评下一页
			getEvaLast: function() {
				if (!this.evaLastId) return this.getLastStatus = false // 这里必须设定为未进行请求下一页，若不设置，则有可能上一次没lastId，而刷新一次后有了，不能请求下一页。
				this.getEva()
			},
			
			// 获取baner
			getBanner: function() {
				const self = this
				return this.dramaRequest('/activity/u/getOnlineActivity', 'GET', {}).then(function ({list}) {
					self.banners = list
				}).catch(e => {
					console.error(e)
					uni.showToast({
					    title: '获取banner失败',
						icon: 'none'
					});
				})
			},
			
			// 获取评测
			getEva: function() {
				const self = this
				const noLoginUrl = '/article/getOnlineArticle'
				const url = '/article/getOnlineArticle'
				// const url = '/article/u/getOnlineArticle'
				const uid = getApp().globalData.userData.uid
				return this.dramaRequest(uid ? url : noLoginUrl, 'POST', {uid, lastId: this.evaLastId}).then(function ({list, lastId}) {
					// 当未有lastId，证明是第一次获取
					if (!self.evaLastId) {
						while(self.evas.length>list.length){
							self.evas.pop()
						}
						for(let i=0;i<list.length;i++){
							if(i<self.evas.length){
								self.evas.splice(i,1,list[i])
							}else{
								self.evas.push(list[i])
							}
						}
						// 停止下拉刷新动画
						uni.stopPullDownRefresh();
					} else { // 有lastId，证明是取下一页
						for (let el of list) {
							self.evas.push(el)
						}
						// 上拉请求状态改为，未在请求
						self.getLastStatus = false
					}
					// 记录下次请求id
					self.evaLastId = lastId || 0
				}).catch(e => {
					console.error(e)
					uni.showToast({
					    title: '获取评测失败',
						icon: 'none'
					});
					// 停止下拉刷新动画
					uni.stopPullDownRefresh();
					// 上拉请求状态改为，未在请求
					self.getLastStatus = false
				})
			},
			
			// 跳往对应的swiper-item
			bannerJump: function (e) {
				const index = e.currentTarget.id
				this.bannerAutoplay = false
				this.bannerCurrent = index
				setTimeout((self) => {
					self.bannerAutoplay = true
				}, this.bannerInterval / 10, this)
			},
			
			// 当banner改变
			onBannerChange: function (event) {
				this.bannerIndex = event.detail.current
			},
			
			// 请求评测
			getEvaInfo: function (_id) {
				return this.dramaRequest('/article/u/getOneArticleById', 'POST', {
					uid: getApp().globalData.userData.uid,
					_id,
				}).then((info) =>{
					// 存入 文章进全局变量
					getApp().globalData.articleInfo = info
					// 跳往文章页
					uni.navigateTo({
						url: `/pages/article/article?id=${_id}`
					})
				}).catch((e) => {
					if(e.data && e.data.info && e.data.info.errcode=="1009"){
						uni.showModal({
							title:"提示",
							content:"需要进行身份验证后方可浏览剧本测评，是否立即进行验证？",
							success:(res)=>{
								if(res.confirm)
								uni.navigateTo({
									url: "../indentify/indentify"
								})
							}
						})
					}else if(e.data && e.data.info && e.data.info.errcode=="1014"){
						uni.showToast({
							title:"您的身份验证正在审核中，暂时无法查看剧本测评。",
							icon: 'none',
							mask: true,
						})
					}else if(e.data && e.data.info && e.data.info.errcode=="1015"){
						uni.showModal({
							title:"提示",
							content:"您的身份验证失败，是否重新进行验证？",
							success:(res)=>{
								if(res.confirm)
								uni.navigateTo({
									url: "../indentify/indentify"
								})
							}
						})
					} else {
						uni.showToast({
						    title: '获取评测详情失败',
							icon: 'none'
						});
					}
				})
			},
			
			// 获取平台信息
			getPlat: function () {
				const {platform} = uni.getSystemInfoSync()
				switch (platform){
					case 'ios':
						this.platId = 1
						break;
					case 'android':
						this.platId = 2
						break;
				}	
			},
				
			// 开始h5登录
			goLogin: async function () {
				try {
					if (this.loginStatus) return
					this.loginStatus = 1
					uni.showToast({
					    title: '正在登录',
						icon: 'loading',
						duration: 5000
					});
					// 检查有无用户信息
					let userInfo = uni.getStorageSync('h5UserInfo')
					if (userInfo) {
						userInfo = JSON.parse(userInfo)
					} else {
						const code = new Date().getTime() + Math.floor(Math.random() * 1000) +  + ''
						// 服务器 检查code，获取uid
						const {uid} = await this.checkLoginCode({code, loginType: 0})
						userInfo = {uid, nickname: code, portrait: 'https://cdn.xijing01.com/imgs/appHead/deer.png', sex: 1}
					}
					// 服务器 登录
					const result = await this.login({loginType: 0, ...userInfo})
					userInfo.realNameVerify = result.realNameVerify;
					userInfo.sweepFlag = result.sweepFlag;
					// 储存用户信息
					uni.setStorageSync('h5UserInfo', JSON.stringify(userInfo));
					getApp().globalData.userData = userInfo
					getApp().globalData.indentifySwitch = result.verifyIDCardSwitch
					// 清除隐形的登录按钮
					this.clearLoginBtn()
					this.loginStatus = 0
					uni.hideToast()
				} catch (e) {
					this.loginStatus = 0
					if (e === 'reLogin') return // 重新登录错误兜底，不进行任何操作
					uni.showToast({
					    title: '登录失败',
						icon: 'none'
					});
				}
			},
			
			// 开始微信登录
			goWxLogin: async function (wxUserInfo, quickLogin) {
				try{
					if (this.loginStatus) return
					this.loginStatus = 1
					uni.showToast({
					    title: '正在登录',
						icon: 'loading',
						duration: 5000
					});
					// 获取已储存的uid
					let uid = uni.getStorageSync('uid')
					// 检查session
					const sStatus = await wxLogin.checkWxSession()
					// 若session不存在或过期，或没有uid，要去取一次uid
					if (!sStatus || !uid) {
						// 获取code
						const {code} = await wxLogin.getWxCode()
						// 服务器 检查code，获取uid
						const checkRes = await this.checkLoginCode({code, loginType: 1})
						uid = checkRes.uid
					}
					// 非快速登录，才记录wx用户信息
					if (!quickLogin) uni.setStorageSync('wxUserInfo', JSON.stringify(wxUserInfo));
					// 服务器 登录
					const {userInfo, encryptedData, iv} = wxUserInfo.detail
					const userData = {
						uid,
						nickname: userInfo.nickName,
						portrait: userInfo.avatarUrl,
						sex: userInfo.gender
					}
					const result = await this.login({loginType: 1, ...userData, encryptedData, iv})
					userData.realNameVerify = result.realNameVerify;
					userData.sweepFlag = result.sweepFlag;
					getApp().globalData.indentifySwitch = result.verifyIDCardSwitch
					// 储存用户信息
					this.saveUserInfo(userData)
					// 清除隐形的登录按钮
					this.clearLoginBtn()
					this.loginStatus = 0
					uni.hideToast()
				}catch(e){
					this.loginStatus = 0 // 允许请求
					if (e === 'reLogin') return // 重新登录错误兜底，不进行任何操作
					uni.showToast({
					    title: '登录失败',
						icon: 'none'
					});
				}
			},
			
			// 服务器 检查code，获取uid
			checkLoginCode: function ({code, loginType}) {
				return this.dramaRequest('/login/checkLoginCode', 'POST', {
					code,
					loginType,
					platId: this.platId
				})
			},
			
			// 服务器 登录
			login: function ({loginType, uid, nickname, portrait, sex, encryptedData, iv}) {
				return this.dramaRequest('/login/userLogin', 'POST', {
					loginType, // 游客0 微信1
					uid,
					nickname,
					portrait,
					sex,
					encryptedData,
					iv
				})
			},
			
			// 存数据
			saveUserInfo: function ({uid, nickname, portrait, sex, realNameVerify}) {
				uni.setStorageSync('uid', uid);
				getApp().globalData.userData = {
					uid,
					nickname,
					portrait,
					sex,
					realNameVerify
				}
			},
			
			// 清除按钮
			clearLoginBtn: function(){
				this.loginBtnClass = 'smallLoginBtn'
				const userData = getApp().globalData.userData
				this.nickname = userData.nickname
				this.portrait = userData.portrait
			},
			
			// 打开按钮
			openLoginBtn: function(){
				this.loginBtnClass = 'bigLoginBtn'
				getApp().globalData.userData = {}
			},
			
			// 快速登录
			quickLogin: function () {
				// #ifdef H5
				this.goLogin()
				// #endif
				// #ifndef H5
				const wxUserInfoString = uni.getStorageSync('wxUserInfo')
				if (wxUserInfoString) {
					const wxUserInfo = JSON.parse(wxUserInfoString)
					const self = this
					this.goWxLogin(wxUserInfo, 1)
				}
				// #endif
			}
		},
		
		// 生命周期
		onLoad() {
			const self = this
			self.getPlat()
			uni.$on('onGetEvaInfo', (_id) => {
				self.getEvaInfo(_id)
			})
			uni.$on('onOpenLoginBtn', () => {
				uni.redirectTo({
					url: '/pages/home/home',
					complete: self.openLoginBtn
				})
			})
			uni.$on('onArticleLike', (_id, like, num) => {
				for (let eva of this.evas) {
					if (eva._id === _id) {
						eva.hasThumb = like
						eva.thumb = num
					}
				}
			})
		},
		// 当页面实例被创建完成
		created() {
			// 快速登录
			this.quickLogin()
			this.getBanner()
			this.getEva()
			// #ifndef H5
			uni.showShareMenu({
            	withShareTicket: true,
            	menus: ['shareAppMessage', 'shareTimeline']
          	})
			// #endif
		},
		
		// 当页面触发下拉刷新页面
		onPullDownRefresh() {
			this.refresher()
		},
		
		// 当页面触发拉到底部请求
		onReachBottom() {
			// 检查是否现在正在进行上拉请求
			if (!this.getLastStatus) {
				this.getLastStatus = true
				this.getEvaLast()
			}
		}
		
	}
</script>
