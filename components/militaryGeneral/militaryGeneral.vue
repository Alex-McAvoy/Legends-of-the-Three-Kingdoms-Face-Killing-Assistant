<template>
	<view class="military-container">
		<!-- 武将图容器 -->
		<view class="military-image-container" @click="preView">
			<!-- 边框盒子 -->
			<view class="border-box">
				<image class="border-image" :src="militaryBorder" mode="widthFix"></image>
			</view>
			<!-- 组件盒子 -->
			<view class="component-box">
				<!-- 势力盒子 -->
				<view class="force-box">
					<image class="force-image" :src="militaryForce" mode="widthFix"></image>
				</view>
				<!-- 勾玉盒子 -->
				<view class="magatama-box">
					<view v-if="item.magatama.actual>10" class="magatama">
						<image class="magatama-image" :src="militaryMagatama" mode="widthFix"></image>
						<view class="magatama-text">×{{item.magatama.actual}}</view>
					</view>
					<view v-else class="magatama">
						<image v-for="index in item.magatama.actual" class="magatama-image" :src="militaryMagatama"
							mode="widthFix"></image>
						<image v-for="index in item.magatama.blank" class="magatama-image" :src="militaryMagatamaBlank"
							mode="widthFix"></image>
					</view>
				</view>
			</view>
			<!-- 武将图盒子 -->
			<view class="military-image-box">
				<image class="military-image" :src="item.imgSrc" mode="widthFix"></image>
			</view>
		</view>
		
		<!-- 内容容器 -->
		<view class="content-container">
			<!-- 武将名 -->
			<view class="name">
				<text>{{item.name}}</text>
			</view>
			
			<!-- 武将数据标签栏 -->
			<u-tabs :list="militaryTabsList" class="tabs" lineColor="#60544a" lineWidth="50"
			:activeStyle="tabsActive" @click="getTabsList"></u-tabs>
			
			<!-- 介绍容器 -->
			<view class="intro-container">
				<!-- 分隔 -->
				<image class="intro-divide" :src="divideImgSrc" mode="widthFix"></image>

				<!-- 技能容器 -->
				<view v-if="tabsIndex==0" class="skills-container" v-for="skill in item.skills">
					<!-- 技能名 -->
					<view class="skill-name"
						:style="{backgroundImage: 'url('+skillBackground+')', color: skillFontColor }">
						{{skill.name}}
					</view>
					<!-- 技能描述 -->
					<view class="skill-description" v-html="dealDescriptions(skill.descriptions)"></view>
				</view>
				<!-- 语音容器 -->
				<view v-if="tabsIndex==1" class="dialogues-container" v-for="dialogue in item.dialogues">
					<!-- 技能名 -->
					<view class="dialogue-name">
						{{dialogue.name}}
					</view>
					<!-- 语音容器 -->
					<view class="dialogue-description-container">
						<view v-for="(description,index) in dialogue.descriptions" class="dialogue-description">
							<!-- 语音 -->
							<text>{{description}}</text>
							<!-- 音频 -->
							<mine-audio class="dialogue-audio" :src="dialogue.srcs[index]"
								:audio-id="'audio_' + item.index + '_' + index"></mine-audio>
						</view>
					</view>
				</view>
				<!-- 武将列传容器 -->
				<view v-if="tabsIndex==2" class="history-container">
					<text class="history-text">{{item.history}}</text>
				</view>
			</view>
		</view>
		
		<!-- 技能描述中的概念模态框-->
		<view>
			<u-modal :show="modal.show" :title="modal.title" :content="modal.content"
			:showConfirmButton="false" :closeOnClickOverlay="true" @close="closeModal">
			</u-modal>
		</view>
	</view>
</template>

<script>
	import mineAudio from "@/components/mineAudio/mineAudio.vue"
	
	export default {
		name: "militaryGeneral",
		components: {
			mineAudio
		},
		props: {
			item: {
				type: Object,
				default: {
					// id
					id: '',
					// 武将名
					name: '',
					// 武将图片
					imgSrc: '',
					// 技能
					skills: [{
						// 技能名
						name: '',
						// 技能描述
						description: ''
					}],
					// 语音
					dialogues: [{
						// 技能名
						name: "",
						// 语音
						descriptions: [],
						// 音频地址
						srcs: []
					}]
				}
			}
		},
		data() {
			return {
				// 武将边框图
				militaryBorder: "/static/images/common/militaryGeneral/border.png",
				// 武将势力图
				militaryForce: "",
				// 武将数据标签栏
				militaryTabsList: [{
					name: "技能",
				},{
					name: "语音",
				},{
					name: "列传",
				}],
				// 标签栏选中样式
				tabsActive: {
					padding: '0 10rpx',
					color: '#1e1b1b',
					fontWeight: 'bold',
					fontWeight: '600',
					transform: 'scale(1.3)',
				},
				// 标签栏未选中样式
				tabsInactive: {
					padding: '10rpx',
					fontSize: '30rpx',
					fontWeight: '600',
				},
				// 标签索引
				tabsIndex: 0,
				// 武将技能背景图
				skillBackground: "",
				// 武将技能字体颜色
				skillFontColor: "",
				// 武将勾玉图
				militaryMagatama: "",
				// 武将空白勾玉图
				militaryMagatamaBlank: "/static/images/common/militaryGeneral/magatama_blank.png",
				// 分隔图
				divideImgSrc: "",
				// 技能描述中关键字的模态框
				modal: {
					show: false,
					title: "标题",
					content: "内容"
				}
			};
		},
		mounted() {
			// 动态加载武将技能背景图
			this.skillBackground = require('@/static/images/common/militaryGeneral/skill_' + this.$props.item.force +
				'.png')
			// 动态加载武将数据
			this.getTabsList({index: 0})
			// 动态加载武将技能字体颜色
			if (this.$props.item.force == "shen") {
				this.skillFontColor = "white"
			} else {
				this.skillFontColor = "black"
			}
			// 动态加载武将势力图
			this.militaryForce = require('@/static/images/common/militaryGeneral/force_' + this.$props.item.force +
				'.png')
			// 动态加载武将勾玉图
			this.militaryMagatama = require('@/static/images/common/militaryGeneral/magatama_' + this.$props.item.force +
				'.png')

			// 将vue实例方法绑定到window对象上
			window.showZhuGongModal = this.showZhuGongModal
			window.showSuoDingModal = this.showSuoDingModal
			window.showXianDingModal = this.showXianDingModal
			window.showJueXingModal = this.showJueXingModal
			window.showZhuanHuanModal = this.showZhuanHuanModal
			window.showZongZuModal = this.showZongZuModal
			window.showYinNiModal = this.showYinNiModal
		},
		computed: {
			imgSrc() {
				return this.$props.item.imgSrc
			}
		},
		methods: {
			// 点击查看图片
			preView() {
				uni.previewImage({
					urls: [this.imgSrc],
					longPressActions: true
				})
			},
			// 获取武将数据
			getTabsList(item) {
				this.tabsIndex = item.index
				// 根据tabs的index，显示分隔图
				if (item.index == 0) { // 技能
					this.divideImgSrc = "/static/images/common/militaryGeneral/divide_skill.png"
				} else if (item.index == 1) { // 语音
					this.divideImgSrc = "/static/images/common/militaryGeneral/divide_dialogue.png"
				} else if (item.index == 2) { // 武将列传
					this.divideImgSrc = "/static/images/common/militaryGeneral/divide_history.png"
				}
			},
			// 处理技能描述中的概念
			dealDescriptions(descriptions) {
				if (descriptions.includes("主公技")) {
					descriptions = descriptions.replace(
						"主公技",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showZhuGongModal()' >主公技</font>"
						)
				}
				if (descriptions.includes("锁定技")) {
					descriptions = descriptions.replace(
						"锁定技",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showSuoDingModal()' >锁定技</font>"
						)
				}
				if (descriptions.includes("限定技")) {
					descriptions = descriptions.replace(
						"限定技",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showXianDingModal()' >限定技</font>"
						)
				}
				if (descriptions.includes("觉醒技")) {
					descriptions = descriptions.replace(
						"觉醒技",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showJueXingModal()' >觉醒技</font>"
						)
				}
				if (descriptions.includes("转换技")) {
					descriptions = descriptions.replace(
						"转换技",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showZhuanHuanModal()' >转换技</font>"
						)
				}
				if (descriptions.includes("宗族技")) {
					descriptions = descriptions.replace(
						"宗族技",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showZongZuModal()' >宗族技</font>")
				}
				if (descriptions.includes("隐匿")) {
					descriptions = descriptions.replace(
						"隐匿",
						"<font style='text-decoration: underline;font-weight:bold;' onclick='showYinNiModal()' >隐匿</font>")
				}
				return descriptions
			},
			// 主公技模态框
			showZhuGongModal() {
				this.modal.show = true
				this.modal.title = "主公技"
				this.modal.content = "技能的标签之一。带有此标签的技能只有在该角色的身份为主公时才能拥有。若该角色在游戏开始时没有此技能，则不受此限制。"
			},
			// 锁定技模态框
			showSuoDingModal() {
				this.modal.show = true
				this.modal.title = "锁定技"
				this.modal.content = "技能的标签之一。请注意：带有“锁定技”标签的技能不一定强制发动，不带有“锁定技”标签的技能未必不强制发动。锁定技在优先级计算上，完全没有任何特权。"
			},
			// 限定技模态框
			showXianDingModal() {
				this.modal.show = true
				this.modal.title = "限定技"
				this.modal.content = "技能的标签之一。带有此标签的技能于一局游戏内仅能发动一次。"
			},
			// 觉醒技模态框
			showJueXingModal() {
				this.modal.show = true
				this.modal.title = "觉醒技"
				this.modal.content = "技能的标签之一。当符合觉醒条件时，玩家必须“觉醒”。带有此标签的技能视为附带一个“锁定技”标签和一个“限定技”标签。"
			},
			// 转换技模态框
			showZhuanHuanModal() {
				this.modal.show = true
				this.modal.title = "转换技"
				this.modal.content = "技能的标签之一。该技能拥有“阳”、“阴”两个状态。游戏开始时，该技能处于“阳”状态。转换技发动后，技能会自动切换到另一种形态，不把阴技能发动就不能使用阳技能，反之亦然。"
			},
			// 宗族技模态框
			showZongZuModal() {
				this.modal.show = true
				this.modal.title = "宗族技"
				this.modal.content = "技能的标签之一。属于同一宗族的族武将拥有相同的宗族技。"
			},
			// 隐匿模态框
			showYinNiModal() {
				this.modal.show = true
				this.modal.title = "隐匿"
				this.modal.content = "在游戏开始时，改为对外亮出“隐匿牌”作为武将牌。“隐匿”状态下的武将没有体力值（血量显示为一个面具），性别为男。当在“隐匿”状态下扣减体力时，防止之并亮出真正的武将牌，称为“登场”。回合开始时，若你处于“隐匿”状态，你“登场”。"
			},
			// 关闭模态框
			closeModal() {
				this.modal.show = false
			}
		}
	}
</script>

<style lang="scss" scoped>
	/* 容器 */
	.military-container {
		display: flex;
		flex-direction: column;
		margin: 30rpx;
		padding: 20px;

		background-color: blanchedalmond;
		background-image: url("@/static/images/common/archive/list_bg.png");
		background-position: center;
		background-repeat: no-repeat;
		background-attachment: fixed;
		background-size: cover;

		box-shadow: 3px -3px 5px rgba(0, 0, 0, 0.5);
		border-radius: 10px;
		border-style: groove;
		border-color: #413430;

		/* 武将图容器 */
		.military-image-container {
			display: grid;
			grid-template-columns: 1fr 4fr 1fr;
			grid-template-rows: 8fr;

			/* 武将边框盒子 */
			.border-box {
				grid-area: 1/2/2/2;
				z-index: 2;

				/* 武将边框图 */
				.border-image {
					width: 100%;
					height: auto;
					align-self: center;
					box-shadow: 0 -3px 5px rgba(0, 0, 0, 0.5);
					border-radius: 5%;
				}
			}

			/* 组件盒子 */
			.component-box {
				grid-area: 1/2/2/2;
				z-index: 3;
				display: grid;
				grid-template-columns: 2fr 6fr 2fr;
				grid-template-rows: 2fr 8fr;

				/* 武将势力盒子 */
				.force-box {
					grid-area: 1/1/1/1;
					z-index: 3;

					/* 武将势力图 */
					.force-image {
						width: 100%;
						height: auto;
						align-self: center;
					}
				}

				/* 武将勾玉盒子 */
				.magatama-box {
					grid-area: 1/2/1/2;
					z-index: 3;

					.magatama {
						display: flex;
						flex-direction: row;

						/* 勾玉图 */
						.magatama-image {
							width: 10%;
							height: auto;
							margin-top: 5px;
							align-self: center;
						}

						/* 勾玉文本 */
						.magatama-text {
							color: white;
						}
					}
				}

			}


			/* 武将盒子 */
			.military-image-box {
				grid-area: 1/2/2/2;
				z-index: 0;

				/* 武将图 */
				.military-image {
					width: 100%;
					height: auto;
					align-self: center;
					border-radius: 5%;
				}
			}
		}

		/* 标签栏 */
		.tabs {
			height: 80rpx;
			padding: 5px 10px 0px 10px;
			font-family: huaWenXingKai;
			background-color: #ffffff;
			margin-bottom: 20px;
		}
		::v-deep .u-tabs__wrapper__nav {
			width: 100%;
			display: flex;
			flex-direction: row;
			position: relative;
			justify-content: space-between;
		}
		::v-deep .u-tabs__wrapper__nav__item {
			flex: 1 1 0% !important;
		}
		
		/* 内容容器 */
		.content-container {
			display: flex;
			flex-direction: column;

			margin-top: 10px;
			box-shadow: 0 -3px 5px rgba(0, 0, 0, 0.5);

			background-image: url("@/static/images/common/militaryGeneral/top.png"), url("@/static/images/common/militaryGeneral/bottom.png");
			background-position: top left, bottom right;
			background-repeat: no-repeat;
			background-color: #ffffff;
			padding: 20px;

			/* 武将名 */
			.name {
				font-family: heiTi;
				color: black;
				font-size: 30px;
				font-weight: bold;
				padding: 20px 10px;
				text-align: center;
			}

			/* 介绍容器 */
			.intro-container {
				display: flex;
				flex-direction: column;
				gap: 20px;

				/* 分隔 */
				.intro-divide {
					justify-self: center;
					align-self: center;
					width: 100%;
					height: auto;
				}

				/* 技能容器 */
				.skills-container {
					display: flex;
					gap: 20px;
					padding-top: 10px;
					font-size: 30rpx;

					/* 技能名 */
					.skill-name {
						flex-shrink: 0;
						align-self: flex-start;
						background-repeat: round;
						width: 70px;
						padding: 5px 0px;

						height: 33px;
						text-align: center;
						padding-right: 4px;
						font-weight: bold;
						font-size: small;

						box-sizing: border-box;
						direction: ltr;
					}

					/* 技能描述 */
					.skill-description {
						align-self: center;
					}
				}

				/* 语音容器 */
				.dialogues-container {
					display: flex;
					gap: 20px;
					padding-top: 10px;
					font-size: 30rpx;

					/* 技能名 */
					.dialogue-name {
						flex-shrink: 0;
						align-self: flex-start;
						background-image: url("@/static/images/common/militaryGeneral/dialogue.png");
						background-repeat: round;
						width: 70px;
						padding: 5px 0px;

						color: white;
						height: 33px;
						text-align: center;
						padding-right: 4px;
						font-size: small;

						box-sizing: border-box;
						direction: ltr;
					}

					/* 语音描述容器 */
					.dialogue-description-container {
						display: flex;
						flex-direction: column;
						justify-content: center;
						align-items: flex-start;
						gap: 20px;

						/* 语音描述 */
						.dialogue-description {
							display: flex;
							flex-direction: row;
							justify-content: center;

							/* 语音音频 */
							.dialogue-audio {
								margin: 5rpx 0px 0px 0px;
							}
						}
					}
				}
			
				/* 武将列传容器 */
				.history-container {
					display: flex;
					gap: 20px;
					padding-top: 10px;
					font-size: 30rpx;
					
					/* 武将列传文本 */
					.history-text {
						text-indent:2em;
					}
				}
			}
		}
	}
</style>