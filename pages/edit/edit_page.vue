<template>
	<view class="content">
		<view class="preview" id="saveview">
			<image class="photo w335" :src="imagePath"></image>
			<view class="exif_border">
				<view class="model_info">
					<view class="model_name">{{ inputModel ? inputModel : exif.Model }}</view>
					
				</view>
				<view class="logo_info">
					<image :style="{width: brandData.width}" :src="brandData.logoSrc"></image>
					<view class="line"></view>
					<view class="exit_info">
						<view class="photo_exif">{{`${focalNum}mm  f/${fNumber} ${ExposureTimeStr}s ISO${exif.ISOSpeedRatings}`}}</view>
						<view class="model_time">{{exif.DateTimeOriginal}}</view>
						<!-- <view class="photo_location">49.11N 116.89W</view> -->
					</view>
				</view>
			</view>
		</view>
		<view class="operation_area w335">
			<view class="brand_choose ">
				<view class="uni-list-cell-left brand_input_name">
					品牌
				</view>
				<view class="uni-list-cell-db">
					<picker @change="bindPickerChange" :value="brandIndex" :range="brandsNameList">
						<view class="uni-input brand_input_name ml30">
						{{ inputMake || exif.Make || "未知" }}
						</view>
					</picker>
				</view>
			</view>
			<view class="brand_choose">
				<view class="uni-list-cell-left brand_input_name">型号</view>
				<input class="uni-input brand_input_name ml30" placeholder="型号" @input="onKeyInput" maxlength="10" v-model="inputModel" />
			</view>
			
		</view>
		
		<button class="save_button" @click="onclickSave()">保存照片</button>
		<button class="save_button" @click="gowebView()">H5页面</button>
		<canvas id="myCanvas" style="display: none;"></canvas>
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
				getImageData(this.imagePath).then(res=>{
				    console.log(res);
					// let {exif} = getFloatLocationByExif(res.exif);
					this.exif = res.exif;
					let timeStr = '';
					if (this.exif.ExposureTime.denominator == 1) {
						timeStr = `${this.exif.ExposureTime.numerator}`;
					} else {
						timeStr = `${this.exif.ExposureTime.numerator}/${this.exif.ExposureTime.denominator}`;
					}

					this.focalNum = removeTrailingZeros((this.exif.FocalLength.numerator / this.exif.FocalLength.denominator).toFixed(1));
					this.fNumber = removeTrailingZeros((this.exif.FNumber.numerator / this.exif.FNumber.denominator).toFixed(2));
					
					this.ExposureTimeStr = timeStr;
					this.inputModel = this.exif.Model || '';
					if (this.exif.Make) {
						this.brandData = brands[this.exif.Make] || {};
					} else {
						uni.showToast({
				            title: '未检测到相机数据, 请手动选择。',
				            icon: 'none'
				        });
					}
					
				}).catch(e=>{
				    console.log(e)
				})
			}); 
		},
		methods: {
			onclickSave() {
				
					// 获取<canvas>标签
				    const canvas = uni.createCanvasContext('myCanvas', this);
					const query = uni.createSelectorQuery().in(this);
					
				      // 获取<view>标签的内容
				      query.select('#saveview')
				        .fields({ node: true, size: true })
				        .exec((res) => {
							console.log("save====>",res);
				          const content = res[0];
						  if (!content) {
						                console.log('获取<view>内容失败');
						                return;
						 }
				          const width = content.width;
				          const height = content.height;
				
				          // 绘制<view>的内容到<canvas>
				          canvas.drawImage(content.node, 0, 0, width, height, 0, 0, width, height);
				          canvas.draw(false, () => {
				            // 将<canvas>转换为图片并保存到相册
				            uni.canvasToTempFilePath({
				              canvasId: 'myCanvas',
				              success: (res) => {
				                uni.saveImageToPhotosAlbum({
				                  filePath: res.tempFilePath,
				                  success: () => {
				                    uni.showToast({ title: '图片保存成功', icon: 'success' });
				                  },
				                  fail: () => {
				                    uni.showToast({ title: '图片保存失败', icon: 'none' });
				                  },
				                });
				              },
				              fail: () => {
				                uni.showToast({ title: '转换图片失败', icon: 'none' });
				              },
				            }, this);
				          });
				        });
			},
			gowebView() {
				uni.navigateTo({
					url: '/pages/edit/web_view',
					success: (res) => {
						
					}
				})
			},
			bindPickerChange: function(e) {
			    console.log('picker发送选择改变，携带值为', e.detail.value);
			    this.brandIndex = e.detail.value;
				const brandName = this.brandsNameList[this.brandIndex] || '';
				this.inputMake = brandName;
				this.brandData = brands[brandName] || {};
			},
			
			onKeyInput: function(event) {
			    this.inputModel = event.target.value
			},
			
		}
	}
</script>

<style>
	.w335{
		width: 335px;
	}
	.ml30 {
		margin-left: 30px;
	}
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
		color: #404040;
		padding-bottom: 5px;
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
		font-size: 10px;
		color: #8F8F8F;
	}
	.photo_location {
		font-size: 6px;
		color: #8F8F8F;
	}
	
	.operation_area {
		margin-top: 20px;
		display: flex;
		/* justify-content: space-between; */
	}
	
	.brand_choose {
		display: flex;
		flex-direction: row;
		align-items: center;
		width: 50%;
	}
	.brand_input {
		display: flex;
		color: white;
	}
	.brand_input_name {
		display: flex;
		color: white;
		white-space: nowrap;
	}
	
	.save_button {
		margin-top: 30px;
	}
	
	.info {
		color: white;
	}

</style>