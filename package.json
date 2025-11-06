import React, { useState, useEffect, useRef, useCallback } from 'react';
import { Target, Zap, Shield, Skull, Trophy, Clock } from 'lucide-react';

export default function BattlefrontRoyale() {
  const canvasRef = useRef(null);
  const [gameState, setGameState] = useState('menu');
  const [stats, setStats] = useState({ kills: 0, time: 0, energy: 100 });
  const [towers, setTowers] = useState({ ally: [100, 100, 100], enemy: [100, 100, 100] });
  const gameRef = useRef(null);

  const initGame = useCallback(() => {
    const mapSize = 200;

    gameRef.current = {
      player: {
        x: 10, y: 1.7, z: mapSize / 2, vx: 0, vy: 0, vz: 0,
        health: 100, ammo: 30, maxAmmo: 30, reloading: false, reloadTime: 0, alive: true, onGround: true
      },
      camera: { yaw: 0, pitch: 0 },
      keys: {},
      mouse: { down: false },

      towers: {
        ally: [
          { x: 5, z: 30, hp: 100, maxHp: 100, lane: 'left' },
          { x: 5, z: 100, hp: 100, maxHp: 100, lane: 'mid' },
          { x: 5, z: 170, hp: 100, maxHp: 100, lane: 'right' }
        ],
        enemy: [
          { x: mapSize - 5, z: 30, hp: 100, maxHp: 100, lane: 'left' },
          { x: mapSize - 5, z: 100, hp: 100, maxHp: 100, lane: 'mid' },
          { x: mapSize - 5, z: 170, hp: 100, maxHp: 100, lane: 'right' }
        ]
      },

      allies: [],
      enemies: [],
      bullets: [],
      particles: [],

      energy: 100,
      maxEnergy: 100,
      energyRegen: 0.1,

      startTime: Date.now(),
      lastSpawn: Date.now(),
      spawnInterval: 3000,

      kills: 0
    };

    // Spawn initial units
    for (let i = 0; i < 15; i++) {
      const lane = i % 3;
      const laneZ = lane === 0 ? 30 : lane === 1 ? 100 : 170;

      gameRef.current.allies.push({
        x: 20 + Math.random() * 10, y: 0, z: laneZ + (Math.random() - 0.5) * 20,
        vx: 0, vz: 0, hp: 50, maxHp: 50, targetX: mapSize - 20, targetZ: laneZ,
        lane: lane, team: 'ally', shootCooldown: 0
      });

      gameRef.current.enemies.push({
        x: mapSize - 20 - Math.random() * 10, y: 0, z: laneZ + (Math.random() - 0.5) * 20,
        vx: 0, vz: 0, hp: 50, maxHp: 50, targetX: 20, targetZ: laneZ,
        lane: lane, team: 'enemy', shootCooldown: 0
      });
    }

    setStats({ kills: 0, time: 0, energy: 100 });
    setTowers({ ally: [100, 100, 100], enemy: [100, 100, 100] });
    setGameState('playing');
  }, []);

  useEffect(() => {
    if (gameState !== 'playing') return;

    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    let animationId;
    let lastTime = Date.now();

    const handleKeyDown = (e) => {
      if (gameRef.current) {
        gameRef.current.keys[e.key.toLowerCase()] = true;
        if (e.key.toLowerCase() === 'r' && !gameRef.current.player.reloading) {
          gameRef.current.player.reloading = true;
          gameRef.current.player.reloadTime = 60;
        }
      }
    };
    const handleKeyUp = (e) => {
      if (gameRef.current) gameRef.current.keys[e.key.toLowerCase()] = false;
    };
    const handleMouseDown = () => {
      if (gameRef.current) gameRef.current.mouse.down = true;
    };
    const handleMouseUp = () => {
      if (gameRef.current) gameRef.current.mouse.down = false;
    };
    const handleMouseMove = (e) => {
      if (document.pointerLockElement === canvas && gameRef.current) {
        gameRef.current.camera.yaw += e.movementX * 0.002;
        gameRef.current.camera.pitch += e.movementY * 0.002;
        gameRef.current.camera.pitch = Math.max(-1.2, Math.min(1.2, gameRef.current.camera.pitch));
      }
    };
    const handleClick = () => canvas.requestPointerLock();

    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
    canvas.addEventListener('mousedown', handleMouseDown);
    canvas.addEventListener('mouseup', handleMouseUp);
    document.addEventListener('mousemove', handleMouseMove);
    canvas.addEventListener('click', handleClick);

    const loop = () => {
      const now = Date.now();
      const dt = Math.min((now - lastTime) / 16.67, 2);
      lastTime = now;

      const game = gameRef.current;
      if (!game) return;

      const player = game.player;
      const MOVE_SPEED = 0.2;
      const GRAVITY = 0.04;

      // Energy regen + auto spawn allies
      if (now - game.lastSpawn > game.spawnInterval && game.energy >= 20) {
        const lane = Math.floor(Math.random() * 3);
        const laneZ = lane === 0 ? 30 : lane === 1 ? 100 : 170;
        game.allies.push({
          x: 20, y: 0, z: laneZ,
          vx: 0, vz: 0, hp: 50, maxHp: 50,
          targetX: 180, targetZ: laneZ,
          lane: lane, team: 'ally', shootCooldown: 0
        });
        game.lastSpawn = now;
        game.energy -= 20;
      }
      game.energy = Math.min(game.maxEnergy, game.energy + game.energyRegen * dt);

      // Player movement + shooting
      if (player.alive) {
        const forward = { x: Math.sin(game.camera.yaw), z: Math.cos(game.camera.yaw) };
        const right = { x: Math.cos(game.camera.yaw), z: -Math.sin(game.camera.yaw) };

        if (game.keys['w']) { player.vx += forward.x * 0.015; player.vz += forward.z * 0.015; }
        if (game.keys['s']) { player.vx -= forward.x * 0.015; player.vz -= forward.z * 0.015; }
        if (game.keys['a']) { player.vx += right.x * 0.015; player.vz += right.z * 0.015; }
        if (game.keys['d']) { player.vx -= right.x * 0.015; player.vz -= right.z * 0.015; }
        if (game.keys[' '] && player.onGround) { player.vy = 0.4; player.onGround = false; }

        const speed = Math.sqrt(player.vx ** 2 + player.vz ** 2);
        if (speed > MOVE_SPEED) { player.vx = (player.vx / speed) * MOVE_SPEED; player.vz = (player.vz / speed) * MOVE_SPEED; }

        player.vy -= GRAVITY * dt;
        player.vx *= 0.9; player.vz *= 0.9;
        player.x += player.vx * dt; player.y += player.vy * dt; player.z += player.vz * dt;

        if (player.y <= 0) { player.y = 0; player.vy = 0; player.onGround = true; }

        player.x = Math.max(5, Math.min(195, player.x));
        player.z = Math.max(5, Math.min(195, player.z));

        if (player.reloading) {
          player.reloadTime -= dt;
          if (player.reloadTime <= 0) { player.reloading = false; player.ammo = player.maxAmmo; }
        }

        if (game.mouse.down && !player.reloading && player.ammo > 0) {
          if (!player.lastShot || now - player.lastShot > 100) {
            player.ammo--;
            player.lastShot = now;

            const dir = {
              x: Math.sin(game.camera.yaw) * Math.cos(game.camera.pitch),
              y: -Math.sin(game.camera.pitch),
              z: Math.cos(game.camera.yaw) * Math.cos(game.camera.pitch)
            };

            game.bullets.push({
              x: player.x + dir.x * 2,
              y: player.y + 1.5 + dir.y * 2,
              z: player.z + dir.z * 2,
              vx: dir.x * 2,
              vy: dir.y * 2,
              vz: dir.z * 2,
              team: 'ally',
              life: 100
            });

            if (player.ammo === 0) { player.reloading = true; player.reloadTime = 60; }
          }
        }
      }

      // UPDATE AND RENDER LOGIC OMITTED FOR BREVITY – copy your previous bullets, units, towers render here

      setStats({
        kills: game.kills,
        time: Math.floor((now - game.startTime) / 1000),
        energy: Math.floor(game.energy)
      });

      setTowers({
        ally: game.towers.ally.map(t => Math
