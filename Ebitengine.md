# Ebitengine v2.10 CheatSheet

**기준 버전: Ebitengine v2.10.2 — 2026-09-14 릴리스**
현재 공식 Go 모듈 최신 안정 버전도 `v2.10.2`로 확인됩니다. Ebitengine 2.10부터는 **Go 1.25 이상**이 필요합니다. ([Go Packages][1])

## 1. 설치 / 프로젝트 시작

```bash
mkdir mygame
cd mygame

go mod init example.com/mygame
go get github.com/hajimehoshi/ebiten/v2@v2.10.2
```

실행:

```bash
go run .
```

빌드:

```bash
go build .
```

Ebitengine 2.10은 Windows뿐 아니라 macOS, Linux/BSD 데스크톱에서도 Cgo 의존성을 제거해 Pure Go 빌드가 가능해졌습니다. 모바일은 여전히 Cgo 및 플랫폼 도구가 필요합니다. ([Ebitengine][2])

---

# 2. 최소 게임 템플릿

Ebitengine의 핵심은 `ebiten.Game`의 세 메서드입니다.

```go
package main

import (
	"log"

	"github.com/hajimehoshi/ebiten/v2"
	"github.com/hajimehoshi/ebiten/v2/ebitenutil"
)

const (
	screenWidth  = 320
	screenHeight = 240
)

type Game struct {
	x float64
	y float64
}

func (g *Game) Update() error {
	if ebiten.IsKeyPressed(ebiten.KeyArrowLeft) {
		g.x -= 2
	}
	if ebiten.IsKeyPressed(ebiten.KeyArrowRight) {
		g.x += 2
	}

	return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
	ebitenutil.DebugPrint(screen, "Hello Ebitengine")
}

func (g *Game) Layout(outsideWidth, outsideHeight int) (int, int) {
	return screenWidth, screenHeight
}

func main() {
	ebiten.SetWindowSize(screenWidth*3, screenHeight*3)
	ebiten.SetWindowTitle("Ebitengine")

	if err := ebiten.RunGame(&Game{}); err != nil {
		log.Fatal(err)
	}
}
```

`Update()`는 기본적으로 **60 TPS의 고정 timestep**으로 호출되며, 게임 로직에서는 일반적으로 OS timer로 delta time을 직접 계산할 필요가 없습니다. `Draw()`는 화면 프레임에 맞춰 호출됩니다. ([Ebitengine][3])

---

# 3. Game Loop

```go
type Game interface {
	Update() error
	Draw(screen *ebiten.Image)
	Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int)
}
```

핵심 개념:

```text
Update()
 └─ 게임 상태 변경
 └─ 입력 처리
 └─ 물리/AI/충돌

Draw()
 └─ 렌더링 전용

Layout()
 └─ 논리 해상도 결정
```

TPS 조정:

```go
ebiten.SetTPS(60)
```

현재 설정:

```go
tps := ebiten.TPS()
```

실제 성능 확인:

```go
fps := ebiten.ActualFPS()
tps := ebiten.ActualTPS()
```

디버그용으로만 사용하고 게임 로직을 `ActualTPS()`에 의존시키지 않는 것이 권장됩니다. ([Go Packages][1])

---

# 4. Window

```go
ebiten.SetWindowSize(1280, 720)
ebiten.SetWindowTitle("My Game")
ebiten.SetWindowResizable(true)
ebiten.SetFullscreen(true)
ebiten.SetVsyncEnabled(true)
```

상태 조회:

```go
ebiten.IsFullscreen()
ebiten.IsFocused()
ebiten.IsVsyncEnabled()
ebiten.IsWindowMaximized()
ebiten.IsWindowMinimized()
ebiten.IsWindowVisible()
```

논리 해상도와 실제 Window 크기는 별개로 생각하는 것이 좋습니다.

예:

```go
func (g *Game) Layout(w, h int) (int, int) {
	return 320, 180
}
```

실제 Window:

```text
1280 × 720
```

논리 게임 화면:

```text
320 × 180
```

---

# 5. Image

생성:

```go
img := ebiten.NewImage(64, 64)
```

Go `image.Image` → Ebitengine:

```go
img := ebiten.NewImageFromImage(src)
```

크기:

```go
w, h := img.Size()
```

색으로 채우기:

```go
img.Fill(color.RGBA{
	R: 255,
	G: 0,
	B: 0,
	A: 255,
})
```

초기화:

```go
img.Clear()
```

Sprite sheet:

```go
sprite := sheet.SubImage(
	image.Rect(0, 0, 32, 32),
).(*ebiten.Image)
```

`SubImage`는 원본과 pixel storage를 공유합니다. ([Ebitengine][4])

---

# 6. DrawImage

기본:

```go
op := &ebiten.DrawImageOptions{}

op.GeoM.Translate(100, 50)

screen.DrawImage(playerImage, op)
```

주요 옵션:

```go
type DrawImageOptions struct {
	GeoM       ebiten.GeoM
	ColorScale ebiten.ColorScale
	Blend      ebiten.Blend
	Filter     ebiten.Filter
}
```

현재 API에서는 과거의 `ColorM` 필드보다 `ColorScale` 또는 `colorm` 패키지를 사용하는 것이 권장됩니다. ([Go Packages][1])

---

# 7. GeoM — 이동 / 회전 / 확대

이동:

```go
op.GeoM.Translate(x, y)
```

확대:

```go
op.GeoM.Scale(2, 2)
```

회전:

```go
op.GeoM.Rotate(math.Pi / 4)
```

주의할 점은 **호출 순서가 중요**하다는 것입니다.

Sprite 중심 회전:

```go
w, h := img.Size()

op := &ebiten.DrawImageOptions{}

op.GeoM.Translate(
	-float64(w)/2,
	-float64(h)/2,
)

op.GeoM.Rotate(angle)

op.GeoM.Translate(x, y)

screen.DrawImage(img, op)
```

공식 `GeoM`의 기본값은 identity matrix입니다. ([Ebitengine][3])

---

# 8. Sprite Flip

좌우 반전:

```go
w, _ := img.Size()

op := &ebiten.DrawImageOptions{}

op.GeoM.Scale(-1, 1)
op.GeoM.Translate(float64(w), 0)

screen.DrawImage(img, op)
```

게임 위치까지 적용:

```go
op.GeoM.Scale(-1, 1)
op.GeoM.Translate(float64(w), 0)
op.GeoM.Translate(x, y)
```

---

# 9. Color / Alpha

투명도:

```go
op.ColorScale.ScaleAlpha(0.5)
```

RGBA Scale:

```go
op.ColorScale.Scale(
	1.0,
	0.5,
	0.5,
	1.0,
)
```

복잡한 HSV/Hue/색상행렬 처리는:

```go
"github.com/hajimehoshi/ebiten/v2/colorm"
```

```go
var cm colorm.ColorM

cm.RotateHue(math.Pi / 2)

colorm.DrawImage(screen, img, cm, nil)
```

`ColorScale`은 premultiplied-alpha 색상에 적용된다는 점도 알아두면 좋습니다. ([Go Packages][1])

---

# 10. Keyboard Input

누르고 있는 동안:

```go
if ebiten.IsKeyPressed(ebiten.KeyW) {
	y--
}
```

한 번 눌렀을 때:

```go
import "github.com/hajimehoshi/ebiten/v2/inpututil"

if inpututil.IsKeyJustPressed(ebiten.KeySpace) {
	jump()
}
```

놓았을 때:

```go
if inpututil.IsKeyJustReleased(ebiten.KeySpace) {
}
```

누른 시간:

```go
ticks := inpututil.KeyPressDuration(ebiten.KeySpace)
```

여러 키 조회:

```go
var keys []ebiten.Key

keys = inpututil.AppendPressedKeys(keys[:0])
```

할당 최소화:

```go
type Game struct {
	keys []ebiten.Key
}

func (g *Game) Update() error {
	g.keys = inpututil.AppendPressedKeys(g.keys[:0])

	for _, key := range g.keys {
		// ...
	}

	return nil
}
```

`inpututil`의 JustPressed/Pressed API들은 `Update()`에서 호출해야 합니다. ([Go Packages][5])

---

# 11. Mouse

좌표:

```go
x, y := ebiten.CursorPosition()
```

v2.10 고정밀 좌표:

```go
x, y := ebiten.CursorPositionF()
```

클릭 유지:

```go
if ebiten.IsMouseButtonPressed(ebiten.MouseButtonLeft) {
}
```

클릭 순간:

```go
if inpututil.IsMouseButtonJustPressed(
	ebiten.MouseButtonLeft,
) {
}
```

휠:

```go
xoff, yoff := ebiten.Wheel()
```

`CursorPositionF()`는 Ebitengine 2.10에서 추가된 고정밀 logical cursor 좌표 API입니다. ([Go Packages][1])

---

# 12. Text Input

실제 문자를 받아야 할 경우 `KeyA`, `KeyB`를 조합하지 말고:

```go
var chars []rune

chars = ebiten.AppendInputChars(chars[:0])

for _, r := range chars {
	text += string(r)
}
```

`ebiten.Key`는 물리적인 US 키보드 기준 key이고, `AppendInputChars`는 현재 locale에 따라 Unicode 입력을 반환합니다. ([Go Packages][1])

IME를 사용하는 편집 UI라면 v2.10부터 experimental:

```text
exp/textinput.Composer
```

사용을 검토할 수 있습니다. 기존 `textinput.Field`는 v2.10에서 deprecated 되었습니다. ([Ebitengine][2])

---

# 13. Touch

현재 touch 목록:

```go
ids := ebiten.AppendTouchIDs(nil)

for _, id := range ids {
	x, y := ebiten.TouchPosition(id)

	_, _ = x, y
}
```

새 touch:

```go
ids := inpututil.AppendJustPressedTouchIDs(nil)
```

릴리즈:

```go
ids := inpututil.AppendJustReleasedTouchIDs(nil)
```

---

# 14. Gamepad

연결된 패드:

```go
ids := ebiten.AppendGamepadIDs(nil)
```

표준 Gamepad layout 권장:

```go
for _, id := range ids {
	if !ebiten.IsStandardGamepadLayoutAvailable(id) {
		continue
	}

	if ebiten.IsStandardGamepadButtonPressed(
		id,
		ebiten.StandardGamepadButtonRightBottom,
	) {
		jump()
	}
}
```

Stick:

```go
x := ebiten.StandardGamepadAxisValue(
	id,
	ebiten.StandardGamepadAxisLeftStickHorizontal,
)

y := ebiten.StandardGamepadAxisValue(
	id,
	ebiten.StandardGamepadAxisLeftStickVertical,
)
```

Axis 값은 일반적으로:

```text
-1.0 ───────── 0 ───────── +1.0
```

입니다. ([Go Packages][1])

---

# 15. Vector Graphics

```go
import "github.com/hajimehoshi/ebiten/v2/vector"
```

사각형:

```go
vector.FillRect(
	screen,
	10, 10,
	100, 50,
	color.White,
	false,
)
```

원:

```go
vector.FillCircle(
	screen,
	100, 100,
	30,
	color.White,
	true,
)
```

선:

```go
vector.StrokeLine(
	screen,
	10, 10,
	200, 100,
	2,
	color.White,
	true,
)
```

테두리:

```go
vector.StrokeRect(
	screen,
	10, 10,
	100, 50,
	2,
	color.White,
	true,
)
```

예전의:

```go
vector.DrawFilledRect()
vector.DrawFilledCircle()
```

보다는 현재 `FillRect`, `FillCircle` 계열을 사용하는 것이 좋습니다. ([Go Packages][6])

---

# 16. text/v2

신규 프로젝트에서는:

```go
"github.com/hajimehoshi/ebiten/v2/text/v2"
```

사용을 권장합니다.

Font 로딩:

```go
fontSource, err := text.NewGoTextFaceSource(
	bytes.NewReader(fontData),
)
if err != nil {
	panic(err)
}
```

Face:

```go
face := &text.GoTextFace{
	Source: fontSource,
	Size:   24,
}
```

렌더링:

```go
op := &text.DrawOptions{}

op.GeoM.Translate(20, 50)

text.Draw(
	screen,
	"Hello Ebitengine",
	face,
	op,
)
```

측정:

```go
w, h := text.Measure(
	"Hello",
	face,
	24,
)
```

v2.10에서는 color emoji 지원과 bidi text 처리 개선이 추가됐습니다. 또한 기존 `text.Advance()`는 deprecated 되었으며 caret/selection 계산에는 `text.AdvanceAt()`을 사용합니다. ([Go Packages][7])

---

# 17. Audio

```go
import "github.com/hajimehoshi/ebiten/v2/audio"
```

Context:

```go
const sampleRate = 48000

audioContext := audio.NewContext(sampleRate)
```

Player:

```go
player, err := audioContext.NewPlayer(stream)
if err != nil {
	return err
}

player.Play()
```

상태:

```go
player.IsPlaying()
player.Pause()
player.Rewind()
player.Position()
player.SetPosition(...)
player.SetVolume(0.5)
```

v2.10부터:

```go
player.Close()
```

는 deprecated이며 대신:

```go
player.PauseAndStopReading()
```

을 사용합니다. ([Go Packages][8])

짧은 SE의 경우 byte data에서 Player를 매번 생성하는 것도 공식 performance guide에서 허용되는 패턴입니다. ([Ebitengine][9])

---

# 18. Shader / Kage

Shader 생성:

```go
shader, err := ebiten.NewShader(shaderSrc)
if err != nil {
	panic(err)
}
```

렌더링:

```go
op := &ebiten.DrawRectShaderOptions{}

op.Uniforms = map[string]any{
	"Time": time,
}

screen.DrawRectShader(
	screenWidth,
	screenHeight,
	shader,
	op,
)
```

Kage:

```go
//kage:unit pixels

package main

var Time float

func Fragment(
	dstPos vec4,
	srcPos vec2,
	color vec4,
) vec4 {
	r := sin(Time) * 0.5 + 0.5

	return vec4(r, 0.2, 1.0, 1.0)
}
```

이미지:

```go
op.Images[0] = image
```

Kage:

```go
func Fragment(dstPos vec4, srcPos vec2) vec4 {
	return imageSrc0At(srcPos)
}
```

새 Kage 프로그램은 공식 문서에서도:

```go
//kage:unit pixels
```

pixel mode 사용을 강하게 권장합니다. ([Ebitengine][10])

---

# 19. Ebitengine 2.10 Kage 변경

v2.10부터:

```go
for i := range 10 {
	// ...
}
```

고정 배열:

```go
for i := range arr {
}
```

지원.

Bitwise:

```text
&^
^
```

추가.

Multi-source image에서는 기존:

```text
imageSrc1At()
imageSrc2At()
imageSrc3At()
```

계열 대신:

```text
imageSrc1AtFromSrc0Pos()
imageSrc2AtFromSrc0Pos()
imageSrc3AtFromSrc0Pos()
```

사용 권장.

Texture 크기:

```text
imageDstTextureSize()

imageSrc0TextureSize()
imageSrc1TextureSize()
imageSrc2TextureSize()
imageSrc3TextureSize()
```

도 v2.10에서 제공됩니다. ([Ebitengine][10])

---

# 20. Shader Precompile — v2.10

새 experimental package:

```go
github.com/hajimehoshi/ebiten/v2/exp/shaderprecomp
```

목적:

```text
Kage
  ↓
shadercollector
  ↓
backend shader source
  ↓
DirectX / Metal native compile
  ↓
runtime shader compilation 감소
```

단:

```text
experimental API
```

이며 Ebitengine 버전을 변경하면 shader를 다시 collect/precompile 해야 합니다. OpenGL driver의 GLSL compile/link까지 완전히 제거되는 것은 아닙니다. ([Ebitengine][2])

---

# 21. Collision

Ebitengine 자체는 ECS나 physics engine을 강제하지 않으므로 간단한 AABB는 직접 구현하기 편합니다.

```go
func overlaps(
	ax, ay, aw, ah,
	bx, by, bw, bh float64,
) bool {
	return ax < bx+bw &&
		ax+aw > bx &&
		ay < by+bh &&
		ay+ah > by
}
```

Circle:

```go
func circleCollision(
	ax, ay, ar,
	bx, by, br float64,
) bool {
	dx := ax - bx
	dy := ay - by

	r := ar + br

	return dx*dx+dy*dy <= r*r
}
```

---

# 22. Animation

전형적인 sprite sheet:

```go
const (
	frameW = 32
	frameH = 32
)

frame := (tick / 6) % frameCount

sx := frame * frameW

sprite := sheet.SubImage(
	image.Rect(
		sx,
		0,
		sx+frameW,
		frameH,
	),
).(*ebiten.Image)

screen.DrawImage(sprite, op)
```

Ebitengine가 고정 TPS이므로 간단한 애니메이션은 tick 기반으로 구현하기 좋습니다.

```go
g.tick++
```

```go
frame := (g.tick / ticksPerFrame) % frameCount
```

---

# 23. Camera

간단한 2D camera:

```go
type Camera struct {
	X     float64
	Y     float64
	Zoom  float64
	Angle float64
}
```

World → Screen:

```go
op.GeoM.Translate(-camera.X, -camera.Y)
op.GeoM.Scale(camera.Zoom, camera.Zoom)

screen.DrawImage(world, op)
```

중심 확대/회전:

```go
op.GeoM.Translate(
	-float64(screenWidth)/2,
	-float64(screenHeight)/2,
)

op.GeoM.Rotate(camera.Angle)
op.GeoM.Scale(camera.Zoom, camera.Zoom)

op.GeoM.Translate(
	float64(screenWidth)/2,
	float64(screenHeight)/2,
)
```

---

# 24. Assets — `embed`

Go의 `embed`와 궁합이 좋습니다.

```go
import _ "embed"

//go:embed assets/player.png
var playerPNG []byte
```

로드:

```go
img, _, err := image.Decode(
	bytes.NewReader(playerPNG),
)
if err != nil {
	panic(err)
}

playerImage := ebiten.NewImageFromImage(img)
```

Folder:

```go
//go:embed assets/*
var assets embed.FS
```

---

# 25. WebAssembly

개발 서버는 공식 문서 기준 다음 방법이 가장 간단합니다.

```bash
go run github.com/hajimehoshi/wasmserve@latest .
```

접속:

```text
http://localhost:8080/
```

직접 빌드:

```bash
GOOS=js GOARCH=wasm \
go build -o game.wasm .
```

Go 1.25 기준 `wasm_exec.js`:

```bash
cp "$(go env GOROOT)/lib/wasm/wasm_exec.js" .
```

Ebitengine 공식 문서 역시 browser embedding에는 `iframe` 방식을 권장합니다. ([Ebitengine][11])

---

# 26. Cross Compile

v2.10 Pure Go Desktop 덕분에 이전보다 간단합니다.

Windows:

```bash
CGO_ENABLED=0 \
GOOS=windows \
GOARCH=amd64 \
go build -o game.exe .
```

Linux:

```bash
CGO_ENABLED=0 \
GOOS=linux \
GOARCH=amd64 \
go build -o game .
```

macOS ARM64:

```bash
CGO_ENABLED=0 \
GOOS=darwin \
GOARCH=arm64 \
go build -o game .
```

단, 타깃 OS에는 그래픽/오디오 등의 **runtime library**가 여전히 필요할 수 있습니다. ([Ebitengine][12])

---

# 27. Mobile

설치:

```bash
go install \
github.com/hajimehoshi/ebiten/v2/cmd/ebitenmobile@latest
```

Android:

```bash
ebitenmobile bind \
-target android \
-javapkg com.example.game \
-o game.aar \
./mobile
```

iOS:

```bash
ebitenmobile bind \
-target ios \
-o Game.xcframework \
./mobile
```

모바일 package에서는:

```go
func init() {
	mobile.SetGame(&game.Game{})
}
```

을 사용하고 직접:

```go
ebiten.RunGame(...)
```

을 호출하지 않습니다. ([Ebitengine][13])

---

# 28. v2.10 중요 변경사항

현재 v2.10 계열에서 특히 기억할 부분은 다음과 같습니다. ([Ebitengine][2])

| 항목                | v2.10                     |
| ----------------- | ------------------------- |
| Go                | **1.25+ 필수**              |
| Windows Desktop   | Pure Go                   |
| macOS Desktop     | **Pure Go**               |
| Linux/BSD Desktop | **Pure Go**               |
| Application VM    | 신규                        |
| Shader precompile | 신규 / experimental         |
| Color Emoji       | `text/v2` 지원              |
| IME               | Linux/Unix/Android/iOS 확대 |
| Kage `range`      | integer / fixed array 지원  |
| Gamepad           | vibration 지원 확대           |
| Hidden Window     | 지원                        |
| Cursor            | `CursorPositionF()` 추가    |

Deprecated:

```text
audio.Player.Close()
    ↓
PauseAndStopReading()

text.Advance()
    ↓
text.AdvanceAt()

textinput.Field
    ↓
textinput.Composer
```

---

# 29. Performance 핵심

Ebitengine에서는 draw 호출 횟수 자체보다 **draw command batching이 깨지는 조건**을 이해하는 것이 중요합니다.

가능하면 연속된 `DrawImage` 호출에서 다음을 동일하게 유지합니다.

```text
Render Target
Blend
Filter
```

그러면 여러 draw가 내부적으로 batch될 수 있습니다. ([Ebitengine][9])

특히 피할 것:

```go
A.DrawImage(B, op)
B.DrawImage(C, op)
```

render source였던 `B`를 다시 변경하면 context restore 관리 비용이 커질 수 있습니다.

또한:

```go
img.ReplacePixels(...)
```

을 매 프레임 과도하게 호출하지 말 것.

```go
img.At(...)
```

도 GPU 결과를 동기화해야 하므로 hot path에서 반복 호출하지 않는 것이 좋습니다. ([Ebitengine][9])

---

# 30. 자주 쓰는 Import

```go
import (
	"github.com/hajimehoshi/ebiten/v2"
	"github.com/hajimehoshi/ebiten/v2/audio"
	"github.com/hajimehoshi/ebiten/v2/audio/mp3"
	"github.com/hajimehoshi/ebiten/v2/audio/vorbis"
	"github.com/hajimehoshi/ebiten/v2/audio/wav"
	"github.com/hajimehoshi/ebiten/v2/colorm"
	"github.com/hajimehoshi/ebiten/v2/ebitenutil"
	"github.com/hajimehoshi/ebiten/v2/inpututil"
	"github.com/hajimehoshi/ebiten/v2/text/v2"
	"github.com/hajimehoshi/ebiten/v2/vector"
)
```

---

# 31. API Quick Reference

```text
GAME
────────────────────────────────────
ebiten.RunGame(game)
Update()
Draw(screen)
Layout(w, h)

TIME
────────────────────────────────────
ebiten.SetTPS()
ebiten.TPS()
ebiten.ActualTPS()
ebiten.ActualFPS()

WINDOW
────────────────────────────────────
ebiten.SetWindowSize()
ebiten.SetWindowTitle()
ebiten.SetWindowResizable()
ebiten.SetFullscreen()
ebiten.SetVsyncEnabled()

IMAGE
────────────────────────────────────
ebiten.NewImage()
ebiten.NewImageFromImage()
Image.Fill()
Image.Clear()
Image.Size()
Image.SubImage()
Image.DrawImage()

TRANSFORM
────────────────────────────────────
GeoM.Translate()
GeoM.Scale()
GeoM.Rotate()

COLOR
────────────────────────────────────
ColorScale.Scale()
ColorScale.ScaleAlpha()

KEYBOARD
────────────────────────────────────
ebiten.IsKeyPressed()
inpututil.IsKeyJustPressed()
inpututil.IsKeyJustReleased()
inpututil.KeyPressDuration()
inpututil.AppendPressedKeys()

MOUSE
────────────────────────────────────
ebiten.CursorPosition()
ebiten.CursorPositionF()
ebiten.IsMouseButtonPressed()
ebiten.Wheel()

TEXT INPUT
────────────────────────────────────
ebiten.AppendInputChars()

TOUCH
────────────────────────────────────
ebiten.AppendTouchIDs()
ebiten.TouchPosition()

GAMEPAD
────────────────────────────────────
ebiten.AppendGamepadIDs()
ebiten.IsStandardGamepadButtonPressed()
ebiten.StandardGamepadAxisValue()

VECTOR
────────────────────────────────────
vector.FillRect()
vector.FillCircle()
vector.StrokeRect()
vector.StrokeCircle()
vector.StrokeLine()

TEXT
────────────────────────────────────
text.NewGoTextFaceSource()
text.GoTextFace
text.Draw()
text.Measure()
text.AdvanceAt()

AUDIO
────────────────────────────────────
audio.NewContext()
Context.NewPlayer()
Player.Play()
Player.Pause()
Player.Rewind()
Player.SetPosition()
Player.SetVolume()
Player.PauseAndStopReading()

SHADER
────────────────────────────────────
ebiten.NewShader()
Image.DrawRectShader()
Image.DrawTrianglesShader()
```

공식 Cheat Sheet도 현재 `Game`, `Image`, `DrawImageOptions`, `GeoM`, 입력 API를 중심으로 구성되어 있습니다. 위 버전은 여기에 **v2.10의 `text/v2`, 최신 vector API, audio deprecation, Kage 변경, WebAssembly/모바일/성능 포인트**를 추가한 실전용 확장판입니다. ([Ebitengine][4])

참고 문서: [Ebitengine v2.10 Release Notes](https://ebitengine.org/en/documents/2.10.html?utm_source=chatgpt.com) · [공식 Cheat Sheet](https://ebitengine.org/en/documents/cheatsheet.html?utm_source=chatgpt.com) · [API Reference](https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2?utm_source=chatgpt.com) · [Performance Tips](https://ebitengine.org/en/documents/performancetips.html?utm_source=chatgpt.com)

다음 단계로는 이 CheatSheet를 기반으로 **`Ebitengine v2.10 프로젝트 스타터 템플릿`**을 만들어 두면 좋습니다. `Scene Manager + Camera + Sprite Animation + Input Manager + Audio + Kage Shader + embed Assets`까지 한 번에 쓸 수 있는 구조로 확장할 수 있습니다.

[1]: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2?utm_source=chatgpt.com "ebiten package - github.com/hajimehoshi/ebiten/v2 - Go Packages"
[2]: https://ebitengine.org/en/documents/2.10.html?utm_source=chatgpt.com "Ebitengine 2.10 Release Notes - Ebitengine"
[3]: https://ebitengine.org/en/documents/cheatsheet.html "Cheat Sheet - Ebitengine"
[4]: https://ebitengine.org/en/documents/cheatsheet.html?utm_source=chatgpt.com "Cheat Sheet - Ebitengine"
[5]: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2/inpututil?utm_source=chatgpt.com "inpututil package - github.com/hajimehoshi/ebiten/v2/inpututil - Go Packages"
[6]: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2%40v2.9.9/vector?utm_source=chatgpt.com "vector package - github.com/hajimehoshi/ebiten/v2/vector - Go Packages"
[7]: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2%40v2.10.1/text/v2?utm_source=chatgpt.com "text package - github.com/hajimehoshi/ebiten/v2/text/v2 - Go Packages"
[8]: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2%40v2.10.1/audio?utm_source=chatgpt.com "audio package - github.com/hajimehoshi/ebiten/v2/audio - Go Packages"
[9]: https://ebitengine.org/en/documents/performancetips.html?utm_source=chatgpt.com "Performance Tips - Ebitengine"
[10]: https://ebitengine.org/en/documents/shader.html?utm_source=chatgpt.com "Shader - Ebitengine"
[11]: https://ebitengine.org/en/documents/webassembly.html?utm_source=chatgpt.com "WebAssembly - Ebitengine"
[12]: https://ebitengine.org/en/documents/install.html?utm_source=chatgpt.com "Install - Ebitengine"
[13]: https://ebitengine.org/en/documents/mobile.html?utm_source=chatgpt.com "Mobile - Ebitengine"
