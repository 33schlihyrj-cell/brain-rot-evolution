```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D Brainrot Chaos</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    overflow: hidden;
    background: #000;
    font-family: Arial, sans-serif;
    color: white;
}

canvas {
    display: block;
}

#hud {
    position: fixed;
    top: 12px;
    left: 12px;
    z-index: 20;
    font-size: 16px;
    line-height: 1.5;
    font-weight: bold;
    text-shadow: 2px 2px 4px #000;
    pointer-events: none;
}

#message {
    position: fixed;
    top: 55px;
    width: 100%;
    z-index: 30;
    text-align: center;
    font-size: 34px;
    font-weight: bold;
    text-shadow: 3px 3px 5px #000;
    pointer-events: none;
}

#crosshair {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    z-index: 25;
    font-size: 28px;
    pointer-events: none;
}

#bossBar {
    display: none;
    position: fixed;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    width: 500px;
    z-index: 40;
}

#bossName {
    text-align: center;
    font-weight: bold;
    font-size: 20px;
    text-shadow: 2px 2px 4px black;
}

#bossOuter {
    width: 100%;
    height: 20px;
    background: #222;
    border: 2px solid white;
}

#bossInner {
    width: 100%;
    height: 100%;
    background: red;
}

#help {
    position: fixed;
    bottom: 10px;
    left: 10px;
    color: #ddd;
    font-size: 13px;
    z-index: 20;
    text-shadow: 2px 2px 3px black;
}

#menu {
    display: none;
    position: fixed;
    inset: 0;
    z-index: 100;
    background: rgba(0,0,0,.85);
    align-items: center;
    justify-content: center;
}

.panel {
    width: 650px;
    max-height: 85vh;
    overflow-y: auto;
    background: #181818;
    border: 3px solid white;
    border-radius: 15px;
    padding: 25px;
    text-align: center;
}

button {
    color: white;
    background: #333;
    border: 2px solid #777;
    padding: 12px 18px;
    margin: 6px;
    border-radius: 8px;
    cursor: pointer;
    font-weight: bold;
}

button:hover {
    background: #555;
}

.item {
    background: #252525;
    border: 1px solid #555;
    border-radius: 8px;
    padding: 10px;
    margin: 8px;
}
</style>
</head>

<body>

<div id="hud"></div>
<div id="message"></div>
<div id="crosshair">+</div>

<div id="bossBar">
    <div id="bossName">BOSS</div>
    <div id="bossOuter">
        <div id="bossInner"></div>
    </div>
</div>

<div id="help">
    WASD = Move |
    Mouse = Look |
    Click = Shoot |
    SPACE = Dash |
    E = Interact |
    I = Inventory |
    P = Pause |
    1/2/3 = Weapons
</div>

<div id="menu">
    <div class="panel" id="menuContent"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* =========================================================
   3D BRAINROT CHAOS
   ========================================================= */

let scene;
let camera;
let renderer;
let clock;

let player;

let keys = {};
let mouse = {
    down: false,
    x: 0,
    y: 0
};

let gameRunning = true;
let paused = false;

let score = 0;
let coins = 100;
let xp = 0;
let level = 1;

let wave = 1;
let enemiesKilled = 0;
let combo = 0;
let bestCombo = 0;

let shake = 0;

let bullets = [];
let enemies = [];
let powerups = [];
let loot = [];
let particles = [];
let npcs = [];

let boss = null;

let messageTimer = 0;

let currentWeapon = "pistol";

let inventory = {
    potions: 2,
    bombs: 1,
    crystals: 0,
    braincells: 0
};

let weapons = {

    pistol: {
        name: "Sigma Blaster",
        damage: 25,
        fireRate: 250,
        speed: 1
    },

    shotgun: {
        name: "Rizz Shotgun",
        damage: 14,
        fireRate: 700,
        speed: 1
    },

    laser: {
        name: "Ohio Laser",
        damage: 8,
        fireRate: 80,
        speed: 1
    }
};

let playerStats = {

    hp: 100,
    maxHp: 100,

    speed: 7,

    armor: 0,

    damageMultiplier: 1,

    criticalChance: .1,

    criticalDamage: 2,

    fireCooldown: 0,

    dashCooldown: 0,

    invincible: 0
};

const brainrotLines = [

    "SKIBIDI!",
    "SIGMA MODE!",
    "RIZZ +1000!",
    "OHIO HAS AWAKENED!",
    "BRO IS COOKING!",
    "ABSOLUTE CINEMA!",
    "FANUM TAX!",
    "GYATT DETECTED!",
    "AURA +500!",
    "WHAT THE SIGMA?!",
    "BRO GOT THAT DAWG!",
    "NAH BRO 💀",
    "EMOTIONAL DAMAGE!",
    "WE ARE SO BACK!",
    "MAXIMUM BRAINROT!",
    "YOU ARE HIM!",
    "OHIO FINAL BOSS!",
    "THE GOAT HAS ARRIVED!"
];

const enemyTypes = [

    {
        name: "Skibidi Goblin",
        hp: 60,
        speed: 3,
        damage: 8,
        color: 0xff3333,
        xp: 15
    },

    {
        name: "Ohio Gremlin",
        hp: 110,
        speed: 2.5,
        damage: 12,
        color: 0xaa44ff,
        xp: 25
    },

    {
        name: "Rizz Demon",
        hp: 180,
        speed: 2,
        damage: 18,
        color: 0xff9900,
        xp: 40
    },

    {
        name: "Sigma Chud",
        hp: 300,
        speed: 1.4,
        damage: 25,
        color: 0x3399ff,
        xp: 70
    }
];

/* =========================================================
   BASIC FUNCTIONS
   ========================================================= */

function random(min, max) {
    return Math.random() * (max - min) + min;
}

function randomInt(min, max) {
    return Math.floor(
        random(min, max + 1)
    );
}

function showMessage(text, duration = 120) {

    document.getElementById(
        "message"
    ).textContent = text;

    messageTimer = duration;
}

function brainrot() {

    showMessage(
        brainrotLines[
            randomInt(
                0,
                brainrotLines.length - 1
            )
        ]
    );
}

/* =========================================================
   3D INITIALIZATION
   ========================================================= */

function init3D() {

    scene = new THREE.Scene();

    scene.background =
        new THREE.Color(0x101018);

    scene.fog =
        new THREE.Fog(
            0x101018,
            30,
            180
        );

    camera =
        new THREE.PerspectiveCamera(
            70,
            window.innerWidth /
            window.innerHeight,
            .1,
            500
        );

    renderer =
        new THREE.WebGLRenderer({
            antialias: true
        });

    renderer.setSize(
        window.innerWidth,
        window.innerHeight
    );

    renderer.shadowMap.enabled = true;

    document.body.appendChild(
        renderer.domElement
    );

    clock =
        new THREE.Clock();

    createLighting();

    createWorld();

    createPlayer();

    createNPCs();

    window.addEventListener(
        "resize",
        resize
    );
}

/* =========================================================
   LIGHTING
   ========================================================= */

function createLighting() {

    const ambient =
        new THREE.AmbientLight(
            0xffffff,
            .55
        );

    scene.add(ambient);

    const sun =
        new THREE.DirectionalLight(
            0xffffff,
            1
        );

    sun.position.set(
        30,
        60,
        20
    );

    sun.castShadow = true;

    scene.add(sun);

    const neon =
        new THREE.PointLight(
            0xff00ff,
            40,
            80
        );

    neon.position.set(
        0,
        10,
        0
    );

    scene.add(neon);
}

/* =========================================================
   WORLD
   ========================================================= */

function createWorld() {

    const groundGeometry =
        new THREE.PlaneGeometry(
            300,
            300
        );

    const groundMaterial =
        new THREE.MeshStandardMaterial({
            color: 0x202020,
            roughness: .9
        });

    const ground =
        new THREE.Mesh(
            groundGeometry,
            groundMaterial
        );

    ground.rotation.x =
        -Math.PI / 2;

    ground.receiveShadow = true;

    scene.add(ground);

    // Grid

    const grid =
        new THREE.GridHelper(
            300,
            60,
            0x444444,
            0x222222
        );

    scene.add(grid);

    // Buildings / obstacles

    for (let i = 0; i < 40; i++) {

        const height =
            random(3, 20);

        const geometry =
            new THREE.BoxGeometry(
                random(3, 10),
                height,
                random(3, 10)
            );

        const material =
            new THREE.MeshStandardMaterial({
                color:
                    Math.random() >
                    .5
                        ? 0x303040
                        : 0x402040
            });

        const building =
            new THREE.Mesh(
                geometry,
                material
            );

        building.position.set(
            random(-120, 120),
            height / 2,
            random(-120, 120)
        );

        building.castShadow = true;

        scene.add(building);
    }

    // Neon pillars

    for (let i = 0; i < 20; i++) {

        const geometry =
            new THREE.CylinderGeometry(
                1,
                1,
                12,
                12
            );

        const material =
            new THREE.MeshStandardMaterial({
                color: 0x00ffff,
                emissive: 0x004444
            });

        const pillar =
            new THREE.Mesh(
                geometry,
                material
            );

        pillar.position.set(
            random(-120, 120),
            6,
            random(-120, 120)
        );

        scene.add(pillar);
    }
}

/* =========================================================
   PLAYER
   ========================================================= */

function createPlayer() {

    const group =
        new THREE.Group();

    const bodyGeometry =
        new THREE.BoxGeometry(
            1.5,
            2,
            1.5
        );

    const bodyMaterial =
        new THREE.MeshStandardMaterial({
            color: 0x2288ff
        });

    const body =
        new THREE.Mesh(
            bodyGeometry,
            bodyMaterial
        );

    body.position.y = 1;

    body.castShadow = true;

    group.add(body);

    // Head

    const headGeometry =
        new THREE.SphereGeometry(
            .65,
            20,
            20
        );

    const headMaterial =
        new THREE.MeshStandardMaterial({
            color: 0xffccaa
        });

    const head =
        new THREE.Mesh(
            headGeometry,
            headMaterial
        );

    head.position.y = 2.3;

    head.castShadow = true;

    group.add(head);

    // Hat

    const hatGeometry =
        new THREE.CylinderGeometry(
            .7,
            .8,
            .4,
            16
        );

    const hatMaterial =
        new THREE.MeshStandardMaterial({
            color: 0x111111
        });

    const hat =
        new THREE.Mesh(
            hatGeometry,
            hatMaterial
        );

    hat.position.y = 3;

    group.add(hat);

    group.position.set(
        0,
        0,
        0
    );

    player = group;

    scene.add(player);
}

/* =========================================================
   CAMERA
   ========================================================= */

let cameraYaw = 0;
let cameraPitch = .3;

function updateCamera() {

    const distance = 14;

    const height = 8;

    const offsetX =
        Math.sin(cameraYaw) *
        distance;

    const offsetZ =
        Math.cos(cameraYaw) *
        distance;

    camera.position.x =
        player.position.x +
        offsetX;

    camera.position.y =
        player.position.y +
        height;

    camera.position.z =
        player.position.z +
        offsetZ;

    camera.lookAt(
        player.position.x,
        player.position.y + 1,
        player.position.z
    );
}

/* =========================================================
   MOUSE
   ========================================================= */

document.addEventListener(
    "mousemove",
    e => {

        if (
            document.pointerLockElement
            === renderer.domElement
        ) {

            cameraYaw -=
                e.movementX *
                .002;

            cameraPitch -=
                e.movementY *
                .002;

            cameraPitch =
                Math.max(
                    -.5,
                    Math.min(
                        1,
                        cameraPitch
                    )
                );
        }
    }
);

rendererReady = false;

/* =========================================================
   CLICK
   ========================================================= */

document.addEventListener(
    "mousedown",
    e => {

        if (e.button !== 0)
            return;

        mouse.down = true;

        if (
            document.pointerLockElement
            !== renderer.domElement
        ) {

            renderer.domElement.requestPointerLock();
        }
    }
);

document.addEventListener(
    "mouseup",
    e => {

        if (e.button === 0)
            mouse.down = false;
    }
);

/* =========================================================
   KEYBOARD
   ========================================================= */

document.addEventListener(
    "keydown",
    e => {

        keys[e.key] = true;

        if (
            [
                "ArrowUp",
                "ArrowDown",
                "ArrowLeft",
                "ArrowRight",
                " "
            ].includes(e.key)
        ) {
            e.preventDefault();
        }

        if (e.key.toLowerCase() === "p")
            togglePause();

        if (e.key.toLowerCase() === "i")
            openInventory();

        if (e.key.toLowerCase() === "e")
            interact();

        if (e.key === "1")
            currentWeapon = "pistol";

        if (e.key === "2")
            currentWeapon = "shotgun";

        if (e.key === "3")
            currentWeapon = "laser";
    }
);

document.addEventListener(
    "keyup",
    e => {
        keys[e.key] = false;
    }
);

/* =========================================================
   PLAYER MOVEMENT
   ========================================================= */

function updatePlayer(delta) {

    if (!player)
        return;

    let x = 0;
    let z = 0;

    if (
        keys["w"] ||
        keys["ArrowUp"]
    )
        z -= 1;

    if (
        keys["s"] ||
        keys["ArrowDown"]
    )
        z += 1;

    if (
        keys["a"] ||
        keys["ArrowLeft"]
    )
        x -= 1;

    if (
        keys["d"] ||
        keys["ArrowRight"]
    )
        x += 1;

    if (x !== 0 || z !== 0) {

        const length =
            Math.hypot(x, z);

        x /= length;
        z /= length;

        const angle =
            cameraYaw;

        const moveX =
            x * Math.cos(angle) -
            z * Math.sin(angle);

        const moveZ =
            x * Math.sin(angle) +
            z * Math.cos(angle);

        player.position.x +=
            moveX *
            playerStats.speed *
            delta;

        player.position.z +=
            moveZ *
            playerStats.speed *
            delta;
    }

    if (
        keys[" "] &&
        playerStats.dashCooldown <= 0
    ) {

        dash();

        keys[" "] = false;
    }

    playerStats.fireCooldown -=
        delta * 1000;

    playerStats.dashCooldown -=
        delta * 1000;

    playerStats.invincible -=
        delta * 1000;
}

/* =========================================================
   DASH
   ========================================================= */

function dash() {

    player.position.x +=
        Math.sin(cameraYaw) *
        12;

    player.position.z +=
        Math.cos(cameraYaw) *
        12;

    playerStats.dashCooldown =
        1500;

    playerStats.invincible =
        400;

    createExplosion(
        player.position,
        0x00ffff
    );

    showMessage(
        "⚡ SIGMA DASH!"
    );
}

/* =========================================================
   SHOOTING
   ========================================================= */

function shoot() {

    if (
        playerStats.fireCooldown > 0 ||
        !gameRunning ||
        paused
    )
        return;

    const weapon =
        weapons[currentWeapon];

    const direction =
        new THREE.Vector3();

    camera.getWorldDirection(
        direction
    );

    let shots = 1;

    if (
        currentWeapon ===
        "shotgun"
    )
        shots = 7;

    for (
        let i = 0;
        i < shots;
        i++
    ) {

        const dir =
            direction.clone();

        if (shots > 1) {

            dir.x +=
                random(-.12, .12);

            dir.y +=
                random(-.08, .08);

            dir.z +=
                random(-.12, .12);

            dir.normalize();
        }

        const bullet =
            createBullet(
                player.position.clone()
                    .add(
                        new THREE.Vector3(
                            0,
                            1.5,
                            0
                        )
                    ),
                dir,
                weapon.damage *
                playerStats.damageMultiplier
            );

        bullets.push(
            bullet
        );
    }

    playerStats.fireCooldown =
        weapon.fireRate;
}

function createBullet(
    position,
    direction,
    damage
) {

    const geometry =
        new THREE.SphereGeometry(
            currentWeapon === "laser"
                ? .12
                : .18,
            8,
            8
        );

    const material =
        new THREE.MeshBasicMaterial({
            color:
                currentWeapon === "laser"
                    ? 0x00ffff
                    : 0xffff00
        });

    const mesh =
        new THREE.Mesh(
            geometry,
            material
        );

    mesh.position.copy(
        position
    );

    scene.add(mesh);

    return {

        mesh,

        velocity:
            direction.multiplyScalar(
                currentWeapon === "laser"
                    ? 60
                    : 35
            ),

        damage,

        life: 2
    };
}

/* =========================================================
   BULLETS
   ========================================================= */

function updateBullets(delta) {

    for (
        let i = bullets.length - 1;
        i >= 0;
        i--
    ) {

        const b =
            bullets[i];

        b.mesh.position.add(
            b.velocity.clone()
                .multiplyScalar(delta)
        );

        b.life -= delta;

        let remove = false;

        for (
            let j = enemies.length - 1;
            j >= 0;
            j--
        ) {

            const enemy =
                enemies[j];

            if (
                b.mesh.position.distanceTo(
                    enemy.mesh.position
                ) <
                enemy.size
            ) {

                let damage =
                    b.damage;

                if (
                    Math.random() <
                    playerStats.criticalChance
                ) {

                    damage *=
                        playerStats.criticalDamage;

                    showMessage(
                        "💥 CRITICAL RIZZ!"
                    );
                }

                enemy.hp -= damage;

                createParticles(
                    enemy.mesh.position,
                    5,
                    0xffaa00
                );

                remove = true;

                if (
                    enemy.hp <= 0
                ) {

                    killEnemy(
                        enemy,
                        j
                    );
                }

                break;
            }
        }

        if (
            boss &&
            !remove &&
            b.mesh.position.distanceTo(
                boss.mesh.position
            ) < boss.size
        ) {

            boss.hp -=
                b.damage;

            createParticles(
                boss.mesh.position,
                8,
                0xff0000
            );

            remove = true;

            if (boss.hp <= 0)
                killBoss();
        }

        if (
            b.life <= 0
        )
            remove = true;

        if (remove) {

            scene.remove(
                b.mesh
            );

            bullets.splice(
                i,
                1
            );
        }
    }
}

/* =========================================================
   ENEMIES
   ========================================================= */

function spawnEnemy() {

    const type =
        enemyTypes[
            randomInt(
                0,
                Math.min(
                    enemyTypes.length - 1,
                    2 + Math.floor(
                        wave / 5
                    )
                )
            )
        ];

    const angle =
        random(
            0,
            Math.PI * 2
        );

    const radius =
        random(
            50,
            90
        );

    const geometry =
        new THREE.SphereGeometry(
            1.5,
            16,
            16
        );

    const material =
        new THREE.MeshStandardMaterial({
            color: type.color
        });

    const mesh =
        new THREE.Mesh(
            geometry,
            material
        );

    mesh.castShadow = true;

    mesh.position.set(
        player.position.x +
            Math.cos(angle) *
            radius,

        1.5,

        player.position.z +
            Math.sin(angle) *
            radius
    );

    scene.add(mesh);

    enemies.push({

        mesh,

        hp:
            type.hp *
            (1 + wave * .1),

        maxHp:
            type.hp *
            (1 + wave * .1),

        speed:
            type.speed *
            (1 + wave * .025),

        damage:
            type.damage,

        size:
            2,

        xp:
            type.xp,

        name:
            type.name
    });
}

function updateEnemies(delta) {

    for (const enemy of enemies) {

        const direction =
            player.position.clone()
                .sub(
                    enemy.mesh.position
                );

        direction.y = 0;

        const length =
            direction.length();

        if (length > 2) {

            direction.normalize();

            enemy.mesh.position.add(
                direction.multiplyScalar(
                    enemy.speed *
                    delta
                )
            );
        }

        if (
            playerStats.invincible <= 0 &&
            enemy.mesh.position.distanceTo(
                player.position
            ) < 2.5
        ) {

            damagePlayer(
                enemy.damage
            );
        }
    }
}

function killEnemy(
    enemy,
    index
) {

    scene.remove(
        enemy.mesh
    );

    enemies.splice(
        index,
        1
    );

    enemiesKilled++;

    combo++;

    bestCombo =
        Math.max(
            bestCombo,
            combo
        );

    score +=
        10 +
        combo * 2;

    coins +=
        randomInt(
            2,
            10
        );

    gainXP(
        enemy.xp
    );

    createExplosion(
        enemy.mesh.position,
        0xff3333
    );

    if (
        Math.random() <
        .2
    ) {

        spawnLoot(
            enemy.mesh.position
        );
    }

    if (
        combo > 0 &&
        combo % 10 === 0
    ) {

        brainrot();
    }
}

/* =========================================================
   DAMAGE
   ========================================================= */

function damagePlayer(
    amount
) {

    if (
        playerStats.invincible > 0
    )
        return;

    const damage =
        Math.max(
            1,
            amount -
            playerStats.armor
        );

    playerStats.hp -=
        damage;

    combo = 0;

    playerStats.invincible =
        500;

    shake = 1;

    if (
        playerStats.hp <= 0
    )
        endGame();
}

/* =========================================================
   XP
   ========================================================= */

function gainXP(amount) {

    xp += amount;

    const required =
        level * 100;

    if (
        xp >= required
    ) {

        xp -= required;

        level++;

        playerStats.maxHp +=
            10;

        playerStats.hp =
            playerStats.maxHp;

        playerStats.damageMultiplier +=
            .08;

        coins += 50;

        showMessage(
            "⭐ LEVEL " +
            level +
            "!"
        );

        createExplosion(
            player.position,
            0x00ffff
        );
    }
}

/* =========================================================
   LOOT
   ========================================================= */

function spawnLoot(position) {

    const types = [
        "coin",
        "crystal",
        "braincell",
        "potion",
        "bomb"
    ];

    const type =
        types[
            randomInt(
                0,
                types.length - 1
            )
        ];

    const geometry =
        new THREE.BoxGeometry(
            .5,
            .5,
            .5
        );

    const material =
        new THREE.MeshStandardMaterial({
            color:
                type === "coin"
                    ? 0xffd700
                    : type === "crystal"
                    ? 0x00ffff
                    : 0xff00ff
        });

    const mesh =
        new THREE.Mesh(
            geometry,
            material
        );

    mesh.position.copy(
        position
    );

    mesh.position.y = 1;

    scene.add(mesh);

    loot.push({

        mesh,

        type,

        life: 30
    });
}

function updateLoot(delta) {

    for (
        let i = loot.length - 1;
        i >= 0;
        i--
    ) {

        const item =
            loot[i];

        item.life -= delta;

        item.mesh.rotation.y +=
            delta * 3;

        if (
            item.mesh.position.distanceTo(
                player.position
            ) < 3
        ) {

            collectLoot(
                item
            );

            scene.remove(
                item.mesh
            );

            loot.splice(
                i,
                1
            );
        }

        else if (
            item.life <= 0
        ) {

            scene.remove(
                item.mesh
            );

            loot.splice(
                i,
                1
            );
        }
    }
}

function collectLoot(item) {

    if (
        item.type === "coin"
    )
        coins += 25;

    if (
        item.type === "crystal"
    )
        inventory.crystals++;

    if (
        item.type === "braincell"
    )
        inventory.braincells++;

    if (
        item.type === "potion"
    )
        inventory.potions++;

    if (
        item.type === "bomb"
    )
        inventory.bombs++;

    showMessage(
        "✨ +" +
        item.type
    );
}

/* =========================================================
   POWERUPS
   ========================================================= */

function spawnPowerup() {

    const types = [
        "speed",
        "heal",
        "damage",
        "nuke",
        "shield",
        "money"
    ];

    const type =
        types[
            randomInt(
                0,
                types.length - 1
            )
        ];

    const geometry =
        new THREE.IcosahedronGeometry(
            1,
            1
        );

    const material =
        new THREE.MeshStandardMaterial({
            color:
                type === "speed"
                    ? 0x00ffff
                    : type === "heal"
                    ? 0x00ff00
                    : type === "damage"
                    ? 0xff3300
                    : type === "nuke"
                    ? 0xff8800
                    : type === "shield"
                    ? 0x0088ff
                    : 0xffff00,

            emissive:
                type === "speed"
                    ? 0x004444
                    : 0x000000
        });

    const mesh =
        new THREE.Mesh(
            geometry,
            material
        );

    mesh.position.set(
        random(-70, 70),
        1,
        random(-70, 70)
    );

    scene.add(mesh);

    powerups.push({

        mesh,

        type,

        life: 30
    });
}

function updatePowerups(delta) {

    for (
        let i = powerups.length - 1;
        i >= 0;
        i--
    ) {

        const p =
            powerups[i];

        p.life -= delta;

        p.mesh.rotation.y +=
            delta * 2;

        if (
            p.mesh.position.distanceTo(
                player.position
            ) < 3
        ) {

            usePowerup(
                p.type
            );

            scene.remove(
                p.mesh
            );

            powerups.splice(
                i,
                1
            );
        }

        else if (
            p.life <= 0
        ) {

            scene.remove(
                p.mesh
            );

            powerups.splice(
                i,
                1
            );
        }
    }
}

function usePowerup(type) {

    if (
        type === "speed"
    ) {

        playerStats.speed = 12;

        setTimeout(
            () => {
                playerStats.speed = 7;
            },
            7000
        );

        showMessage(
            "⚡ SPEED RIZZ!"
        );
    }

    if (
        type === "heal"
    ) {

        playerStats.hp =
            Math.min(
                playerStats.maxHp,
                playerStats.hp + 50
            );

        showMessage(
            "❤️ GYATT HEALTH!"
        );
    }

    if (
        type === "damage"
    ) {

        playerStats.damageMultiplier +=
            .5;

        setTimeout(
            () => {
                playerStats.damageMultiplier -=
                    .5;
            },
            8000
        );

        showMessage(
            "🔥 DAMAGE MODE!"
        );
    }

    if (
        type === "nuke"
    ) {

        for (
            const e of enemies
        ) {

            score += 20;

            createExplosion(
                e.mesh.position,
                0xff8800
            );

            scene.remove(
                e.mesh
            );
        }

        enemies = [];

        showMessage(
            "💥 OHIO NUKE!"
        );
    }

    if (
        type === "shield"
    ) {

        playerStats.invincible =
            800;

        showMessage(
            "🛡️ UNTOUCHABLE!"
        );
    }

    if (
        type === "money"
    ) {

        coins += 500;

        showMessage(
            "💰 FANUM TAX +500!"
        );
    }
}

/* =========================================================
   BOSS
   ========================================================= */

function spawnBoss() {

    const geometry =
        new THREE.SphereGeometry(
            7,
            24,
            24
        );

    const material =
        new THREE.MeshStandardMaterial({
            color: 0xff0055,
            emissive: 0x440011
        });

    const mesh =
        new THREE.Mesh(
            geometry,
            material
        );

    mesh.position.set(
        player.position.x,
        7,
        player.position.z - 60
    );

    scene.add(mesh);

    boss = {

        mesh,

        hp:
            3000 +
            wave * 600,

        maxHp:
            3000 +
            wave * 600,

        speed: 4,

        size: 8,

        attackTimer: 2
    };

    document.getElementById(
        "bossBar"
    ).style.display =
        "block";

    document.getElementById(
        "bossName"
    ).textContent =
        wave % 2 === 0
            ? "THE OHIO OVERLORD"
            : "SIGMA SUPREME";

    showMessage(
        "🚨 BOSS HAS ENTERED THE CHAT 🚨",
        180
    );
}

function updateBoss(delta) {

    if (!boss)
        return;

    const direction =
        player.position.clone()
            .sub(
                boss.mesh.position
            );

    direction.y = 0;

    if (
        direction.length() > 10
    ) {

        direction.normalize();

        boss.mesh.position.add(
            direction.multiplyScalar(
                boss.speed *
                delta
            )
        );
    }

    boss.attackTimer -=
        delta;

    if (
        boss.attackTimer <= 0
    ) {

        bossAttack();

        boss.attackTimer =
            random(
                1,
                2.5
            );
    }

    const percentage =
        Math.max(
            0,
            boss.hp /
            boss.maxHp *
            100
        );

    document.getElementById(
        "bossInner"
    ).style.width =
        percentage + "%";
}

function bossAttack() {

    const choice =
        randomInt(
            0,
            2
        );

    if (choice === 0) {

        if (
            boss.mesh.position.distanceTo(
                player.position
            ) < 25
        ) {

            damagePlayer(40);
        }

        createExplosion(
            boss.mesh.position,
            0xff0000
        );
    }

    if (choice === 1) {

        for (
            let i = 0;
            i < 6;
            i++
        )
            spawnEnemy();

        showMessage(
            "💀 THE BOSS SUMMONED MINIONS!"
        );
    }

    if (choice === 2) {

        damagePlayer(25);

        showMessage(
            "🔴 OHIO LASER!"
        );
    }
}

function killBoss() {

    score += 2000;

    coins += 1000;

    gainXP(1500);

    createExplosion(
        boss.mesh.position,
        0xffd700
    );

    scene.remove(
        boss.mesh
    );

    boss = null;

    document.getElementById(
        "bossBar"
    ).style.display =
        "none";

    wave++;

    showMessage(
        "👑 BOSS DESTROYED! 👑",
        180
    );

    startWave();
}

/* =========================================================
   WAVES
   ========================================================= */

function startWave() {

    const amount =
        5 +
        wave * 2;

    for (
        let i = 0;
        i < amount;
        i++
    )
        spawnEnemy();

    if (
        wave % 5 === 0
    ) {

        setTimeout(
            () => {
                spawnBoss();
            },
            1500
        );
    }
}

/* =========================================================
   PARTICLES
   ========================================================= */

function createParticles(
    position,
    amount,
    color
) {

    for (
        let i = 0;
        i < amount;
        i++
    ) {

        const geometry =
            new THREE.BoxGeometry(
                .15,
                .15,
                .15
            );

        const material =
            new THREE.MeshBasicMaterial({
                color
            });

        const mesh =
            new THREE.Mesh(
                geometry,
                material
            );

        mesh.position.copy(
            position
        );

        scene.add(mesh);

        particles.push({

            mesh,

            velocity:
                new THREE.Vector3(
                    random(-5, 5),
                    random(1, 7),
                    random(-5, 5)
                ),

            life: 1
        });
    }
}

function createExplosion(
    position,
    color
) {

    createParticles(
        position,
        30,
        color
    );

    shake = 1;
}

function updateParticles(delta) {

    for (
        let i = particles.length - 1;
        i >= 0;
        i--
    ) {

        const p =
            particles[i];

        p.mesh.position.add(
            p.velocity.clone()
                .multiplyScalar(
                    delta
                )
        );

        p.velocity.y -=
            10 * delta;

        p.life -=
            delta;

        if (
            p.life <= 0
        ) {

            scene.remove(
                p.mesh
            );

            particles.splice(
                i,
                1
            );
        }
    }
}

/* =========================================================
   NPCS
   ========================================================= */

function createNPCs() {

    const data = [

        {
            name: "Rizzler",
            x: 20,
            z: 20,
            text:
                "You got that DAWG in you."
        },

        {
            name: "Ohio Man",
            x: -30,
            z: 40,
            text:
                "Welcome to Ohio."
        },

        {
            name: "Sigma Sage",
            x: 40,
            z: -30,
            text:
                "Never stop grinding."
        }
    ];

    for (
        const n of data
    ) {

        const geometry =
            new THREE.CapsuleGeometry(
                .7,
                1.5,
                4,
                8
            );

        const material =
            new THREE.MeshStandardMaterial({
                color: 0x55ff99
            });

        const mesh =
            new THREE.Mesh(
                geometry,
                material
            );

        mesh.position.set(
            n.x,
            1.3,
            n.z
        );

        scene.add(mesh);

        npcs.push({

            mesh,

            name: n.name,

            text: n.text
        });
    }
}

function interact() {

    for (
        const npc of npcs
    ) {

        if (
            npc.mesh.position.distanceTo(
                player.position
            ) < 5
        ) {

            showMessage(
                npc.name +
                ": " +
                npc.text,
                180
            );

            coins += 5;

            return;
        }
    }
}

/* =========================================================
   INVENTORY
   ========================================================= */

function openInventory() {

    paused = true;

    const menu =
        document.getElementById(
            "menu"
        );

    const content =
        document.getElementById(
            "menuContent"
        );

    content.innerHTML = `

        <h1>🎒 INVENTORY</h1>

        <div class="item">
            ❤️ Potions:
            ${inventory.potions}

            <button onclick="
                usePotion()
            ">
                USE
            </button>
        </div>

        <div class="item">
            💣 Bombs:
            ${inventory.bombs}

            <button onclick="
                useBomb()
            ">
                USE
            </button>
        </div>

        <div class="item">
            💎 Crystals:
            ${inventory.crystals}
        </div>

        <div class="item">
            🧠 Braincells:
            ${inventory.braincells}
        </div>

        <h2>Weapons</h2>

        <button onclick="
            currentWeapon='pistol';
            closeMenu();
        ">
            1 — Sigma Blaster
        </button>

        <button onclick="
            currentWeapon='shotgun';
            closeMenu();
        ">
            2 — Rizz Shotgun
        </button>

        <button onclick="
            currentWeapon='laser';
            closeMenu();
        ">
            3 — Ohio Laser
        </button>

        <br>

        <button onclick="
            closeMenu()
        ">
            CLOSE
        </button>
    `;

    menu.style.display =
        "flex";
}

function usePotion() {

    if (
        inventory.potions <= 0
    )
        return;

    inventory.potions--;

    playerStats.hp =
        Math.min(
            playerStats.maxHp,
            playerStats.hp + 50
        );

    openInventory();
}

function useBomb() {

    if (
        inventory.bombs <= 0
    )
        return;

    inventory.bombs--;

    for (
        const e of enemies
    ) {

        e.hp -= 150;
    }

    createExplosion(
        player.position,
        0xff8800
    );

    openInventory();
}

/* =========================================================
   SHOP
   ========================================================= */

function openShop() {

    paused = true;

    const menu =
        document.getElementById(
            "menu"
        );

    const content =
        document.getElementById(
            "menuContent"
        );

    content.innerHTML = `

        <h1>🏪 BRAINROT SHOP</h1>

        <h2>
            💰 Coins: ${coins}
        </h2>

        <div class="item">
            ❤️ Max HP +20
            <br>
            <button onclick="
                buyHP()
            ">
                100 COINS
            </button>
        </div>

        <div class="item">
            ⚔️ Damage +10%
            <br>
            <button onclick="
                buyDamage()
            ">
                150 COINS
            </button>
        </div>

        <div class="item">
            🛡️ Armor +2
            <br>
            <button onclick="
                buyArmor()
            ">
                200 COINS
            </button>
        </div>

        <button onclick="
            closeMenu()
        ">
            LEAVE SHOP
        </button>
    `;

    menu.style.display =
        "flex";
}

function buyHP() {

    if (coins < 100)
        return;

    coins -= 100;

    playerStats.maxHp +=
        20;

    playerStats.hp =
        playerStats.maxHp;

    openShop();
}

function buyDamage() {

    if (coins < 150)
        return;

    coins -= 150;

    playerStats.damageMultiplier +=
        .1;

    openShop();
}

function buyArmor() {

    if (coins < 200)
        return;

    coins -= 200;

    playerStats.armor +=
        2;

    openShop();
}

/* =========================================================
   SAVE / LOAD
   ========================================================= */

function saveGame() {

    const data = {

        score,
        coins,
        xp,
        level,
        wave,
        enemiesKilled,
        bestCombo,

        playerStats,

        inventory,

        currentWeapon
    };

    localStorage.setItem(
        "brainrot3D",
        JSON.stringify(data)
    );

    showMessage(
        "💾 GAME SAVED!"
    );
}

function loadGame() {

    const raw =
        localStorage.getItem(
            "brainrot3D"
        );

    if (!raw)
        return;

    const data =
        JSON.parse(raw);

    score =
        data.score || 0;

    coins =
        data.coins || 100;

    xp =
        data.xp || 0;

    level =
        data.level || 1;

    wave =
        data.wave || 1;

    enemiesKilled =
        data.enemiesKilled || 0;

    bestCombo =
        data.bestCombo || 0;

    if (
        data.playerStats
    ) {

        Object.assign(
            playerStats,
            data.playerStats
        );
    }

    if (
        data.inventory
    ) {

        inventory =
            data.inventory;
    }

    currentWeapon =
        data.currentWeapon ||
        "pistol";
}

/* =========================================================
   PAUSE / MENU
   ========================================================= */

function togglePause() {

    if (
        document.getElementById(
            "menu"
        ).style.display ===
        "flex"
    ) {

        closeMenu();

        return;
    }

    paused = true;

    document.getElementById(
        "menuContent"
    ).innerHTML = `

        <h1>⏸️ PAUSED</h1>

        <button onclick="
            closeMenu()
        ">
            CONTINUE
        </button>

        <button onclick="
            openInventory()
        ">
            🎒 INVENTORY
        </button>

        <button onclick="
            openShop()
        ">
            🏪 SHOP
        </button>

        <button onclick="
            saveGame()
        ">
            💾 SAVE GAME
        </button>
    `;

    document.getElementById(
        "menu"
    ).style.display =
        "flex";
}

function closeMenu() {

    document.getElementById(
        "menu"
    ).style.display =
        "none";

    paused = false;
}

/* =========================================================
   HUD
   ========================================================= */

function updateHUD() {

    const weapon =
        weapons[currentWeapon];

    document.getElementById(
        "hud"
    ).innerHTML = `

        🧠 SCORE: ${score}

        <br>

        💰 COINS: ${coins}

        <br>

        ❤️ HP:
        ${Math.max(
            0,
            Math.floor(
                playerStats.hp
            )
        )}
        /
        ${playerStats.maxHp}

        <br>

        ⭐ LEVEL: ${level}

        <br>

        ✨ XP:
        ${Math.floor(xp)}
        /
        ${level * 100}

        <br>

        🌊 WAVE: ${wave}

        <br>

        🔥 COMBO: ${combo}

        <br>

        👑 BEST: ${bestCombo}

        <br>

        🔫 ${weapon.name}

        <br>

        🛡️ ARMOR:
        ${playerStats.armor}
    `;
}

/* =========================================================
   RANDOM EVENTS
   ========================================================= */

let eventTimer = 0;
let eventName = "";

function randomEvent() {

    const events = [

        "OHIO MODE",
        "DOUBLE XP",
        "ENEMY RUSH",
        "COIN RAIN",
        "SUPER SPEED",
        "NUCLEAR CHAOS"
    ];

    eventName =
        events[
            randomInt(
                0,
                events.length - 1
            )
        ];

    eventTimer =
        20;

    showMessage(
        "🌎 EVENT: " +
        eventName,
        150
    );

    if (
        eventName ===
        "ENEMY RUSH"
    ) {

        for (
            let i = 0;
            i < 20;
            i++
        )
            spawnEnemy();
    }

    if (
        eventName ===
        "COIN RAIN"
    ) {

        for (
            let i = 0;
            i < 20;
            i++
        ) {

            spawnLoot(
                new THREE.Vector3(
                    random(
                        -70,
                        70
                    ),
                    0,
                    random(
                        -70,
                        70
                    )
                )
            );
        }
    }
}

/* =========================================================
   GAME OVER
   ========================================================= */

function endGame() {

    gameRunning = false;

    document.getElementById(
        "menuContent"
    ).innerHTML = `

        <h1>💀 YOU GOT COOKED 💀</h1>

        <h2>
            Score:
            ${score}
        </h2>

        <p>
            Level:
            ${level}
        </p>

        <p>
            Enemies destroyed:
            ${enemiesKilled}
        </p>

        <p>
            Best combo:
            ${bestCombo}
        </p>

        <button onclick="
            location.reload()
        ">
            🔄 PLAY AGAIN
        </button>

        <button onclick="
            saveGame()
        ">
            💾 SAVE SCORE
        </button>
    `;

    document.getElementById(
        "menu"
    ).style.display =
        "flex";
}

/* =========================================================
   RESIZE
   ========================================================= */

function resize() {

    camera.aspect =
        window.innerWidth /
        window.innerHeight;

    camera.updateProjectionMatrix();

    renderer.setSize(
        window.innerWidth,
        window.innerHeight
    );
}

/* =========================================================
   GAME LOOP
   ========================================================= */

function animate() {

    requestAnimationFrame(
        animate
    );

    const delta =
        Math.min(
            clock.getDelta(),
            .05
        );

    if (
        gameRunning &&
        !paused
    ) {

        updatePlayer(delta);

        updateCamera();

        updateShooting();

        updateBullets(delta);

        updateEnemies(delta);

        updateBoss(delta);

        updateLoot(delta);

        updatePowerups(delta);

        updateParticles(delta);

        if (
            eventTimer > 0
        ) {

            eventTimer -= delta;

            if (
                eventTimer <= 0
            ) {

                eventName = "";
            }
        }

        if (
            Math.random() <
            .0003
        ) {

            randomEvent();
        }

        if (
            Math.random() <
            .0005
        ) {

            spawnPowerup();
        }

        updateHUD();

        if (
            messageTimer > 0
        ) {

            messageTimer--;

        } else {

            document.getElementById(
                "message"
            ).textContent = "";
        }
    }

    renderer.render(
        scene,
        camera
    );
}

function updateShooting() {

    if (
        mouse.down
    ) {

        shoot();
    }
}

/* =========================================================
   ENEMY SPAWNER
   ========================================================= */

setInterval(
    () => {

        if (
            gameRunning &&
            !paused &&
            enemies.length <
            5 + wave * 2
        ) {

            spawnEnemy();
        }

    },
    1000
);

/* =========================================================
   AUTOSAVE
   ========================================================= */

setInterval(
    () => {

        if (
            gameRunning
        ) {

            saveGame();
        }

    },
    30000
);

/* =========================================================
   START
   ========================================================= */

init3D();

loadGame();

startWave();

brainrot();

animate();

</script>

</body>
</html>
```
