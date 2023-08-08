<template>
		<view class="content">
			<image class="preview_img" :src="preview_src"></image>
			<button class="save_button" @click="onclickSave()">保存照片</button>
			<canvas canvas-id="canvas" :style="{ width: imageWidthPXStr, height: imageHeightPXStr}"  class="myCanvas">
				
			</canvas>
		</view>
	
</template>

<script>
	import { callWithAsyncErrorHandling } from "vue";
import {getImageData, getFloatLocationByExif}  from '../../static/izExif.js';
	export default {
		data() {
			return {
				title: '编辑',
				imagePath: '',
				imageWidthPX: 0,
				imageHeightPX: 0,
				imageWidthPXStr: '0px',
				imageHeightPXStr: '0px',
				preview_src: '',
				exif: {
					PixelXDimension: 0, //宽度
					PixelYDimension: 0, //高度
					ImageWidth: 0, //宽度
					ImageHeight: 0, //高度
					Make: '', // 品牌
					Model: '', //型号
					FocalLength: {
						denominator: 1, //倍数吧
						numerator: 0 //焦距值
					},
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
					DateTimeOriginal: '',
					DateTimeDigitized: '',
				},
				ExposureTimeStr: '', //快门时间
				brandData: {
					logoSrc: '',
					width: '50px'
				},
				brandIndex: 0,
				brandsNameList: ['未知','Apple','SONY','Canon','Nikon','Hasu','Fujifilm','Pentax','Olympus','Zeiss'],
				inputMake: '',
				inputModel: '',
				focalNum: '',
				fNumber: '',
			}
		},
		onLoad() {
			const eventChannel = this.getOpenerEventChannel();
			eventChannel.on('sendImage', (data) => { 
				this.imagePath = data.imagePath; 
				console.log("getImageData==>",this.imagePath);
				getImageData(this.imagePath).then(res=>{
					// let {exif} = getFloatLocationByExif(res.exif);
					this.exif = res.exif;
					console.log("exif===>", this.exif);
					
					uni.getImageInfo({
						src: this.imagePath,
						success: (reImg) => {
							this.imageWidthPX = reImg.width || 0;
							this.imageHeightPX = reImg.height || 0;
							const scale = uni.getSystemInfoSync().devicePixelRatio;
							this.imageWidthPXStr = `${this.imageWidthPX / scale}px`;
							this.imageHeightPXStr = `${this.imageHeightPX / scale + 240}px`;
							this.drawImg();
						}
					});
					
					

					
				});
			});
		},
		methods: {
			onclickSave() {
				uni.saveImageToPhotosAlbum({
				  filePath: this.preview_src,
				  success: () => {
				    uni.showToast({ title: '图片保存成功', icon: 'success' });
				  },
				  fail: () => {
				    uni.showToast({ title: '图片保存失败', icon: 'none' });
				  },
				});
			},
			drawImg() {
				
				this.$nextTick().then(() => {
					// 图片地址
					const imageUrl = this.imagePath;
					console.log("画图===>",imageUrl);
					// 创建Canvas绘图上下文
					const ctx = uni.createCanvasContext('canvas');
					// ctx.hidpi = false;
					// 缩放画布坐标，适应高分辨率屏幕
					const scale = uni.getSystemInfoSync().devicePixelRatio;
					// ctx.scale(scale, scale);

					// 将之前在绘图上下文中的描述（路径、变形、样式）画到 canvas 中
					
					// 绘制图片到Canvas
					const imgWidth = this.imageWidthPX / scale;
					const imgHeight = this.imageHeightPX / scale;
					console.log("size", {imgWidth, imgHeight});
					ctx.drawImage(this.imagePath, 0, 0, imgWidth, imgHeight);
					// 保存当前状态
					ctx.save();
					
					ctx.setFontSize(60);
					ctx.setFillStyle('#606060');
					ctx.fillText(this.exif.Model, 50, imgHeight + 124);
					ctx.save();

					ctx.fillText('MINA', 100, 100);
					ctx.draw(true,(file) => {
						console.log("draw=>",file);
						uni.canvasToTempFilePath({
						  canvasId: 'canvas',
						  fileType: 'jpg',
						  quality: 1.0,
						  success: res => {
						    // 在H5平台下，tempFilePath 为 base64
						    console.log("canvasToTempFilePath=>",res);
							// 将Canvas内容导出为图片
							this.preview_src = res.tempFilePath;
							
						  } 
						})
					});
					
				});
			},
		},
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
		background-color: black;
		height: 100%;
	}
	.myCanvas{
		/* z-index: -1;
		position: relative; */
		position: absolute; 
		top: -9999px; 
		left: -9999px
	}
	/* .preview {
		display: none;
		height: 100px;
		width: 100px;
		margin-top: 40px;
		margin-left: 20px;
		margin-right: 20px; 
	} */
	.preview_img {
		background-color: bisque;
		object-fit: cover;
		display: f;
	}
</style>