<template>
	<web-view :src="url"></web-view>
</template>

<script>
	export default {
		data() {
			return {
				url: ''
			}
		},
		onLoad(options) {
			if (options.url) {
				this.url = decodeURIComponent(options.url)
			}
			if (options.title) {
				uni.setNavigationBarTitle({
					title: decodeURIComponent(options.title)
				})
			}
		},
		onReady() {
			// #ifdef APP-PLUS
			this.hookWechatOAuth()
			// #endif
		},
		methods: {
			// #ifdef APP-PLUS
			hookWechatOAuth() {
				const self = this
				let count = 0
				const tryHook = () => {
					const pageWebview = self.$scope.$getAppWebview()
					const children = pageWebview && pageWebview.children()
					if (children && children.length) {
						const wv = children[0]
						// 只拦截 weixin:// 协议，拉起微信客户端；其余所有页面（含 open.weixin.qq.com 授权页）均留在当前 WebView 加载
						wv.overrideUrlLoading({
							mode: 'reject',
							match: '^weixin://'
						}, function(e) {
							plus.runtime.openURL(e.url)
						})
						return true
					}
					return false
				}
				const timer = setInterval(() => {
					count++
					if (tryHook() || count > 20) {
						clearInterval(timer)
					}
				}, 200)
			}
			// #endif
		}
	}
</script>

<style>
</style>
