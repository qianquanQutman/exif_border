<template>
	<view class="content">
		<image :src="preview_src" :style="{objectFit: 'cover', width: `${preview_width}px`, height: `${preview_height}px`}"></image>
		<button class="save_button" @click="onclickSave()">保存照片</button>
		<button class="save_button" @click="saveImage()">保存h5照片</button>
		<canvas canvas-id="canvas" :style="{ width: imageWidthPXStr, height: imageHeightPXStr }" class="myCanvas">

		</canvas>
	</view>
</template>

<script>
import { callWithAsyncErrorHandling } from "vue";
import { getImageData, getFloatLocationByExif } from '../../static/izExif.js';
import brands from './brand.json';

export default {
	data() {
		return {
			title: '编辑',
			imagePath: '',
			imageWidthPX: 0,
			imageHeightPX: 0,
			imageWidthPXStr: '0px',
			imageHeightPXStr: '0px',
			takePhotoTime: '',
			preview_src: '',
			preview_width: 320,
			preview_height: 0,
			brandData: {
				logoSrc: '',
				logoPng: '',
				width: '50px',
				widthNum: 50
			},
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
			brandsNameList: ['未知', 'Apple', 'SONY', 'Canon', 'Nikon', 'Hasu', 'Fujifilm', 'Pentax', 'Olympus', 'Zeiss'],
			inputMake: '',
			inputModel: '',
			focalNum: '',
			fNumber: '',
		}
	},
	onLoad() {
		function removeTrailingZeros(str) {
			// 使用正则表达式去除末尾的0
			// 使用正则表达式去除末尾的0，并判断是否以.0结尾
			const result = str.replace(/(\.\d*?[1-9])0+$/, "$1");
			// 如果末尾是.0，则去除.0部分
			return result.endsWith('.0') ? result.slice(0, -2) : result;
		};
		const eventChannel = this.getOpenerEventChannel();
		eventChannel.on('sendImage', (data) => {

			this.imagePath = data.imagePath;
			console.log("getImageData==>", this.imagePath);
			getImageData(this.imagePath).then(res => {
				// let {exif} = getFloatLocationByExif(res.exif);
				this.exif = res.exif;
				console.log("exif===>", this.exif);

				let timeStr = '';
				if (this.exif.ExposureTime.denominator == 1) {
					timeStr = `${this.exif.ExposureTime.numerator}`;
				} else {
					timeStr = `${this.exif.ExposureTime.numerator}/${this.exif.ExposureTime.denominator}`;
				}
				this.ExposureTimeStr = timeStr;

				this.focalNum = removeTrailingZeros((this.exif.FocalLength.numerator / this.exif.FocalLength.denominator).toFixed(1));
				this.fNumber = removeTrailingZeros((this.exif.FNumber.numerator / this.exif.FNumber.denominator).toFixed(2));

				this.imageWidthPX = this.exif.PixelXDimension || 0;
				this.imageHeightPX = this.exif.PixelYDimension || 0;

				if (this.exif.Make) {
					this.brandData = brands[this.exif.Make] || {};
				} else {
					uni.showToast({
						title: '未检测到相机数据, 请手动选择。',
						icon: 'none'
					});
				}

				const moment = require('moment');
				this.takePhotoTime = moment(this.date).format('YYYY-MM-DD HH:mm:ss');

				uni.getImageInfo({
					src: this.imagePath,
					success: (reImg) => {
					console.log("getImageInfo==>", reImg);
					const scale = uni.getSystemInfoSync().devicePixelRatio;
						
					const canvasWidth = parseInt(reImg.width / scale);
					const canvasHeight = parseInt(reImg.height / scale + 240);
					this.imageWidthPXStr = `${canvasWidth}px`;
					this.imageHeightPXStr = `${canvasHeight}px`;

					this.preview_height = parseInt(canvasHeight / canvasWidth * 320);

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
		saveImage() {
			const imageBlob = this.base64ToBlob(this.preview_src);
			const imageUrl = URL.createObjectURL(imageBlob);

			const a = document.createElement('a');
			a.href = imageUrl;
			a.download = 'image.jpg';
			a.click();

			URL.revokeObjectURL(imageUrl);
		},
		base64ToBlob(base64) {
			const parts = base64.split(';base64,');
			const contentType = parts[0].split(':')[1];
			const byteCharacters = atob(parts[1]);
			const byteArrays = new Uint8Array(byteCharacters.length);

			for (let i = 0; i < byteCharacters.length; i++) {
				byteArrays[i] = byteCharacters.charCodeAt(i);
			}

			return new Blob([byteArrays], { type: contentType });
		},
		drawImg() {

			this.$nextTick().then(() => {
				// 图片地址
				const imageUrl = this.imagePath;
				console.log("画图===>", imageUrl);
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
				console.log("size", { imgWidth, imgHeight });
				ctx.drawImage(this.imagePath, 0, 0, imgWidth, imgHeight);
				// 保存当前状态
				ctx.save();

				ctx.setFontSize(50);
				ctx.setFillStyle('#606060');
				ctx.fillText(this.exif.Model, 50, imgHeight + 124);
				const cameraEx = `${this.focalNum}mm  f/${this.fNumber} ${this.ExposureTimeStr}s ISO${this.exif.ISOSpeedRatings}`;
				const cameraExSize = ctx.measureText(cameraEx); // 文字宽度
				ctx.fillText(cameraEx, imgWidth - cameraExSize.width - 50, imgHeight + 110);
				ctx.save();
				console.log("画时间=>", this.takePhotoTime);
				ctx.setFontSize(32);
				ctx.setFillStyle('#8F8F8F');
				ctx.fillText(this.takePhotoTime, imgWidth - cameraExSize.width - 50, imgHeight + 170);
				ctx.save();
				console.log("画分割线=>");
				ctx.setFillStyle('#D8D8D8');
				ctx.fillRect(imgWidth - cameraExSize.width - 100, imgHeight + 70, 3, 100);
				ctx.save();

				const { logoPng, widthNum } = this.brandData;
				console.log("画logo=>", { logoPng, widthNum });

				ctx.drawImage(logoPng, parseInt(imgWidth - cameraExSize.width - 100 - 53 - widthNum * 3), parseInt(imgHeight + 120 - widthNum * 3 / 2), widthNum * 3, widthNum * 3);
				ctx.save();
				console.log("画完毕");

				ctx.draw(true, (file) => {
					console.log("draw=>", file);
					uni.canvasToTempFilePath({
						canvasId: 'canvas',
						fileType: 'jpg',
						quality: 1.0,
						success: res => {
							// 在H5平台下，tempFilePath 为 base64
							console.log("canvasToTempFilePath=>", res);
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

.myCanvas {
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
</style>