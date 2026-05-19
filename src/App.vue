<template>
  <div id="app">
    <div class="game-wrapper" ref="gameWrapper">
      <div class="game-header">
        <span class="title">⚔️ 英雄对决 ⚔️</span>
        <span class="round-indicator">🏆 三局两胜 · 第 {{ currentRound }} 局</span>
        <span class="tips">🎮 查看底部操作说明</span>
      </div>

      <div class="arena-container" ref="arenaContainer">
        <div class="arena-ground"></div>
        <div class="ground-lines"></div>
        <div class="spotlight"></div>

        <div class="particles-layer">
          <span
            v-for="p in particles"
            :key="p.id"
            class="particle"
            :style="{
              left: p.x + 'px',
              top: p.y + 'px',
              width: p.size + 'px',
              height: p.size + 'px',
              background: p.color,
              '--dx': p.dx + 'px',
              '--dy': p.dy + 'px',
              animationDuration: p.duration + 's',
            }"
          ></span>
        </div>

        <div class="damage-numbers-layer">
          <span
            v-for="d in damageNumbers"
            :key="d.id"
            class="damage-number"
            :class="{ crit: d.isCrit }"
            :style="{
              left: d.x + 'px',
              top: d.y + 'px',
              color: d.color,
            }"
            >{{ d.text }}</span
          >
        </div>

        <div class="special-text-layer">
          <span
            v-for="s in specialTexts"
            :key="s.id"
            class="special-text"
            :style="{
              left: s.x + 'px',
              top: s.y + 'px',
              color: s.color,
            }"
            >{{ s.text }}</span
          >
        </div>

        <div class="player-hud p1-hud" :style="{ left: player1.x - 50 + 'px', bottom: '190px' }">
          <span class="hud-name">⚔️ 圣骑士</span>
          <div class="hp-bar-outer">
            <div
              class="hp-bar-inner"
              :style="{ width: (player1.hp / player1.maxHp) * 100 + '%' }"
            ></div>
          </div>
          <span class="hp-text">{{ Math.max(0, player1.hp) }} / {{ player1.maxHp }}</span>
          <div class="energy-bar-outer">
            <div
              class="energy-bar-inner"
              :style="{ width: (player1.energy / player1.maxEnergy) * 100 + '%' }"
            ></div>
          </div>
          <div class="skill-indicators">
            <div class="skill-icon" :class="{ ready: player1.skill1Cd <= 0 }">
              K
              <div
                class="cooldown-overlay"
                :style="{ height: (player1.skill1Cd / player1.skill1MaxCd) * 100 + '%' }"
              ></div>
              <span class="cd-text" v-if="player1.skill1Cd > 0">{{
                player1.skill1Cd.toFixed(1)
              }}</span>
            </div>
            <div class="skill-icon" :class="{ ready: player1.skill2Cd <= 0 }">
              L
              <div
                class="cooldown-overlay"
                :style="{ height: (player1.skill2Cd / player2.skill2MaxCd) * 100 + '%' }"
              ></div>
              <span class="cd-text" v-if="player1.skill2Cd > 0">{{
                player1.skill2Cd.toFixed(1)
              }}</span>
            </div>
            <div
              class="skill-icon ultimate"
              :class="{ ready: player1.skill3Cd <= 0 && player1.energy >= player1.skill3Cost }"
            >
              I
              <div
                class="cooldown-overlay"
                :style="{ height: (player1.skill3Cd / player1.skill3MaxCd) * 100 + '%' }"
              ></div>
              <span class="cd-text" v-if="player1.skill3Cd > 0">{{
                player1.skill3Cd.toFixed(1)
              }}</span>
            </div>
          </div>
        </div>

        <div class="player-hud p2-hud" :style="{ left: player2.x - 50 + 'px', bottom: '190px' }">
          <span class="hud-name">🗡️ 暗影武士</span>
          <div class="hp-bar-outer">
            <div
              class="hp-bar-inner"
              :style="{ width: (player2.hp / player2.maxHp) * 100 + '%' }"
            ></div>
          </div>
          <span class="hp-text">{{ Math.max(0, player2.hp) }} / {{ player2.maxHp }}</span>
          <div class="energy-bar-outer">
            <div
              class="energy-bar-inner"
              :style="{ width: (player2.energy / player2.maxEnergy) * 100 + '%' }"
            ></div>
          </div>
          <div class="skill-indicators">
            <div class="skill-icon" :class="{ ready: player2.skill1Cd <= 0 }">
              2
              <div
                class="cooldown-overlay"
                :style="{ height: (player2.skill1Cd / player2.skill1MaxCd) * 100 + '%' }"
              ></div>
              <span class="cd-text" v-if="player2.skill1Cd > 0">{{
                player2.skill1Cd.toFixed(1)
              }}</span>
            </div>
            <div class="skill-icon" :class="{ ready: player2.skill2Cd <= 0 }">
              3
              <div
                class="cooldown-overlay"
                :style="{ height: (player2.skill2Cd / player2.skill2MaxCd) * 100 + '%' }"
              ></div>
              <span class="cd-text" v-if="player2.skill2Cd > 0">{{
                player2.skill2Cd.toFixed(1)
              }}</span>
            </div>
            <div
              class="skill-icon ultimate"
              :class="{ ready: player2.skill3Cd <= 0 && player2.energy >= player2.skill3Cost }"
            >
              4
              <div
                class="cooldown-overlay"
                :style="{ height: (player2.skill3Cd / player2.skill3MaxCd) * 100 + '%' }"
              ></div>
              <span class="cd-text" v-if="player2.skill3Cd > 0">{{
                player2.skill3Cd.toFixed(1)
              }}</span>
            </div>
          </div>
        </div>

        <div
          class="player-character p1"
          :class="{
            attacking: player1.isAttacking,
            blocking: player1.isBlocking && !player1.isPerfectBlock,
            'perfect-block': player1.isPerfectBlock,
            shaking: player1.isShaking,
          }"
          :style="{ left: player1.x - 30 + 'px', '--lunge-dir': '25px', '--swing-rot': '35deg' }"
        >
          <div class="character-sprite">
            <div class="char-head"></div>
            <div class="char-body"></div>
            <div class="char-weapon"></div>
            <div class="char-legs">
              <div class="char-leg"></div>
              <div class="char-leg"></div>
            </div>
          </div>
        </div>

        <div
          class="player-character p2"
          :class="{
            attacking: player2.isAttacking,
            blocking: player2.isBlocking && !player2.isPerfectBlock,
            'perfect-block': player2.isPerfectBlock,
            shaking: player2.isShaking,
          }"
          :style="{ left: player2.x - 30 + 'px', '--lunge-dir': '-25px', '--swing-rot': '-35deg' }"
        >
          <div class="character-sprite">
            <div class="char-head"></div>
            <div class="char-body"></div>
            <div class="char-weapon"></div>
            <div class="char-legs">
              <div class="char-leg"></div>
              <div class="char-leg"></div>
            </div>
          </div>
        </div>

        <div class="controls-hint">
          <span>🟦 P1: A/D移动 S格挡 J普攻 K一技 L二技 I三技</span>
          <span>🟥 P2: ←→移动 ↓格挡 1普攻 2一技 3二技 4三技</span>
        </div>

        <div class="game-over-overlay" v-if="gameOver" @click.stop>
          <div class="game-over-text" :class="winner === 1 ? 'winner-p1' : 'winner-p2'">
            {{ winner === 1 ? '🏆 圣骑士胜利！' : '🏆 暗影武士胜利！' }}
          </div>
          <div style="color: #ccc; font-size: 1em; letter-spacing: 2px">
            {{ winner === 1 ? '玩家1' : '玩家2' }} 赢得了第 {{ currentRound }} 局
          </div>
          <div style="color: #ffd740; font-size: 0.9em">
            总比分: 玩家1 {{ score.p1 }} - {{ score.p2 }} 玩家2
          </div>
          <button class="restart-btn" @click.stop="restartGame">🔄 再来一局</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted, onUnmounted } from 'vue'

const ARENA_LEFT = 40
const ARENA_RIGHT = 880
const MIN_DISTANCE = 65
const MOVE_SPEED = 320
const ATTACK_RANGE_NORMAL = 85
const ATTACK_RANGE_SKILL1 = 105
const ATTACK_RANGE_SKILL2 = 130
const ATTACK_RANGE_SKILL3 = 250
const COMBO_TIMEOUT = 1.6
const PERFECT_BLOCK_WINDOW = 0.28
const CRIT_CHANCE = 0.16
const CRIT_MULTIPLIER = 1.75

const gameWrapper = ref(null)
const arenaContainer = ref(null)
const gameOver = ref(false)
const winner = ref(0)
const currentRound = ref(1)
const score = reactive({ p1: 0, p2: 0 })

const player1 = reactive({
  x: 200,
  hp: 100,
  maxHp: 100,
  energy: 0,
  maxEnergy: 100,
  isAttacking: false,
  isBlocking: false,
  isPerfectBlock: false,
  isShaking: false,
  comboCount: 0,
  lastHitTime: 0,
  blockStartTime: 0,
  normalCd: 0,
  normalMaxCd: 0.45,
  skill1Cd: 0,
  skill1MaxCd: 3.2,
  skill1Cost: 18,
  skill2Cd: 0,
  skill2MaxCd: 5.5,
  skill2Cost: 30,
  skill3Cd: 0,
  skill3MaxCd: 11,
  skill3Cost: 55,
  moveLeft: false,
  moveRight: false,
  blockHeld: false,
  stunnedUntil: 0,
})

const player2 = reactive({
  x: 700,
  hp: 100,
  maxHp: 100,
  energy: 0,
  maxEnergy: 100,
  isAttacking: false,
  isBlocking: false,
  isPerfectBlock: false,
  isShaking: false,
  comboCount: 0,
  lastHitTime: 0,
  blockStartTime: 0,
  normalCd: 0,
  normalMaxCd: 0.45,
  skill1Cd: 0,
  skill1MaxCd: 3.2,
  skill1Cost: 18,
  skill2Cd: 0,
  skill2MaxCd: 5.5,
  skill2Cost: 30,
  skill3Cd: 0,
  skill3MaxCd: 11,
  skill3Cost: 55,
  moveLeft: false,
  moveRight: false,
  blockHeld: false,
  stunnedUntil: 0,
})

const particles = ref([])
const damageNumbers = ref([])
const specialTexts = ref([])
let particleIdCounter = 0
let dmgIdCounter = 0
let specialIdCounter = 0
let lastFrameTime = 0
let animFrameId = null
let gameActive = true

// ====================== 工具函数（全部提前定义，不会再报错）======================
function clamp(val, min, max) {
  return Math.max(min, Math.min(max, val))
}

function distanceBetween() {
  return Math.abs(player1.x - player2.x)
}

function isInRange(attacker, defender, range) {
  if (attacker === player1) {
    return defender.x - attacker.x < range && defender.x - attacker.x > 0
  } else {
    return attacker.x - defender.x < range && attacker.x - defender.x > 0
  }
}

function spawnParticles(x, y, count, color) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2
    const speed = 40 + Math.random() * 120
    particles.value.push({
      id: ++particleIdCounter,
      x,
      y,
      size: 3 + Math.random() * 7,
      color,
      dx: Math.cos(angle) * speed,
      dy: Math.sin(angle) * speed - 30,
      duration: 0.4 + Math.random() * 0.5,
    })
  }
  if (particles.value.length > 40) {
    particles.value.splice(0, particles.value.length - 40)
  }
}

function spawnDamageNumber(x, y, amount, isCrit = false, isHeal = false) {
  const color = isHeal ? '#4cff4c' : isCrit ? '#ffd740' : '#ffffff'
  const text = isHeal ? `+${amount}` : isCrit ? `${amount} 暴击!` : `${amount}`
  damageNumbers.value.push({
    id: ++dmgIdCounter,
    x,
    y: y - 30,
    text,
    color,
    isCrit,
  })
  if (damageNumbers.value.length > 15) {
    damageNumbers.value.splice(0, damageNumbers.value.length - 15)
  }
}

function spawnSpecialText(x, y, text, color) {
  specialTexts.value.push({
    id: ++specialIdCounter,
    x,
    y,
    text,
    color,
  })
  if (specialTexts.value.length > 8) {
    specialTexts.value.splice(0, specialTexts.value.length - 8)
  }
}

function cleanupEffects() {
  const now = Date.now()
  particles.value = particles.value.filter((p) => now - p.id < 1200)
  damageNumbers.value = damageNumbers.value.filter((d) => now - d.id < 1500)
  specialTexts.value = specialTexts.value.filter((s) => now - s.id < 1800)
}

function enforceMinDistance() {
  const dist = distanceBetween()
  if (dist < MIN_DISTANCE) {
    const midX = (player1.x + player2.x) / 2
    player1.x = midX - MIN_DISTANCE / 2
    player2.x = midX + MIN_DISTANCE / 2
    player1.x = clamp(player1.x, ARENA_LEFT, ARENA_RIGHT)
    player2.x = clamp(player2.x, ARENA_LEFT, ARENA_RIGHT)
  }
}

function triggerScreenShake() {
  const el = arenaContainer.value
  if (!el) return
  el.style.transform = 'translateX(-6px)'
  setTimeout(() => {
    el.style.transform = 'translateX(5px)'
  }, 50)
  setTimeout(() => {
    el.style.transform = 'translateX(-3px)'
  }, 100)
  setTimeout(() => {
    el.style.transform = 'translateX(2px)'
  }, 150)
  setTimeout(() => {
    el.style.transform = 'translateX(0)'
  }, 200)
}

// ====================== 核心战斗逻辑 ======================
function dealDamage(attacker, defender, baseDamage, isSkill = false, skillLevel = 0) {
  let range = ATTACK_RANGE_NORMAL
  if (skillLevel === 1) range = ATTACK_RANGE_SKILL1
  if (skillLevel === 2) range = ATTACK_RANGE_SKILL2
  if (skillLevel === 3) range = ATTACK_RANGE_SKILL3

  if (!isInRange(attacker, defender, range)) {
    spawnParticles(attacker === player1 ? attacker.x + 35 : attacker.x - 35, 110, 5, '#888888')
    return false
  }

  const now = performance.now() / 1000
  const blockDuration = now - defender.blockStartTime
  const isPerfectBlock = defender.isBlocking && blockDuration < PERFECT_BLOCK_WINDOW

  if (isPerfectBlock) {
    defender.isPerfectBlock = true
    setTimeout(() => {
      defender.isPerfectBlock = false
    }, 400)
    const reflectDmg = Math.floor(baseDamage * 0.25)
    attacker.hp -= reflectDmg
    attacker.isShaking = true
    setTimeout(() => {
      attacker.isShaking = false
    }, 250)
    spawnDamageNumber(attacker.x, 80, reflectDmg, false, false)
    spawnSpecialText(defender.x, 60, '完美格挡!', '#ffffff')
    spawnParticles(defender.x, 100, 20, '#ffffff')
    attacker.comboCount = 0
    attacker.lastHitTime = 0
    return 'perfect_blocked'
  }

  let finalDamage = baseDamage
  if (defender.isBlocking && !isPerfectBlock) {
    finalDamage = Math.floor(baseDamage * 0.4)
    spawnSpecialText(defender.x, 60, '格挡', '#88ccff')
    spawnParticles(defender.x, 100, 8, '#aaddff')
  }

  let isCrit = false
  if (!defender.isBlocking) {
    if (Math.random() < CRIT_CHANCE) {
      finalDamage = Math.floor(finalDamage * CRIT_MULTIPLIER)
      isCrit = true
    }
  }

  if (attacker.comboCount > 1 && !defender.isBlocking) {
    const comboBonus = Math.min(attacker.comboCount * 0.04, 0.25)
    finalDamage = Math.floor(finalDamage * (1 + comboBonus))
  }

  defender.hp -= finalDamage
  defender.isShaking = true
  setTimeout(() => {
    defender.isShaking = false
  }, 250)

  const knockback = isSkill ? (skillLevel === 3 ? 40 : skillLevel === 2 ? 20 : 10) : 4
  if (attacker === player1) {
    defender.x = clamp(defender.x + knockback, ARENA_LEFT, ARENA_RIGHT)
  } else {
    defender.x = clamp(defender.x - knockback, ARENA_LEFT, ARENA_RIGHT)
  }
  enforceMinDistance()

  defender.energy = Math.min(defender.maxEnergy, defender.energy + Math.floor(finalDamage * 0.35))

  const hitX = defender.x
  const hitY = 95 + Math.random() * 25
  const particleColor = isCrit ? '#ffd740' : attacker === player1 ? '#ff8888' : '#ff6666'
  spawnParticles(hitX, hitY, isCrit ? 18 : 10, particleColor)
  spawnDamageNumber(hitX, hitY - 10, finalDamage, isCrit)

  if (isCrit) {
    spawnSpecialText(hitX, hitY - 40, '💥暴击!', '#ffd740')
  }

  const now2 = performance.now() / 1000
  if (now2 - attacker.lastHitTime < COMBO_TIMEOUT) {
    attacker.comboCount++
  } else {
    attacker.comboCount = 1
  }
  attacker.lastHitTime = now2

  if (attacker.comboCount >= 5) {
    spawnSpecialText(attacker.x, 40, `${attacker.comboCount} 连击!`, '#ff9800')
  }

  return true
}

function performAttack(attacker, defender, skillLevel = 0) {
  const now = performance.now() / 1000
  if (attacker.stunnedUntil > now) return

  let damage = 0
  let energyGain = 0
  let cdRef = null
  let maxCd = 0
  let cost = 0

  if (skillLevel === 0) {
    if (attacker.normalCd > 0) return
    damage = 13 + Math.floor(Math.random() * 10)
    energyGain = 14
    cdRef = 'normalCd'
    maxCd = attacker.normalMaxCd
    cost = 0
  } else if (skillLevel === 1) {
    if (attacker.skill1Cd > 0) return
    if (attacker.energy < attacker.skill1Cost) return
    damage = 22 + Math.floor(Math.random() * 14)
    energyGain = 8
    cdRef = 'skill1Cd'
    maxCd = attacker.skill1MaxCd
    cost = attacker.skill1Cost
  } else if (skillLevel === 2) {
    if (attacker.skill2Cd > 0) return
    if (attacker.energy < attacker.skill2Cost) return
    damage = 28 + Math.floor(Math.random() * 18)
    energyGain = 5
    cdRef = 'skill2Cd'
    maxCd = attacker.skill2MaxCd
    cost = attacker.skill2Cost
  } else if (skillLevel === 3) {
    if (attacker.skill3Cd > 0) return
    if (attacker.energy < attacker.skill3Cost) return
    damage = 38 + Math.floor(Math.random() * 25)
    energyGain = 0
    cdRef = 'skill3Cd'
    maxCd = attacker.skill3MaxCd
    cost = attacker.skill3Cost
  }

  attacker.energy -= cost
  attacker[cdRef] = maxCd

  attacker.isAttacking = true
  setTimeout(() => {
    attacker.isAttacking = false
  }, 300)

  const result = dealDamage(attacker, defender, damage, skillLevel > 0, skillLevel)

  if (result === true || result === false) {
    const actualGain = result === true ? energyGain : Math.floor(energyGain * 0.3)
    attacker.energy = Math.min(attacker.maxEnergy, attacker.energy + actualGain)
  }

  if (skillLevel === 3 && result === true) {
    defender.stunnedUntil = performance.now() / 1000 + 0.8
    spawnSpecialText(defender.x, 50, '⚡眩晕!', '#ff6d00')
    triggerScreenShake()
  }
}

// ====================== 游戏循环 ======================
function updatePlayer(player, opponent, dt, now) {
  if (player.stunnedUntil > now) {
    player.isBlocking = false
    player.blockHeld = false
    return
  }

  if (player.moveLeft && !player.moveRight) {
    player.x -= MOVE_SPEED * dt
  } else if (player.moveRight && !player.moveLeft) {
    player.x += MOVE_SPEED * dt
  }
  player.x = clamp(player.x, ARENA_LEFT, ARENA_RIGHT)

  player.isBlocking = player.blockHeld && player.stunnedUntil <= now
  if (player.blockHeld && !player.isBlocking && player.stunnedUntil <= now) {
    player.blockStartTime = now
  }
  if (!player.blockHeld) {
    player.isBlocking = false
    player.isPerfectBlock = false
  }

  if (player.normalCd > 0) player.normalCd = Math.max(0, player.normalCd - dt)
  if (player.skill1Cd > 0) player.skill1Cd = Math.max(0, player.skill1Cd - dt)
  if (player.skill2Cd > 0) player.skill2Cd = Math.max(0, player.skill2Cd - dt)
  if (player.skill3Cd > 0) player.skill3Cd = Math.max(0, player.skill3Cd - dt)

  player.energy = Math.min(player.maxEnergy, player.energy + 1.8 * dt)

  if (player.lastHitTime > 0 && now - player.lastHitTime > COMBO_TIMEOUT) {
    player.comboCount = 0
  }
}

function checkGameOver() {
  if (player1.hp <= 0 || player2.hp <= 0) {
    gameActive = false
    gameOver.value = true
    if (player1.hp <= 0 && player2.hp <= 0) {
      winner.value = player1.hp > player2.hp ? 1 : 2
    } else if (player1.hp <= 0) {
      winner.value = 2
      score.p2++
    } else {
      winner.value = 1
      score.p1++
    }
    player1.hp = Math.max(0, player1.hp)
    player2.hp = Math.max(0, player2.hp)
  }
}

function gameLoop(timestamp) {
  if (!gameActive) {
    animFrameId = requestAnimationFrame(gameLoop)
    return
  }
  if (lastFrameTime === 0) lastFrameTime = timestamp
  let dt = (timestamp - lastFrameTime) / 1000
  if (dt <= 0) dt = 0.016
  if (dt > 0.1) dt = 0.1
  lastFrameTime = timestamp

  const now = timestamp / 1000

  updatePlayer(player1, player2, dt, now)
  updatePlayer(player2, player1, dt, now)

  enforceMinDistance()
  player1.x = clamp(player1.x, ARENA_LEFT, ARENA_RIGHT - MIN_DISTANCE)
  player2.x = clamp(player2.x, ARENA_LEFT + MIN_DISTANCE, ARENA_RIGHT)
  enforceMinDistance()

  checkGameOver()
  cleanupEffects()

  animFrameId = requestAnimationFrame(gameLoop)
}

// ====================== 重启游戏 ======================
function restartGame() {
  player1.hp = player1.maxHp
  player2.hp = player2.maxHp
  player1.energy = 0
  player2.energy = 0
  player1.x = 200
  player2.x = 700
  player1.normalCd = 0
  player1.skill1Cd = 0
  player1.skill2Cd = 0
  player1.skill3Cd = 0
  player2.normalCd = 0
  player2.skill1Cd = 0
  player2.skill2Cd = 0
  player2.skill3Cd = 0
  player1.comboCount = 0
  player2.comboCount = 0
  player1.lastHitTime = 0
  player2.lastHitTime = 0
  player1.isAttacking = false
  player2.isAttacking = false
  player1.isBlocking = false
  player2.isBlocking = false
  player1.isPerfectBlock = false
  player2.isPerfectBlock = false
  player1.isShaking = false
  player2.isShaking = false
  player1.blockHeld = false
  player2.blockHeld = false
  player1.stunnedUntil = 0
  player2.stunnedUntil = 0
  player1.moveLeft = false
  player1.moveRight = false
  player2.moveLeft = false
  player2.moveRight = false
  particles.value = []
  damageNumbers.value = []
  specialTexts.value = []
  gameOver.value = false
  winner.value = 0
  gameActive = true
  lastFrameTime = 0

  if (score.p1 >= 2 || score.p2 >= 2) {
    score.p1 = 0
    score.p2 = 0
    currentRound.value = 1
  } else {
    currentRound.value++
  }
}

// ====================== 键盘事件 ======================
const handleKeyDown = (e) => {
  if (gameOver.value) {
    if (e.key === 'Enter' || e.key === ' ') {
      restartGame()
    }
    return
  }
  if (!gameActive) return

  const key = e.key.toLowerCase()
  const gameKeys = [
    'a',
    'd',
    's',
    'j',
    'k',
    'l',
    'i',
    'arrowleft',
    'arrowright',
    'arrowdown',
    '1',
    '2',
    '3',
    '4',
  ]
  if (gameKeys.includes(key)) e.preventDefault()

  if (key === 'a') player1.moveLeft = true
  if (key === 'd') player1.moveRight = true
  if (key === 's') {
    player1.blockHeld = true
    if (!player1.isBlocking) player1.blockStartTime = performance.now() / 1000
  }
  if (key === 'j') performAttack(player1, player2, 0)
  if (key === 'k') performAttack(player1, player2, 1)
  if (key === 'l') performAttack(player1, player2, 2)
  if (key === 'i') performAttack(player1, player2, 3)

  if (key === 'arrowleft') player2.moveLeft = true
  if (key === 'arrowright') player2.moveRight = true
  if (key === 'arrowdown') {
    player2.blockHeld = true
    if (!player2.isBlocking) player2.blockStartTime = performance.now() / 1000
  }
  if (key === '1') performAttack(player2, player1, 0)
  if (key === '2') performAttack(player2, player1, 1)
  if (key === '3') performAttack(player2, player1, 2)
  if (key === '4') performAttack(player2, player1, 3)
}

const handleKeyUp = (e) => {
  const key = e.key.toLowerCase()
  const gameKeys = ['a', 'd', 's', 'arrowleft', 'arrowright', 'arrowdown']
  if (gameKeys.includes(key)) e.preventDefault()

  if (key === 'a') player1.moveLeft = false
  if (key === 'd') player1.moveRight = false
  if (key === 's') {
    player1.blockHeld = false
    player1.isBlocking = false
  }
  if (key === 'arrowleft') player2.moveLeft = false
  if (key === 'arrowright') player2.moveRight = false
  if (key === 'arrowdown') {
    player2.blockHeld = false
    player2.isBlocking = false
  }
}

// ====================== 生命周期 ======================
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', handleKeyUp)
  animFrameId = requestAnimationFrame(gameLoop)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
  if (animFrameId) cancelAnimationFrame(animFrameId)
  gameActive = false
})
</script>

<style scoped>
:root {
  --bg: #1a1a2e;
  --arena: #16213e;
  --ground: #2d2d3d;
  --p1-color: #4da6ff;
  --p1-glow: #2b6cb0;
  --p2-color: #ff5252;
  --p2-glow: #b71c1c;
  --gold: #ffd740;
  --text: #e0e0e0;
  --ui-bg: rgba(0, 0, 0, 0.7);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  width: 100%;
  max-width: 960px;
  padding: 10px;
  margin: 0 auto;
}

body {
  background: #0a0a14;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  font-family: 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  user-select: none;
  overflow: hidden;
  background-image:
    radial-gradient(ellipse at center, #1a1a35 0%, #0a0a14 70%),
    repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(255, 255, 255, 0.008) 2px,
      rgba(255, 255, 255, 0.008) 4px
    );
}

.game-wrapper {
  position: relative;
  background: var(--arena);
  border-radius: 20px;
  overflow: hidden;
  box-shadow:
    0 0 60px rgba(30, 60, 120, 0.4),
    0 0 120px rgba(0, 0, 0, 0.6),
    inset 0 0 80px rgba(0, 0, 0, 0.3);
  border: 3px solid #2a2a45;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 24px;
  background: rgba(0, 0, 0, 0.5);
  border-bottom: 2px solid #2a2a45;
  flex-wrap: wrap;
  gap: 8px;
}

.game-header .title {
  font-size: 1.4em;
  font-weight: 900;
  letter-spacing: 3px;
  background: linear-gradient(135deg, #4da6ff, #ff5252);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.game-header .round-indicator {
  color: #ffd740;
  font-weight: 700;
  font-size: 0.95em;
  letter-spacing: 2px;
  animation: pulse-gold 2s infinite;
}

@keyframes pulse-gold {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.game-header .tips {
  color: #888;
  font-size: 0.75em;
  letter-spacing: 1px;
}

.arena-container {
  position: relative;
  width: 100%;
  height: 520px;
  background:
    radial-gradient(ellipse at 50% 70%, rgba(40, 40, 70, 0.8) 0%, transparent 60%),
    linear-gradient(180deg, #1a1a30 0%, #1e1e38 40%, #252540 100%);
  cursor: default;
  overflow: hidden;
}

.arena-ground {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 80px;
  background: linear-gradient(180deg, #3a3a50 0%, #2a2a3d 30%, #1e1e30 100%);
  border-top: 3px solid #4a4a60;
  box-shadow: inset 0 2px 20px rgba(0, 0, 0, 0.5);
}

.arena-ground::before {
  content: '';
  position: absolute;
  top: -20px;
  left: 0;
  right: 0;
  height: 20px;
  background: linear-gradient(180deg, transparent, rgba(255, 255, 255, 0.03));
  pointer-events: none;
}

.ground-lines {
  position: absolute;
  bottom: 78px;
  left: 5%;
  right: 5%;
  height: 2px;
  background: repeating-linear-gradient(
    90deg,
    rgba(255, 255, 255, 0.15) 0px,
    rgba(255, 255, 255, 0.15) 30px,
    transparent 30px,
    transparent 80px
  );
  pointer-events: none;
}

.spotlight {
  position: absolute;
  top: -100px;
  left: 50%;
  transform: translateX(-50%);
  width: 500px;
  height: 300px;
  background: radial-gradient(ellipse, rgba(255, 255, 200, 0.06) 0%, transparent 70%);
  pointer-events: none;
  border-radius: 50%;
}

.particles-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 10;
}

.particle {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  animation: particle-fly 0.6s ease-out forwards;
}

@keyframes particle-fly {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translate(var(--dx), var(--dy)) scale(0);
    opacity: 0;
  }
}

.damage-numbers-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 15;
}

.damage-number {
  position: absolute;
  font-weight: 900;
  font-size: 1.6em;
  pointer-events: none;
  animation: dmg-float 1s ease-out forwards;
  text-shadow:
    0 0 8px currentColor,
    0 0 16px currentColor;
  white-space: nowrap;
}

.damage-number.crit {
  font-size: 2.2em;
  animation: dmg-float-crit 1s ease-out forwards;
}

.damage-number.heal {
  color: #4cff4c !important;
}

@keyframes dmg-float {
  0% {
    transform: translateY(0) scale(0.5);
    opacity: 1;
  }
  30% {
    transform: translateY(-30px) scale(1.3);
    opacity: 1;
  }
  100% {
    transform: translateY(-90px) scale(0.8);
    opacity: 0;
  }
}

@keyframes dmg-float-crit {
  0% {
    transform: translateY(0) scale(0.3) rotate(-10deg);
    opacity: 1;
  }
  30% {
    transform: translateY(-35px) scale(1.6) rotate(5deg);
    opacity: 1;
  }
  100% {
    transform: translateY(-100px) scale(1) rotate(0deg);
    opacity: 0;
  }
}

.special-text-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 20;
}

.special-text {
  position: absolute;
  font-weight: 900;
  font-size: 1.3em;
  letter-spacing: 3px;
  pointer-events: none;
  animation: special-pop 1.2s ease-out forwards;
}

@keyframes special-pop {
  0% {
    transform: translate(-50%, -50%) scale(0.2);
    opacity: 1;
  }
  40% {
    transform: translate(-50%, -50%) scale(1.6);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(1.8);
    opacity: 0;
  }
}

.player-character {
  position: absolute;
  bottom: 75px;
  z-index: 5;
  transition: left 0.05s linear;
  pointer-events: none;
}

.player-character.shaking {
  animation: shake 0.25s ease-out;
}

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  20% {
    transform: translateX(-8px);
  }
  40% {
    transform: translateX(8px);
  }
  60% {
    transform: translateX(-6px);
  }
  80% {
    transform: translateX(4px);
  }
}

.player-character.attacking {
  animation: attack-lunge 0.3s ease-out;
}

@keyframes attack-lunge {
  0% {
    transform: translateX(0) scale(1);
  }
  40% {
    transform: translateX(var(--lunge-dir, 20px)) scale(1.1);
  }
  100% {
    transform: translateX(0) scale(1);
  }
}

.player-character.blocking {
  filter: brightness(1.3) drop-shadow(0 0 12px cyan);
}

.player-character.perfect-block {
  filter: brightness(1.8) drop-shadow(0 0 25px #fff) drop-shadow(0 0 40px gold);
  animation: perfect-flash 0.5s ease-out;
}

@keyframes perfect-flash {
  0%,
  100% {
    filter: brightness(1.3);
  }
  50% {
    filter: brightness(2.5) drop-shadow(0 0 40px #fff);
  }
}

.character-sprite {
  position: relative;
  width: 60px;
  height: 90px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.char-head {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  position: relative;
  z-index: 2;
  border: 3px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

.p1 .char-head {
  background: linear-gradient(180deg, #6db9ff 0%, #3d8fd9 100%);
  border-color: #7cc4ff;
}

.p2 .char-head {
  background: linear-gradient(180deg, #ff7070 0%, #d32f2f 100%);
  border-color: #ff8888;
}

.char-head::after {
  content: '';
  position: absolute;
  top: 6px;
  left: 50%;
  transform: translateX(-50%);
  width: 14px;
  height: 6px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 3px;
}

.char-body {
  width: 44px;
  height: 38px;
  border-radius: 8px;
  margin-top: -4px;
  position: relative;
  z-index: 1;
  border: 2px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.4);
}

.p1 .char-body {
  background: linear-gradient(180deg, #5a9fd4 0%, #3b7db8 100%);
  border-color: #6aafdf;
}

.p2 .char-body {
  background: linear-gradient(180deg, #e05555 0%, #b52525 100%);
  border-color: #e56565;
}

.char-weapon {
  position: absolute;
  z-index: 3;
  border-radius: 3px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
  transition: transform 0.1s;
}

.p1 .char-weapon {
  right: -28px;
  top: 30px;
  width: 32px;
  height: 7px;
  background: linear-gradient(90deg, #c0c0c0, #e8e8e8, #a0a0a0);
  border-radius: 4px 1px 1px 4px;
  transform-origin: left center;
}

.p2 .char-weapon {
  left: -28px;
  top: 30px;
  width: 32px;
  height: 7px;
  background: linear-gradient(270deg, #c0c0c0, #e8e8e8, #a0a0a0);
  border-radius: 1px 4px 4px 1px;
  transform-origin: right center;
}

.player-character.attacking .char-weapon {
  animation: weapon-swing 0.3s ease-out;
}

@keyframes weapon-swing {
  0% {
    transform: rotate(0);
  }
  50% {
    transform: rotate(var(--swing-rot, 30deg));
  }
  100% {
    transform: rotate(0);
  }
}

.char-legs {
  display: flex;
  gap: 6px;
  margin-top: -2px;
  z-index: 0;
}

.char-leg {
  width: 14px;
  height: 20px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
}

.p1 .char-leg {
  background: #3b6d9e;
}
.p2 .char-leg {
  background: #9e3535;
}

.player-hud {
  position: absolute;
  z-index: 8;
  pointer-events: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  transition: left 0.05s linear;
}

.hud-name {
  font-weight: 700;
  font-size: 0.8em;
  letter-spacing: 2px;
  text-shadow: 0 0 8px currentColor;
}

.p1-hud .hud-name {
  color: #7cc4ff;
}
.p2-hud .hud-name {
  color: #ff8888;
}

.hp-bar-outer {
  width: 100px;
  height: 10px;
  background: #1a1a1a;
  border-radius: 5px;
  overflow: hidden;
  border: 2px solid #444;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.hp-bar-inner {
  height: 100%;
  border-radius: 3px;
  transition:
    width 0.3s ease-out,
    background 0.3s;
}

.p1-hud .hp-bar-inner {
  background: linear-gradient(90deg, #3a8fd4, #5db8fe);
  box-shadow: 0 0 10px rgba(77, 166, 255, 0.6);
}

.p2-hud .hp-bar-inner {
  background: linear-gradient(90deg, #d43a3a, #fe5d5d);
  box-shadow: 0 0 10px rgba(255, 82, 82, 0.6);
}

.hp-text {
  font-size: 0.7em;
  font-weight: 700;
  color: #ccc;
  text-shadow: 0 0 4px #000;
  letter-spacing: 1px;
}

.energy-bar-outer {
  width: 80px;
  height: 5px;
  background: #1a1a1a;
  border-radius: 3px;
  overflow: hidden;
  border: 1px solid #333;
}

.energy-bar-inner {
  height: 100%;
  border-radius: 2px;
  transition: width 0.2s;
  background: linear-gradient(90deg, #b8860b, #ffd740, #ffea80);
  box-shadow: 0 0 6px rgba(255, 215, 64, 0.5);
}

.skill-indicators {
  display: flex;
  gap: 6px;
  margin-top: 2px;
}

.skill-icon {
  width: 28px;
  height: 28px;
  border-radius: 6px;
  background: #2a2a3a;
  border: 2px solid #555;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 0.7em;
  color: #aaa;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.4);
}

.skill-icon.ready {
  border-color: #ffd740;
  box-shadow: 0 0 12px rgba(255, 215, 64, 0.5);
  animation: ready-glow 1.5s infinite;
}

@keyframes ready-glow {
  0%,
  100% {
    box-shadow: 0 0 8px rgba(255, 215, 64, 0.4);
  }
  50% {
    box-shadow: 0 0 20px rgba(255, 215, 64, 0.8);
  }
}

.skill-icon .cooldown-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background: rgba(0, 0, 0, 0.7);
  transition: height 0.3s linear;
  pointer-events: none;
}

.skill-icon .cd-text {
  position: absolute;
  font-size: 0.65em;
  font-weight: 700;
  color: #fff;
  z-index: 2;
  pointer-events: none;
  text-shadow: 0 0 4px #000;
}

.skill-icon.ultimate {
  border-color: #ff6d00;
  background: #1a0a00;
  color: #ff9800;
}

.skill-icon.ultimate.ready {
  border-color: #ff9800;
  box-shadow: 0 0 20px rgba(255, 152, 0, 0.7);
  animation: ult-glow 0.8s infinite;
}

@keyframes ult-glow {
  0%,
  100% {
    box-shadow: 0 0 12px rgba(255, 152, 0, 0.5);
  }
  50% {
    box-shadow:
      0 0 28px rgba(255, 152, 0, 1),
      0 0 50px rgba(255, 100, 0, 0.6);
  }
}

.controls-hint {
  position: absolute;
  bottom: 8px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 25;
  display: flex;
  gap: 30px;
  pointer-events: none;
  font-size: 0.7em;
  color: rgba(255, 255, 255, 0.5);
  letter-spacing: 1px;
  background: rgba(0, 0, 0, 0.5);
  padding: 6px 16px;
  border-radius: 20px;
}

.game-over-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.75);
  z-index: 30;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  pointer-events: all;
}

.game-over-text {
  font-size: 3em;
  font-weight: 900;
  letter-spacing: 4px;
  animation: result-bounce 0.6s ease-out;
}

@keyframes result-bounce {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  60% {
    transform: scale(1.2);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.winner-p1 {
  color: #5db8fe;
  text-shadow:
    0 0 30px #4da6ff,
    0 0 60px #2b6cb0;
}

.winner-p2 {
  color: #fe5d5d;
  text-shadow:
    0 0 30px #ff5252,
    0 0 60px #b71c1c;
}

.restart-btn {
  padding: 12px 36px;
  font-size: 1.1em;
  font-weight: 700;
  letter-spacing: 2px;
  border: 2px solid #ffd740;
  background: rgba(0, 0, 0, 0.6);
  color: #ffd740;
  border-radius: 30px;
  cursor: pointer;
  pointer-events: all;
  transition: all 0.3s;
  text-transform: uppercase;
}

.restart-btn:hover {
  background: #ffd740;
  color: #000;
  box-shadow: 0 0 30px rgba(255, 215, 64, 0.6);
  transform: scale(1.05);
}

@media (max-width: 700px) {
  .arena-container {
    height: 400px;
  }
  .player-character {
    bottom: 55px;
  }
  .char-head {
    width: 28px;
    height: 28px;
  }
  .char-body {
    width: 34px;
    height: 28px;
  }
  .char-leg {
    width: 10px;
    height: 14px;
  }
  .char-weapon {
    width: 22px;
    height: 5px;
  }
  .p1 .char-weapon {
    right: -20px;
    top: 22px;
  }
  .p2 .char-weapon {
    left: -20px;
    top: 22px;
  }
  .hp-bar-outer {
    width: 70px;
    height: 8px;
  }
  .skill-icon {
    width: 22px;
    height: 22px;
    font-size: 0.6em;
  }
  .controls-hint {
    font-size: 0.6em;
    gap: 10px;
    padding: 4px 10px;
  }
}
</style>
