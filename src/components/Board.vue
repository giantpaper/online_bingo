<template>
	<h1>Week of <span>{{ bingodate.week() }}</span></h1>
	<div v-if="list !== false" class="board_container relative my-16">
		<ul class="board">
			<li v-for="(item, i) in list" :ref="li => (liList[i] = li)">
				<div class="free-space checked" v-if="item.label===`Free Space`">
					FREE SPACE
				</div>
				<label v-else :class="{ checked: item.checked, }">
					<span class="bg_coords">{{ `${item.col}-${item.row}` }}</span>
					<input type="checkbox" v-model="item.checked" @click="ifWin(list, i, 'clicked')" v-if="item.disabled===false" />
					<span v-html="displayLabel(item.label, i)"></span>
				</label>
			</li>
		</ul>
	</div>
	<div v-else class="error">
		<h3>Ruh Roh</h3>
		<p>Major error! Check back soon!</p>
	</div>
	<p v-if="list !== false" class="countdown"><strong>{{ COUNTDOWN }}</strong><br /><span>Till new bingo cards</span></p>
	<WinCond />
	<Info />

	<Footer />
</template>
<style lang="scss" scoped>
	.labels {
		display: flex;
		span {
			display: block;
			height: 100%;
		}
	}
	.letters {
		position: absolute;
		align-items: center;
		justify-content: space-around;
		bottom: 100%;
		left: 0;
		right: 0;
		&:nth-child(1) {
			bottom: 100%;
		}
		&:nth-child(2) {
			top: 100%;
		}
	}
	.numbers {
		position: absolute;
		justify-content: center;
		align-items: stretch;
		top: 0;
		bottom: 0;
		flex-direction: column;
		&:nth-child(3) {
			left: 100%;
		}
		&:nth-child(4) {
			right: 100%;
		}
	}
	.error {
		text-align: center;
	}
	.board {
		border-top: 3px var(--color-border) solid;
		border-left: 3px var(--color-border) solid;
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		margin: 0 auto;
		width: 105ch;
		overflow-x: auto;
		max-width: 100%;
		aspect-ratio: 1/1;
		list-style: none;
		margin-top: 0;
		margin-bottom: 0;
		padding: 0;
		min-height: 100vh;
		@media (min-width: 768px) {
			/* Remove scrollbar on larger displays */
			min-height: unset;
			overflow: unset;
		}
		li {
			border-right: 3px var(--color-border) solid;
			border-bottom: 3px var(--color-border) solid;
			transition: background 0.2s;
			min-width: calc(105ch / 5);
			aspect-ratio: 1/1;
			&.bingo {
				.checked {
					color: white;
				}
				label {
					background: var(--background-win) !important;
					cursor: revert;
					span em {
						color: var(--vt-c-white);
					}
				}
			}
		}
	}
	.free-space,
	label {
		aspect-ratio: 1/1;
		display: flex;
		font-size: calc(20rem / 16);
		align-items: center;
		justify-content: center;
		text-align: center;
		padding: 1rem;
		position: relative;
		font-weight: 800;
		&::before,
		&::after {
			width: 0;
			background: var(--ts-color-magenta);
			content: '';
			display: block;
			position: absolute;
			z-index: 5;
			transition: width 0.2s, 0.15s height 0.2s;
			will-change: width;
			opacity: 0;
			mix-blend-mode: multiply;
		}
		&::before {
			top: 41%;
			left: 1%;
			mask: url('@/assets/brush_stroke.svg') no-repeat center / 100% 100%;
			transform: scaleX(-100%);
		}
		&::after {
			mask: url('@/assets/brush_stroke_02.svg') no-repeat center / 100% 100%;
			bottom: 41%;
			right: 1%;
			transition-delay: 0.2s;
			transform: rotate(180deg);
		}
		&.checked {
			&::before,
			&::after {
				width: 98%;
				height: 10%;
				opacity: 0.5;
			}
		}
		input[type="checkbox"] {
			appearance: none;
		}
	}
	label {
		background: var(--background-box);
	}
	.free-space {
		background: rgba(0,0,0,0.0625) !important;
	}
	.countdown {
		text-transform: uppercase;
		text-align: center;
		margin-bottom: 2rem;
		strong {
			font-size: 1rem;
			font-weight: 900;
		}
		span {
		}
	}
	.bg_coords {
		font-size: 4rem;
		display: block;
		position: absolute;
		top: 1rem;
		left: 0.25rem;
		opacity: 0.0625;
	}
	.bingo {
		background: var(--background-win);
		.free-space {
			span em {
				color: var(--vt-c-white);
			}
		}
		div,
		label {
			&::before,
			&::after {
				display: none;
			}
		}
	}
</style>
<script setup>
import { ref } from "vue"
import DefaultValues from "../assets/_default.js"
// import { updateProgress, getProgress, setProgress } from "../assets/_progress.js"
import Bingo from "../assets/_bingo.js"
import BingoDate from "../assets/_bingodate.js"
import { range } from 'mathjs'

import Countdown from '../assets/_countdown.js'

import Info from './Info.vue'
import WinCond from './WinCond.vue'
import Footer from './Footer.vue'

import emojiRegex from 'emoji-regex'

const list = ref([]) // progress
const liList = ref([])
const disabled = ref(false)

const bingo = new Bingo(list.value)
const bingodate = new BingoDate()

const letters_i = ref(0)
const numbers_i = ref(0)

list.value = bingo.list()

ifWin(list.value)

function displayLabel (item, i) {
	// https://www.freecodecamp.org/news/how-to-use-regex-to-match-emoji-including-discord-emotes/
	let label = item
	for (const match of label.matchAll(emojiRegex())) {
		let regex = new RegExp(`(${match[0]})`, 'g')
		label = label.replace(regex, "<i>$1</i>")
	}

	label = label.replace(/\^([^\^]+)\^/, '<em>$1</em>')

	label = label.replace(/"([^"]+)"/g, `“$1”`)
	label = label.replace(/'([^']+)'/g, `‘$1’`)
	label = label.replace(/'s/g, `‘s`)

	// Fixes obscenely-long text not breaking
	label = label.replace(/([^<])\/([^>])/g, "$1/<wbr>$2")

	// Makes arrows look sexier
	label = label.replace(/->/g, "→")

	// For debugging/dev purposes
	if (import.meta.env.MODE === 'development') {
		label = i + ': ' + label
	}

	return label
}

const date = new Date()

function getExampleWin(keyword, i) {
	return bingo.winCond(keyword).includes(i)
}
function ifWin(list, i, event) {
	// Apparently this was running before the list = ref even had a chance to update
	// For some reason, setTimeout() works and onMounted() doesn't. I dunno why
	setTimeout(() => {
		if (list.value === false) {
			return false
		}

		let win = bingo.ifWin(list, i, event)
		list.value = bingo.list()

		document.querySelectorAll('.board > li').forEach((el, i) => {
			if (el.classList.contains('bingo')) {
				list.value[i].disabled = true
			}
		})
	})
}

const COUNTDOWN = ref(null)

let timer = new Countdown()

setInterval(() => {
	COUNTDOWN.value = timer.get()
})


</script>
