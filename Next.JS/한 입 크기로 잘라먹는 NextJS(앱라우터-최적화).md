# 이미지 최적화

- http archive에 따르면 이미지는 평균적으로 웹페이지 용량의 58%를 차지.
- 다양한 이미지 최적화 기법들.
  - webp, AVIF 등의 차세대 형식으로 변환.
  - 디바이스 사이즈에 맞는 이미지 호출.
  - 레이지 로딩 적용.
  - 블러 이미지 활용.
  - ...=> Next에서는 해당 기능을 자체적으로 제공.

<br>

- `next/image`

```tsx
      // before
      <img src={coverImgUrl} />
      // after
      <Image src={coverImgUrl} width={80} height={105} alt={`도서 ${title}의 표지 이미지`} />
```

```tsx
// next.config.mjs
const nextConfig = {
  ...
  images: {
    domains: ['이미지를 불러오는 도메인'],
  },
};

export default nextConfig;
```

# 검색 엔진 최적화 SEO

- 방법.
  - sitemap 설정.
  - RSS 발행.
  - 시멘틱 태그 설정.
  - 메타 데이터 설정.
  - ...

## 메타 데이터 설정

- 정적 메타데이터 설정.
  - 페이지나 레이아웃에 입력.

```tsx
export const metadata: Metadata = {
  title: "사이트 제목",
  description: "사이트 설명.",
  openGraph: {
    title: "사이트 제목",
    description: "사이트 설명.",
    images: ["/thumbnamil.png"],
  },
};
```

- 동적 메타데이터 설정.

```tsx
// page와 동일한 props를 전달 받음.
export function generateMetadata({ searchParams }: Props): Metadata {
  return {
    title: `${searchParams.q} : 검색`,
    description: `${searchParams.q} 검색 결과입니다.`,
    openGraph: {
      title: `${searchParams.q} : 검색`,
      description: `${searchParams.q} 검색 결과입니다.`,
      images: ["/thumbnail.png"],
    },
  };
}

// async를 적용해야 할 때.
export async function generateMetadata({
  params,
}: {
  params: { id: string };
}): Promise<Metadata | null> {
  const response = await fetch(`${API_URL}`, {
    cache: "force-cache",
  });
  const book: BookData = await response.json();
  return {
    title: `${book.title}`,
    description: `${book.description}`,
    openGraph: {
      title: `${book.title}`,
      description: `${book.description}`,
      images: [book.coverImgUrl],
    },
  };
}
```
