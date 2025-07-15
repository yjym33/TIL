![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/fileUpload.png)

<br>

![alt text](/Nest/image/fileUpload2.png)
<br>

![alt text](/Nest/image/fileUpload3.png)

<br>

![alt text](/Nest/image/fileUpload4.png)

<br>

![alt text](/Nest/image/fileUpload5.png)

<br>

![alt text](/Nest/image/fileUpload6.png)

## 추가설명

## 1. 전송방식 개요

🔸 클라이언트 → 서버로 파일을 전송하는 방식
웹 브라우저나 앱은 파일을 업로드할 때 주로 HTTP POST 요청으로 전송하며, multipart/form-data 형식으로 인코딩됩니다.

이때 파일과 함께 다른 일반적인 필드도 같이 전송 가능 (title, description 등)

<br>

## 2. NestJS에서 사용하는 모듈

NestJS는 기본적으로 @nestjs/platform-express와 multer를 이용하여 파일 업로드 기능을 제공합니다.

<br>

## 📦 주요 구성 요소

| 구성 요소                           | 설명                                                                  |
| ----------------------------------- | --------------------------------------------------------------------- |
| `@UseInterceptors(FileInterceptor)` | 파일 업로드를 가로채서 처리하는 인터셉터                              |
| `multer`                            | 실제 파일 업로드를 처리하는 미들웨어 (Express용)                      |
| `diskStorage`                       | 파일을 로컬 디스크에 저장하는 설정                                    |
| `memoryStorage`                     | 파일을 메모리에 저장 (직접 처리하거나 외부 저장소로 전송할 경우 사용) |

<br>

## 3. NestJS 파일 업로드 - 기본 코드 흐름 (파일명 포함)

### 1. 필요 모듈 설치 (이미 설치되어 있으면 생략)

```bash
npm install @nestjs/platform-express multer
```

<br>

## 2. upload.controller.ts

```ts
import {
  Controller,
  Post,
  UploadedFile,
  UseInterceptors,
} from "@nestjs/common";
import { FileInterceptor } from "@nestjs/platform-express";
import { diskStorage } from "multer";
import { extname } from "path";

@Controller("upload")
export class UploadController {
  @Post("single")
  @UseInterceptors(
    FileInterceptor("file", {
      storage: diskStorage({
        destination: "./uploads", // 저장 경로
        filename: (req, file, cb) => {
          // 📌 파일명: 현재 시간 + 원래 확장자
          const uniqueSuffix =
            Date.now() + "-" + Math.round(Math.random() * 1e9);
          const ext = extname(file.originalname); // ex: .jpg, .png
          cb(null, `${uniqueSuffix}${ext}`);
        },
      }),
      limits: {
        fileSize: 5 * 1024 * 1024, // 5MB 제한
      },
      fileFilter: (req, file, cb) => {
        // 예시: 이미지 파일만 허용
        if (!file.mimetype.match(/\/(jpg|jpeg|png)$/)) {
          return cb(new Error("이미지 파일만 업로드 가능합니다."), false);
        }
        cb(null, true);
      },
    })
  )
  async uploadFile(@UploadedFile() file: Express.Multer.File) {
    return {
      message: "파일 업로드 성공",
      originalName: file.originalname,
      fileName: file.filename,
      size: file.size,
      path: file.path,
    };
  }
}
```

<br>

## 3. 업로드 디렉토리 만들기 (선택 사항)

```bash
mkdir uploads
```

<br>

- uploads/ 폴더가 존재하지 않으면 오류가 날 수 있습니다. 동적으로 만들고 싶다면:

<br>

```ts
import * as fs from "fs";
const destination = "./uploads";
if (!fs.existsSync(destination)) {
  fs.mkdirSync(destination, { recursive: true });
}
```

<br>

## 4. 반환 결과 예시

### 설명 및 의도

| 항목                      | 의도                                                             |
| ------------------------- | ---------------------------------------------------------------- |
| `diskStorage.destination` | 업로드된 파일을 서버에 저장할 경로 지정                          |
| `filename` 콜백           | 원본 파일명이 중복되거나 한글 등으로 충돌나는 것을 방지하기 위해 |
|                           | `Date.now()` + 랜덤값 + 확장자 형태로 저장                       |
| `fileFilter`              | 특정 파일만 허용하기 위해 MIME 타입 검사                         |
| `limits.fileSize`         | 업로드 가능한 최대 파일 크기 제한 (예: 5MB)                      |
| `@UploadedFile()`         | 업로드된 파일의 정보에 접근하는 데 사용됨                        |
