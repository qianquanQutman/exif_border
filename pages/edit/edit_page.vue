<template>
	<view class="content">
		<image class="image" :src="imagePath"></image>
		<button class="save_button" @click="onclickSave()">保存照片</button>
		<text class="info">{{exif.Make}}</text>
		<button class="save_button" @click="gowebView()">H5页面</button>
	</view>
</template>

<script>
	import {getImageData, getFloatLocationByExif}  from '../../static/izExif.js'
	export default {
		data() {
			return {
				imagePath: '',
				title: '编辑',
				exif: {
					Make: '',
					Model: ''
				}
			}
		},
		onLoad() { 
			const eventChannel = this.getOpenerEventChannel(); 
			eventChannel.on('sendImage', (data) => { 
				this.imagePath = data.imagePath; 
				getImageData(this.imagePath).then(res=>{
				    console.log(res);
					// let {exif} = getFloatLocationByExif(res.exif);
					this.exif = res.exif;
				}).catch(e=>{
				    console.log(e)
				})
			}); 
		},
		methods: {
			onclickSave() {
				uni.saveImageToPhotosAlbum({
				        filePath: this.imagePath,
				        success(res) {
				          uni.showToast({
				            title: '保存成功',
				            icon: 'success'
				          });
				        },
				        fail(err) {
				          uni.showToast({
				            title: '保存失败',
				            icon: 'none'
				          });
				        }
				});
			},
			gowebView() {
				uni.navigateTo({
					url: '/pages/edit/web_view',
					success: (res) => {
						
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
	
	.image {
		width: 200px;
		height: 150px;
		background-color: white;
	}
	
	.save_button {
		margin-top: 30px;
	}
	
	.info {
		color: white;
	}

</style>