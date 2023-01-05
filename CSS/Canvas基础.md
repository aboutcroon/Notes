

当第一个 canvas 报错的时候，后面的就不会执行了

```js
initCanvas () {
      this.getCanvas('canvas-vgpu', this.vgpuPercent / 100)
      this.getCanvas('canvas-ratio', this.ratioPercent / 100)
      this.getCanvas('canvas-memory', this.memoryPercent / 100)
    },
```



环形百分比图

```js
 getCanvas (name, percent) {
      const ctx = this.$refs[name].getContext('2d')
      // 外圆环
      ctx.beginPath()
      ctx.arc(32, 32, 32, 0, 2 * Math.PI)
      ctx.strokeStyle = '#ffffff'
      ctx.fillStyle = '#F0F2F5'
      ctx.fill()
      ctx.stroke()
      // 内圆环
      ctx.beginPath()
      ctx.arc(32, 32, 19, 0, 2 * Math.PI)
      ctx.strokeStyle = '#ffffff'
      ctx.fillStyle = '#ffffff'
      ctx.fill()
      ctx.stroke()
      // 环形图的进度条
      ctx.beginPath()
      ctx.arc(32, 32, 25, -Math.PI / 2, -Math.PI / 2 + percent * (Math.PI * 2))
      ctx.lineWidth = 12
      ctx.strokeStyle = '#0083FF'
      ctx.stroke()
    },
```

