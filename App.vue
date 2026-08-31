<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style>
	/*每个页面公共css */
	page,
	body,
	html {
		width: 100%;
		max-width: 100%;
		overflow-x: hidden;
	}

	uni-page-body {
		width: 100%;
		max-width: 100%;
		overflow-x: hidden;
	}
</style>
