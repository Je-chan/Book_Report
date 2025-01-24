# CSS

Next.js는 CSS를 사용하여 애플리케이션을 스타일링하는 여러 가지 방법을 제공합니다:

- [CSS 모듈](#css-모듈)
- [전역 CSS](#전역-css)
- [외부 스타일시트](#외부-스타일시트)
- [Tailwind CSS](/docs/app/guides/tailwind-css.md)
- [Sass](/docs/app/guides/sass.md)
- [CSS-in-JS](/docs/app/guides/css-in-js.md)

## CSS 모듈

CSS 모듈은 고유한 클래스 이름을 생성하여 CSS를 로컬 범위로 지정합니다. 이를 통해 이름 충돌을 걱정하지 않고 서로 다른 파일에서 동일한 클래스를 사용할 수 있습니다.

CSS 모듈 사용을 시작하려면 `.module.css` 확장자로 새 파일을 만들고 `app` 디렉토리 내의 모든 컴포넌트로 가져오세요:

```css
/* app/blog/blog.module.css */
.blog {
  padding: 24px;
}
```

```tsx
// app/blog/page.tsx
import styles from './blog.module.css'

export default function Page() {
  return <main className={styles.blog}></main>
}
```

> 💡 **실무 팁**
>
> **CSS 모듈 활용 전략:**
> - **명명 규칙**: `component-name.module.css` 형태로 일관된 네이밍을 사용하세요
> - **컴포넌트별 분리**: 각 컴포넌트마다 별도의 CSS 모듈을 만들어 유지보수성을 높이세요
> - **복합 클래스**: `clsx`나 `classnames` 라이브러리로 조건부 스타일링을 효율적으로 처리하세요

## 전역 CSS

전역 CSS를 사용하여 애플리케이션 전체에 스타일을 적용할 수 있습니다.

`app/global.css` 파일을 만들고 루트 레이아웃에서 가져와서 애플리케이션의 **모든 라우트**에 스타일을 적용하세요:

```css
/* app/global.css */
body {
  padding: 20px 20px 60px;
  max-width: 680px;
  margin: 0 auto;
}
```

```tsx
// app/layout.tsx
// 이러한 스타일은 애플리케이션의 모든 라우트에 적용됩니다
import './global.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

> **알아두면 좋은 점**: 전역 스타일은 `app` 디렉토리 내의 모든 레이아웃, 페이지 또는 컴포넌트로 가져올 수 있습니다. 하지만 Next.js가 Suspense와 통합하기 위해 React의 스타일시트 지원을 사용하므로, 현재 라우트 간 탐색 시 스타일시트를 제거하지 않아 충돌이 발생할 수 있습니다. *진정한* 전역 CSS에는 전역 스타일을, 범위가 지정된 CSS에는 **CSS 모듈**을 사용하는 것을 권장합니다.

## 외부 스타일시트

외부 패키지에서 발행한 스타일시트는 함께 위치한 컴포넌트를 포함하여 `app` 디렉토리 어디에서나 가져올 수 있습니다:

```tsx
// app/layout.tsx
import 'bootstrap/dist/css/bootstrap.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className="container">{children}</body>
    </html>
  )
}
```

> **알아두면 좋은 점**: React 19에서는 `<link rel="stylesheet" href="..." />`도 사용할 수 있습니다. 자세한 정보는 React `link` 문서를 참조하세요.

> 💡 **실무 팁**
>
> **외부 스타일시트 관리:**
> - **선택적 임포트**: 필요한 컴포넌트만 임포트하여 번들 크기를 최적화하세요
> - **CDN 활용**: 자주 사용되는 라이브러리는 CDN을 고려해보세요
> - **버전 관리**: 외부 라이브러리 업데이트 시 스타일 호환성을 확인하세요

## 순서 지정 및 병합

Next.js는 프로덕션 빌드 중에 스타일시트를 자동으로 청크화(병합)하여 CSS를 최적화합니다. **CSS의 순서**는 **코드에서 스타일을 가져오는 순서**에 따라 달라집니다.

예를 들어, `<BaseButton>`이 `page.module.css`보다 먼저 가져와지므로 `base-button.module.css`가 `page.module.css`보다 먼저 순서가 지정됩니다:

```tsx
// page.tsx
import { BaseButton } from './base-button'
import styles from './page.module.css'

export default function Page() {
  return <BaseButton className={styles.primary} />
}
```

```tsx
// base-button.tsx
import styles from './base-button.module.css'

export function BaseButton() {
  return <button className={styles.primary} />
}
```

### 권장사항

CSS 순서를 예측 가능하게 유지하려면:

- CSS 임포트를 단일 JavaScript 또는 TypeScript 진입 파일에 포함하도록 노력하세요
- 애플리케이션의 루트에서 전역 스타일과 Tailwind 스타일시트를 가져오세요
- 중첩된 컴포넌트에는 전역 스타일 대신 CSS 모듈을 사용하세요
- CSS 모듈에 일관된 명명 규칙을 사용하세요. 예: `<name>.tsx`보다 `<name>.module.css` 사용
- 공유 스타일을 공유 컴포넌트로 추출하여 중복 임포트를 방지하세요
- ESLint의 `sort-imports`와 같이 임포트를 자동 정렬하는 린터나 포매터를 끄세요
- `next.config.js`의 `cssChunking` 옵션을 사용하여 CSS가 청크되는 방식을 제어할 수 있습니다

> 💡 **실무 팁**
>
> **CSS 순서 관리 전략:**
> - **임포트 순서 일관성**: 팀 내에서 임포트 순서에 대한 컨벤션을 정하고 준수하세요
> - **스타일 우선순위**: 더 구체적인 스타일이 나중에 로드되도록 구조를 설계하세요
> - **빌드 테스트**: 개발 환경과 프로덕션 환경에서 스타일이 다르게 적용될 수 있으므로 빌드 후 확인하세요

## 개발 vs 프로덕션

- 개발 환경(`next dev`)에서는 CSS 업데이트가 Fast Refresh로 즉시 적용됩니다
- 프로덕션 환경(`next build`)에서는 모든 CSS 파일이 자동으로 **여러 개의 압축되고 코드 분할된** `.css` 파일로 연결되어 라우트에 최소한의 CSS만 로드되도록 합니다
- 프로덕션에서는 JavaScript가 비활성화되어도 CSS가 로드되지만, 개발에서는 Fast Refresh를 위해 JavaScript가 필요합니다
- CSS 순서는 개발 환경에서 다르게 작동할 수 있으므로, 항상 빌드(`next build`)를 확인하여 최종 CSS 순서를 검증하세요

> **개발 vs 프로덕션 차이점**
>
> 개발 환경에서는 빠른 개발 경험을 위해 스타일이 개별적으로 로드되지만, 프로덕션에서는 성능 최적화를 위해 번들링됩니다. 이로 인해 스타일 적용 순서가 달라질 수 있으므로 배포 전 반드시 프로덕션 빌드를 테스트하세요.

## 다음 단계

애플리케이션에서 CSS를 사용할 수 있는 대안적인 방법에 대해 자세히 알아보세요.

- **Tailwind CSS** - Tailwind CSS를 사용하여 Next.js 애플리케이션 스타일링
- **Sass** - Sass를 사용하여 Next.js 애플리케이션 스타일링
- **CSS-in-JS** - Next.js와 함께 CSS-in-JS 라이브러리 사용

> 📚 **스타일링 방법 선택 가이드**
>
> **프로젝트에 맞는 스타일링 방법 선택:**
> - **CSS 모듈**: 전통적인 CSS에 익숙하고 범위 지정이 필요한 경우
> - **Tailwind CSS**: 유틸리티 우선 접근법을 선호하고 빠른 프로토타이핑이 필요한 경우
> - **CSS-in-JS**: 동적 스타일링과 테마 지원이 중요한 경우
> - **Sass**: 변수, 믹스인 등 고급 CSS 기능이 필요한 경우

---

**원문**: [Next.js 공식 문서 - CSS](https://nextjs.org/docs/app/getting-started/css)