# Dark Mode

## 📌 모바일 브라우저별 다크모드 동작 차이 이슈

이번 프로젝트에서는 **별도의 다크모드를 구현하지 않았음에도**, 모바일 테스트 중 브라우저에 따라 화면 표시가 다르게 보이는 현상을 발견하여 기록한다.

모바일 테스트 중

- **삼성 인터넷 앱**에서는 다크 테마 선택 시 웹페이지 전체가 다크모드처럼 표시되었고
- **크롬 앱**에서는 동일한 환경에서도 라이트 모드 그대로 표시되었다

이로 인해 초기에는 “디자이너가 다크모드를 적용한 것이 아닌가”라고 오해했으나, 실제로는 프로젝트 내부에 다크모드 관련 코드나 스타일은 존재하지 않았다.

확인 결과, 이는 **브라우저의 다크모드 처리 방식 차이**에서 발생한 현상이었다.

---

## 🌙 삼성 인터넷 브라우저의 특징

삼성 인터넷은 **브라우저 레벨에서 웹페이지를 강제로 다크 테마로 변환**하는 기능을 제공한다.

- 시스템 다크 모드 또는 브라우저 다크 모드가 활성화되면
    
    → **웹사이트 콘텐츠 색상을 브라우저가 직접 재연산하여 전체 페이지를 어둡게 표시**
    
- 웹사이트가 다크모드를 전혀 구현하지 않았더라도 동작함
    
    → CSS `prefers-color-scheme` 미지원 사이트도 **강제로 다크 처리**
    

즉, **웹사이트 구현 여부와 관계없이 브라우저가 후처리 방식으로 다크모드를 적용**한다.

이로 인해

- 배경색이 어두워지고
- 텍스트 색이 밝게 바뀌며
- 일부 이미지나 아이콘 색상이 의도와 다르게 보일 수 있다

---

## 🌞 크롬 브라우저의 특징

크롬은 다크모드를 **브라우저 UI에만 적용**하며, 웹사이트 콘텐츠에는 기본적으로 영향을 주지 않는다.

- 다크모드 적용 범위:
    
    → 주소창, 탭, 메뉴 등 **브라우저 인터페이스 영역만 변경**
    
- 웹페이지 내부는 사이트 구현을 따름
    
    → 사이트가 `prefers-color-scheme: dark` 기반 다크모드를 구현한 경우에만 다크 테마 적용
    
    → 그렇지 않으면 **항상 라이트 모드로 표시**
    

즉, 크롬은 **“브라우저 테마”와 “웹페이지 테마”를 완전히 분리**해서 처리한다.

---

## 🔍 브라우저별 동작 차이 요약

| 구분 | 삼성 인터넷 | 크롬 |
| --- | --- | --- |
| 브라우저 UI 다크모드 | 적용 | 적용 |
| 웹페이지 다크모드 자동 적용 | **브라우저가 강제 변환** | 적용 안 함 |
| 사이트에 다크모드 구현 없을 때 | 다크처럼 보임 | 라이트 그대로 |
| 색상 처리 방식 | 브라우저 후처리(강제 다크) | 사이트 CSS 기준 |

---

## 📌 Difference in Dark Mode Behavior Between Mobile Browsers

In this project, **we did not implement a dedicated dark mode**, but during mobile testing we discovered that the UI appeared differently depending on the browser. This is being documented as a technical note.

During testing:

- In **Samsung Internet**, when dark theme was enabled, the entire webpage appeared in dark mode.
- In **Chrome**, under the same device and system settings, the page remained in light mode.

At first, this led to confusion, as it seemed like a dark mode had been applied by design. However, after reviewing the codebase, we confirmed that **there was no dark mode–related implementation** in the project.

The root cause turned out to be **differences in how browsers handle dark mode rendering**.

---

## 🌙 Characteristics of Samsung Internet

Samsung Internet provides a feature that **forces dark mode at the browser level**, regardless of website implementation.

- When system dark mode or browser dark mode is enabled
    
    → The browser **recalculates and transforms website colors** to make the entire page appear dark.
    
- This works even if the website does not support dark mode
    
    → Sites without CSS `prefers-color-scheme` support are still **forcibly rendered in dark mode**
    

In other words, **the browser applies post-processing to simulate dark mode**, independent of the site’s design.

As a result:

- Backgrounds become darker
- Text colors are adjusted to be lighter
- Some images, icons, or brand colors may appear distorted or unintended

---

## 🌞 Characteristics of Chrome

Chrome applies dark mode **only to the browser UI**, not to the webpage content by default.

- Dark mode affects:
    
    → Address bar, tabs, menus, and other **browser interface elements**
    
- Webpage content follows the site’s implementation:
    - If the site supports dark mode using `prefers-color-scheme: dark`, it will render in dark theme
    - Otherwise, the page remains in **light mode**

In short, Chrome **clearly separates “browser theme” from “website theme.”**

---

## 🔍 Summary of Browser Differences

| Category | Samsung Internet | Chrome |
| --- | --- | --- |
| Browser UI Dark Mode | Applied | Applied |
| Automatic Dark Mode for Web Content | **Forced by browser** | Not applied |
| When site has no dark mode | Appears dark | Remains light |
| Color processing method | Browser post-processing | Website CSS-based |