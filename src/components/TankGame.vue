<template>
  <div class="tank-game">
    <div class="game-header">
      <div class="score-display">
        <span class="label">分数:</span>
        <span class="value">{{ score }}</span>
      </div>
      <div class="level-display">
        <span class="label">关卡:</span>
        <span class="value">{{ level }}</span>
      </div>
      <div class="lives-display">
        <span class="label">生命:</span>
        <span class="value">{{ lives }}</span>
      </div>
    </div>
    
    <canvas ref="gameCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    
    <div v-if="gameState === 'menu'" class="menu-overlay">
      <div class="menu-content">
        <h1 class="game-title">坦克大战</h1>
        <p class="game-subtitle">TANK BATTLE</p>
        <button class="start-btn" @click="startGame">开始游戏</button>
        
        <div class="help-panel">
          <h3 class="help-title">🎮 操作指南</h3>
          <div class="help-section">
            <div class="help-item">
              <span class="key-badge">W</span>
              <span class="key-badge">↑</span>
              <span class="key-desc">向上移动</span>
            </div>
            <div class="help-item">
              <span class="key-badge">S</span>
              <span class="key-badge">↓</span>
              <span class="key-desc">向下移动</span>
            </div>
            <div class="help-item">
              <span class="key-badge">A</span>
              <span class="key-badge">←</span>
              <span class="key-desc">向左移动</span>
            </div>
            <div class="help-item">
              <span class="key-badge">D</span>
              <span class="key-badge">→</span>
              <span class="key-desc">向右移动</span>
            </div>
            <div class="help-item">
              <span class="key-badge wide">空格</span>
              <span class="key-desc">发射子弹</span>
            </div>
            <div class="help-item">
              <span class="key-badge">P</span>
              <span class="key-desc">暂停/继续</span>
            </div>
          </div>

          <h3 class="help-title">📜 游戏规则</h3>
          <div class="rules-section">
            <p>1. 消灭所有敌人即可进入下一关</p>
            <p>2. 被敌人子弹或敌人本体击中会损失一条生命</p>
            <p>3. 生命值归零时游戏结束</p>
            <p>4. 收集道具可以获得各种增益效果</p>
            <p>5. 钢墙无法被破坏，水域可以通行但子弹会穿过</p>
          </div>

          <h3 class="help-title">⭐ 道具说明</h3>
          <div class="powerup-section">
            <div class="powerup-item">
              <span class="powerup-icon" style="color: #00ffff;">⚡</span>
              <span class="powerup-name">速度提升</span>
              <span class="powerup-desc">增加移动速度</span>
            </div>
            <div class="powerup-item">
              <span class="powerup-icon" style="color: #ff00ff;">★</span>
              <span class="powerup-name">火力增强</span>
              <span class="powerup-desc">减少射击冷却</span>
            </div>
            <div class="powerup-item">
              <span class="powerup-icon" style="color: #00ff00;">♥</span>
              <span class="powerup-name">生命恢复</span>
              <span class="powerup-desc">增加一条生命</span>
            </div>
            <div class="powerup-item">
              <span class="powerup-icon" style="color: #ffff00;">◆</span>
              <span class="powerup-name">护盾加分</span>
              <span class="powerup-desc">直接获得 50 分</span>
            </div>
          </div>

          <h3 class="help-title">👾 敌人类型</h3>
          <div class="enemy-section">
            <div class="enemy-item">
              <div class="enemy-preview" style="background: #ff4444;"></div>
              <div class="enemy-info">
                <span class="enemy-name">普通敌人</span>
                <span class="enemy-desc">速度中等，1发子弹摧毁</span>
                <span class="enemy-score">100 分</span>
              </div>
            </div>
            <div class="enemy-item">
              <div class="enemy-preview" style="background: #ffaa00;"></div>
              <div class="enemy-info">
                <span class="enemy-name">快速敌人</span>
                <span class="enemy-desc">速度很快，1发子弹摧毁</span>
                <span class="enemy-score">200 分</span>
              </div>
            </div>
            <div class="enemy-item">
              <div class="enemy-preview" style="background: #aa00ff;"></div>
              <div class="enemy-info">
                <span class="enemy-name">重型敌人</span>
                <span class="enemy-desc">速度较慢，3发子弹摧毁</span>
                <span class="enemy-score">300 分</span>
              </div>
            </div>
          </div>

          <h3 class="help-title">🧱 墙壁类型</h3>
          <div class="wall-section">
            <div class="wall-item">
              <div class="wall-preview" style="background: #cc8844;"></div>
              <div class="wall-info">
                <span class="wall-name">砖墙</span>
                <span class="wall-desc">可被子弹破坏（2发）</span>
              </div>
            </div>
            <div class="wall-item">
              <div class="wall-preview" style="background: #888888;"></div>
              <div class="wall-info">
                <span class="wall-name">钢墙</span>
                <span class="wall-desc">无法被破坏</span>
              </div>
            </div>
            <div class="wall-item">
              <div class="wall-preview" style="background: #4488ff;"></div>
              <div class="wall-info">
                <span class="wall-name">水域</span>
                <span class="wall-desc">可通行，子弹穿过</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div v-if="gameState === 'paused'" class="pause-overlay">
      <div class="pause-content">
        <h2>游戏暂停</h2>
        <button class="resume-btn" @click="resumeGame">继续游戏</button>
      </div>
    </div>
    
    <div v-if="gameState === 'gameover'" class="gameover-overlay">
      <div class="gameover-content">
        <h2>游戏结束</h2>
        <p class="final-score">最终分数: {{ score }}</p>
        <p class="final-level">到达关卡: {{ level }}</p>
        <button class="restart-btn" @click="restartGame">重新开始</button>
      </div>
    </div>
    
    <div v-if="gameState === 'levelup'" class="levelup-overlay">
      <div class="levelup-content">
        <h2>关卡 {{ level }} 完成!</h2>
        <p class="level-score">当前分数: {{ score }}</p>
        <button class="next-level-btn" @click="nextLevel">下一关</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const canvasWidth = 800
const canvasHeight = 600
const gameCanvas = ref<HTMLCanvasElement>()
const gameState = ref<'menu' | 'playing' | 'paused' | 'gameover' | 'levelup'>('menu')
const score = ref(0)
const level = ref(1)
const lives = ref(3)

// 游戏上下文
let ctx: CanvasRenderingContext2D | null = null
let animationId: number | null = null

// 游戏对象
const player = ref({
  x: canvasWidth / 2 - 20,
  y: canvasHeight - 60,
  width: 40,
  height: 40,
  speed: 5,
  direction: 'up',
  color: '#00ff88',
  shootCooldown: 0,
  shootDelay: 15
})

const bullets: Bullet[] = []
const enemies: Enemy[] = []
const walls: Wall[] = []
const particles: Particle[] = []
const powerUps: PowerUp[] = []

// 输入状态
const keys: { [key: string]: boolean } = {}

// 类型定义
interface Bullet {
  x: number
  y: number
  width: number
  height: number
  speed: number
  direction: 'up' | 'down' | 'left' | 'right'
  isPlayer: boolean
  color: string
}

interface Enemy {
  x: number
  y: number
  width: 35
  height: 35
  speed: number
  direction: 'up' | 'down' | 'left' | 'right'
  color: string
  health: number
  shootCooldown: number
  moveTimer: number
  type: 'normal' | 'fast' | 'heavy'
}

interface Wall {
  x: number
  y: number
  width: number
  height: number
  type: 'brick' | 'steel' | 'water'
  health: number
  color: string
}

interface Particle {
  x: number
  y: number
  vx: number
  vy: number
  life: number
  maxLife: number
  color: string
  size: number
}

interface PowerUp {
  x: number
  y: number
  width: 25
  height: 25
  type: 'speed' | 'power' | 'life' | 'shield'
  color: string
}

// 关卡配置
const levelConfigs = [
  { enemies: 3, enemySpeed: 1.5, walls: 8 },
  { enemies: 5, enemySpeed: 2, walls: 10 },
  { enemies: 7, enemySpeed: 2.5, walls: 12 },
  { enemies: 10, enemySpeed: 3, walls: 15 },
  { enemies: 12, enemySpeed: 3.5, walls: 18 },
  { enemies: 15, enemySpeed: 4, walls: 20 },
]

// 初始化关卡
function initLevel() {
  const config = levelConfigs[Math.min(level.value - 1, levelConfigs.length - 1)]
  
  // 重置游戏对象
  enemies.length = 0
  bullets.length = 0
  walls.length = 0
  powerUps.length = 0
  
  // 重置玩家位置
  player.value.x = canvasWidth / 2 - 20
  player.value.y = canvasHeight - 60
  player.value.direction = 'up'
  
  // 创建敌人
  const enemyTypes: ('normal' | 'fast' | 'heavy')[] = ['normal', 'normal', 'fast', 'heavy']
  for (let i = 0; i < config.enemies; i++) {
    const type = enemyTypes[i % enemyTypes.length]
    let color = '#ff4444'
    let health = 1
    let speed = config.enemySpeed
    
    if (type === 'fast') {
      color = '#ffaa00'
      speed *= 1.5
    } else if (type === 'heavy') {
      color = '#aa00ff'
      health = 3
      speed *= 0.7
    }
    
    enemies.push({
      x: 50 + (i % 5) * 150,
      y: 50 + Math.floor(i / 5) * 100,
      width: 35,
      height: 35,
      speed,
      direction: 'down',
      color,
      health,
      shootCooldown: Math.random() * 60 + 60,
      moveTimer: 0,
      type
    })
  }
  
  // 创建墙壁
  for (let i = 0; i < config.walls; i++) {
    const wallType = ['brick', 'brick', 'steel', 'water'][Math.floor(Math.random() * 4)] as Wall['type']
    let health = wallType === 'steel' ? 999 : wallType === 'water' ? 999 : 2
    let color = wallType === 'steel' ? '#888888' : wallType === 'water' ? '#4488ff' : '#cc8844'
    
    walls.push({
      x: 100 + Math.random() * (canvasWidth - 200),
      y: 150 + Math.random() * (canvasHeight - 300),
      width: 40 + Math.random() * 40,
      height: 40 + Math.random() * 40,
      type: wallType,
      health,
      color
    })
  }
}

// 创建粒子效果
function createParticles(x: number, y: number, color: string, count: number = 10) {
  for (let i = 0; i < count; i++) {
    const angle = (Math.PI * 2 * i) / count
    const speed = 2 + Math.random() * 3
    particles.push({
      x,
      y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 30 + Math.random() * 20,
      maxLife: 50,
      color,
      size: 2 + Math.random() * 3
    })
  }
}

// 创建爆炸效果
function createExplosion(x: number, y: number, size: number = 1) {
  const colors = ['#ff4444', '#ffaa00', '#ffff00', '#ffffff']
  for (let i = 0; i < 15 * size; i++) {
    const angle = Math.random() * Math.PI * 2
    const speed = 2 + Math.random() * 4 * size
    particles.push({
      x,
      y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 40 + Math.random() * 30,
      maxLife: 70,
      color: colors[Math.floor(Math.random() * colors.length)],
      size: 2 + Math.random() * 4 * size
    })
  }
}

// 碰撞检测
function checkCollision(a: any, b: any): boolean {
  return a.x < b.x + b.width &&
         a.x + a.width > b.x &&
         a.y < b.y + b.height &&
         a.y + a.height > b.y
}

// 更新游戏
function update() {
  if (gameState.value !== 'playing') return
  
  // 更新玩家
  if (player.value.shootCooldown > 0) player.value.shootCooldown--
  
  let dx = 0
  let dy = 0
  
  if (keys['w'] || keys['arrowup']) { dy = -player.value.speed; player.value.direction = 'up' }
  if (keys['s'] || keys['arrowdown']) { dy = player.value.speed; player.value.direction = 'down' }
  if (keys['a'] || keys['arrowleft']) { dx = -player.value.speed; player.value.direction = 'left' }
  if (keys['d'] || keys['arrowright']) { dx = player.value.speed; player.value.direction = 'right' }
  
  // 玩家移动和碰撞检测
  const newX = player.value.x + dx
  const newY = player.value.y + dy
  
  let canMove = true
  const testPlayer = { ...player.value, x: newX, y: newY }
  
  for (const wall of walls) {
    if (wall.type !== 'water' && checkCollision(testPlayer, wall)) {
      canMove = false
      break
    }
  }
  
  if (canMove) {
    player.value.x = Math.max(0, Math.min(canvasWidth - player.value.width, newX))
    player.value.y = Math.max(0, Math.min(canvasHeight - player.value.height, newY))
  }
  
  // 更新子弹
  for (let i = bullets.length - 1; i >= 0; i--) {
    const bullet = bullets[i]
    
    switch (bullet.direction) {
      case 'up': bullet.y -= bullet.speed; break
      case 'down': bullet.y += bullet.speed; break
      case 'left': bullet.x -= bullet.speed; break
      case 'right': bullet.x += bullet.speed; break
    }
    
    // 子弹出界
    if (bullet.x < 0 || bullet.x > canvasWidth || bullet.y < 0 || bullet.y > canvasHeight) {
      bullets.splice(i, 1)
      continue
    }
    
    // 子弹与墙壁碰撞
    let hitWall = false
    for (let j = walls.length - 1; j >= 0; j--) {
      const wall = walls[j]
      if (checkCollision(bullet, wall)) {
        if (wall.type !== 'water') {
          wall.health--
          createParticles(bullet.x, bullet.y, wall.color, 5)
          if (wall.health <= 0) {
            createExplosion(wall.x + wall.width / 2, wall.y + wall.height / 2)
            walls.splice(j, 1)
          }
        }
        hitWall = true
        break
      }
    }
    
    if (hitWall) {
      bullets.splice(i, 1)
      continue
    }
    
    // 玩家子弹与敌人碰撞
    if (bullet.isPlayer) {
      for (let j = enemies.length - 1; j >= 0; j--) {
        const enemy = enemies[j]
        if (checkCollision(bullet, enemy)) {
          enemy.health--
          createParticles(bullet.x, bullet.y, enemy.color, 8)
          
          if (enemy.health <= 0) {
            createExplosion(enemy.x + enemy.width / 2, enemy.y + enemy.height / 2)
            score.value += enemy.type === 'heavy' ? 300 : enemy.type === 'fast' ? 200 : 100
            
            // 随机掉落道具
            if (Math.random() < 0.15) {
              const types: PowerUp['type'][] = ['speed', 'power', 'life', 'shield']
              const colors = ['#00ffff', '#ff00ff', '#00ff00', '#ffff00']
              const idx = Math.floor(Math.random() * types.length)
              powerUps.push({
                x: enemy.x,
                y: enemy.y,
                width: 25,
                height: 25,
                type: types[idx],
                color: colors[idx]
              })
            }
            
            enemies.splice(j, 1)
          }
          
          bullets.splice(i, 1)
          break
        }
      }
    } else {
      // 敌人子弹与玩家碰撞
      if (checkCollision(bullet, player.value)) {
        createParticles(bullet.x, bullet.y, player.value.color, 10)
        bullets.splice(i, 1)
        lives.value--
        createExplosion(player.value.x + player.value.width / 2, player.value.y + player.value.height / 2, 0.5)
        
        if (lives.value <= 0) {
          gameState.value = 'gameover'
        }
      }
    }
  }
  
  // 更新敌人
  for (const enemy of enemies) {
    // 敌人射击
    enemy.shootCooldown--
    if (enemy.shootCooldown <= 0) {
      const bulletX = enemy.x + enemy.width / 2 - 3
      const bulletY = enemy.y + enemy.height / 2 - 6
      bullets.push({
        x: bulletX,
        y: bulletY,
        width: 6,
        height: 12,
        speed: 5,
        direction: enemy.direction,
        isPlayer: false,
        color: '#ff6666'
      })
      enemy.shootCooldown = 60 + Math.random() * 60
    }
    
    // 敌人移动
    enemy.moveTimer--
    if (enemy.moveTimer <= 0) {
      const directions: ('up' | 'down' | 'left' | 'right')[] = ['up', 'down', 'left', 'right']
      enemy.direction = directions[Math.floor(Math.random() * 4)]
      enemy.moveTimer = 30 + Math.random() * 60
    }
    
    let edx = 0, edy = 0
    switch (enemy.direction) {
      case 'up': edy = -enemy.speed; break
      case 'down': edy = enemy.speed; break
      case 'left': edx = -enemy.speed; break
      case 'right': edx = enemy.speed; break
    }
    
    const newEX = enemy.x + edx
    const newEY = enemy.y + edy
    
    let canEMove = true
    const testEnemy = { ...enemy, x: newEX, y: newEY }
    
    for (const wall of walls) {
      if (wall.type !== 'water' && checkCollision(testEnemy, wall)) {
        canEMove = false
        break
      }
    }
    
    if (canEMove) {
      enemy.x = Math.max(0, Math.min(canvasWidth - enemy.width, newEX))
      enemy.y = Math.max(0, Math.min(canvasHeight - enemy.height, newEY))
    } else {
      enemy.moveTimer = 0
    }
    
    // 敌人与玩家碰撞
    if (checkCollision(enemy, player.value)) {
      createExplosion(player.value.x + player.value.width / 2, player.value.y + player.value.height / 2)
      lives.value--
      if (lives.value <= 0) {
        gameState.value = 'gameover'
      }
    }
  }
  
  // 更新道具
  for (let i = powerUps.length - 1; i >= 0; i--) {
    const powerUp = powerUps[i]
    if (checkCollision(powerUp, player.value)) {
      switch (powerUp.type) {
        case 'speed': player.value.speed = Math.min(8, player.value.speed + 1); break
        case 'power': player.value.shootDelay = Math.max(5, player.value.shootDelay - 3); break
        case 'life': lives.value++; break
        case 'shield': score.value += 50; break
      }
      createParticles(powerUp.x + 12, powerUp.y + 12, powerUp.color, 15)
      powerUps.splice(i, 1)
    }
  }
  
  // 更新粒子
  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i]
    p.x += p.vx
    p.y += p.vy
    p.vx *= 0.95
    p.vy *= 0.95
    p.life--
    if (p.life <= 0) {
      particles.splice(i, 1)
    }
  }
  
  // 检查关卡完成
  if (enemies.length === 0) {
    gameState.value = 'levelup'
  }
}

// 绘制游戏
function draw() {
  if (!ctx) return
  
  // 清空画布
  ctx.fillStyle = '#1a1a2e'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  // 绘制网格背景
  ctx.strokeStyle = '#2a2a4e'
  ctx.lineWidth = 1
  for (let x = 0; x < canvasWidth; x += 40) {
    ctx.beginPath()
    ctx.moveTo(x, 0)
    ctx.lineTo(x, canvasHeight)
    ctx.stroke()
  }
  for (let y = 0; y < canvasHeight; y += 40) {
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(canvasWidth, y)
    ctx.stroke()
  }
  
  // 绘制墙壁
  for (const wall of walls) {
    ctx.fillStyle = wall.color
    ctx.fillRect(wall.x, wall.y, wall.width, wall.height)
    
    // 墙壁边框
    ctx.strokeStyle = '#ffffff33'
    ctx.lineWidth = 2
    ctx.strokeRect(wall.x, wall.y, wall.width, wall.height)
    
    // 砖块纹理
    if (wall.type === 'brick') {
      ctx.fillStyle = '#00000033'
      for (let bx = wall.x; bx < wall.x + wall.width; bx += 20) {
        for (let by = wall.y; by < wall.y + wall.height; by += 10) {
          ctx.fillRect(bx + 2, by + 2, 16, 6)
        }
      }
    } else if (wall.type === 'steel') {
      ctx.fillStyle = '#ffffff44'
      ctx.fillRect(wall.x + 5, wall.y + 5, wall.width - 10, wall.height - 10)
    } else if (wall.type === 'water') {
      ctx.fillStyle = '#66aaff44'
      const time = Date.now() / 500
      for (let i = 0; i < 3; i++) {
        ctx.beginPath()
        ctx.arc(
          wall.x + wall.width / 2 + Math.sin(time + i) * 10,
          wall.y + wall.height / 2 + Math.cos(time + i * 1.5) * 10,
          5,
          0,
          Math.PI * 2
        )
        ctx.fill()
      }
    }
  }
  
  // 绘制道具
  for (const powerUp of powerUps) {
    ctx.fillStyle = powerUp.color
    ctx.beginPath()
    ctx.arc(powerUp.x + 12, powerUp.y + 12, 12, 0, Math.PI * 2)
    ctx.fill()
    
    // 道具光晕
    const gradient = ctx.createRadialGradient(
      powerUp.x + 12, powerUp.y + 12, 0,
      powerUp.x + 12, powerUp.y + 12, 20
    )
    gradient.addColorStop(0, powerUp.color + '88')
    gradient.addColorStop(1, powerUp.color + '00')
    ctx.fillStyle = gradient
    ctx.beginPath()
    ctx.arc(powerUp.x + 12, powerUp.y + 12, 20, 0, Math.PI * 2)
    ctx.fill()
    
    // 道具图标
    ctx.fillStyle = '#ffffff'
    ctx.font = 'bold 14px Arial'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    const icons = { speed: '⚡', power: '★', life: '♥', shield: '◆' }
    ctx.fillText(icons[powerUp.type], powerUp.x + 12, powerUp.y + 12)
  }
  
  // 绘制玩家
  drawTank(player.value.x, player.value.y, player.value.width, player.value.height, player.value.direction, player.value.color, true)
  
  // 绘制敌人
  for (const enemy of enemies) {
    drawTank(enemy.x, enemy.y, enemy.width, enemy.height, enemy.direction, enemy.color, false)
    
    // 敌人血条
    const maxHealth = enemy.type === 'heavy' ? 3 : 1
    const healthBarWidth = enemy.width * (enemy.health / maxHealth)
    ctx.fillStyle = '#ff0000'
    ctx.fillRect(enemy.x, enemy.y - 8, enemy.width, 4)
    ctx.fillStyle = '#00ff00'
    ctx.fillRect(enemy.x, enemy.y - 8, healthBarWidth, 4)
  }
  
  // 绘制子弹
  for (const bullet of bullets) {
    ctx.fillStyle = bullet.color
    ctx.shadowColor = bullet.color
    ctx.shadowBlur = 10
    ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height)
    ctx.shadowBlur = 0
  }
  
  // 绘制粒子
  for (const p of particles) {
    const alpha = p.life / p.maxLife
    ctx.fillStyle = p.color + Math.floor(alpha * 255).toString(16).padStart(2, '0')
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.size * alpha, 0, Math.PI * 2)
    ctx.fill()
  }
}

// 绘制坦克
function drawTank(x: number, y: number, width: number, height: number, direction: string, color: string, isPlayer: boolean) {
  ctx!.save()
  ctx!.translate(x + width / 2, y + height / 2)
  
  // 旋转
  const rotations: { [key: string]: number } = { up: 0, right: Math.PI / 2, down: Math.PI, left: -Math.PI / 2 }
  ctx!.rotate(rotations[direction])
  
  // 坦克主体
  ctx!.fillStyle = color
  ctx!.shadowColor = color
  ctx!.shadowBlur = isPlayer ? 15 : 10
  ctx!.fillRect(-width / 2, -height / 2, width, height)
  ctx!.shadowBlur = 0
  
  // 坦克履带
  ctx!.fillStyle = '#333333'
  ctx!.fillRect(-width / 2 - 3, -height / 2, 6, height)
  ctx!.fillRect(width / 2 - 3, -height / 2, 6, height)
  
  // 坦克炮塔
  ctx!.fillStyle = isPlayer ? '#00aa55' : '#aa3333'
  ctx!.beginPath()
  ctx!.arc(0, 0, width / 3, 0, Math.PI * 2)
  ctx!.fill()
  
  // 坦克炮管
  ctx!.fillStyle = '#666666'
  ctx!.fillRect(-3, -height / 2 - 10, 6, height / 2)
  
  // 炮管尖端
  ctx!.fillStyle = isPlayer ? '#00ff88' : '#ff4444'
  ctx!.beginPath()
  ctx!.arc(0, -height / 2 - 10, 4, 0, Math.PI * 2)
  ctx!.fill()
  
  ctx!.restore()
}

// 游戏循环
function gameLoop() {
  update()
  draw()
  
  animationId = requestAnimationFrame(gameLoop)
}

// 开始游戏
function startGame() {
  score.value = 0
  level.value = 1
  lives.value = 3
  player.value.speed = 5
  player.value.shootDelay = 15
  gameState.value = 'playing'
  initLevel()
}

// 暂停游戏
function pauseGame() {
  if (gameState.value === 'playing') {
    gameState.value = 'paused'
  }
}

// 继续游戏
function resumeGame() {
  if (gameState.value === 'paused') {
    gameState.value = 'playing'
  }
}

// 重新开始
function restartGame() {
  startGame()
}

// 下一关
function nextLevel() {
  level.value++
  gameState.value = 'playing'
  initLevel()
}

// 射击
function shoot() {
  if (gameState.value !== 'playing') return
  if (player.value.shootCooldown > 0) return
  
  const bulletX = player.value.x + player.value.width / 2 - 3
  const bulletY = player.value.y + player.value.height / 2 - 6
  
  let bx = bulletX, by = bulletY
  switch (player.value.direction) {
    case 'up': by = player.value.y - 10; break
    case 'down': by = player.value.y + player.value.height; break
    case 'left': bx = player.value.x - 10; break
    case 'right': bx = player.value.x + player.value.width; break
  }
  
  bullets.push({
    x: bx,
    y: by,
    width: 6,
    height: 12,
    speed: 8,
    direction: player.value.direction as 'up' | 'down' | 'left' | 'right',
    isPlayer: true,
    color: '#00ff88'
  })
  
  player.value.shootCooldown = player.value.shootDelay
  createParticles(bx, by, '#00ff88', 5)
}

// 键盘事件
function handleKeyDown(e: KeyboardEvent) {
  keys[e.key.toLowerCase()] = true
  
  if (e.key === ' ') {
    e.preventDefault()
    shoot()
  }
  
  if (e.key.toLowerCase() === 'p') {
    if (gameState.value === 'playing') {
      pauseGame()
    } else if (gameState.value === 'paused') {
      resumeGame()
    }
  }
}

function handleKeyUp(e: KeyboardEvent) {
  keys[e.key.toLowerCase()] = false
}

onMounted(() => {
  ctx = gameCanvas.value?.getContext('2d') || null
  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', handleKeyUp)
  animationId = requestAnimationFrame(gameLoop)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
})
</script>

<style scoped>
.tank-game {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.game-header {
  display: flex;
  gap: 40px;
  padding: 15px 30px;
  background: linear-gradient(135deg, #2a2a4e 0%, #1a1a3e 100%);
  border-radius: 15px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
  border: 2px solid #3a3a6e;
}

.score-display, .level-display, .lives-display {
  display: flex;
  align-items: center;
  gap: 10px;
}

.label {
  color: #8888aa;
  font-size: 16px;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.value {
  color: #00ff88;
  font-size: 24px;
  font-weight: bold;
  text-shadow: 0 0 10px #00ff88;
  min-width: 60px;
}

canvas {
  border-radius: 10px;
  box-shadow: 0 0 30px rgba(0, 255, 136, 0.3);
  border: 3px solid #3a3a6e;
}

.menu-overlay, .pause-overlay, .gameover-overlay, .levelup-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(10, 10, 30, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}

.menu-content, .pause-content, .gameover-content, .levelup-content {
  text-align: center;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.game-title {
  font-size: 64px;
  font-weight: bold;
  background: linear-gradient(135deg, #00ff88 0%, #00aaff 50%, #ff00aa 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 10px;
  text-shadow: 0 0 30px rgba(0, 255, 136, 0.5);
  animation: glow 2s ease-in-out infinite alternate;
}

@keyframes glow {
  from {
    filter: drop-shadow(0 0 10px #00ff88);
  }
  to {
    filter: drop-shadow(0 0 30px #00aaff);
  }
}

.game-subtitle {
  font-size: 24px;
  color: #8888aa;
  letter-spacing: 10px;
  margin-bottom: 40px;
}

.start-btn, .resume-btn, .restart-btn, .next-level-btn {
  padding: 15px 50px;
  font-size: 20px;
  font-weight: bold;
  color: #ffffff;
  background: linear-gradient(135deg, #00ff88 0%, #00aaff 100%);
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 5px 20px rgba(0, 255, 136, 0.4);
  margin-bottom: 30px;
}

.start-btn:hover, .resume-btn:hover, .restart-btn:hover, .next-level-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 30px rgba(0, 255, 136, 0.6);
}

.start-btn:active, .resume-btn:active, .restart-btn:active, .next-level-btn:active {
  transform: translateY(0);
}

.help-panel {
  background: rgba(20, 20, 40, 0.95);
  border: 2px solid #3a3a6e;
  border-radius: 15px;
  padding: 20px;
  margin-top: 20px;
  max-width: 500px;
  max-height: 400px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #3a3a6e #1a1a3e;
}

.help-panel::-webkit-scrollbar {
  width: 8px;
}

.help-panel::-webkit-scrollbar-track {
  background: #1a1a3e;
  border-radius: 4px;
}

.help-panel::-webkit-scrollbar-thumb {
  background: #3a3a6e;
  border-radius: 4px;
}

.help-panel::-webkit-scrollbar-thumb:hover {
  background: #4a4a8e;
}

.help-title {
  font-size: 18px;
  color: #00ff88;
  margin: 15px 0 10px 0;
  padding-bottom: 5px;
  border-bottom: 1px solid #3a3a6e;
}

.help-section {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  margin-bottom: 15px;
}

.help-item {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(40, 40, 80, 0.5);
  padding: 8px;
  border-radius: 8px;
}

.key-badge {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  min-width: 28px;
  height: 28px;
  background: linear-gradient(135deg, #3a3a6e 0%, #2a2a4e 100%);
  border: 1px solid #4a4a8e;
  border-radius: 6px;
  color: #ffffff;
  font-size: 12px;
  font-weight: bold;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}

.key-badge.wide {
  min-width: 50px;
}

.key-desc {
  color: #aaaacc;
  font-size: 13px;
}

.rules-section {
  background: rgba(40, 40, 80, 0.5);
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 15px;
}

.rules-section p {
  color: #aaaacc;
  font-size: 13px;
  margin: 6px 0;
  line-height: 1.5;
}

.powerup-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 15px;
}

.powerup-item {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(40, 40, 80, 0.5);
  padding: 10px;
  border-radius: 8px;
}

.powerup-icon {
  font-size: 20px;
  width: 30px;
  text-align: center;
}

.powerup-name {
  color: #ffffff;
  font-size: 14px;
  font-weight: bold;
  flex: 1;
}

.powerup-desc {
  color: #8888aa;
  font-size: 12px;
}

.enemy-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 15px;
}

.enemy-item {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(40, 40, 80, 0.5);
  padding: 10px;
  border-radius: 8px;
}

.enemy-preview {
  width: 30px;
  height: 30px;
  border-radius: 4px;
  border: 2px solid #ffffff33;
}

.enemy-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
}

.enemy-name {
  color: #ffffff;
  font-size: 14px;
  font-weight: bold;
}

.enemy-desc {
  color: #8888aa;
  font-size: 12px;
}

.enemy-score {
  color: #00ff88;
  font-size: 12px;
  font-weight: bold;
}

.wall-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.wall-item {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(40, 40, 80, 0.5);
  padding: 10px;
  border-radius: 8px;
}

.wall-preview {
  width: 30px;
  height: 30px;
  border-radius: 4px;
  border: 2px solid #ffffff33;
}

.wall-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
}

.wall-name {
  color: #ffffff;
  font-size: 14px;
  font-weight: bold;
}

.wall-desc {
  color: #8888aa;
  font-size: 12px;
}

.pause-content h2, .gameover-content h2, .levelup-content h2 {
  font-size: 48px;
  color: #ffffff;
  margin-bottom: 30px;
}

.final-score, .final-level, .level-score {
  font-size: 24px;
  color: #00ff88;
  margin-bottom: 20px;
}
</style>
