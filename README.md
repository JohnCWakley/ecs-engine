# ecs-engine

Welcome! This project is an attempt to create an ECS framework that has an easy to use functional programing interface. The design idea is that the programming flow is easy to understand, and easy to work with.

### Installation:
* npm: `$ npm install @n3rdw1z4rd/ecs-engine`
* yarn: `$ yarn install @n3rdw1z4rd/ecs-engine`

### Usage:
The code below can be found in `src/test/index.ts`, as well as it's dependencies:
```typescript
// src/test/index.ts

import { Engine, Entity } from '..';
import { CanvasRenderer } from './canvas-renderer';
import { StatsDiv } from './stats-div';

const renderer: CanvasRenderer = new CanvasRenderer();
renderer.appendTo(document.body);

const statsDiv: StatsDiv = new StatsDiv();

const engine: Engine = Engine.instance;

engine
    .createComponent('Position', {
        x: renderer.width / 2,
        y: renderer.height / 2,
    })
    .createComponent('Velocity', {
        x: (): number => (engine.rng.nextf * 4 - 2),
        y: (): number => (engine.rng.nextf * 4 - 2),
    })
    .createComponent('Appearance', {
        color: () => engine.rng.choose(['red', 'green', 'blue', 'yellow']),
        size: 2,
    })
    .includeAsDefaultComponents('Position', 'Velocity', 'Appearance')
    .createSystem('Move', 'Position', 'Velocity', (entity: Entity, { Position, Velocity }) => {
        if (Position.x + Velocity.x > renderer.width || Position.x + Velocity.x < 0) {
            Velocity.x = -Velocity.x;
        }

        if (Position.y + Velocity.y > renderer.height || Position.y + Velocity.y < 0) {
            Velocity.y = -Velocity.y;
        }

        Position.x += Velocity.x;
        Position.y += Velocity.y;
    })
    .createSystem('Draw', 'Position', 'Appearance', (entity: Entity, { Position, Appearance }) => {
        renderer.drawCircle(Position.x, Position.y, Appearance.color, Appearance.size);
    })
    .createEntities(1000)
    .onBeforeRun(() => {
        engine.log.debug('entities:', engine.entities);
    })
    .beforeTick(() => {
        renderer.clear();
        renderer.resize();
    })
    .afterTick(() => {
        statsDiv.update(engine);
    })
    .run();
```

### Screenshot:
![image](https://github.com/JohnCWakley/ecs-engine/assets/33690133/d6e07110-33f6-4b87-a569-6239e540affe)