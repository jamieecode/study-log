# iOS Safari에서 input focus 시 화면 확대(Zoom-in) 이슈 대응

### 문제 현상

모바일 웹 프로젝트에서 기본 폰트 크기를 **14px**로 사용하고 있었는데, **iOS Safari 환경에서 `input`, `textarea` 요소를 터치하여 포커스할 경우 화면이 자동으로 확대(zoom-in)** 되는 현상이 발생했다.

이는 iOS Safari의 기본 동작으로, **입력 필드의 폰트 크기가 16px 미만일 때 가독성을 보장하기 위해 브라우저가 자동 확대**를 수행하기 때문이라고 한다.

---

### 원인

- iOS Safari는 사용자의 입력 편의성을 위해
    
    **폼 요소(`input`, `textarea`)의 font-size가 16px 미만이면 자동 줌**을 적용한다.
    
- 이 동작은 브라우저 정책이며, CSS나 JS 오류가 아니다.

---

### 해결 방법 검토

### 1️⃣ 폰트 사이즈를 16px 이상으로 조정 (✅ 적용)

**장점**

- 브라우저 기본 동작을 존중하는 방식
- 추가적인 접근성 문제 없음
- 가장 안정적이고 표준적인 해결 방법

**단점**

- 기존 디자인 시스템(14px 기준)과의 차이 발생 가능

👉 **이번 프로젝트에서는 접근성과 안정성을 고려해 해당 방법을 채택**

---

### 2️⃣ viewport meta 태그로 확대 제한

```html
<metaname="viewport"content="width=device-width, initial-scale=1, maximum-scale=1">
```

**장점**

- 포커스 시 확대 현상을 강제로 방지 가능

**단점**

- 사용자의 수동 확대(핀치 줌)까지 제한됨
- **웹 접근성(WCAG) 측면에서 권장되지 않는 방식**
- 저시력 사용자에게 불편을 초래할 수 있음

👉 접근성 문제로 인해 **적용하지 않음**

---

### 3️⃣ `transform: scale()`을 이용한 우회 처리

```css
input {
	font-size:16px;
	transform:scale(0.875);/* 14px처럼 보이도록 축소 */
	transform-origin: left top;
}
```

**원리**

- 실제 폰트 크기는 16px로 유지하여 Safari의 확대 조건을 회피
- 시각적으로만 축소하여 기존 디자인과 유사하게 보이도록 처리

**장점**

- 디자인 유지 가능

**단점**

- 레이아웃 계산 복잡도 증가
- 클릭 영역, caret 위치, line-height 등 UI 어긋남 발생 가능
- 유지보수 비용 증가

👉 일정 및 안정성을 고려해 **이번 프로젝트에서는 적용하지 않음**

---

### Issue

In this mobile web project, the base font size was **14px**. On **iOS Safari**, focusing on `input` or `textarea` fields caused the screen to **automatically zoom in**.

This happens because iOS Safari automatically enlarges form fields when their **font size is smaller than 16px**, ensuring text readability during input.

---

### Cause

- iOS Safari applies **automatic zoom** when a focused form control has a `font-size` less than **16px**
- This is a **browser behavior**, not a layout or script bug

---

### Considered Solutions

### 1️⃣ Increase font size to 16px or larger (✅ Adopted)

**Pros**

- Respects browser accessibility behavior
- No negative impact on zoom accessibility
- Most stable and standards-friendly solution

**Cons**

- May slightly differ from the original design system (14px baseline)

👉 Chosen for stability and accessibility

---

### 2️⃣ Disable zoom via viewport meta tag

```html
<metaname="viewport"content="width=device-width, initial-scale=1, maximum-scale=1">
```

**Pros**

- Prevents auto zoom behavior

**Cons**

- Also disables user pinch-to-zoom
- **Not recommended for accessibility (WCAG concerns)**
- Can negatively affect low-vision users

👉 Rejected due to accessibility concerns

---

### 3️⃣ Use `transform: scale()` as a workaround

```css
input {
font-size:16px;
transform:scale(0.875);
transform-origin: left top;
}
```

**Concept**

- Keeps the actual font size at 16px to avoid Safari zoom
- Visually scales it down to look like 14px

**Pros**

- Preserves original visual design

**Cons**

- Can cause layout inconsistencies
- Caret position, click area, and line-height issues
- Higher maintenance complexity

👉 Not applied due to time constraints and stability concerns

---