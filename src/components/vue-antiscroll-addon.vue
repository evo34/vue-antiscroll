<script>
	import T from '../libs/t'
	import Antiscroll from '../libs/antiscroll'
	import ResizeSensor from 'css-element-queries/src/ResizeSensor'

	export default {
		props: {
			height: {
				required: false,
				type: null
			},
			width: {
				required: false,
				type: null
			},
			onScrolling: {
				required: false,
				type: Function
			},
			onScrollToBottom: {
				required: false,
				type: Function
			},
			initialDisplay: {
				required: false,
				type: Boolean,
				default: () => Boolean(1)
			}
		},
		data() {
			return {}
		},
		mounted() {
			let {initialDisplay} = this
			this.scroller = new Antiscroll(this.$el, {initialDisplay})
			this._onScroll = T.proxy(this, 'onScroll')
			this.scroller.inner.addEventListener('scroll', this._onScroll, false)
			this.attachDimensionChangeEvent()
		},
		beforeDestroy() {
			this.scroller && this.scroller.destroy()
			this.scroller && this.scroller.inner.removeEventListener('scroll', this._onScroll, false)
			Object.keys(this.resizeSensors).forEach(key => this.resizeSensors[key].detach())
			this.resizeSensors = null
			this.scroller = null
			this._onScroll = null
		},
		watch: {
			'height': 'refresh',
			'width': 'refresh'
		},
		computed: {
			_$styObj() {
				let {_$height, _$width} = this
				let hash = {height: _$height}
				_$width && (hash['width'] = _$width)
				return hash
			},
			_$height() {
				let {height} = this
				height = height + ''
                height = parseFloat(height) + + Antiscroll.scrollbarSize
				return height + 'px'
			},
			_$width() {
				let {width} = this
				width = width && width + ''
				if (!width) return null
                width = parseFloat(width) + Antiscroll.scrollbarSize
				return width + 'px'
			}
		},
		methods: {
			onScroll(evt) {
				let innerHeight = T.getStyle(this.scroller.inner, 'height')
				let scrollHeight = T.getStyle(this.scroller.inner, 'scrollHeight')
				let scrollTop = T.getStyle(this.scroller.inner, 'scrollTop')

				if (typeof this.onScrolling === 'function') {
					this.onScrolling.call(this, this.scroller, evt)
				}
				if (scrollHeight <= innerHeight + scrollTop) {
					if (typeof this.onScrollToBottom === 'function') {
						this.onScrollToBottom.call(this, this.scroller, evt)
					}
				}
			},
			scrollTo(placement) {
				let scroller = this.scroller
				if (scroller) {
					scroller.scrollTo(placement)
				}
			},

			refresh() {
				let scroller = this.scroller
				if (scroller) {
					scroller.refresh()
					this.detachDimensionChangeEvent()
					this.attachDimensionChangeEvent()
				}
			},
			attachDimensionChangeEvent() {
				try {
					var resizeSensors = this.resizeSensors = {}
					var scroller = this.scroller
					var innerChild = scroller.inner.childNodes[0]
					resizeSensors.innerChildObserver = new ResizeSensor(innerChild, () => scroller.refresh({updatable: false}))
				} catch (e) {
					console.info('滚动条错误辣!');
				}
			},
			detachDimensionChangeEvent() {
				let resizeSensors = this.resizeSensors
				Object.keys(resizeSensors).forEach(key => this.resizeSensors[key].detach())
			}
		}

	}
</script>

<template>
    <div class="antiscroll-wrap">
        <div class="antiscroll-inner" :style="_$styObj">
            <div class="inner-in-fact">
                <slot></slot>
            </div>
        </div>
    </div>
</template>