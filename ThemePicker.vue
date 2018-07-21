<template>
	<el-color-picker class="theme-picker" popper-class="theme-picker-dropdown" v-model="theme"></el-color-picker>
</template>

<script>
	const version = require('element-ui/package.json').version // element-ui version from node_modules
	const ORIGINAL_THEME = '#409EFF' // default color

	export default {
		data() {
			return {
				chalk: '', // content of theme-chalk css
				theme: this.$store.getters.themeColor || ORIGINAL_THEME
			}
		},
		created() {
			if(this.$store.getters.themeColor) this.initThemeColor(this.$store.getters.themeColor, '#409EFF', 'default')
		},
		//		mounted() {
		//			this.$nextTick(() => {
		//			})
		//		},
		watch: {
			theme(newVal, oldVal) {
				this.initThemeColor(newVal, oldVal)
			},
			
//			$route() {
//				this.initThemeColor(this.theme, ORIGINAL_THEME, 'default')
//			} 
		},

		methods: {
			initThemeColor(newVal, oldVal, type) {
				if(typeof newVal !== 'string') return
				const themeCluster = this.getThemeCluster(newVal.replace('#', ''))
				const originalCluster = this.getThemeCluster(oldVal.replace('#', ''))
				console.log(themeCluster, originalCluster)
				const getHandler = (variable, id) => {
					return() => {
						const originalCluster = this.getThemeCluster(ORIGINAL_THEME.replace('#', ''))
						const newStyle = this.updateStyle(this[variable], originalCluster, themeCluster)

						let styleTag = document.getElementById(id)
						if(!styleTag) {
							styleTag = document.createElement('style')
							styleTag.setAttribute('id', id)
							document.head.appendChild(styleTag)
						}
						styleTag.innerText = newStyle
					}
				}

				const chalkHandler = getHandler('chalk', 'chalk-style')

				if(!this.chalk) {
					const url = `https://unpkg.com/element-ui@${version}/lib/theme-chalk/index.css`
					this.getCSSString(url, chalkHandler, 'chalk') //将element-ui的颜色样式替换并加载到id=chalk的style标签里
				} else {
					chalkHandler()
				}

				const styles = [].slice.call(document.querySelectorAll('style'))
					.filter(style => {
						const text = style.innerText
						return new RegExp(oldVal, 'i').test(text) && !/Chalk Variables/.test(text) //获取所有的style下的匹配上的列表
					})
				styles.forEach(style => {
					const {
						innerText
					} = style
					if(typeof innerText !== 'string') return
					style.innerText = this.updateStyle(innerText, originalCluster, themeCluster) //将匹配上的颜色替换成新的颜色
				})
				this.$store.dispatch('setThemeColor', newVal)
				if(type !== 'default') {
					//					window.location.reload()
					this.$message({
						message: '换肤成功',
						type: 'success'
					})
				}
				//localStorage.themeColor = newVal
			},
			updateStyle(style, oldCluster, newCluster) {
				let newStyle = style
				oldCluster.forEach((color, index) => {
					newStyle = newStyle.replace(new RegExp(color, 'ig'), newCluster[index])
				})
				return newStyle
			},

			getCSSString(url, callback, variable) {
				const xhr = new XMLHttpRequest()
				xhr.onreadystatechange = () => {
					if(xhr.readyState === 4 && xhr.status === 200) {
						this[variable] = xhr.responseText.replace(/@font-face{[^}]+}/, '')
						callback()
					}
				}
				xhr.open('GET', url)
				xhr.send()
			},

			getThemeCluster(theme) {
				const tintColor = (color, tint) => {
					let red = parseInt(color.slice(0, 2), 16)
					let green = parseInt(color.slice(2, 4), 16)
					let blue = parseInt(color.slice(4, 6), 16)

					if(tint === 0) { // when primary color is in its rgb space
						return [red, green, blue].join(',')
					} else {
						red += Math.round(tint * (255 - red))
						green += Math.round(tint * (255 - green))
						blue += Math.round(tint * (255 - blue))

						if(red < 16) red = '0' + red
						if(green < 16) green = '0' + green
						if(blue < 16) blue = '0' + blue

						red = red.toString(16)
						green = green.toString(16)
						blue = blue.toString(16)

						return `#${red}${green}${blue}`
					}
				}

				const shadeColor = (color, shade) => {
					let red = parseInt(color.slice(0, 2), 16)
					let green = parseInt(color.slice(2, 4), 16)
					let blue = parseInt(color.slice(4, 6), 16)

					red = Math.round(shade * red)
					green = Math.round(shade * green)
					blue = Math.round(shade * blue)
					if(red < 16) red = '0' + red
					if(green < 16) green = '0' + green
					if(blue < 16) blue = '0' + blue
					red = red.toString(16);
					green = green.toString(16);
					blue = blue.toString(16);

					return `#${red}${green}${blue}`
				}

				const clusters = [theme]
				for(let i = 0; i <= 9; i++) {
					clusters.push(tintColor(theme, Number((i / 10).toFixed(2))))
				}
				clusters.push(shadeColor(theme, 0.9)) //不能删
				return clusters
			}
		}
	}
</script>

<style>
	.theme-picker .el-color-picker__trigger {
		vertical-align: middle;
	}
	
	.theme-picker-dropdown .el-color-dropdown__link-btn {
		display: none;
	}
</style>