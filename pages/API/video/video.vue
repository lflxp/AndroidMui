<template>
	<view>
		<page-head :title="title"></page-head>
		<view class="page-body">
			<view class="page-section">
				<view class="uni-list">
					<view class="uni-list-cell">
						<view class="uni-list-cell-left">
							<view class="uni-label">视频来源</view>
						</view>
						<view class="uni-list-cell-right">
							<picker :range="sourceType" @change="sourceTypeChange" :value="sourceTypeIndex">
								<view class="uni-input">{{sourceType[sourceTypeIndex]}}</view>
							</picker>
						</view>
					</view>
				</view>
				<view class="page-body-info">
					<block v-if="src === ''">
						<view class="image-plus image-plus-nb" @tap="chooseVideo">
							<view class="image-plus-horizontal"></view>
							<view class="image-plus-vertical"></view>
						</view>
						<view class="image-plus-text">添加视频</view>
					</block>
					<block v-if="src != ''">
						<video :src="src" class="video"></video>
					</block>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	import pageHead from '../../../components/page-head.vue'

	var sourceType = [
		['camera'],
		['album'],
		['camera', 'album']
	]
	export default {
		data() {
			return {
				title: 'chooseVideo',
				sourceTypeIndex: 2,
				sourceType: ['拍摄', '相册', '拍摄或相册'],
				src: ''
			}
		},
		onUnload(){
			this.src = '',
			this.sourceTypeIndex = 2,
			this.sourceType=['拍摄', '相册', '拍摄或相册'];
		},
		methods: {
			sourceTypeChange: function (e) {
				this.sourceTypeIndex = e.target.value
			},
			chooseVideo: function () {
				uni.chooseVideo({
					sourceType: sourceType[this.sourceTypeIndex],
					success: (res) => {
						this.src = res.tempFilePath
					}
				})
			}
		},
		components: {
			pageHead
		}
	}
</script>

<style>
	@import "../../../common/uni.css";
	.page-body-info {
		display: flex;
		margin-top: 40px;
		padding: 0;
		height: 360px;
		border-top: 1px solid #D9D9D9;
		border-bottom: 1px solid #D9D9D9;
		align-items: center;
		justify-content: center;
	}
</style>
