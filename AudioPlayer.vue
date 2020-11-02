<template>
	<div>
		<span>Play the Audio</span>
		<div id="audio-element">
			<audio 
				id="player"
				ref="audio" 
				:src="section.audio"
				@ended="restart"
				@timeupdate="getAudioProgress"
			/>
			<div>
				<svg 
					height="100" 
					width="100"
				>
					<circle 
						v-if="audioProgress > 0"
						stroke="#EC4C24"
						stroke-width="2"
						fill="transparent"
						cx="50"
						cy="50"
						:r="normalizedRadius"
						:style="{ strokeDashoffset }"
						:stroke-dasharray="circumference + ' ' + circumference"
					/>
					<circle 
						stroke="rgba(255, 255, 255, 0.15)"
						stroke-width="2"
						fill="transparent"
						cx="50"
						cy="50"
						:r="normalizedRadius"
						@click="seekTo" 
					/>
				</svg>
				<div class="player-elements">
					<button 
						class="play-button"
						@click="play"
					>
						<div 
							:class="isPlaying ? 'pause-icon' : 'play-icon'"
						/>
					</button>	
					<img 
						:src="section.portrait" 
					>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: "MediaPlayer",
	props: {
		section: {
			type: Object,
			default: () => ({}),
		},
	},

	data: function () {
		const normalizedRadius = 48 - 4 * 2
		const circumference = normalizedRadius * 2 * Math.PI
		return {
			circumference,
			normalizedRadius,
			audioProgress: 0,
			realTime: 0,
			audioLength: 0,
			isPlaying: false,
		}
	},

	computed: {
		strokeDashoffset() {
			var amountMoved = this.audioProgress * this.circumference
			return this.circumference - amountMoved
		},
	},

	watch: {
		audioProgress: function () {
			this.$emit("current-time", this.audioProgress)
		},
	},

	methods: {
		play() {
			this.isPlaying = !this.isPlaying

			if (this.isPlaying) {
				document.getElementById("player").play()
			} else {
				document.getElementById("player").pause()
			}
		},

		restart() {
			this.isPlaying = !this.isPlaying
			this.audioProgress = 0
		},

		getAudioProgress() {
			this.audioLength = document.getElementById("player").duration
			this.realTime = document.getElementById("player").currentTime
			this.audioProgress = this.realTime / this.audioLength
		},

		seekTo(event) {
			let theta = Math.atan2(event.offsetX - 50, event.offsetY - 50)
			let unormalizedCircumference = 48 * 2 * Math.PI
			let reverse = theta * 48 + unormalizedCircumference / 2

			let arcProgress =
				(unormalizedCircumference - reverse) / unormalizedCircumference
			let arcTime = arcProgress * this.audioLength

			document.getElementById("player").currentTime = arcTime
			this.audioProgress = arcProgress
		},
	},
}
</script>

<style lang="scss" scoped>
.play-button {
	width: 68px;
	height: 68px;
	border-radius: 50%;
	background-color: rgba(255, 255, 255, 0.3);
	border-style: solid;
	position: absolute;
	margin: 16px 1px;

	&:hover {
		border-color: rgba(255, 255, 255, 0.6);
	}

	&:focus {
		outline: 0;
	}
}

.play-icon {
	border-top: 12px solid transparent;
	border-left: 20px solid #ffffff;
	border-bottom: 12px solid transparent;
	position: relative;
	margin: auto;
	left: 6%;
	display: inline-block;
}

.pause-icon {
	width: 22px;
	height: 22px;
	border-style: solid;
	border-width: 7px;
	border-color: #ffffff;
	border-style: double;
	border-width: 0px 0px 0px 15px;
	position: relative;
	margin: auto;
	left: 20%;
	display: inline-block;
}

.player-elements {
	left: 75px;
	position: absolute;
	display: inline;
}

img {
	width: 60px;
	height: 60px;
	border-radius: 50%;
	display: inline-block;
	position: absolute;
	margin: 19px auto auto 110px;
	border-style: solid;
	border-width: 1px;
	border-color: white;
}

circle {
	transform: rotate(-90deg);
	transform-origin: 50% 50%;
	transition: stroke-dashoffset 0.23s linear;
}
</style>