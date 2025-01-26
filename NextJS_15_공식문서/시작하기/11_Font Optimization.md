# 폰트 최적화

`next/font` 모듈은 폰트를 자동으로 최적화하고 개선된 프라이버시와 성능을 위해 외부 네트워크 요청을 제거합니다.

모든 폰트 파일에 대한 **내장 셀프 호스팅**을 포함합니다. 이는 레이아웃 시프트 없이 웹 폰트를 최적으로 로드할 수 있다는 것을 의미합니다.

`next/font` 사용을 시작하려면 [`next/font/local`](#로컬-폰트) 또는 [`next/font/google`](#google-폰트)에서 임포트하고, 적절한 옵션으로 함수로 호출한 다음, 폰트를 적용하려는 요소의 `className`을 설정하세요. 예를 들어:

```tsx
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={geist.className}>
      <body>{children}</body>
    </html>
  )
}
```

폰트는 사용된 컴포넌트에 범위가 지정됩니다. 전체 애플리케이션에 폰트를 적용하려면 [루트 레이아웃](/docs/app/api-reference/file-conventions/layout.md#root-layout)에 추가하세요.

> 💡 **실무 팁**
>
> **폰트 최적화 기본 전략:**
> - **루트 레이아웃 적용**: 전역 폰트는 루트 레이아웃에서 한 번만 정의하여 성능을 최적화하세요
> - **서브셋 활용**: 사용하는 언어에 맞는 서브셋만 로드하여 번들 크기를 줄이세요
> - **폰트 로딩 전략**: 중요한 텍스트에는 빠른 로딩을, 장식적 요소에는 지연 로딩을 고려하세요

## Google 폰트

모든 Google 폰트를 자동으로 셀프 호스팅할 수 있습니다. 폰트는 정적 자산으로 포함되어 배포와 동일한 도메인에서 제공되므로, 사용자가 사이트를 방문할 때 브라우저가 Google에 요청을 보내지 않습니다.

Google 폰트 사용을 시작하려면 `next/font/google`에서 선택한 폰트를 임포트하세요:

```tsx
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={geist.className}>
      <body>{children}</body>
    </html>
  )
}
```

최고의 성능과 유연성을 위해 [가변 폰트](https://fonts.google.com/variablefonts)를 사용하는 것을 권장합니다. 하지만 가변 폰트를 사용할 수 없다면 가중치(weight)를 지정해야 합니다:

```tsx
import { Roboto } from 'next/font/google'

const roboto = Roboto({
  weight: '400',
  subsets: ['latin'],
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={roboto.className}>
      <body>{children}</body>
    </html>
  )
}
```

> 💡 **실무 팁**
>
> **Google 폰트 최적화 전략:**
> - **가변 폰트 우선**: 가능하면 가변 폰트를 선택하여 다양한 가중치와 스타일을 하나의 파일로 해결하세요
> - **필요한 가중치만 선택**: 실제로 사용하는 폰트 가중치만 지정하여 로딩 시간을 단축하세요
> - **프리로드 고려**: 주요 폰트는 `preload` 옵션을 활용하여 우선 로딩하세요

> **가변 폰트의 장점**
>
> 가변 폰트는 하나의 파일에서 다양한 가중치, 너비, 스타일을 제공하여 네트워크 요청을 줄이고 로딩 성능을 향상시킵니다. Google Fonts에서 "Variable" 태그가 있는 폰트를 찾아 활용하세요.

## 로컬 폰트

로컬 폰트를 사용하려면 `next/font/local`에서 폰트를 임포트하고 로컬 폰트 파일의 [`src`](/docs/app/api-reference/components/font.md#src)를 지정하세요. 폰트는 [`public`](/docs/app/api-reference/file-conventions/public-folder.md) 폴더에 저장하거나 `app` 폴더 내에 함께 위치시킬 수 있습니다. 예를 들어:

```tsx
import localFont from 'next/font/local'

const myFont = localFont({
  src: './my-font.woff2',
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={myFont.className}>
      <body>{children}</body>
    </html>
  )
}
```

단일 폰트 패밀리에 여러 파일을 사용하려면 `src`를 배열로 설정할 수 있습니다:

```js
const roboto = localFont({
  src: [
    {
      path: './Roboto-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './Roboto-Italic.woff2',
      weight: '400',
      style: 'italic',
    },
    {
      path: './Roboto-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
    {
      path: './Roboto-BoldItalic.woff2',
      weight: '700',
      style: 'italic',
    },
  ],
})
```

> 💡 **실무 팁**
>
> **로컬 폰트 관리 전략:**
> - **파일 위치 최적화**: 자주 사용되는 폰트는 `public`에, 컴포넌트별 폰트는 해당 폴더에 위치시키세요
> - **폰트 포맷 선택**: WOFF2를 우선 사용하고, 구형 브라우저 지원이 필요하면 WOFF도 함께 제공하세요
> - **폰트 파일 크기 관리**: 필요한 글자만 포함된 서브셋 폰트를 사용하여 파일 크기를 최적화하세요
> - **캐싱 전략**: 폰트 파일에 적절한 캐시 헤더를 설정하여 재방문 시 성능을 향상시키세요

> **폰트 파일 포맷 가이드**
> - **WOFF2**: 가장 최신이고 압축률이 높은 포맷 (권장)
> - **WOFF**: 광범위한 브라우저 지원
> - **TTF/OTF**: 원본 포맷이지만 파일 크기가 큼

## 고급 폰트 최적화 기법

> 📚 **폰트 성능 최적화 고급 전략**
>
> **프로덕션 레벨 폰트 최적화 가이드:**
> - **폰트 디스플레이 전략**: `font-display: swap`을 활용하여 폰트 로딩 중에도 텍스트를 표시하세요
> - **프리로딩**: 중요한 폰트는 `<link rel="preload">`로 우선 로딩하세요
> - **폰트 메트릭스 조정**: 웹 폰트와 시스템 폰트의 크기 차이를 줄여 레이아웃 시프트를 최소화하세요
> - **조건부 로딩**: 사용자의 네트워크 상태에 따라 폰트 로딩을 조건부로 처리하세요

## API 참조

Next.js 폰트의 전체 기능 세트에 대한 API 참조를 확인하세요.

- [Font](/docs/app/api-reference/components/font.md) - 내장된 `next/font` 로더로 웹 폰트 로딩을 최적화하세요.

> 🚀 **폰트 최적화 체크리스트**
>
> **폰트 성능 최적화를 위한 필수 체크 항목:**
> - **폰트 선택**: 가변 폰트를 우선 고려하고, 필요한 가중치만 선택하세요
> - **서브셋 설정**: 사용하는 언어와 문자에 맞는 서브셋만 로드하세요
> - **로딩 전략**: 중요한 폰트는 프리로드, 장식적 폰트는 지연 로딩을 적용하세요
> - **폴백 폰트**: 웹 폰트 로딩 실패에 대비한 적절한 시스템 폰트를 설정하세요
> - **성능 모니터링**: 폰트 로딩 시간과 레이아웃 시프트를 지속적으로 모니터링하세요

---

**원문**: [Next.js 공식 문서 - Font Optimization](https://nextjs.org/docs/app/getting-started/fonts)