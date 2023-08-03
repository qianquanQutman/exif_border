<template>
	<view class="content">
		<view class="preview">
			<image class="photo" :src="imagePath"></image>
			<view class="exif_border">
				<view class="model_info">
					<view class="model_name">{{exif.Model}}</view>
					<view class="model_time">{{exif.DateTime}}</view>
				</view>
				<view class="logo_info">
					<image :style="{width: brandData.width}" :src="brandData.logoSrc"></image>
					<view class="line"></view>
					<view class="exit_info">
						<view class="photo_exif">{{`${exif.FocalLengthIn35mmFilm}mm  f/${exif.FNumber.numerator / exif.FNumber.denominator} ${ExposureTimeStr} ISO${exif.ISOSpeedRatings}`}}</view>
						<view class="photo_location">49.11N 116.89W</view>
					</view>
				</view>
			</view>
		</view>
		<button class="save_button" @click="onclickSave()">保存照片</button>
		<text class="info">{{exif.Make}}</text>
		<button class="save_button" @click="gowebView()">H5页面</button>
	</view>
</template>

<script>
	import {getImageData, getFloatLocationByExif}  from '../../static/izExif.js';
	import brands from './brand.json';
	
	export default {
		data() {
			return {
				imagePath: '',
				title: '编辑',
				exif: {
					Make: '', // 品牌
					Model: '', //型号
					FocalLengthIn35mmFilm: '', //焦距
					FNumber: { //光圈
						denominator: 1, //倍数吧
						numerator: 0 //光圈值
					},
					ExposureTime: { // 快门时间
						denominator: 1,
						numerator: 0
					},
					ISOSpeedRatings: 100, //ISO
					DateTime: '',
				},
				ExposureTimeStr: '', //快门时间
				brandData: {
					logoSrc: '',
					width: '50px'
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
					let timeStr = '';
					if (this.exif.ExposureTime.denominator == 1) {
						timeStr = `${this.exif.ExposureTime.numerator}s`;
					} else {
						timeStr = `${this.exif.ExposureTime.numerator}/${this.exif.ExposureTime.denominator}s`;
					}
					this.ExposureTimeStr = timeStr;
					this.brandData = brands[this.exif.Make];
					console.log(brands);
					console.log("logo===>", this.brandData );
					
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
		/* justify-content: center; */
		background-color: black;
		height: 100%;
	}
	
	.preview {
		display: flex;
		margin-top: 40px;
		margin-left: 20px;
		margin-right: 20px;
		flex-direction: column;
	}
	
	.photo {
		margin: 0px;
		padding: 0px;
		background-color: darkgrey;
	}
	
	.exif_border {
		display: flex;
		height: 42px;
		background-color: white;
		flex-direction: row;
		align-items: center;
		justify-content: space-between;
	}
	
	.model_info {
		display: flex;
		align-self: flex-start;
		height: 42px;
		margin-left: 12px;
		flex-direction: column;
		justify-content: center;
		/* background-color: antiquewhite; */
	}
	
	.model_name {
		font-size: 12px;
		font-family: 'PingFang-RE';
		color: #606060;
	}
	
	.model_time {
		font-size: 6px;
		font-family: 'PingFang-RE';
		color: #606060;
	}
	
	.logo_info {
		display: flex;
		align-self: flex-end;
		height: 42px;
		margin-right: 12px;
		flex-direction: row;
		align-items: center;
	}
	
	.line {
		width: 1px;
		height: 18px;
		margin-left: 5px;
		margin-right: 5px;
		background-color: #CCC;
	}
	
	.photo_exif {
		font-size: 12px;
		color: #8F8F8F;
	}
	.photo_location {
		font-size: 6px;
		color: #8F8F8F;
	}
	
	.save_button {
		margin-top: 30px;
	}
	
	.info {
		color: white;
	}

</style>