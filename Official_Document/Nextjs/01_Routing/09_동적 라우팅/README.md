빌드 전 정확한 Segement 의 이름을 모르거나, 동적인 데이터로부터 경로를 만들고 싶을 때, 요청시에 채워지거나 빌드 타임에 미리 렌더링되는 Dynamic Segement 를 사용할 수 있다.

---

## **1. 규칙**

동적 Segement 는 대괄호로 폴더의 이름을 감싸면 된다. 예를 들면 [id], [slug]

동적 Segement 는 layout.js, page.js, route.js 및 generateMetadata 함수의 params 속성으로 전달된다.

---

## **2. 예시**

예를 들면, 블로그는 다음과 같은 경로를 포함할 수 있다. `app/blog/[slug]/page.js` 여기에서 [slug] 가 블로그 게시물을 위한 동적 Segment 다.

```
export default function Page({ params }: { params: { slug: string } }) {
  return <div>My Post: {params.slug}</div>
}
```

| 경로(폴더/파일 구조) | URL 예시 | params |
| --- | --- | --- |
| app/blog/[slug]/page.js | /blog/a | { slug: a } |
| app/blog/[slug]/page.js | /blog/b | { slug: b } |
| app/blog/[slug]/page.js | /blog/c | { slug: c } |

---

## **3. Static Params 생성**

**generateStaticParams()* 함수를 사용해서 동적 경로 Segement 와 조합해 요청 시간이 아닌 빌드 타임에 정적으로 route 를 생성할 수 있다.

```
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

**generateStaticParams** 의 이점은 데이터를 똑똑하게 검색한다는 것이다. **generateStaticParams** 함수 내에서 fetch 요청을 해 콘텐츠를 가져오는 경우, 요청은 자동으로 저장(Memoization)된다. 이는 여러 **generateStaticParams**, 레이아웃, 페이지에서 동일한 인자로 fetch 요청이 한 번만 이루어진다는 것을 뜻하고 이 방식을 통해 빌드 시간을 줄인다.

---

## **4. 모든 Segment 가져오기**

동적 Segement는 대괄호 내부에 점 세 개를 추가해 뒤에 따라오는 Segment 를 모두 수집할 수 있다.

예를 들어서 `app/shop/[...slug]/page.js` 를 사용하면 `/shop/clothes`, `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts` 와 같은 URL 을 만들 때 활용할 수 있다.

| 경로(폴더/파일 구조) | URL 예시 | params |
| --- | --- | --- |
| app/shop/[...slug]/page.js | /shop/a | { slug: ['a'] } |
| app/shop/[...slug]/page.js | /shop/a/b | { slug: ['a', 'b'] } |
| app/shop/[...slug]/page.js | /shop/a/b/c | { slug: ['a', 'b', 'c'] } |

---

## **5. Optional로 모든 Segement 들 가져오기**

대괄호를 중첩해서 사용하면 모든 파라미터를 가져올 때 Optional 로 가져올 수 있다. (* 굳이 '선택적으로'라는 뜻이 아닌 Optional 이라고 표기한 이유는, Optional 연산자인 ? 를 연상하게 하기 위함이다.)

| 경로(폴더/파일 구조) | URL 예시 | params |
| --- | --- | --- |
| app/shop/[[...slug]]/page.js | /shop | {} |
| app/shop/[[...slug]]/page.js | /shop/a | { slug: ['a'] } |
| app/shop/[[...slug]]/page.js | /shop/a/b | { slug: ['a', 'b'] } |
| app/shop/[[...slug]]/page.js | /shop/a/b/c | { slug: ['a', 'b', 'c'] } |

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes#catch-all-segments**

[Routing: Dynamic Routes | Next.js
Dynamic Routes can be used to programmatically generate route segments from dynamic data.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes#catch-all-segments)