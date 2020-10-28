<template>
	<view class="bannerInfo">
		<web-view :src="h5Url" v-if="h5Url"></web-view>
		<image v-for="url of bigUrls" :src="url" v-if="url"></image>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				bigUrl: '',
				h5Url: ''
			}
		},
		computed: {
			bigUrls: function () {
				return this.bigUrl.split('|')
			}
		},
		// 生命周期
		mounted() {
			const bannerInfo = getApp().globalData.bannerInfo
			const bigUrl = bannerInfo.bigUrl
			const h5Url = bannerInfo.h5Url
			const title = bannerInfo.title
			bigUrl && (this.bigUrl = bigUrl)
			h5Url && (this.h5Url = h5Url)
			title && uni.setNavigationBarTitle({
				title
			})
		}
	}
</script>

<style scoped>
	.bannerInfo {
		width: 100vw;
		height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	
	.bannerInfo web-view {
		width: 100%;
		height: 100%;
	}
</style>
