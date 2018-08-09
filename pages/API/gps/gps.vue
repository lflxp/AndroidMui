
<template>
	<view>
		<page-head :title="title"></page-head>
		<view class="page-body">
			<view class="page-section">
				<view class="page-body-info">
					<text class="page-body-text-small">当前位置经纬度</text>
					<block v-if="hasLocation === false">
						<text class="page-body-text">未获取</text>
					</block>
					<block v-if="hasLocation === true">
						<view class="page-body-text-location">
							<text>E: {{location.longitude[0]}}°{{location.longitude[1]}}′</text>
							<text>N: {{location.latitude[0]}}°{{location.latitude[1]}}′</text>
						</view>
					</block>
				</view>
				<view class="btn-area">
					<button type="primary" @tap="getLocation">获取位置</button>
					<button @tap="clear">清空</button>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
    /*
    {"key":"date",
    "tablename":"gps",
    "value":"asd"
    }
    */
	import pageHead from '../../../components/page-head.vue';

	var util = require('../../../common/util.js');
	var formatLocation = util.formatLocation;

	export default {
		data() {
			return {
				title: 'getLocation',
				hasLocation: false,
				location: {}
			}
		},
		methods: {
			getLocation: function () {
				uni.getLocation({
					success: (res) => {
						this.hasLocation = true,
                            this.location = formatLocation(res.longitude, res.latitude)
                            var dataed = {"key":new Date().getTime()+ "",
                            "value":"E "+this.location.longitude[0]+','+this.location.longitude[1]+' N '+this.location.latitude[0]+','+this.location.latitude[1],
                            "tablename":"gps"}
                            uni.request({
                                url: 'http://192.168.1.2:8080/v1/db/kv',
                                data: dataed,
                                method: 'POST',
                                success: (data) => {
                                    if (data.statusCode == 200) {
                                        console.log(data.data)
                                    }
                                },
                                fail: (data,code) => {
                                    console.log('fail'+JSON.stringify(data));
                                }
                            })
					}
				})
			},
			clear: function () {
				this.hasLocation = false
			}
		},
		components: {
			pageHead

		}
	}
</script>

<style>
	.page-body-info {
		height: 250px;
	}

	.page-body-text-small {
		font-size: 24px;
		color: #000;
		margin-bottom: 100px;
	}

	.page-body-text-location {
		display: flex;
		font-size: 50px;
	}

	.page-body-text-location text {
		margin: 10px;
	}
</style>
