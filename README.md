# test
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>p5.js 彩色粒子流动动画</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #111; /* 页面背景深灰，突出画布 */
            overflow: hidden;
        }
        canvas {
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
        }
    </style>
    <!-- 引入 p5.js CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
</head>
<body>

<script>
    let particles = [];
    const numParticles = 1500; // 粒子数量
    const noiseScale = 0.01;   // 噪声缩放比例，影响流动的平滑度

    function setup() {
        createCanvas(600, 400);
        colorMode(HSB, 360, 100, 100, 100); // 使用 HSB 颜色模式，方便生成彩虹色
        
        // 初始化粒子
        for (let i = 0; i < numParticles; i++) {
            particles.push(new Particle());
        }
        
        background(0);
    }

    function draw() {
        // 使用半透明黑色背景，产生拖尾效果
        noStroke();
        fill(0, 0, 0, 10); // 最后一个参数是透明度，越小拖尾越长
        rect(0, 0, width, height);

        // 更新并显示所有粒子
        for (let p of particles) {
            p.update();
            p.show();
        }
    }

    class Particle {
        constructor() {
            this.pos = createVector(random(width), random(height));
            this.vel = createVector(0, 0);
            this.acc = createVector(0, 0);
            this.maxSpeed = 2; // 最大速度
            this.prevPos = this.pos.copy();
            
            // 为每个粒子分配一个基础色相，使其颜色相对固定但随位置微调
            this.hue = random(360); 
        }

        update() {
            // 保存上一帧位置用于画线
            this.prevPos.x = this.pos.x;
            this.prevPos.y = this.pos.y;

            // 基于当前位置计算噪声角度
            let n = noise(this.pos.x * noiseScale, this.pos.y * noiseScale, frameCount * 0.002);
            let angle = n * TWO_PI * 2; // 将噪声映射到角度

            // 根据角度设置加速度
            this.acc = p5.Vector.fromAngle(angle);
            this.acc.setMag(0.5); // 加速度大小

            // 物理更新
            this.vel.add(this.acc);
            this.vel.limit(this.maxSpeed);
            this.pos.add(this.vel);

            // 边界处理：环绕屏幕
            this.handleEdges();
        }

        show() {
            // 颜色动态变化：基础色相 + 速度/位置偏移
            let h = (this.hue + this.pos.x * 0.1 + frameCount * 0.5) % 360;
            stroke(h, 80, 100, 50); // 高饱和度，中等亮度，半透明
            strokeWeight(1.5);
            
            // 绘制从上一位置到当前位置的线，比点更流畅
            line(this.prevPos.x, this.prevPos.y, this.pos.x, this.pos.y);
        }

        handleEdges() {
            if (this.pos.x > width) {
                this.pos.x = 0;
                this.prevPos.x = 0;
            }
            if (this.pos.x < 0) {
                this.pos.x = width;
                this.prevPos.x = width;
            }
            if (this.pos.y > height) {
                this.pos.y = 0;
                this.prevPos.y = 0;
            }
            if (this.pos.y < 0) {
                this.pos.y = height;
                this.prevPos.y = height;
            }
        }
    }
</script>

</body>
</html>
