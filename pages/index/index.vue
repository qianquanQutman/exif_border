<template>
	<view class="content" @click="onClickPage">
		<image class="btn" src="../../static/btn_add.png"></image>
		<text class="text">点按任意位置即可打开照片</text>
	</view>
	
</template>

<script>
	
	export default {
		data() {
			return {
				title: 'logo border'
			}
		},
		onLoad() {

		},
		methods: {
			onClickPage() {
				uni.chooseImage({
					count: 1,
					sizeType: ['original'],
					sourceType: ['album'],
					success: (res) => {
						let tempFilePaths = res.tempFilePaths;
						console.log("imgs===",res);
						uni.navigateTo({
							url: '/pages/edit/canvas_edit_page',
							success: (res) => {
								res.eventChannel.emit('sendImage', { imagePath: tempFilePaths[0] }); 
							}
						})
					}
				})
			}
		}
	}
</script>

<style>
	page {
		background-color: black;
		display: flex;
		flex-direction: column;
		height: 100%;
	}
	.content {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background-color: black;
		height: 100%;
	}

	.btn {
		width: 108px;
		height: 108px;
	}
	
	.text {
		margin-top: 30px;
		color: #E6E6E6;
	}

</style>
