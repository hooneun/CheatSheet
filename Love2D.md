# LÖVE 2D 11.5 Cheat Sheet

> **대상:** LÖVE 11.5 / Lua
> LÖVE는 Lua 기반의 오픈소스 2D 게임 프레임워크이며 Windows, macOS, Linux, Android, iOS를 지원한다. 현재 공식 사이트에서 제공하는 안정 버전은 **11.5**다.

---

## 1. 기본 프로젝트 구조

```text
my-game/
├── main.lua
├── conf.lua
├── assets/
│   ├── images/
│   ├── sounds/
│   └── fonts/
└── src/
    ├── player.lua
    └── game.lua
```

최소한 `main.lua`만 있으면 실행할 수 있다.

```lua
function love.load()
    -- 최초 1회 초기화
end

function love.update(dt)
    -- 게임 로직 업데이트
end

function love.draw()
    -- 화면 렌더링
end
```

핵심 콜백은 `love.load`, `love.update`, `love.draw`이며 각각 초기화, 상태 업데이트, 렌더링에 사용한다.

---

# 2. 기본 게임 루프

```lua
local x = 100
local speed = 200

function love.update(dt)
    x = x + speed * dt
end

function love.draw()
    love.graphics.circle("fill", x, 200, 20)
end
```

### `dt`

`dt`는 이전 업데이트 이후 흐른 시간(초)이다.

```lua
position = position + speed * dt
```

프레임률에 독립적인 이동을 구현할 때 사용한다.

```lua
-- BAD
x = x + 5

-- GOOD
x = x + 300 * dt
```

---

# 3. Graphics

`love.graphics`는 선, 도형, 텍스트, 이미지 등의 화면 렌더링을 담당하는 핵심 모듈이다.

## 색상

LÖVE 11.x에서는 RGB 값을 `0 ~ 1`로 지정한다.

```lua
love.graphics.setColor(1, 0, 0)
love.graphics.setColor(0.2, 0.5, 1)
love.graphics.setColor(1, 1, 1, 0.5)
```

원래 색상으로 복구:

```lua
love.graphics.setColor(1, 1, 1, 1)
```

---

## 사각형

```lua
love.graphics.rectangle("fill", x, y, width, height)

love.graphics.rectangle("line", x, y, width, height)
```

예:

```lua
love.graphics.rectangle("fill", 100, 100, 200, 80)
```

---

## 원

```lua
love.graphics.circle("fill", x, y, radius)

love.graphics.circle("line", x, y, radius)
```

---

## 선

```lua
love.graphics.line(x1, y1, x2, y2)
```

```lua
love.graphics.setLineWidth(3)
```

---

## 점

```lua
love.graphics.points(x, y)
```

---

# 4. Text

```lua
love.graphics.print("Hello LÖVE", 100, 100)
```

공식 문서상 `love.graphics.print`는 화면에 텍스트를 출력하며 UTF-8 텍스트도 지원한다. 적절한 글꼴에 해당 글리프가 포함되어 있어야 한다.

### printf

너비와 정렬 지정:

```lua
love.graphics.printf(
    "Game Over",
    0,
    200,
    love.graphics.getWidth(),
    "center"
)
```

정렬:

```text
left
center
right
justify
```

---

# 5. Font

```lua
local font

function love.load()
    font = love.graphics.newFont(24)
end

function love.draw()
    love.graphics.setFont(font)
    love.graphics.print("Score: 100", 20, 20)
end
```

파일 사용:

```lua
font = love.graphics.newFont(
    "assets/fonts/game.ttf",
    24
)
```

---

# 6. Image

이미지 로드:

```lua
local player

function love.load()
    player = love.graphics.newImage(
        "assets/images/player.png"
    )
end
```

렌더링:

```lua
function love.draw()
    love.graphics.draw(player, 100, 200)
end
```

공식 예제에서도 `newImage`로 이미지를 로드하고 `love.graphics.draw`로 렌더링한다.

---

# 7. love.graphics.draw

기본 시그니처:

```lua
love.graphics.draw(
    drawable,
    x,
    y,
    rotation,
    scaleX,
    scaleY,
    originX,
    originY
)
```

`draw`는 Image, Canvas, SpriteBatch, ParticleSystem, Mesh, Text, Video 등의 `Drawable` 객체를 그릴 수 있다.

### 회전

```lua
love.graphics.draw(
    player,
    x,
    y,
    math.rad(45)
)
```

### 확대

```lua
love.graphics.draw(
    player,
    x,
    y,
    0,
    2,
    2
)
```

### 이미지 중심 기준 회전

```lua
local w = player:getWidth()
local h = player:getHeight()

love.graphics.draw(
    player,
    x,
    y,
    rotation,
    1,
    1,
    w / 2,
    h / 2
)
```

---

# 8. 화면 크기

```lua
local width = love.graphics.getWidth()
local height = love.graphics.getHeight()
```

동시에:

```lua
local width, height =
    love.graphics.getDimensions()
```

---

# 9. Transform

## 이동

```lua
love.graphics.translate(100, 50)
```

## 회전

```lua
love.graphics.rotate(math.rad(45))
```

## 확대

```lua
love.graphics.scale(2, 2)
```

### push / pop

특정 오브젝트에만 변환 적용:

```lua
love.graphics.push()

love.graphics.translate(x, y)
love.graphics.rotate(rotation)

love.graphics.rectangle(
    "fill",
    -25,
    -25,
    50,
    50
)

love.graphics.pop()
```

매우 자주 쓰는 패턴:

```lua
love.graphics.push()
-- transform
-- draw
love.graphics.pop()
```

---

# 10. Keyboard Input

## 키가 눌려 있는지 검사

```lua
if love.keyboard.isDown("left") then
    x = x - speed * dt
end
```

여러 키:

```lua
if love.keyboard.isDown("left", "a") then
    x = x - speed * dt
end
```

`love.keyboard.isDown`은 해당 키가 현재 눌려 있는지를 반환한다. 일회성 입력 이벤트인 `love.keypressed`와 용도가 다르다.

---

## 키를 누른 순간

```lua
function love.keypressed(key)
    if key == "space" then
        jump()
    end

    if key == "escape" then
        love.event.quit()
    end
end
```

---

## 키를 뗀 순간

```lua
function love.keyreleased(key)
    print(key .. " released")
end
```

---

# 11. 이동 입력 패턴

```lua
function love.update(dt)

    local dx = 0
    local dy = 0

    if love.keyboard.isDown("a") then
        dx = dx - 1
    end

    if love.keyboard.isDown("d") then
        dx = dx + 1
    end

    if love.keyboard.isDown("w") then
        dy = dy - 1
    end

    if love.keyboard.isDown("s") then
        dy = dy + 1
    end

    player.x = player.x + dx * player.speed * dt
    player.y = player.y + dy * player.speed * dt

end
```

### 대각선 이동 속도 보정

```lua
local length = math.sqrt(dx * dx + dy * dy)

if length > 0 then
    dx = dx / length
    dy = dy / length
end
```

---

# 12. Mouse

현재 마우스 좌표:

```lua
local mx, my = love.mouse.getPosition()
```

또는:

```lua
local mx = love.mouse.getX()
local my = love.mouse.getY()
```

`love.mouse.getPosition`은 현재 마우스의 x/y 좌표를 반환한다.

---

## 클릭

```lua
function love.mousepressed(x, y, button)

    if button == 1 then
        print("Left Click")
    end

end
```

```text
1 = Left
2 = Right
3 = Middle
```

---

## 마우스 버튼 상태

```lua
if love.mouse.isDown(1) then
    -- left button
end
```

---

# 13. Mouse → Object 방향 구하기

```lua
local mx, my = love.mouse.getPosition()

local angle =
    math.atan2(
        my - player.y,
        mx - player.x
    )
```

렌더링:

```lua
love.graphics.draw(
    player.image,
    player.x,
    player.y,
    angle
)
```

---

# 14. Audio

오디오 소스 생성:

```lua
local sound =
    love.audio.newSource(
        "assets/sounds/hit.wav",
        "static"
    )
```

`love.audio.newSource`는 파일 경로 등의 입력으로 재생 가능한 `Source` 객체를 생성한다.

재생:

```lua
sound:play()
```

중지:

```lua
sound:stop()
```

일시 정지:

```lua
sound:pause()
```

---

## static vs stream

효과음:

```lua
love.audio.newSource(
    "hit.wav",
    "static"
)
```

배경 음악:

```lua
love.audio.newSource(
    "bgm.ogg",
    "stream"
)
```

공식 사이트의 기본 예제도 음악 파일을 `stream` Source로 생성해 재생하는 방식을 보여준다.

---

## 반복 재생

```lua
music:setLooping(true)
music:play()
```

볼륨:

```lua
music:setVolume(0.5)
```

---

# 15. Window

`conf.lua`:

```lua
function love.conf(t)

    t.window.title = "My Game"

    t.window.width = 1280
    t.window.height = 720

    t.window.resizable = true

end
```

런타임 변경:

```lua
love.window.setMode(
    1280,
    720,
    {
        resizable = true,
        vsync = 1
    }
)
```

제목:

```lua
love.window.setTitle("My Game")
```

---

# 16. FPS

```lua
local fps = love.timer.getFPS()

love.graphics.print(
    "FPS: " .. fps,
    10,
    10
)
```

---

# 17. Random

```lua
local value = love.math.random(1, 100)
```

0 ~ 1:

```lua
local value = love.math.random()
```

랜덤 위치:

```lua
local x =
    love.math.random(
        0,
        love.graphics.getWidth()
    )
```

---

# 18. Math 자주 쓰는 함수

```lua
math.floor(x)

math.ceil(x)

math.abs(x)

math.min(a, b)

math.max(a, b)

math.sqrt(x)

math.sin(x)

math.cos(x)

math.atan2(y, x)

math.rad(90)

math.deg(angle)
```

---

# 19. 거리 계산

```lua
local dx = x2 - x1
local dy = y2 - y1

local distance =
    math.sqrt(dx * dx + dy * dy)
```

충돌 체크 등에 자주 사용한다.

```lua
if distance < radius1 + radius2 then
    -- collision
end
```

---

# 20. AABB 충돌

사각형 충돌의 가장 기본적인 형태:

```lua
function checkCollision(a, b)

    return
        a.x < b.x + b.width and
        b.x < a.x + a.width and
        a.y < b.y + b.height and
        b.y < a.y + a.height

end
```

사용:

```lua
if checkCollision(player, enemy) then
    print("Collision")
end
```

---

# 21. Clamp

값 제한:

```lua
function clamp(value, min, max)

    return math.max(
        min,
        math.min(max, value)
    )

end
```

예:

```lua
player.x = clamp(
    player.x,
    0,
    love.graphics.getWidth()
)
```

---

# 22. Lerp

부드러운 이동:

```lua
function lerp(a, b, t)
    return a + (b - a) * t
end
```

사용:

```lua
camera.x =
    lerp(
        camera.x,
        player.x,
        5 * dt
    )
```

---

# 23. Filesystem

저장:

```lua
love.filesystem.write(
    "save.txt",
    "100"
)
```

`love.filesystem.write`는 LÖVE의 save directory에 데이터를 기록하며 같은 파일이 이미 존재하면 내용을 교체한다.

읽기:

```lua
local data =
    love.filesystem.read(
        "save.txt"
    )
```

존재 여부:

```lua
local info =
    love.filesystem.getInfo(
        "save.txt"
    )

if info then
    print("exists")
end
```

---

# 24. Module

`src/player.lua`

```lua
local Player = {}

function Player.new()

    return {
        x = 100,
        y = 100,
        speed = 200
    }

end

function Player.update(player, dt)

    if love.keyboard.isDown("d") then
        player.x =
            player.x +
            player.speed * dt
    end

end

return Player
```

`main.lua`

```lua
local Player =
    require("src.player")

local player

function love.load()
    player = Player.new()
end

function love.update(dt)
    Player.update(player, dt)
end
```

---

# 25. Lua Class 패턴

LÖVE 자체에는 클래스 시스템이 없기 때문에 Lua table과 metatable 패턴을 자주 사용한다.

```lua
Player = {}
Player.__index = Player

function Player:new(x, y)

    local obj = setmetatable({}, self)

    obj.x = x
    obj.y = y
    obj.speed = 200

    return obj

end

function Player:update(dt)

    if love.keyboard.isDown("d") then
        self.x =
            self.x +
            self.speed * dt
    end

end

function Player:draw()

    love.graphics.rectangle(
        "fill",
        self.x,
        self.y,
        32,
        32
    )

end
```

사용:

```lua
local player =
    Player:new(100, 100)

player:update(dt)
player:draw()
```

---

# 26. Game State 패턴

```lua
local state = "menu"

function love.update(dt)

    if state == "game" then
        updateGame(dt)
    end

end

function love.draw()

    if state == "menu" then
        drawMenu()

    elseif state == "game" then
        drawGame()

    elseif state == "gameover" then
        drawGameOver()
    end

end
```

---

# 27. Sprite Animation

Sprite Sheet:

```text
+-------+-------+-------+-------+
|   1   |   2   |   3   |   4   |
+-------+-------+-------+-------+
```

Quad 생성:

```lua
local frames = {}

frames[1] =
    love.graphics.newQuad(
        0,
        0,
        32,
        32,
        image:getDimensions()
    )
```

렌더링:

```lua
love.graphics.draw(
    image,
    frames[currentFrame],
    x,
    y
)
```

간단한 애니메이션:

```lua
timer = timer + dt

if timer >= 0.1 then

    timer = 0

    currentFrame =
        currentFrame + 1

    if currentFrame > #frames then
        currentFrame = 1
    end

end
```

---

# 28. Canvas

오프스크린 렌더링:

```lua
local canvas

function love.load()

    canvas =
        love.graphics.newCanvas(
            320,
            180
        )

end
```

Canvas에 그리기:

```lua
love.graphics.setCanvas(canvas)

love.graphics.clear()

love.graphics.rectangle(
    "fill",
    10,
    10,
    50,
    50
)

love.graphics.setCanvas()
```

화면에 출력:

```lua
love.graphics.draw(
    canvas,
    0,
    0,
    0,
    4,
    4
)
```

픽셀 아트 게임의 내부 해상도를 고정할 때 유용하다.

---

# 29. Physics 기본

```lua
local world

function love.load()

    world =
        love.physics.newWorld(
            0,
            9.81 * 64,
            true
        )

end

function love.update(dt)
    world:update(dt)
end
```

Body:

```lua
local body =
    love.physics.newBody(
        world,
        100,
        100,
        "dynamic"
    )
```

Shape:

```lua
local shape =
    love.physics.newRectangleShape(
        32,
        32
    )
```

Fixture:

```lua
local fixture =
    love.physics.newFixture(
        body,
        shape
    )
```

---

# 30. Debug 출력

```lua
print("Player:", player.x, player.y)
```

화면 디버그:

```lua
function love.draw()

    love.graphics.print(
        string.format(
            "x: %.2f\ny: %.2f\nFPS: %d",
            player.x,
            player.y,
            love.timer.getFPS()
        ),
        10,
        10
    )

end
```

---

# 31. Fullscreen

```lua
love.window.setFullscreen(true)
```

토글:

```lua
love.window.setFullscreen(
    not love.window.getFullscreen()
)
```

예:

```lua
function love.keypressed(key)

    if key == "f11" then

        love.window.setFullscreen(
            not love.window.getFullscreen()
        )

    end

end
```

---

# 32. Quit

```lua
love.event.quit()
```

```lua
function love.keypressed(key)

    if key == "escape" then
        love.event.quit()
    end

end
```

---

# 33. 실전 Player 예제

```lua
local player = {
    x = 400,
    y = 300,
    width = 32,
    height = 32,
    speed = 250
}

function love.update(dt)

    local dx = 0
    local dy = 0

    if love.keyboard.isDown("a") then
        dx = dx - 1
    end

    if love.keyboard.isDown("d") then
        dx = dx + 1
    end

    if love.keyboard.isDown("w") then
        dy = dy - 1
    end

    if love.keyboard.isDown("s") then
        dy = dy + 1
    end

    local length =
        math.sqrt(
            dx * dx +
            dy * dy
        )

    if length > 0 then

        dx = dx / length
        dy = dy / length

    end

    player.x =
        player.x +
        dx *
        player.speed *
        dt

    player.y =
        player.y +
        dy *
        player.speed *
        dt

end

function love.draw()

    love.graphics.rectangle(
        "fill",
        player.x,
        player.y,
        player.width,
        player.height
    )

end
```

---

# 34. 추천 코드 구조

작은 프로젝트:

```text
main.lua
conf.lua
player.lua
enemy.lua
```

중간 규모:

```text
main.lua

src/
├── game.lua
├── player.lua
├── enemy.lua
├── bullet.lua
├── collision.lua
├── camera.lua
└── states/
    ├── menu.lua
    ├── play.lua
    └── gameover.lua

assets/
├── images/
├── audio/
├── fonts/
└── shaders/
```

---

# 35. 자주 쓰는 API 요약

| 목적       | API                             |
| -------- | ------------------------------- |
| 초기화      | `love.load()`                   |
| 게임 업데이트  | `love.update(dt)`               |
| 렌더링      | `love.draw()`                   |
| 키 상태     | `love.keyboard.isDown()`        |
| 키 입력 이벤트 | `love.keypressed()`             |
| 마우스 위치   | `love.mouse.getPosition()`      |
| 마우스 클릭   | `love.mousepressed()`           |
| 이미지 로드   | `love.graphics.newImage()`      |
| 이미지 출력   | `love.graphics.draw()`          |
| 사각형      | `love.graphics.rectangle()`     |
| 원        | `love.graphics.circle()`        |
| 텍스트      | `love.graphics.print()`         |
| 색상       | `love.graphics.setColor()`      |
| 폰트       | `love.graphics.newFont()`       |
| 화면 크기    | `love.graphics.getDimensions()` |
| FPS      | `love.timer.getFPS()`           |
| 소리 생성    | `love.audio.newSource()`        |
| 랜덤       | `love.math.random()`            |
| 파일 저장    | `love.filesystem.write()`       |
| 파일 읽기    | `love.filesystem.read()`        |
| 종료       | `love.event.quit()`             |

---

# 36. 가장 많이 사용하는 패턴

### Update

```lua
function love.update(dt)

    handleInput(dt)

    player:update(dt)

    enemies:update(dt)

    collision:update()

end
```

### Draw

```lua
function love.draw()

    world:draw()

    player:draw()

    enemies:draw()

    ui:draw()

end
```

즉:

```text
INPUT
  ↓
UPDATE
  ↓
COLLISION / PHYSICS
  ↓
DRAW
  ↓
NEXT FRAME
```

---

# 37. 기억할 핵심 10개

```text
1. main.lua가 진입점이다.

2. 초기화는 love.load().

3. 게임 로직은 love.update(dt).

4. 렌더링은 love.draw().

5. 이동 계산에는 dt를 곱한다.

6. 지속 입력은 love.keyboard.isDown().

7. 단발 입력은 love.keypressed().

8. 이미지는 newImage() → draw().

9. 효과음은 static, 긴 음악은 주로 stream Source를 사용한다.

10. 게임 상태와 렌더링 로직을 분리하면 규모가 커져도 관리하기 쉽다.
```

---

## 초압축 버전

```lua
local player = {
    x = 100,
    y = 100,
    speed = 200
}

function love.load()
end

function love.update(dt)

    if love.keyboard.isDown("a") then
        player.x = player.x - player.speed * dt
    end

    if love.keyboard.isDown("d") then
        player.x = player.x + player.speed * dt
    end

    if love.keyboard.isDown("w") then
        player.y = player.y - player.speed * dt
    end

    if love.keyboard.isDown("s") then
        player.y = player.y + player.speed * dt
    end

end

function love.draw()

    love.graphics.rectangle(
        "fill",
        player.x,
        player.y,
        32,
        32
    )

    love.graphics.print(
        "FPS: " .. love.timer.getFPS(),
        10,
        10
    )

end

function love.keypressed(key)

    if key == "escape" then
        love.event.quit()
    end

end
```

**LÖVE 핵심 공식:**

```text
love.load()
      ↓
love.update(dt)
      ↓
love.draw()
      ↓
repeat
```

LÖVE 공식 사이트는 `love`, `audio`, `filesystem`, `graphics`, `keyboard`, `mouse`, `physics`, `timer`, `window` 등을 주요 API 모듈로 제공한다.
