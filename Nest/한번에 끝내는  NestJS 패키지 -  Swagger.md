![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/swagger.png)

## DocumentBuilder

- Swagger Documentation을 초기화하고 생성하는데 사용된다.

```tsx
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle("SNS API")
    .setDescription("API documentation for SNS project")
    .setVersion("1.0")
    .addTag("posts")
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup("api", app, document);

  await app.listen(3000);
}
bootstrap();
```

<br>

## @ApiTags

- 여러 API를 그룹화하는데 사용된다. 일반적으로 Controller에 직접 적용해서 관련 엔드포인트들을 하나로 묶는다.

```tsx
import { Controller, Get } from "@nestjs/common";
import { ApiTags } from "@nestjs/swagger";

@ApiTags("posts")
@Controller("posts")
export class PostsController {
  @Get()
  findAll(): string {
    return "This action returns all posts";
  }
}
```

<br>

## @ApiOperation

- 엔드포인트가 어떤 역할을 하는지 정의 할 수 있다. 짧은 요약을 작성하는데 종종 사용된다.

```tsx
import { Get, Controller } from "@nestjs/common";
import { ApiOperation } from "@nestjs/swagger";

@Controller("posts")
export class PostsController {
  @Get()
  @ApiOperation({ summary: "Retrieve all posts" })
  findAll(): string {
    return "This action returns all posts";
  }
}
```

<br>

## @ApiResponse

- 엔드포인트의 응답에 대한 정의를 할 수 있다. 경우의수에 따라 하나 이상 정의 가능하다

```tsx
import { Get, Controller } from "@nestjs/common";
import { ApiResponse } from "@nestjs/swagger";

@Controller("posts")
export class PostsController {
  @Get()
  @ApiResponse({ status: 200, description: "Successfully retrieved posts." })
  @ApiResponse({ status: 404, description: "Posts not found." })
  findAll(): string {
    return "This action returns all posts";
  }
}
```

<br>

## @ApiQuery

- API의 쿼리 파라미터에 대한 정의를 할 수 있다.

```tsx
import { Controller, Get, Query } from "@nestjs/common";
import { ApiQuery } from "@nestjs/swagger";

@Controller("posts")
export class PostsController {
  @Get()
  @ApiQuery({ name: "author", required: false, type: String })
  findAll(@Query("author") author: string): string {
    return `This action returns all posts by author ${author}`;
  }
}
```

<br>

## @ApiParam

- API의 Path Parameter에 대한 정의를 할 수 있다.

```tsx
import { Controller, Get, Param } from "@nestjs/common";
import { ApiParam } from "@nestjs/swagger";

@Controller("posts")
export class PostsController {
  @Get(":id")
  @ApiParam({ name: "id", required: true, type: String })
  findOne(@Param("id") id: string): string {
    return `This action returns a post with ID ${id}`;
  }
}
```

<br>

## @ApiBody

### API의 Body 에 대한 정의를 할 수 있다

```tsx
import { Controller, Post, Body } from "@nestjs/common";
import { ApiBody } from "@nestjs/swagger";

class CreatePostDto {
  title: string;
  content: string;
  author: string;
}

@Controller("posts")
export class PostsController {
  @Post()
  @ApiBody({ type: CreatePostDto })
  create(@Body() createPostDto: CreatePostDto): string {
    return `This action adds a new post: ${createPostDto.title}`;
  }
}
```

<br>

## @ApiProperty

- Model 또는 DTO의 프로퍼티를 태깅 할 수 있다

```tsx
import { ApiProperty } from "@nestjs/swagger";

class CreatePostDto {
  @ApiProperty({ example: "My First Post" })
  title: string;

  @ApiProperty({ example: "This is the content of the post." })
  content: string;

  @ApiProperty({ example: "John Doe" })
  author: string;
}
```

<br>

## @ApiBearerAuth

- Bearer 토큰이 필요한 경우 사용 가능하다.

```tsx
import { Controller, Get } from "@nestjs/common";
import { ApiBearerAuth } from "@nestjs/swagger";

@Controller("posts")
export class PostsController {
  @Get()
  @ApiBearerAuth()
  findAll(): string {
    return "This action returns all posts";
  }
}
```

<br>

## @ApiOAuth2

- OAuth가 필요한 경우 사용할 수 있다.

```tsx
import { Controller, Get } from "@nestjs/common";
import { ApiOAuth2 } from "@nestjs/swagger";

@Controller("posts")
export class PostsController {
  @Get()
  @ApiOAuth2(["read", "write"])
  findAll(): string {
    return "This action returns all posts";
  }
}
```

<br>

## 적용 예제

- 필요한 기능들을 Annotate 하다보면 코드가 매우 복잡해진다.
  특히나 기능과 관련 없는 Annotation이다보니 관리가 부담된다

```tsx
class CreatePostDto {
  @ApiProperty({ example: "My First Post" })
  title: string;

  @ApiProperty({ example: "This is the content of the post." })
  content: string;

  @ApiProperty({ example: "John Doe" })
  author: string;
}

@ApiTags("posts")
@Controller("posts")
export class PostsController {
  @Get()
  @ApiOperation({ summary: "Retrieve all posts" })
  @ApiQuery({ name: "author", required: false, type: String })
  @ApiResponse({ status: 200, description: "Successfully retrieved posts." })
  findAll(@Query("author") author: string): string {
    return `This action returns all posts by author ${author}`;
  }

  @Get(":id")
  @ApiOperation({ summary: "Retrieve a specific post by ID" })
  @ApiParam({ name: "id", required: true, type: String })
  @ApiResponse({ status: 200, description: "Successfully retrieved the post." })
  @ApiResponse({ status: 404, description: "Post not found." })
  findOne(@Param("id") id: string): string {
    return `This action returns a post with ID ${id}`;
  }

  @Post()
  @ApiOperation({ summary: "Create a new post" })
  @ApiBody({ type: CreatePostDto })
  @ApiResponse({
    status: 201,
    description: "The post has been successfully created.",
  })
  create(@Body() createPostDto: CreatePostDto): string {
    return `This action adds a new post: ${createPostDto.title}`;
  }
}
```

<br>

## Open API CLI Plugin

- Typescript의 Reflection 기능을 사용하면 유추 할 수 있는 값들이 많기 때문에 Open API CLI Plugin을 이용하면 많은 기능들을 자동 유추 할 수 있다.

```tsx
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/swagger"]
  }
}
```
