# 이미지 최적화

Next.js `<Image>` 컴포넌트는 HTML `<img>` 요소를 확장하여 다음과 같은 기능을 제공합니다:

- **크기 최적화:** WebP와 같은 최신 이미지 형식을 사용하여 각 디바이스에 맞는 크기의 이미지를 자동으로 제공합니다.
- **시각적 안정성:** 이미지 로딩 시 레이아웃 시프트를 자동으로 방지합니다.
- **빠른 페이지 로드:** 네이티브 브라우저 지연 로딩을 사용하여 뷰포트에 들어올 때만 이미지를 로드하며, 선택적으로 블러 업 플레이스홀더를 제공합니다.
- **자산 유연성:** 원격 서버에 저장된 이미지라도 온디맨드로 크기를 조정할 수 있습니다.

`<Image>` 사용을 시작하려면 `next/image`에서 임포트하고 컴포넌트 내에서 렌더링하세요.

```tsx
import Image from 'next/image'

export default function Page() {
  return <Image src="" alt="" />
}
```

`src` 속성은 [로컬 이미지](#로컬-이미지) 또는 [원격 이미지](#원격-이미지)일 수 있습니다.

> **🎥 동영상 학습**: `next/image` 사용법에 대해 자세히 알아보기 → [YouTube (9분)](https://youtu.be/IU_qq_c_lKA)

## 로컬 이미지

이미지와 폰트 같은 정적 파일을 루트 디렉토리의 `public` 폴더에 저장할 수 있습니다. `public` 내부의 파일들은 기본 URL(`/`)에서 시작하여 코드에서 참조할 수 있습니다.

```tsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```

이미지가 정적으로 임포트되면, Next.js는 자동으로 내재된 `width`와 `height`를 결정합니다. 이 값들은 이미지 비율을 결정하고 이미지 로딩 중 누적 레이아웃 시프트를 방지하는 데 사용됩니다.

```tsx
import Image from 'next/image'
import ProfileImage from './profile.png'

export default function Page() {
  return (
    <Image
      src={ProfileImage}
      alt="Picture of the author"
      // width={500} 자동으로 제공됨
      // height={500} 자동으로 제공됨
      // blurDataURL="data:..." 자동으로 제공됨
      // placeholder="blur" // 로딩 중 선택적 블러 업
    />
  )
}
```

> 💡 **실무 팁**
>
> **로컬 이미지 최적화 전략:**
> - **정적 임포트 활용**: 컴파일 타임에 최적화를 위해 정적 임포트를 우선 사용하세요
> - **이미지 포맷 선택**: WebP, AVIF 등 최신 포맷의 이미지를 준비하면 더 나은 최적화가 가능합니다
> - **적절한 크기**: 원본 이미지는 최대 사용 크기의 2배 정도로 준비하여 고해상도 디스플레이를 지원하세요

## 원격 이미지

원격 이미지를 사용하려면 `src` 속성에 URL 문자열을 제공할 수 있습니다.

```tsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```

Next.js는 빌드 과정에서 원격 파일에 접근할 수 없으므로, `width`, `height` 및 선택적 `blurDataURL` props를 수동으로 제공해야 합니다. `width`와 `height`는 이미지의 올바른 종횡비를 추론하고 이미지 로딩으로 인한 레이아웃 시프트를 방지하는 데 사용됩니다. 또는 `fill` 속성을 사용하여 이미지가 부모 요소의 크기를 채우도록 할 수 있습니다.

원격 서버의 이미지를 안전하게 허용하려면 `next.config.js`에서 지원되는 URL 패턴 목록을 정의해야 합니다. 악의적인 사용을 방지하기 위해 가능한 한 구체적으로 설정하세요. 예를 들어, 다음 구성은 특정 AWS S3 버킷의 이미지만 허용합니다:

```ts
// next.config.ts
import type { NextConfig } from 'next'

const config: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
        search: '',
      },
    ],
  },
}

export default config
```

> 💡 **실무 팁**
>
> **원격 이미지 보안 및 성능 최적화:**
> - **도메인 화이트리스트**: `remotePatterns`을 최대한 구체적으로 설정하여 보안을 강화하세요
> - **CDN 활용**: 이미지 서빙을 위해 CloudFront, Cloudinary 등의 CDN을 활용하세요
> - **이미지 크기 미리 확인**: 원격 이미지의 실제 크기를 확인하여 정확한 width/height를 설정하세요
> - **에러 처리**: 원격 이미지 로딩 실패에 대비한 폴백 이미지를 준비하세요

> **보안 고려사항**
>
> 원격 이미지를 사용할 때는 반드시 신뢰할 수 있는 도메인만 허용하고, 와일드카드 사용을 최소화하세요. 잘못 설정된 패턴은 보안 취약점이 될 수 있습니다.

## 고급 기능 및 최적화

> 📚 **이미지 최적화 고급 전략**
>
> **프로덕션 레벨 이미지 최적화 가이드:**
> - **반응형 이미지**: `sizes` 속성을 활용하여 다양한 화면 크기에 최적화된 이미지를 제공하세요
> - **우선순위 설정**: Above-the-fold 이미지에는 `priority` 속성을 사용하세요
> - **로딩 전략**: `loading="eager"` 또는 `loading="lazy"`를 적절히 활용하세요
> - **플레이스홀더 최적화**: 블러 데이터 URL을 생성하여 더 나은 사용자 경험을 제공하세요

## API 참조

Next.js Image의 전체 기능 세트에 대한 API 참조를 확인하세요.

- **Image 컴포넌트** - 내장된 `next/image` 컴포넌트를 사용하여 Next.js 애플리케이션에서 이미지를 최적화하세요.

> 🚀 **성능 최적화 체크리스트**
>
> **이미지 최적화를 위한 필수 체크 항목:**
> - **alt 텍스트**: 접근성을 위해 모든 이미지에 의미 있는 alt 텍스트를 제공하세요
> - **크기 지정**: CLS(Cumulative Layout Shift)를 방지하기 위해 항상 width와 height를 지정하세요
> - **포맷 최적화**: 브라우저 지원에 따라 WebP, AVIF 등 최신 포맷을 활용하세요
> - **로딩 최적화**: 중요한 이미지는 우선순위를, 나머지는 지연 로딩을 적용하세요

---

**원문**: [Next.js 공식 문서 - Image Optimization](https://nextjs.org/docs/app/getting-started/images)