![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/exceptionFilter.png)

<br>

![alt text](/Nest/image/exceptionFilter2.png)

<br>

## Exception Filter란?

NestJS에서는 자체적으로 예외 레이어를 관리한다. 서버에서 발생한 예외가 따로 핸들링 되지 않으면 NestJS 예외 레이어에서 에러를 사용자 친화적으로 변환해서 응답 할 수 있다.

<br>

## Exception Filter 구현법

```tsx
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
} from "@nestjs/common";
import { Request, Response } from "express";

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```
