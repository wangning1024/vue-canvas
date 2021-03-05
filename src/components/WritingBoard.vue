<template>
  <div>
    <div class="writing-board">
      <canvas @mousedown="mouseDown"
              @mousemove="mouseMove"
              @mouseup="mouseUp"
              @touchstart="touchStart"
              @touchmove="touchMove"
              @touchend="mouseUp"
              id="canvas">您的浏览器不支持 HTML5 canvas 标签</canvas>

      <img id='wbImageElement'/>
    </div>
    <div class="button">
      <button @click="clear">重写</button>
      <button @click="save">保存</button>
    </div>

  </div>

</template>

<script>
    export default {
        name: 'WritingBoard',
        data() {
            return {
                canvas: null,
                ctx: null,
                isDown: false,
                startTime: null,
                startX: 0,
                startY: 0,
                winW: 0,
                winH: 0,
            };
        },
        methods: {
            initCanvas() {
                this.canvas = document.getElementById('canvas');
                this.ctx = this.canvas.getContext('2d');
                this.ctx.lineCap = 'round';
                // this.ctx.lineJoin = 'round';
                // this.winW = window.innerWidth - 10;
                // this.winH = window.innerHeight - 100;
                this.canvas.width = this.winW;
                this.canvas.height = this.winH;

            },
            windowToCanvas(canvas, x, y) {
                let bbox = canvas.getBoundingClientRect(); // 获取canvas元素的边框
                return {
                    // 当canvas元素大小与绘图表面大小不相符时，将这两个坐标进行缩放
                    // x: x - bbox.left * (canvas.width / bbox.width),
                    // y: y - bbox.top * (canvas.height / bbox.height)
                    x: x - bbox.left,
                    y: y - bbox.top
                };

            },
            setLineWidth(x, y) {
                let nowDate = new Date().getTime();
                let timeDistance = 10000;
                if (this.startTime) {
                    timeDistance = nowDate - this.startTime.getTime();
                }
                let lineWidth = 6;
                if (timeDistance < 100) {
                    lineWidth = 3;
                } else if(timeDistance < 500) {
                    lineWidth = 4;
                } else if(timeDistance < 800) {
                    lineWidth = 5;
                }else {
                    let distance = Math.sqrt(Math.pow(x - this.startX, 2) +
                        Math.pow(y - this.startY, 2));
                    // console.log('distance', distance);
                    lineWidth = 6;
                    if (distance <= 1) {
                        lineWidth = 6;
                    } else if (distance > 1 && distance < 5) {
                        lineWidth = 5.5;
                    } else {
                        lineWidth = 4.5;
                    }
                }
                return lineWidth;

            },
            touchStart(ev) {
                ev.preventDefault();
                if (ev.touches.length == 1) {
                    this.startDown(ev.targetTouches[0].clientX, ev.targetTouches[0].clientY)
                }
            },
            mouseDown (ev) {
                ev.preventDefault();
                this.startDown(ev.clientX, ev.clientY)
            },
            startDown(clientX, clientY) {
                let loc = this.windowToCanvas(this.canvas, clientX, clientY);
                this.startX = loc.x;
                this.startY = loc.y;
                this.ctx.moveTo(this.startX, this.startY);
                this.isDown = true;
                this.startTime = new Date();
            },
            touchMove(ev) {
                ev.preventDefault();
                if (ev.touches.length == 1) {
                    this.move(ev.targetTouches[0].clientX, ev.targetTouches[0].clientY)
                }
            },
            mouseMove(ev) {
                ev.preventDefault();
                this.move(ev.clientX, ev.clientY);
            },
            move(clientX, clientY) {
                if (this.isDown) {
                    let loc = this.windowToCanvas(this.canvas, clientX, clientY);
                    // console.log('drawing');
                    this.ctx.beginPath();
                    this.ctx.moveTo(this.startX, this.startY);
                    this.ctx.lineTo(loc.x, loc.y);

                    // 获得笔触粗细
                    this.ctx.lineWidth = this.setLineWidth(loc.x, loc.y);
                    // this.ctx.lineWidth = 6;
                    this.startX = loc.x;
                    this.startY = loc.y;
                    this.ctx.stroke();
                }
            },

            mouseUp(ev) {
                ev.preventDefault();
                this.ctx.beginPath();
                this.ctx.moveTo(this.startX, this.startY);
                this.ctx.lineTo(this.startX + 1, this.startY + 2);
                this.ctx.lineWidth = 3.5;
                this.ctx.stroke();
                this.isDown = false;
            },
            clear() {
                this.ctx.clearRect(0, 0, this.winW, this.winH);
                this.isDown = false;
                this.startTime = null;
                let imageElement = document.getElementById('wbImageElement');
                imageElement.style.display = 'none';
                this.canvas.style.display = 'inline';
            },
            save() {
                let dataURL = this.canvas.toDataURL();
                // window.location.href = dataURL;
                let imageElement = document.getElementById('wbImageElement');
                imageElement.src = dataURL;
                imageElement.style.display = 'inline';
                this.canvas.style.display = 'none';
            }
        },
        mounted() {
            // this.winW = window.innerWidth - 10;
            // this.winH = window.innerHeight - 100;
            this.winW = document.body.clientWidth - 50;
            this.winH = 500;

            this.initCanvas();

        }
    };
</script>

<style scoped>

  .writing-board {
    /*margin: 0 50px;*/
    /*border: 1px black solid;*/
    border: thin solid #aaaaaa;
    -webkit-box-shadow: 4px 4px 8px rgba(0,0,0,0.5);
    -moz-box-shadow: 4px 4px 8px rgba(0,0,0,0.5);
    box-shadow: 4px 4px 8px rgba(0,0,0,0.5);
  }
  #canvas {

  }

  .wbImageElement {
    /*border: 1px black solid;*/
  }
  .button {
    margin: 20px auto;
    width: 140px;
  }
  .button button {
    margin: 0 10px;
  }


</style>
