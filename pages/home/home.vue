<template>
	<view class="page">
		<!-- ===================== 首页 ===================== -->
		<view class="screen home" v-show="!showSub">
			<view class="top" :style="{ paddingTop: statusBarHeight + 'px' }">
				<view class="statusbar">
					<text class="clock">{{ clock }}</text>
					<view class="sb-right">
						<view v-html="icon('signal')" style="color:#fff"></view>
						<view v-html="icon('wifi')" style="color:#fff"></view>
						<view v-html="icon('battery')" style="color:#fff"></view>
					</view>
				</view>
				<view class="home-header">
					<view class="hh-left">
						<view class="hello">早上好，欢迎回来 <text class="wave">👋</text></view>
						<view class="hello-sub">蒙自市崇德学校 · 一体化办公平台</view>
					</view>
					<view class="avatar">崇</view>
				</view>
			</view>

			<view class="searchbar" @click="toast('搜索功能将在下一版本开放')">
				<view class="search-ico" v-html="icon('search')"></view>
				<text>搜索功能 / 表单</text>
			</view>

			<scroll-view class="scroll" scroll-y :show-scrollbar="false">
				<view class="banner">
					<view class="ban-tag">📌 今日</view>
					<view class="ban-title">欢迎使用一体化办公平台</view>
					<view class="ban-date">{{ banDate }}</view>
				</view>

				<view class="section-label">
					<text class="sl-h">快捷入口</text>
					<text class="sl-s">常用功能一键直达</text>
				</view>
				<view class="quick">
					<view class="quick-item" v-for="q in quicks" :key="q.name" @click="openModule(q.id)">
						<view class="quick-ico" :style="{ background: q.grad }" v-html="icon(q.icon)"></view>
						<text>{{ q.name }}</text>
					</view>
				</view>

				<view class="section-label">
					<text class="sl-h">功能模块</text>
					<text class="sl-s">共 3 大类 · 18 项功能</text>
				</view>
				<view class="modules">
					<view class="module-card" v-for="m in modules" :key="m.id" @click="openModule(m.id)">
						<view class="mc-ico" :style="{ background: m.grad }" v-html="icon(m.icon)"></view>
						<view class="mc-body">
							<view class="mc-name">
								<text>{{ m.name }}</text>
								<text class="mc-count">{{ m.count }} 项</text>
							</view>
							<view class="mc-desc">{{ m.desc }}</view>
						</view>
						<view class="mc-arrow" v-html="icon('arrow')"></view>
					</view>
				</view>

				<view class="notice">
					<view class="dot" v-html="icon('bell')"></view>
					<view class="n-body">
						<view class="n-t">系统通知</view>
						<view class="n-s">管理员端功能需向管理员申请权限后使用</view>
					</view>
				</view>
				<view class="scroll-gap"></view>
			</scroll-view>

			<view class="tabbar">
				<view class="tab active" @click="setTab('首页')"><view v-html="icon('home')"></view><text>首页</text></view>
				<view class="tab" @click="setTab('工作台')"><view v-html="icon('grid')"></view><text>工作台</text></view>
				<view class="tab" @click="setTab('消息')"><view v-html="icon('message')"></view><text>消息</text></view>
				<view class="tab" @click="setTab('我的')"><view v-html="icon('user')"></view><text>我的</text></view>
			</view>
		</view>

		<!-- ===================== 二级页 ===================== -->
		<view class="screen sub" v-show="showSub">
			<view class="sub-top" :style="{ paddingTop: statusBarHeight + 'px', background: curGrad }">
				<view class="statusbar" style="background:transparent;color:#fff">
					<text class="clock">{{ clock }}</text>
					<view class="sb-right">
						<view v-html="icon('signal')" style="color:#fff"></view>
						<view v-html="icon('wifi')" style="color:#fff"></view>
						<view v-html="icon('battery')" style="color:#fff"></view>
					</view>
				</view>
				<view class="sub-head">
					<view class="back" @click="goHome"><view v-html="icon('back')"></view></view>
					<view class="t">
						<view class="t-title">{{ curModule ? curModule.name : '' }}</view>
						<view class="t-desc">{{ curModule ? curModule.desc : '' }}</view>
					</view>
				</view>
			</view>

			<scroll-view class="scroll" scroll-y :show-scrollbar="false" style="padding-bottom:40rpx">
				<view v-if="curModule">
					<view class="role-group" v-for="(r, ri) in curModule.roles" :key="ri">
						<view class="role-head">
							<text class="role-chip" :class="r.type === 'staff' ? 'staff' : 'admin'">{{ r.type === 'staff' ? '👨‍🏫 教职工' : '🛡️ 管理人员' }}</text>
						</view>
						<view v-if="r.note" class="role-note">⚠️ {{ r.note }}</view>
						<view class="fn-list">
							<view class="fn-item" v-for="it in r.items" :key="it.n" @click="openFn(ri, it)">
								<view class="fn-ico" :style="{ background: it.c }" v-html="icon(it.i)"></view>
								<view class="fn-body">
									<view class="fn-name">{{ it.n }}</view>
									<view class="fn-desc">{{ it.d }}</view>
								</view>
								<view class="fn-arrow" v-html="icon('arrow')"></view>
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>

		<!-- ===================== 抽屉 / 遮罩 / toast ===================== -->
		<view class="sheet-mask" :class="{ show: sheetShow }" @click="closeSheet"></view>
		<view class="sheet" :class="{ show: sheetShow }">
			<view class="grab"></view>
			<view class="s-ico" :style="{ background: sheetItem ? sheetItem.c : '#3B6DFF' }" v-html="icon(sheetItem ? sheetItem.i : 'link')"></view>
			<view class="s-name">{{ sheetItem ? sheetItem.n : '' }}</view>
			<view class="s-sub">{{ sheetSub }}</view>
			<view class="s-link"><view v-html="icon('link')"></view><text class="s-link-code">{{ sheetItem ? sheetItem.u : '' }}</text></view>
			<view class="s-btn" @click="openLink">打开链接 → 进入表单</view>
			<view class="s-btn-browser" @click="openInBrowser">用默认浏览器打开</view>
			<view class="s-btn-wechat" @click="openInWechat">跳转至微信打开</view>
			<view class="s-cancel" @click="closeSheet">取消</view>
		</view>
		<view class="toast" :class="{ show: toastShow }">{{ toastMsg }}</view>
	</view>
</template>

<script>
	function svg(p) {
		return '<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">' + p + '</svg>'
	}
	const ICON = {
		book: svg('<path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"/><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"/>'),
		tool: svg('<path d="M14.7 6.3a1 1 0 0 0 0 1.4l1.6 1.6a1 1 0 0 0 1.4 0l3.77-3.77a6 6 0 0 1-7.94 7.94l-6.91 6.91a2.12 2.12 0 0 1-3-3l6.91-6.91a6 6 0 0 1 7.94-7.94l-3.76 3.76z"/>'),
		users: svg('<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/>'),
		clipboard: svg('<path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><rect x="8" y="2" width="8" height="4" rx="1"/><line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="13" y2="16"/>'),
		swap: svg('<polyline points="17 1 21 5 17 9"/><path d="M3 11V9a4 4 0 0 1 4-4h14"/><polyline points="7 23 3 19 7 15"/><path d="M21 13v2a4 4 0 0 1-4 4H3"/>'),
		search: svg('<circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>'),
		printer: svg('<polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/><rect x="6" y="14" width="12" height="8"/>'),
		chart: svg('<line x1="12" y1="20" x2="12" y2="10"/><line x1="18" y1="20" x2="18" y2="4"/><line x1="6" y1="20" x2="6" y2="16"/>'),
		check: svg('<path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/>'),
		eye: svg('<path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/>'),
		package: svg('<path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/>'),
		phone: svg('<path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/>'),
		edit: svg('<path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/>'),
		shield: svg('<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>'),
		arrow: svg('<line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>'),
		back: svg('<polyline points="15 18 9 12 15 6"/>'),
		link: svg('<path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/><polyline points="15 3 21 3 21 9"/><line x1="10" y1="14" x2="21" y2="3"/>'),
		bell: svg('<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/>'),
		home: svg('<path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/>'),
		grid: svg('<rect x="3" y="3" width="7" height="7" rx="1.5"/><rect x="14" y="3" width="7" height="7" rx="1.5"/><rect x="14" y="14" width="7" height="7" rx="1.5"/><rect x="3" y="14" width="7" height="7" rx="1.5"/>'),
		message: svg('<path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"/>'),
		user: svg('<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>'),
		signal: '<svg width="17" height="11" viewBox="0 0 17 11" fill="currentColor"><rect x="0" y="6" width="3" height="5" rx="1"/><rect x="4.5" y="4" width="3" height="7" rx="1"/><rect x="9" y="2" width="3" height="9" rx="1"/><rect x="13.5" y="0" width="3" height="11" rx="1"/></svg>',
		wifi: '<svg width="16" height="11" viewBox="0 0 16 11" fill="currentColor"><path d="M8 2.5c2.5 0 4.8 1 6.5 2.6l1.4-1.4C13.9 1.7 11 0.5 8 0.5S2.1 1.7 0.1 3.7l1.4 1.4C3.2 3.5 5.5 2.5 8 2.5z"/><path d="M8 6c1.4 0 2.7 0.6 3.7 1.5l1.4-1.4C11.7 4.7 9.9 4 8 4S4.3 4.7 2.9 6.1l1.4 1.4C5.3 6.6 6.6 6 8 6z"/><circle cx="8" cy="9.3" r="1.6"/></svg>',
		battery: '<svg width="25" height="12" viewBox="0 0 25 12" fill="none"><rect x="0.5" y="0.5" width="21" height="11" rx="3" stroke="currentColor" opacity=".5"/><rect x="2" y="2" width="16" height="8" rx="1.5" fill="currentColor"/><rect x="22.5" y="3.5" width="2" height="5" rx="1" fill="currentColor" opacity=".5"/></svg>'
	}

	const DATA = [{
		id: 'teaching', name: '教学管理', color: '#3B6DFF', grad: 'linear-gradient(135deg,#3B6DFF,#6A8BFF)',
		icon: 'book', desc: '课堂日志 · 调课申请 · 印刷管理', count: 10,
		roles: [{
			name: '任课教师', type: 'staff',
			items: [
				{ n: '课堂日志（初中部）', d: '填写每日课堂情况', i: 'clipboard', c: '#3B6DFF', u: 'https://f.wps.cn/g/kYEy2bJO/' },
				{ n: '教师调课申请', d: '提交调课申请表单', i: 'swap', c: '#00B894', u: 'https://f.wps.cn/g/1L5U5MyE/' },
				{ n: '调课申请审核情况查询', d: '查询调课申请审核进度', i: 'search', c: '#6A7BFF', u: 'https://www.kdocs.cn/wo/sl/v11de30g' },
				{ n: '资料印刷申请', d: '提交资料印制申请', i: 'printer', c: '#FF9F43', u: 'https://f.wps.cn/g/cN8Y1EFe/' },
				{ n: '印刷进度查询', d: '查询印制处理进度', i: 'chart', c: '#F56565', u: 'https://www.kdocs.cn/etapps/query/q/g6nKkxFm' }
			]
		}, {
			name: '管理人员', type: 'admin', note: '初次使用需申请权限，请备注姓名',
			items: [
				{ n: '调课申请审核', d: '审核教师调课申请', i: 'check', c: '#3B6DFF', u: 'https://www.kdocs.cn/wo/sl/v1ZnOGh' },
				{ n: '班级人数修改（班主任）', d: '维护班级学生人数', i: 'users', c: '#6A7BFF', u: 'https://www.kdocs.cn/l/coeb0xCvtMgB' },
				{ n: '日常查课（初中部）', d: '每日巡课记录', i: 'eye', c: '#00B894', u: 'https://www.kdocs.cn/l/cdqOe3fhNPV5' },
				{ n: '资料印刷审核', d: '审核印刷申请', i: 'shield', c: '#FF9F43', u: 'https://www.kdocs.cn/wo/sl/v12LUEIW' },
				{ n: '文印室管理员终端', d: '文印室后台管理', i: 'printer', c: '#F56565', u: 'https://www.kdocs.cn/wo/sl/v1wsEnq' }
			]
		}]
	}, {
		id: 'logistics', name: '后勤管理', color: '#FF9F43', grad: 'linear-gradient(135deg,#FF9F43,#FFB36B)',
		icon: 'tool', desc: '后勤报修 · 物品申请 · 报修管理', count: 6,
		roles: [{
			name: '教职工', type: 'staff',
			items: [
				{ n: '后勤报修', d: '提交设备设施报修', i: 'tool', c: '#FF9F43', u: 'https://www.kdocs.cn/wo/sl/v12Qw43K' },
				{ n: '报修办结情况查询', d: '查询报修处理进度', i: 'search', c: '#6A7BFF', u: 'https://www.kdocs.cn/wo/sl/v12HrUYh' },
				{ n: '物品申请', d: '提交物品领用申请', i: 'package', c: '#00B894', u: 'https://f.wps.cn/g/9BRiWTVp/' },
				{ n: '物品申请审核情况查询', d: '查询物品申请进度', i: 'search', c: '#F56565', u: 'https://www.kdocs.cn/etapps/query/q/np7i289X' }
			]
		}, {
			name: '后勤管理员', type: 'admin', note: '初次使用需申请权限，请备注姓名',
			items: [
				{ n: '报修管理', d: '处理报修工单', i: 'tool', c: '#FF9F43', u: 'https://www.kdocs.cn/l/cgvZwQ0NYF8s' },
				{ n: '物品申请审核', d: '审核物品申请', i: 'check', c: '#00B894', u: 'https://www.kdocs.cn/l/ci1vR1gxhq8H' }
			]
		}]
	}, {
		id: 'admin', name: '行政管理', color: '#8B5CF6', grad: 'linear-gradient(135deg,#8B5CF6,#B18CFF)',
		icon: 'users', desc: '通讯录 · 校内信息管理', count: 2,
		roles: [{
			name: '教职工', type: 'staff',
			items: [
				{ n: '教职工通讯录', d: '查看校内教职工联系方式', i: 'phone', c: '#8B5CF6', u: 'https://web.wps.cn/wo/sl/v32fwWCG?app_id=2jyl5GHXoqWXqHmUxjU9Ur' }
			]
		}, {
			name: '行政管理人员', type: 'admin',
			items: [
				{ n: '教职工校内信息管理', d: '维护教职工档案信息', i: 'edit', c: '#6A7BFF', u: 'https://www.kdocs.cn/l/cod7nafNk5QH' }
			]
		}]
	}]

	const QUICKS = [
		{ name: '课堂日志', id: 'teaching', icon: 'clipboard', grad: 'linear-gradient(135deg,#3B6DFF,#6A8BFF)' },
		{ name: '调课申请', id: 'teaching', icon: 'swap', grad: 'linear-gradient(135deg,#00B894,#3ED0A8)' },
		{ name: '后勤报修', id: 'logistics', icon: 'tool', grad: 'linear-gradient(135deg,#FF9F43,#FFB36B)' },
		{ name: '通讯录', id: 'admin', icon: 'phone', grad: 'linear-gradient(135deg,#8B5CF6,#B18CFF)' }
	]

	// ================= 微信小程序唤起说明 =================
	// 「跳转至微信打开」改为直接用 WebView 加载金山文档分享链接，
	// 由金山文档 H5 分享页自带的「拉起微信小程序」能力处理唤起逻辑，
	// WebView 仅需放行 weixin:// 协议跳转（见 pages/webview/webview.vue）。

	export default {
		data() {
			return {
				statusBarHeight: 0,
				clock: '',
				banDate: '',
				modules: DATA,
				quicks: QUICKS,
				showSub: false,
				curModule: null,
				curGrad: 'linear-gradient(135deg,#3B6DFF,#6A8BFF)',
				curRoleIndex: 0,
				sheetShow: false,
				sheetItem: null,
				sheetSub: '',
				toastShow: false,
				toastMsg: '',
				timer: null,
				toastTimer: null
			}
		},
		onLoad() {
			try {
				const info = uni.getSystemInfoSync()
				this.statusBarHeight = info.statusBarHeight || 0
			} catch (e) {
				this.statusBarHeight = 0
			}
			this.updateClock()
			this.timer = setInterval(() => {
				this.updateClock()
			}, 30000)
			const d = new Date()
			const wd = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六']
			this.banDate = d.getFullYear() + '年' + (d.getMonth() + 1) + '月' + d.getDate() + '日 ' + wd[d.getDay()]
		},
		onUnload() {
			if (this.timer) clearInterval(this.timer)
			if (this.toastTimer) clearTimeout(this.toastTimer)
		},
		onBackPress(options) {
			if (this.showSub) {
				this.showSub = false
				return true
			}
			if (this.sheetShow) {
				this.sheetShow = false
				return true
			}
			return false
		},
		methods: {
			icon(name) {
				return ICON[name] || ''
			},
			updateClock() {
				const d = new Date()
				const pad = n => (n < 10 ? '0' + n : n)
				this.clock = pad(d.getHours()) + ':' + pad(d.getMinutes())
			},
			openModule(id) {
				const m = this.modules.find(x => x.id === id)
				if (!m) return
				this.curModule = m
				this.curGrad = 'linear-gradient(135deg,' + m.color + ',' + m.color + 'cc)'
				this.showSub = true
			},
			goHome() {
				this.showSub = false
			},
			openFn(ri, it) {
				if (!this.curModule) return
				this.sheetItem = it
				this.sheetSub = this.curModule.name + ' · ' + this.curModule.roles[ri].name
				this.sheetShow = true
			},
			closeSheet() {
				this.sheetShow = false
			},
			openLink() {
				if (!this.sheetItem || !this.sheetItem.u) return
				this.sheetShow = false
				uni.navigateTo({
					url: '/pages/webview/webview?url=' + encodeURIComponent(this.sheetItem.u) + '&title=' + encodeURIComponent(this.sheetItem.n)
				})
			},
			openInBrowser() {
				if (!this.sheetItem || !this.sheetItem.u) return
				const url = this.sheetItem.u
				this.sheetShow = false
				// #ifdef APP-PLUS
				plus.runtime.openURL(url)
				// #endif
				// #ifdef H5
				window.open(url, '_blank')
				// #endif
			},
			openInWechat() {
				if (!this.sheetItem || !this.sheetItem.u) return
				const url = this.sheetItem.u
				const title = this.sheetItem.n
				this.sheetShow = false
				// 直接用 WebView 加载文档分享链接，页面自带「拉起微信小程序」能力，
				// WebView 放行 weixin:// 协议即可自动唤起微信小程序。
				uni.navigateTo({
					url: '/pages/webview/webview?url=' + encodeURIComponent(url) + '&title=' + encodeURIComponent(title)
				})
			},
			setTab(name) {
				if (name !== '首页') {
					this.toast('「' + name + '」页面开发中')
				}
			},
			toast(msg) {
				this.toastMsg = msg
				this.toastShow = true
				if (this.toastTimer) clearTimeout(this.toastTimer)
				this.toastTimer = setTimeout(() => {
					this.toastShow = false
				}, 1600)
			}
		}
	}
</script>

<style>
	page {
		background: #F4F6FB;
		height: 100%;
	}

	.page {
		--brand: #3D5AFE;
		--brand-2: #6A7BFF;
		--brand-3: #8E8BFF;
		--ink: #1A2340;
		--muted: #8A93A6;
		--line: #ECF0F7;
		--shadow-sm: 0 6rpx 20rpx rgba(43, 60, 120, .08);
		--shadow: 0 16rpx 48rpx rgba(43, 60, 120, .10);
		position: relative;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}

	/* ============ 屏幕层 ============ */
	.screen {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		width: 100%;
		max-width: 100vw;
		display: flex;
		flex-direction: column;
		background: #F4F6FB;
		transition: transform .48s cubic-bezier(.32, .72, .28, 1);
	}
	.screen.home {
		z-index: 2;
	}
	.screen.sub {
		z-index: 3;
		background: #fff;
		animation: subIn .3s ease;
	}
	@keyframes subIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	/* ============ 顶部渐变区 ============ */
	.top {
		flex: none;
		background: linear-gradient(135deg, #3D5AFE 0%, #6A7BFF 55%, #8E8BFF 120%);
	}
	.statusbar {
		height: 88rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 52rpx;
		font-size: 28rpx;
		font-weight: 600;
		color: #fff;
	}
	.sb-right {
		display: flex;
		align-items: center;
		gap: 12rpx;
	}
	.home-header {
		padding: 8rpx 40rpx 32rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
		color: #fff;
	}
	.hello {
		font-size: 40rpx;
		font-weight: 700;
		letter-spacing: .6rpx;
	}
	.wave {
		display: inline-block;
		animation: wave 2.2s ease infinite;
		transform-origin: 70% 70%;
	}
	@keyframes wave {
		0%, 60%, 100% { transform: rotate(0); }
		10% { transform: rotate(14deg); }
		20% { transform: rotate(-8deg); }
		30% { transform: rotate(14deg); }
		40% { transform: rotate(-4deg); }
		50% { transform: rotate(10deg); }
	}
	.hello-sub {
		font-size: 24rpx;
		opacity: .85;
		margin-top: 6rpx;
	}
	.avatar {
		width: 88rpx;
		height: 88rpx;
		border-radius: 28rpx;
		background: rgba(255, 255, 255, .22);
		border: 2rpx solid rgba(255, 255, 255, .35);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 34rpx;
		font-weight: 700;
		color: #fff;
	}

	/* ============ 搜索栏 ============ */
	.searchbar {
		flex: none;
		margin: 28rpx 40rpx 4rpx;
		height: 84rpx;
		border-radius: 28rpx;
		background: #fff;
		box-shadow: var(--shadow-sm);
		display: flex;
		align-items: center;
		gap: 18rpx;
		padding: 0 28rpx;
		color: var(--muted);
		font-size: 26rpx;
		border: 2rpx solid var(--line);
	}
	.search-ico {
		display: flex;
		align-items: center;
		color: var(--muted);
	}

	.scroll {
		flex: 1;
		height: 0;
		width: 100%;
		max-width: 100%;
		box-sizing: border-box;
		padding: 0 40rpx;
	}
	.scroll-gap {
		height: 32rpx;
	}

	/* ============ 横幅 ============ */
	.banner {
		margin-top: 28rpx;
		border-radius: 36rpx;
		padding: 30rpx 32rpx;
		color: #fff;
		position: relative;
		overflow: hidden;
		background: linear-gradient(120deg, #2b3f9e 0%, #3D5AFE 100%);
		box-shadow: 0 24rpx 52rpx -16rpx rgba(61, 90, 254, .55);
	}
	.banner::after {
		content: "";
		position: absolute;
		right: -52rpx;
		top: -60rpx;
		width: 220rpx;
		height: 220rpx;
		border-radius: 50%;
		background: rgba(255, 255, 255, .12);
	}
	.banner::before {
		content: "";
		position: absolute;
		right: 36rpx;
		bottom: -68rpx;
		width: 160rpx;
		height: 160rpx;
		border-radius: 50%;
		background: rgba(255, 255, 255, .10);
	}
	.ban-tag {
		display: inline-flex;
		align-items: center;
		gap: 10rpx;
		font-size: 22rpx;
		padding: 6rpx 18rpx;
		border-radius: 40rpx;
		background: rgba(255, 255, 255, .2);
		margin-bottom: 14rpx;
	}
	.ban-title {
		font-size: 32rpx;
		font-weight: 700;
		position: relative;
		z-index: 1;
	}
	.ban-date {
		font-size: 23rpx;
		opacity: .85;
		margin-top: 8rpx;
		position: relative;
		z-index: 1;
	}

	/* ============ 分区标题 ============ */
	.section-label {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin: 36rpx 4rpx 20rpx;
	}
	.sl-h {
		font-size: 30rpx;
		font-weight: 700;
		color: var(--ink);
	}
	.sl-s {
		font-size: 22rpx;
		color: var(--muted);
	}

	/* ============ 快捷入口 ============ */
	.quick {
		display: flex;
		flex-wrap: wrap;
	}
	.quick-item {
		width: 25%;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 14rpx;
		margin-bottom: 12rpx;
	}
	.quick-ico {
		width: 104rpx;
		height: 104rpx;
		border-radius: 32rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		color: #fff;
		box-shadow: var(--shadow-sm);
		transition: transform .18s ease;
	}
	.quick-item:active .quick-ico {
		transform: scale(.9);
	}
	.quick-item text {
		font-size: 23rpx;
		color: #3c4660;
		font-weight: 500;
	}

	/* ============ 功能模块卡片 ============ */
	.modules {
		display: flex;
		flex-direction: column;
		gap: 24rpx;
	}
	.module-card {
		display: flex;
		align-items: center;
		gap: 28rpx;
		padding: 32rpx;
		border-radius: 36rpx;
		background: #fff;
		box-shadow: var(--shadow-sm);
		border: 2rpx solid var(--line);
		position: relative;
		overflow: hidden;
	}
	.module-card::before {
		content: "";
		position: absolute;
		left: 0;
		top: 0;
		bottom: 0;
		width: 8rpx;
		background: #3B6DFF;
		border-radius: 0 6rpx 6rpx 0;
	}
	.mc-ico {
		width: 100rpx;
		height: 100rpx;
		border-radius: 30rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		color: #fff;
		flex: none;
	}
	.mc-body {
		flex: 1;
		min-width: 0;
	}
	.mc-name {
		font-size: 31rpx;
		font-weight: 700;
		color: var(--ink);
		display: flex;
		align-items: center;
	}
	.mc-count {
		margin-left: 8rpx;
		font-size: 20rpx;
		padding: 4rpx 14rpx;
		border-radius: 40rpx;
		background: #EEF2FF;
		color: #3B6DFF;
		font-weight: 600;
	}
	.mc-desc {
		font-size: 23rpx;
		color: var(--muted);
		margin-top: 6rpx;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	.mc-arrow {
		color: #c4cad6;
		flex: none;
	}

	/* ============ 通知 ============ */
	.notice {
		margin-top: 32rpx;
		background: #fff;
		border-radius: 32rpx;
		padding: 26rpx 28rpx;
		box-shadow: var(--shadow-sm);
		border: 2rpx solid var(--line);
		display: flex;
		gap: 22rpx;
		align-items: center;
	}
	.notice .dot {
		width: 72rpx;
		height: 72rpx;
		border-radius: 22rpx;
		background: #FFF3E6;
		color: #FF9F43;
		display: flex;
		align-items: center;
		justify-content: center;
		flex: none;
	}
	.n-t {
		font-size: 26rpx;
		font-weight: 600;
		color: var(--ink);
	}
	.n-s {
		font-size: 22rpx;
		color: var(--muted);
		margin-top: 4rpx;
	}

	/* ============ 底部导航 ============ */
	.tabbar {
		flex: none;
		height: 128rpx;
		background: #fff;
		border-top: 2rpx solid var(--line);
		display: flex;
		align-items: center;
		padding-bottom: 12rpx;
	}
	.tab {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 6rpx;
		color: #9aa3b5;
		font-size: 21rpx;
		transition: color .2s;
	}
	.tab.active {
		color: #3D5AFE;
	}

	/* ============ 二级页 ============ */
	.sub-top {
		flex: none;
		background: linear-gradient(135deg, #3D5AFE 0%, #6A7BFF 55%, #8E8BFF 120%);
	}
	.sub-head {
		padding: 4rpx 32rpx 28rpx;
		color: #fff;
		display: flex;
		align-items: center;
		gap: 20rpx;
	}
	.sub-head .back {
		width: 68rpx;
		height: 68rpx;
		border-radius: 22rpx;
		background: rgba(255, 255, 255, .2);
		display: flex;
		align-items: center;
		justify-content: center;
		flex: none;
		color: #fff;
	}
	.sub-head .t {
		flex: 1;
	}
	.t-title {
		font-size: 36rpx;
		font-weight: 700;
	}
	.t-desc {
		font-size: 23rpx;
		opacity: .85;
		margin-top: 4rpx;
	}
	.role-group {
		padding: 32rpx 32rpx 8rpx;
	}
	.role-head {
		display: flex;
		align-items: center;
		gap: 16rpx;
		margin-bottom: 22rpx;
	}
	.role-chip {
		font-size: 22rpx;
		font-weight: 700;
		padding: 6rpx 20rpx;
		border-radius: 16rpx;
	}
	.role-chip.staff {
		background: #EEF2FF;
		color: #3B6DFF;
	}
	.role-chip.admin {
		background: #F3EEFF;
		color: #8B5CF6;
	}
	.role-note {
		font-size: 21rpx;
		color: #E9A23B;
		background: #FFF8EC;
		border: 2rpx solid #FFE7C2;
		border-radius: 16rpx;
		padding: 12rpx 20rpx;
		margin: -4rpx 0 24rpx;
		line-height: 1.5;
	}
	.fn-list {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}
	.fn-item {
		display: flex;
		align-items: center;
		gap: 24rpx;
		background: #fff;
		border: 2rpx solid var(--line);
		border-radius: 30rpx;
		padding: 24rpx 26rpx;
		box-shadow: var(--shadow-sm);
	}
	.fn-ico {
		width: 80rpx;
		height: 80rpx;
		border-radius: 24rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		color: #fff;
		flex: none;
	}
	.fn-body {
		flex: 1;
		min-width: 0;
	}
	.fn-name {
		font-size: 28rpx;
		font-weight: 600;
		color: var(--ink);
	}
	.fn-desc {
		font-size: 22rpx;
		color: var(--muted);
		margin-top: 4rpx;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	.fn-arrow {
		color: #c4cad6;
		flex: none;
	}

	/* ============ 底部抽屉 ============ */
	.sheet-mask {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(10, 15, 35, .45);
		opacity: 0;
		pointer-events: none;
		transition: opacity .3s;
		z-index: 80;
	}
	.sheet-mask.show {
		opacity: 1;
		pointer-events: auto;
	}
	.sheet {
		position: absolute;
		left: 0;
		right: 0;
		bottom: 0;
		background: #fff;
		border-radius: 52rpx 52rpx 0 0;
		padding: 28rpx 40rpx 52rpx;
		transform: translateY(105%);
		transition: transform .38s cubic-bezier(.32, .72, .28, 1);
		z-index: 81;
		box-shadow: 0 -40rpx 100rpx rgba(0, 0, 0, .2);
	}
	.sheet.show {
		transform: translateY(0);
	}
	.sheet .grab {
		width: 80rpx;
		height: 9rpx;
		border-radius: 6rpx;
		background: #E3E8F0;
		margin: 0 auto 28rpx;
	}
	.sheet .s-ico {
		width: 104rpx;
		height: 104rpx;
		border-radius: 30rpx;
		color: #fff;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 22rpx;
		box-shadow: var(--shadow-sm);
	}
	.sheet .s-name {
		font-size: 34rpx;
		font-weight: 700;
		color: var(--ink);
	}
	.sheet .s-sub {
		font-size: 24rpx;
		color: var(--muted);
		margin-top: 8rpx;
	}
	.sheet .s-link {
		display: flex;
		align-items: center;
		gap: 14rpx;
		margin-top: 24rpx;
		font-size: 23rpx;
		color: #3B6DFF;
		background: #EEF2FF;
		border-radius: 20rpx;
		padding: 16rpx 24rpx;
		overflow: hidden;
	}
	.s-link-code {
		font-family: ui-monospace, SFMono-Regular, monospace;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		color: #4a5bb0;
		flex: 1;
	}
	.sheet .s-btn {
		margin-top: 32rpx;
		height: 96rpx;
		border-radius: 28rpx;
		background: linear-gradient(135deg, #3D5AFE, #6A7BFF);
		color: #fff;
		font-size: 30rpx;
		font-weight: 600;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 20rpx 44rpx -16rpx rgba(61, 90, 254, .6);
	}
	.sheet .s-btn:active {
		transform: scale(.98);
	}
	.sheet .s-btn-browser {
		margin-top: 18rpx;
		height: 88rpx;
		border-radius: 28rpx;
		background: #EEF2FF;
		color: #3D5AFE;
		font-size: 28rpx;
		font-weight: 600;
		display: flex;
		align-items: center;
		justify-content: center;
		border: 2rpx solid #D8E0FF;
	}
	.sheet .s-btn-browser:active {
		transform: scale(.98);
	}
	.sheet .s-btn-wechat {
		margin-top: 18rpx;
		height: 88rpx;
		border-radius: 28rpx;
		background: #E8FBF1;
		color: #07C160;
		font-size: 28rpx;
		font-weight: 600;
		display: flex;
		align-items: center;
		justify-content: center;
		border: 2rpx solid #B8F0D3;
	}
	.sheet .s-btn-wechat:active {
		transform: scale(.98);
	}
	.sheet .s-cancel {
		margin-top: 18rpx;
		height: 88rpx;
		border-radius: 28rpx;
		background: #F2F4F9;
		color: #6a7385;
		font-size: 28rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	/* ============ Toast ============ */
	.toast {
		position: absolute;
		left: 50%;
		bottom: 220rpx;
		transform: translate(-50%, 40rpx);
		background: rgba(20, 26, 45, .92);
		color: #fff;
		font-size: 25rpx;
		padding: 20rpx 36rpx;
		border-radius: 24rpx;
		opacity: 0;
		pointer-events: none;
		transition: all .3s;
		z-index: 90;
		white-space: nowrap;
	}
	.toast.show {
		opacity: 1;
		transform: translate(-50%, 0);
	}
</style>
