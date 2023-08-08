<template>
	<view>
		<web-view :webview-styles="webviewStyles" src="http://172.16.9.12:8802/#/chooseImageDemo"></web-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				webviewStyles: {
					progress: {
						color: '#FF3333'
					}
				}
			}
		}
	}
</script>

<style>

</style>