<template>
	<view class="content">
	    <canvas canvas-id="canvas" v-if="imageWidthPX > 0" :width="imageWidthPXStr" :height="imageHeightPXStr" class="myCanvas"></canvas>
		<image class="preview_img" :src="preview_src"></image>
	  </view>
</template>

<script>
	import {getImageData, getFloatLocationByExif}  from '../../static/izExif.js';
	export default {
		data() {
			return {
				title: '编辑',
				imagePath: '',
				imageWidthPX: 0,
				imageHeightPX: 0,
				imageWidthPXStr: '0px',
				imageWidthPXStr: '0px',
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
					this.imageWidthPX = this.exif.ImageWidth || this.exif.PixelXDimension || 0;
					this.imageHeightPX = this.exif.ImageHeight || this.exif.PixelYDimension || 0;
					this.imageWidthPXStr = `${this.imageWidthPX}px`;
					this.imageHeightPXStr = `${this.imageHeightPX}px`
					console.log("开始画图");
					this.drawImg();
				});
			});
		},
		methods: {
			drawImg() {
				
				this.$nextTick().then(() => {
					// 图片地址
					const imageUrl = this.imagePath;
					console.log("画图===>",imageUrl);
					// 创建Canvas绘图上下文
					const ctx = uni.createCanvasContext('canvas');
					
					// 将之前在绘图上下文中的描述（路径、变形、样式）画到 canvas 中
					
					// 绘制图片到Canvas
					const imgWidth = this.imageWidthPX;
					const imgHeight = this.imageHeightPX;
					console.log("size", {imgWidth, imgHeight});
					ctx.drawImage(this.imagePath, 0, 0, imgWidth, imgHeight)
					
					// 设置水印文字、字体、颜色、透明度等属性
					var watermark = 'Bing'
					var font = '20px Arial'
					var color = 'red'
					var alpha = 0.5
					// 计算水印的位置和角度
					var x = imgWidth - 10 // 水平偏移量，可以根据需要调整
					var y = imgHeight - 10 // 垂直偏移量，可以根据需要调整
					var angle = -45 // 旋转角度，可以根据需要调整
					// 保存当前状态
					ctx.save()

					ctx.setFontSize(20)
					ctx.fillText('Hello', 20, 20)
					ctx.fillText('MINA', 100, 100)
					ctx.draw(true,(file) => {
						console.log("draw=>",file);
						uni.canvasToTempFilePath({
						  canvasId: 'canvas',
						  success: res => {
						    // 在H5平台下，tempFilePath 为 base64
						    console.log("canvasToTempFilePath=>",res.tempFilePath);
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
		z-index: -1;
		position: relative;
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
		object-fit: contain;
	}
</style>