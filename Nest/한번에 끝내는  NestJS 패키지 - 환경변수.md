![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br><br>

## 환경변수

환경변수는 프로그램이 동작하는 환경에서 설정값이나 비밀 정보를 저장하고 사용하는 변수다.

<br>

### 환경 변수의 중요성

<div style="border:1px solid #ccc; border-radius:10px; padding:15px; margin:10px 0; background:#1e1e1e; color:white;">
  <strong>🔐 [보안]</strong><br>
  API 키, 데이터베이스 비밀번호 등 민감한 정보를 코드에 직접 작성하지 않고 <strong style="color:#f87171;">환경 변수</strong>로 처리할 수 있다.
</div>

<div style="border:1px solid #ccc; border-radius:10px; padding:15px; margin:10px 0; background:#1e1e1e; color:white;">
  <strong>⚙️ [유연성]</strong><br>
  애플리케이션을 다양한 환경 (개발, 테스트, 운영)에서 <strong style="color:#f87171;">쉽게 설정하고</strong> 실행할 수 있도록 도와준다.
</div>

<div style="border:1px solid #ccc; border-radius:10px; padding:15px; margin:10px 0; background:#1e1e1e; color:white;">
  <strong>🔄 [유지보수성]</strong><br>
  설정 변경 시 코드 수정 없이 환경변수 <strong style="color:#f87171;">파일</strong>을 통해 간단히 업데이트할 수 있어 <strong style="color:#f87171;">유지보수성을 높인다</strong>.
</div>

<div style="border:1px solid #ccc; border-radius:10px; padding:15px; margin:10px 0; background:#1e1e1e; color:white;">
  <strong>📦 [통일성]</strong><br>
  동일한 애플리케이션을 여러 환경에서 <strong style="color:#f87171;">일관된 방식</strong>으로 배포하고 운영할 수 있도록 한다.
</div>

<br><br>

![alt text](/Nest/image/image4.png)

<br><br>

## 환경변수 사용하기 1: 환경변수 파일

```jsx
DB_HOST = localhost;
DB_PORT = 5432;
EXTERNAL_API_KEY = your - api - key;
PORT = 3000;
NODE_ENV = development;
SMTP_HOST = smtp.mailtrap.io;
LOG_LEVEL = debug;
```

<br><br>

## 환경변수 사용하기 2 : 환경변수 모듈 등록하기

```jsx
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

<br><br>

## 환경변수 사용하기 3: 환경변수 사용하기

```jsx
import { ConfigService } from '@nestjs/config';

export class AppService {
	constructor(private configService: ConfigService) {}

	getDatabaseHost(): string {
		return this.configService.get<string>('DB_HOST');
	}

	getApiKey(): string {
		return this.configService.get<string>('EXTERNAL_API_KEY');
	}
}
```
