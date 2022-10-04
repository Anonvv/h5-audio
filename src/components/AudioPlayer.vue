<template>
	<div class="audio-player">
		<!-- 音频列表 -->
		<ul class="audio-list">
			<li v-for="(item, index) in audioList" :class="index === singeIdx ? 'active' : ''"
				@dblclick="toggleSing(index)">
				<span>{{ item.title }}</span>
				<span v-if="index === singeIdx">playing</span>
			</li>
		</ul>
		<!-- 进度条 -->
		<div class="progress" @mouseup="progressChange">
			<div class="active" :style="{ width: `${playingData.currentTime/playingData.duration*100}%`}"></div>
		</div>
		<!-- 播放信息 -->
		<div class="play-panel">
			<!-- 自定义控件 -->
			<div class="tools">
				<span @click="handleCut(0)">&lt;&lt;</span>
				<span @click="handlePlay">
					{{ isPlaying ? 'Pause' : 'Play' }}
				</span>
				<span @click="handleCut(1)">>></span>
			</div>
			<div class="info">
				<!-- 时间 -->
				<span class="time">{{ secondsToMinites(playingData.currentTime) }}/{{
				secondsToMinites(playingData.duration) }}</span>
				<!-- 音量 -->
				<span class="volume">
					<span>)))</span>
					<div class="volume-justify">
						<div class="volume-bar" @mouseup="volumeChange">
							<div class="active" :style="{height: `${currentVolume/1*100}%`}"></div>
						</div>
					</div>
				</span>
			</div>
		</div>
		<!-- 原生控件，太丑了不需要显示(controls=false)-->
		<audio ref="singeBox" :src="playingData.url"></audio>
	</div>
</template>
<script lang="ts" setup>
import { ref, Ref, reactive, onMounted, computed } from 'vue'

const audioList = reactive([ // 音频列表数据
	{
		url: '/src/assets/audio/Joshua Hyslop - Let It Go.mp3',
		title: 'Joshua Hyslop - Let It Go'
	},
	{
		url: '/src/assets/audio/Joshua Hyslop - The Flood.mp3',
		title: 'Joshua Hyslop - The Flood'
	},
])
const singeIdx: Ref<number> = ref(0) // 播放音频下索引
const singeBox: Ref<any> = ref() // audio标签节点
const isPlaying: Ref<boolean> = ref(false) // 是否正在播放
const playingData = reactive({ // 当前播放数据
	duration: 0, // 音频总长时间
	currentTime: 0, // 当前播放时间
	url: computed(() => audioList[singeIdx.value].url),
	title: computed(() => audioList[singeIdx.value].title),
})
const currentVolume: Ref<number> = ref(1) // 音量 (0-1)

onMounted(() => {
	init()
})

// 初始化
const init = () => {
	currentVolume.value = singeBox.value.volume // 获取默认音量
	// 播放器事件监听
	// 当时长有变化时触发，由"NaN"变为实际时长也算
	singeBox.value.ondurationchange = function () {
		playingData.duration = singeBox.value.duration
	}
	// 当前数据可用是触发
	singeBox.value.oncanplay = function () {
		if (isPlaying.value) {
			singeBox.value.play()
		}
	}
	// 播放位置发送改变时触发。
	singeBox.value.ontimeupdate = function () {
		playingData.currentTime = singeBox.value.currentTime
	}
	// 音频播放完毕
	singeBox.value.onended = function () {
		// 切换下一首
		handleCut(1)
	}
	// 音频播放出错
	singeBox.value.onerror = function () {
		console.log('加载出错！')
	}
}
// 播放/暂停
const handlePlay = () => {
	isPlaying.value = !isPlaying.value
	if (isPlaying.value) {
		singeBox.value.play()
	} else {
		singeBox.value.pause()
	}
}
/**
 * 切歌
 * @param type 0：上一首，1：下一首
 */
const handleCut = (type: number) => {
	let index = 0
	if (type) { // 下一首
		if (singeIdx.value < audioList.length - 1) {
			index = singeIdx.value + 1
		}
	} else { // 上一首
		if (singeIdx.value <= 0) {
			index = audioList.length - 1
		} else {
			index = singeIdx.value - 1
		}
	}
	toggleSing(index)
}
/**
 * 切换音频
 * @param index 点击项索引
 */
const toggleSing = (index: number): void => {
	if (index === singeIdx.value) return
	singeIdx.value = index
	isPlaying.value = true
}
// 监听进度条鼠标事件
const progressChange = (e: any): void => {
	const percent = e.offsetX / e.target.clientWidth // 计算鼠标位置占总长度的百分比
	singeBox.value.currentTime = playingData.duration * percent // 按百分比设置已播放时间
}
// 监听音量条鼠标事件
const volumeChange = (e: any): void => {
	const percent = (e.target.clientHeight - e.offsetY) / e.target.clientHeight // 计算鼠标位置占总高度的百分比
	// 按百分比设置音量
	currentVolume.value = 1 * percent
	singeBox.value.volume = currentVolume.value
}

/**
 * 格式化秒数
 * @param totalSeconds 秒
 * @return {String} 'xx:xx'
 */
const secondsToMinites = (totalSeconds: number = 0): string => {
	const minutes = parseInt(totalSeconds / 60)
	const seconds = parseInt(totalSeconds - minutes * 60)
	return `${minutes < 10 ? '0' + minutes : minutes}:${seconds < 10 ? '0' + seconds : seconds}`
}

</script>
<style lang="sass" scoped>
.audio-player 
	padding: 6rem
	color: #fff
	.audio-list
		list-style: none
		li
			display: flex
			justify-content: space-between
			line-height: 4.2rem
			padding: 0 1rem
			cursor: pointer
			user-select: none
			&:hover
				background-color: rgba(black, .2)
			&.active
				color: #2bf
				background-color: rgba(black, .2)
	.progress
		height: 2px
		margin-top: 2rem
		background-color: #ddd
		position: relative
		cursor: pointer
		.active
			position: absolute
			height: 100%
			background-color: #2bf
	.play-panel
		padding: 1rem 0
		display: flex
		justify-content: space-between
		align-items: center
		.tools
			span
				margin: 2rem
				cursor: pointer
				&:hover
					color: #2bf
		.info
			font-size: 1.4rem
			position: relative
			.time
				padding-right: 4rem
			.volume
				cursor: pointer
				position: absolute
				bottom: 0
				right: 1rem
				&:hover
					.volume-justify
						height: 100px
						padding: 1rem
				.volume-justify
					display: flex
					justify-content: center
					position: absolute
					right: 0rem
					bottom: 2rem
					width: 30px
					height: 0
					padding: 0
					border-radius: 3px
					background-color: rgba(black, .2)
					.volume-bar
						width: 4px
						height: 100%
						background-color: #ddd
						border-radius: 3px
						position: relative
						overflow: hidden
						.active
							position: absolute
							bottom: 0
							width: 100%
							background-color: #2bf
</style>


