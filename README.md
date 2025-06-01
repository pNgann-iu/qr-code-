<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Galaxy-Heart QR ✨</title>

<style>
html,body{height:100%;margin:0;background:#000;display:grid;place-items:center;overflow:hidden;font-family:system-ui,sans-serif}
#stage{background:#000;cursor:pointer}
#hint{position:fixed;bottom:1rem;left:50%;transform:translateX(-50%);color:#fff;font-size:.9rem;opacity:.4;user-select:none}
#message{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  color:#ffbbf9;font-size:1.3rem;font-weight:bold;white-space:nowrap;
  text-shadow:0 0 10px #ff69b4,0 0 20px #fff;
  opacity:0;pointer-events:none}
</style>

<script src="https://cdn.jsdelivr.net/npm/qrcode@1.4.4/build/qrcode.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js"></script>
</head>
<body>
  <canvas id="stage" width="600" height="600"></canvas>
  <div id="hint">Chạm / click sau khi quét QR ✨</div>
  <div id="message">Chúc cậu ngày nào cũng thật hạnh phúc, mãi iuuu 💖</div>

<script>
/* ========= CONFIG ========= */
const BASE_URL  = location.href.split('?')[0];
const QR_TEXT   = BASE_URL + '?auto=1';          // QR mã hoá auto=1
const urlParams = new URLSearchParams(location.search);
const AUTO_PLAY = urlParams.has('play') || urlParams.get('auto')==='1';

const NUM_SHOOTERS=180, HEART_PARTS=2300;
const DUR_METEOR=3, DUR_ASSEMBLE=3, DUR_PULSE=7, DUR_FADE=1;

const cvs=document.getElementById('stage'), ctx=cvs.getContext('2d');
const w=cvs.width, h=cvs.height; let time=0, animating=false;

/* ==== Heart helpers ==== */
function inHeart(x,y){return Math.pow(x*x+y*y-1,3)-x*x*y*y*y<=0;}
function randPoint(r = 210) {
  while (true) {
    const t = Math.random() * Math.PI * 2;
    const x = 16 * Math.pow(Math.sin(t), 3);
    const y = -(13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t));
    return [x * r * 0.045, y * r * 0.045];
  }
}

/* ==== Draw square QR ==== */
function drawQR(){
  const off=document.createElement('canvas');
  QRCode.toCanvas(off,QR_TEXT,{margin:1,width:300,color:{dark:"#000",light:"#fff"}},()=>{
    const ox=w/2-off.width/2, oy=h/2-off.height/2;
    ctx.fillStyle='#fff';ctx.fillRect(ox,oy,off.width,off.height);ctx.drawImage(off,ox,oy);
  });
}
if(!AUTO_PLAY)drawQR();else document.getElementById('hint').style.display='none';

/* ==== Arrays ==== */
const shooters=[], hearts=[]; let heartsFading=false;

/* ==== Start show ==== */
function startShow(){
  if(animating)return; animating=true;
  if(!AUTO_PLAY)gsap.to('#hint',{duration:.4,autoAlpha:0});

  /* sao băng */
  for(let i=0;i<NUM_SHOOTERS;i++){
    shooters.push({x:Math.random()*w*2-w*.5,y:-Math.random()*h*.6,
      vx:-(Math.random()*3+2),vy:(Math.random()*3+2),life:Math.random()*2+2});
  }
  gsap.delayedCall(DUR_METEOR,()=>gsap.to(cvs,{duration:DUR_FADE,onUpdate:fadeQR,onComplete:spawnHearts}));
}
cvs.addEventListener('click',startShow);
if(AUTO_PLAY){ctx.clearRect(0,0,w,h);startShow();}

/* ==== Fade QR ==== */
function fadeQR(){
  ctx.globalCompositeOperation='destination-out';
  ctx.fillStyle='rgba(0,0,0,0.08)';
  ctx.fillRect(0,0,w,h);
  ctx.globalCompositeOperation='source-over';
}

/* ==== Draw heart mini ==== */
function tinyHeart(x,y,s,alpha=1,scale=1){
  ctx.fillStyle = `rgba(255, 125, 233, ${alpha})`;
  ctx.beginPath();
  ctx.moveTo(x,y);
  ctx.bezierCurveTo(x-s*scale,y-s*scale,x-2*s*scale,y+s*scale,x,y+2*s*scale);
  ctx.bezierCurveTo(x+2*s*scale,y+s*scale,x+s*scale,y-s*scale,x,y);
  ctx.fill();
}
const easeOut=t=>t*(2-t);

/* ==== Hearts ==== */
function spawnHearts(){
  for(let i=0;i<HEART_PARTS*2.5;i++){ // Tăng số lượng trái tim nhỏ lên đáng kể
    const [targetX, targetY] = randPoint();
    const angle = Math.random() * Math.PI * 2;
    const radius = Math.max(w, h) * 0.8 * Math.random() + 50; // Bắt đầu xa hơn
    hearts.push({
      x: w / 2 + Math.cos(angle) * radius, // Bắt đầu từ ngoài khung hình
      y: h / 2 + Math.sin(angle) * radius,
      targetX: w / 2 + targetX, // Điểm đích bên trong hình trái tim lớn
      targetY: h / 2 + targetY,
      size: Math.random() * 1.2 + 0.3, // Kích thước ban đầu nhỏ hơn
      prog: 0,
      assembled: false,
      startPulse: 0,
      alpha: Math.random() * 0.4 + 0.6, // Độ trong suốt ban đầu cao hơn để rõ hơn
      tw: Math.random() * Math.PI * 2,
      vx: (w / 2 + targetX - (w / 2 + Math.cos(angle) * radius)) * 0.005 + (Math.random() - 0.5) * 4, // Tốc độ ban đầu chậm hơn, ít biến thiên
      vy: (h / 2 + targetY - (h / 2 + Math.sin(angle) * radius)) * 0.005 + (Math.random() - 0.5) * 4,
      delay: Math.random() * 1.8 // Tăng độ trễ để các trái tim bay vào mượt mà, phân tán hơn
    });
  }
  // Kích hoạt phase fade trái tim sau khi assemble và pulse xong
  setTimeout(() => heartsFading = true, (DUR_ASSEMBLE + DUR_PULSE) * 1e3);
  setTimeout(showMessage, (DUR_ASSEMBLE + DUR_PULSE + DUR_FADE) * 1e3);
}

/* ==== Message ==== */
function showMessage(){
  const msg=document.getElementById('message');
  gsap.to(msg,{duration:1,opacity:1});
  gsap.to(msg,{delay:10,duration:4,x:'-120%',opacity:0,ease:'power1.inOut'});
}

/* ==== Loop ==== */
function loop(){requestAnimationFrame(loop);const dt=.016;time+=dt;if(animating)ctx.clearRect(0,0,w,h);

  /* meteors */
  for(let i=shooters.length-1;i>=0;i--){
    const s=shooters.splice(i,1)[0];
    s.x+=s.vx;s.y+=s.vy;s.life-=dt;
    if(s.life<=0||s.x<-80||s.y>h+80) continue;
    ctx.strokeStyle='rgba(255,255,255,0.85)';ctx.lineWidth=2;
    ctx.beginPath();ctx.moveTo(s.x,s.y);ctx.lineTo(s.x-s.vx*7,s.y-s.vy*7);ctx.stroke();
    shooters.push(s);
  }

  /* hearts */
  hearts.forEach(p=>{
    p.delay -= dt;
    if (p.delay > 0) return; // Nếu còn delay thì chưa vẽ

    if(!p.assembled){
      // Tốc độ bay vào và gia tốc nhẹ dần khi gần đến đích
      const speed = 0.5 + Math.max(0, 1 - p.prog) * 0.5; // Giảm tốc độ khi gần đến đích
      p.x += p.vx * speed;
      p.y += p.vy * speed;

      // Tính khoảng cách đến đích
      const dist = Math.sqrt(Math.pow(p.x - p.targetX, 2) + Math.pow(p.y - p.targetY, 2));
      // Tăng prog dựa trên thời gian, không dựa trên khoảng cách
      p.prog = Math.min(p.prog + dt / DUR_ASSEMBLE * 1.5, 1); // Tăng prog nhanh hơn để assemble nhanh hơn
      if (dist < 10 && p.prog >= 0.9) { // assembled khi gần đích và prog đủ lớn
        p.assembled = true;
        p.startPulse = time;
      }
      tinyHeart(p.x, p.y, p.size, p.alpha);
    } else {
      // Hiệu ứng đập rõ ràng hơn và lấp lánh nhẹ
      const pulse = 1 + 0.25 * Math.sin((time - p.startPulse) * 8); // Biên độ và tần số đập
      const blink = (Math.sin(p.tw * 0.5 + time * 5) * 0.5 + 0.5) * 0.3 + 0.7; // Hiệu ứng lấp lánh nhẹ
      const currentAlpha = heartsFading ? Math.max(0, p.alpha - dt / DUR_FADE) : p.alpha;
      ctx.globalAlpha = currentAlpha * blink; // Kết hợp alpha và blink
      tinyHeart(p.x, p.y, p.size * pulse);
      ctx.globalAlpha = 1; // Reset globalAlpha sau khi vẽ
    }
  });
}
requestAnimationFrame(loop);
</script>
</body>
</html>
