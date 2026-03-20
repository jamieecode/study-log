# MUI DataGrid v8 Toolbar Migration Issue

## **배경**

MUI DataGrid v8 업그레이드 후 toolbar 관련 기능이 동작하지 않는 문제가 발생했다. 주요 증상은 다음과 같다:

- 등록 버튼이 화면에 보이지 않음
- CSV 내보내기 버튼이 사라짐
- `MUI X: Missing context. Toolbar subcomponents must be placed within a <Toolbar /> component.` 에러 발생
- 버튼이 toolbar 오른쪽 끝에 표시됨 (기존에는 왼쪽)

---

## **v8 주요 변경사항**

### **1. `showToolbar` prop 추가 (Breaking Change)**

v7까지는 `slots={{ toolbar: MyToolbar }}`만 설정하면 toolbar가 렌더링됐지만, v8부터는 `showToolbar` prop을 명시해야만 toolbar가 표시된다.

```tsx
// ❌ v7 방식 - v8에서 toolbar가 보이지 않음
<DataGrid slots={{ toolbar: MyToolbar }} />

// ✅ v8 방식
<DataGrid showToolbar slots={{ toolbar: MyToolbar }} />
```

### **2. Toolbar 기본 정렬이 `flex-end`로 변경**

v8의 새 `Toolbar` 내부 CSS가 `justifyContent: 'end'`로 변경되어 버튼이 오른쪽으로 밀린다.

```tsx
// ✅ 왼쪽 정렬 유지
<GridToolbarContainer sx={{ justifyContent: "flex-start" }}>
  <Button>등록</Button>
</GridToolbarContainer>
```

### **3. Toolbar Subcomponent Context 필요 (Breaking Change)**

`GridToolbarExport`, `GridToolbarQuickFilter` 등 subcomponent는 반드시 `GridToolbarContainer`(또는 새 `Toolbar`) 안에 있어야 한다. 일반 `<div>`, `<Grid container>` 안에 배치하면 런타임 에러가 발생한다.

```tsx
// ❌ v8에서 에러 발생
<Grid container>
  <GridToolbarExport />
  <GridToolbarQuickFilter />
</Grid>

// ✅ v8 방식
<GridToolbarContainer>
  <GridToolbarExport />
  <GridToolbarQuickFilter />
</GridToolbarContainer>
```