# .pen File Format Specification

> Source: [The .pen Format](https://docs.pencil.dev/for-developers/the-pen-format)

`.pen` 파일은 JSON 기반 텍스트 포맷으로, 디자인을 코드처럼 다룰 수 있다. Git diff/merge/branch 가능.

---

## Root Structure

```json
{
  "version": "1.0",
  "themes": { "mode": ["light", "dark"] },
  "variables": { ... },
  "children": [ ... ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | 포맷 버전 |
| `themes` | object? | 테마 축 정의. `{"축이름": ["값1", "값2"]}` |
| `variables` | object? | 문서 전역 변수 (디자인 토큰) |
| `children` | array | 캔버스에 배치된 최상위 오브젝트 배열 |

---

## Object Types

모든 오브젝트의 공통 속성:

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | 고유 식별자 (슬래시 불가) |
| `type` | string | 오브젝트 타입 |
| `x`, `y` | number | 좌상단 기준 좌표 |
| `name` | string? | 표시 이름 |
| `reusable` | boolean? | `true`이면 재사용 가능 컴포넌트 |
| `opacity` | number \| variable | 0–1 |
| `rotation` | number? | 반시계 방향 각도 |
| `flipX`, `flipY` | boolean? | 좌우/상하 반전 |
| `enabled` | boolean \| variable? | 활성 여부 |
| `theme` | object? | 테마 설정 `{"축": "값"}` |
| `metadata` | object? | 커스텀 메타데이터 |

### rectangle

```json
{ "type": "rectangle", "width": 200, "height": 100, "cornerRadius": 8 }
```

- `width`, `height`: number 또는 SizingBehavior
- `cornerRadius`: 단일값 또는 4값 배열 `[topLeft, topRight, bottomRight, bottomLeft]`
- `fill`, `stroke`, `effect` 속성 사용 가능

### ellipse

```json
{ "type": "ellipse", "width": 100, "height": 100, "innerRadius": 0 }
```

- `innerRadius`: 안쪽/바깥 반지름 비율 (0=채움, 1=빈 원)
- `startAngle`: 호 시작 각도
- `sweepAngle`: 호 길이 (-360 ~ 360)

### line

```json
{ "type": "line", "width": 200, "height": 0 }
```

- `width`, `height`: 바운딩 박스 정의

### polygon

```json
{ "type": "polygon", "width": 100, "height": 100, "polygonCount": 6 }
```

- `polygonCount`: 변의 수
- `cornerRadius`: 선택적 라운딩

### path

```json
{ "type": "path", "geometry": "M0 0 L100 50 L0 100 Z", "fillRule": "nonzero" }
```

- `geometry`: SVG path 문자열
- `fillRule`: `"nonzero"` | `"evenodd"`

### text

```json
{
  "type": "text",
  "content": "Hello World",
  "fontSize": 16,
  "fontFamily": "Inter",
  "fontWeight": 400,
  "textAlign": "left"
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `content` | string \| styled array | 텍스트 내용 |
| `textGrowth` | `"auto"` \| `"fixed-width"` \| `"fixed-width-height"` | 크기 조절 방식 |
| `fontSize` | number | 글꼴 크기 |
| `fontFamily` | string | 글꼴 |
| `fontWeight` | number | 굵기 |
| `fontStyle` | string | 스타일 (italic 등) |
| `letterSpacing` | number | 자간 |
| `lineHeight` | number | 행간 |
| `textAlign` | `"left"` \| `"center"` \| `"right"` \| `"justify"` | 수평 정렬 |
| `textAlignVertical` | `"top"` \| `"middle"` \| `"bottom"` | 수직 정렬 |
| `underline` | boolean | 밑줄 |
| `strikethrough` | boolean | 취소선 |
| `href` | string? | 하이퍼링크 |

### frame

```json
{
  "type": "frame",
  "width": 390, "height": 844,
  "layout": "vertical",
  "gap": 16,
  "padding": [24, 16],
  "children": [ ... ],
  "clip": true
}
```

- `children`: 중첩 오브젝트 배열
- `clip`: overflow 클리핑
- `placeholder`: boolean
- `slot`: 권장 컴포넌트 ID 배열
- 레이아웃 속성 사용 가능 (아래 Layout System 참조)

### group

```json
{ "type": "group", "children": [ ... ] }
```

- `children`: 중첩 오브젝트 배열
- `width`, `height`: SizingBehavior만 사용
- 레이아웃 속성 사용 가능

### ref (인스턴스)

```json
{
  "type": "ref",
  "ref": "component-id",
  "descendants": {
    "child-id": { "fill": "#ff0000" }
  }
}
```

- `ref`: 참조할 컴포넌트 ID
- 속성 직접 추가로 오버라이드
- `descendants`로 하위 요소 커스텀 (아래 Components & Instances 참조)

### iconFont

```json
{
  "type": "iconFont",
  "iconFontName": "home",
  "iconFontFamily": "lucide"
}
```

- `iconFontFamily`: `"lucide"` | `"feather"` | `"Material Symbols Outlined"` | `"Material Symbols Rounded"` | `"Material Symbols Sharp"` | `"phosphor"`
- `weight`: 100–700 (가변 폰트용)

---

## Layout System

frame과 group에 적용되는 Flexbox 기반 레이아웃:

| Property | Values | Default |
|----------|--------|---------|
| `layout` | `"none"` \| `"vertical"` \| `"horizontal"` | frame: `"horizontal"`, group: `"none"` |
| `gap` | number | 자식 요소 간 간격 |
| `padding` | number \| [h, v] \| [top, right, bottom, left] | 내부 여백 |
| `justifyContent` | `"start"` \| `"center"` \| `"end"` \| `"space_between"` \| `"space_around"` | 주축 정렬 |
| `alignItems` | `"start"` \| `"center"` \| `"end"` | 교차축 정렬 |
| `layoutIncludeStroke` | boolean | 스트로크를 레이아웃 계산에 포함 |

### SizingBehavior

`width`와 `height`에 숫자 대신 사용 가능한 특수값:

| Value | Description |
|-------|-------------|
| `"fit_content"` | 자식 크기에 맞춤 |
| `"fit_content(100)"` | 자식 크기에 맞추되 폴백 크기 지정 |
| `"fill_container"` | 부모 크기에 맞춤 |

---

## Fills

단일 fill 또는 fill 배열. fill이 문자열이면 solid color.

### Solid Color

```json
"fill": "#3b82f6"
```

또는 상세 형식:

```json
"fill": { "type": "color", "color": "#3b82f6", "enabled": true, "blendMode": "normal" }
```

색상 포맷: `#RGB`, `#RRGGBB`, `#RRGGBBAA`

### Gradient

```json
"fill": {
  "type": "gradient",
  "gradientType": "linear",
  "rotation": 90,
  "colors": [
    { "color": "#3b82f6", "position": 0 },
    { "color": "#8b5cf6", "position": 1 }
  ]
}
```

- `gradientType`: `"linear"` | `"radial"` | `"angular"`
- `center`, `size`: radial/angular 그래디언트용
- `rotation`: 각도
- `colors`: 색상 스탑 배열

### Image

```json
"fill": { "type": "image", "url": "path/to/image.png", "mode": "fill" }
```

- `mode`: `"stretch"` | `"fill"` | `"fit"`
- `opacity`: number

### Mesh Gradient

```json
"fill": { "type": "mesh_gradient", "columns": 2, "rows": 2, "colors": [...], "points": [...] }
```

---

## Strokes

```json
"stroke": {
  "fill": "#e5e7eb",
  "align": "inside",
  "thickness": 1,
  "join": "miter",
  "cap": "none",
  "dashPattern": [4, 2]
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `align` | `"inside"` \| `"center"` \| `"outside"` | 스트로크 위치 |
| `thickness` | number \| `{top, right, bottom, left}` | 두께 (변별 가능) |
| `join` | `"miter"` \| `"bevel"` \| `"round"` | 모서리 연결 |
| `cap` | `"none"` \| `"round"` \| `"square"` | 끝 처리 |
| `miterAngle` | number | miter 한계 각도 |
| `dashPattern` | number[] | 대시 패턴 |
| `fill` | fill | 스트로크 색상 (멀티 fill 가능) |

---

## Effects

단일 이펙트 또는 이펙트 배열.

### Blur

```json
{ "type": "blur", "radius": 10 }
```

### Background Blur

```json
{ "type": "background_blur", "radius": 20 }
```

### Shadow

```json
{
  "type": "shadow",
  "shadowType": "outer",
  "offset": { "x": 0, "y": 4 },
  "blur": 6,
  "spread": 0,
  "color": "#00000040"
}
```

- `shadowType`: `"inner"` | `"outer"`

### BlendMode (fill/shadow에서 사용)

`"normal"` | `"darken"` | `"multiply"` | `"linearBurn"` | `"colorBurn"` | `"light"` | `"screen"` | `"linearDodge"` | `"colorDodge"` | `"overlay"` | `"softLight"` | `"hardLight"` | `"difference"` | `"exclusion"` | `"hue"` | `"saturation"` | `"color"` | `"luminosity"`

---

## Components & Instances

### 컴포넌트 생성

오브젝트에 `"reusable": true` 설정:

```json
{
  "type": "frame",
  "id": "btn-primary",
  "reusable": true,
  "children": [
    { "type": "text", "id": "btn-label", "content": "Button" }
  ]
}
```

### 인스턴스 생성

`ref` 타입으로 컴포넌트 참조:

```json
{
  "type": "ref",
  "ref": "btn-primary"
}
```

### 속성 오버라이드

ref 오브젝트에 직접 속성 추가하면 컴포넌트 기본값 오버라이드:

```json
{
  "type": "ref",
  "ref": "btn-primary",
  "fill": "#ef4444"
}
```

### descendants 오버라이드

하위 요소의 속성을 커스텀:

```json
{
  "type": "ref",
  "ref": "btn-primary",
  "descendants": {
    "btn-label": { "content": "Submit", "fill": "#ffffff" }
  }
}
```

**규칙:**
- `id`, `type`, `children`은 오버라이드 불가 (부분 오버라이드)
- `type`이 포함되면 **완전 교체** (새 오브젝트로 대체)
- `children`이 포함되면 자식 전체 교체
- 중첩 경로: `"parent-id/child-id"` 슬래시 표기법

### Slots

frame에 `slot` 속성으로 권장 컴포넌트 지정:

```json
{
  "type": "frame",
  "id": "card-content",
  "placeholder": true,
  "slot": ["card-body", "card-footer"]
}
```

---

## Variables & Themes

### Variable 정의

```json
"variables": {
  "primary": {
    "type": "color",
    "value": "#3b82f6"
  },
  "spacing-md": {
    "type": "number",
    "value": 16
  },
  "show-badge": {
    "type": "boolean",
    "value": true
  },
  "font-family": {
    "type": "string",
    "value": "Inter"
  }
}
```

### 4가지 Variable Type

| Type | Value | 용도 |
|------|-------|------|
| `color` | hex string | 색상 토큰 |
| `number` | numeric | 간격, 크기, 폰트 크기 |
| `boolean` | true/false | 가시성, 활성 상태 |
| `string` | text | 폰트명, 텍스트 값 |

### `$variable-name` 바인딩

속성값에 `$`접두사로 변수 참조:

```json
{
  "type": "rectangle",
  "fill": "$primary",
  "width": "$spacing-md"
}
```

### 테마별 값

```json
"variables": {
  "bg-color": {
    "type": "color",
    "value": [
      { "value": "#ffffff", "theme": { "mode": "light" } },
      { "value": "#1a1a1a", "theme": { "mode": "dark" } }
    ]
  }
}
```

테마 축 정의:

```json
"themes": {
  "mode": ["light", "dark"]
}
```

오브젝트에 테마 적용:

```json
{ "type": "frame", "theme": { "mode": "dark" }, ... }
```

마지막으로 매칭되는 테마 값이 적용됨.

---

## .pen → Code 매핑

| .pen Type | HTML/React | Notes |
|-----------|------------|-------|
| `frame` | `<div>` | 레이아웃 컨테이너. flexbox 속성 → CSS flex |
| `text` | `<p>`, `<span>`, `<h1>`–`<h6>` | 내용/맥락에 따라 태그 결정 |
| `rectangle` | `<div>` | CSS로 스타일링 |
| `ellipse` | `<div>` with `border-radius: 50%` | 또는 SVG |
| `ref` | 컴포넌트 import | `<ButtonPrimary />` 등 |
| `group` | `<div>` 또는 Fragment | 레이아웃 없으면 Fragment |
| `iconFont` | 아이콘 컴포넌트 | `<Home />` (Lucide 등) |
| `path` | `<svg>` | SVG path 그대로 |
| `$variable` | CSS custom property | `var(--primary)` 또는 Tailwind 토큰 |
