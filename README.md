<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bouncy Balls with Microphone</title>
    <style>
        canvas {
            display: block;
            background: #000;
            margin: 0 auto;
        }
    </style>
</head>
<body>
    <canvas id="canvas"></canvas>
    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let balls = [];
        const ballCount = 100;

        // Ball class
        class Ball {
            constructor(x, y, radius) {
                this.x = x;
                this.y = y;
                this.radius = radius;
                this.dx = (Math.random() - 0.5) * 2;
                this.dy = (Math.random() - 0.5) * 2;
                this.color = 'rgba(' + Math.random() * 255 + ',' + Math.random() * 255 + ',' + Math.random() * 255 + ', 0.8)';
            }

            update() {
                if (this.x + this.radius > canvas.width || this.x - this.radius < 0) this.dx = -this.dx;
                if (this.y + this.radius > canvas.height || this.y - this.radius < 0) this.dy = -this.dy;
                this.x += this.dx;
                this.y += this.dy;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }
        }

        // Initialize balls
        for (let i = 0; i < ballCount; i++) {
            balls.push(new Ball(Math.random() * canvas.width, Math.random() * canvas.height, Math.random() * 20 + 5));
        }

        // Set up microphone
        navigator.mediaDevices.getUserMedia({ audio: true })
            .then(stream => {
                const audioContext = new (window.AudioContext || window.webkitAudioContext)();
                const analyser = audioContext.createAnalyser();
                const source = audioContext.createMediaStreamSource(stream);
                source.connect(analyser);
                analyser.fftSize = 256;
                const bufferLength = analyser.frequencyBinCount;
                const dataArray = new Uint8Array(bufferLength);

                function animate() {
                    requestAnimationFrame(animate);
                    analyser.getByteFrequencyData(dataArray);
                    const average = dataArray.reduce((a, b) => a + b) / bufferLength;

                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    balls.forEach(ball => {
                        ball.update();
                        ball.radius = average / 10; // Adjust ball size based on average microphone input
                        ball.draw();
                    });
                }

                animate();
            })
            .catch(err => {
                console.error('Error accessing microphone: ', err);
            });
    </script>
</body>
</html>
