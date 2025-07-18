![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

## Task Scheduling

### Task Scheduling란?

- 특정 작업(함수 또는 로직)을 정해진 시간 간격 또는 특정 시점에 자동으로 실행되도록 설정하는 기능
- 백그라운드에서 주기적으로 실행되어야 하는 작업을 예약

<br>

![alt text](/Nest/image/TaskScheduling.png)

<br>

![alt text](/Nest/image/TaskScheduling2.png)

<br>

![alt text](/Nest/image/TaskScheduling3.png)

<br>

## Cron 문법

<br>

![alt text](/Nest/image/TaskScheduling4.png)

<br>

![alt text](/Nest/image/TaskScheduling5.png)

<br>

![alt text](/Nest/image/TaskScheduling6.png)

<br>

![alt text](/Nest/image/TaskScheduling7.png)

<br>

![alt text](/Nest/image/TaskScheduling8.png)

<br>

## NestJS 에서 Task Scheduling 구현 방법

### 1. 설치

```bash

npm install @nestjs/schedule

```

<br>

### 2. 기본 설정

```ts
// app.module.ts
import { Module } from "@nestjs/common";
import { ScheduleModule } from "@nestjs/schedule";
import { TasksModule } from "./tasks/tasks.module";

@Module({
  imports: [
    ScheduleModule.forRoot(), // 스케줄 모듈 등록
    TasksModule,
  ],
})
export class AppModule {}
```

<br>

### 3. 예제: 일정 시간마다 실행되는 작업 만들기

```ts
// tasks.service.ts
import { Injectable, Logger } from "@nestjs/common";
import { Cron, Interval, Timeout } from "@nestjs/schedule";

@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  @Cron("45 * * * * *") // 매 분 45초마다 실행
  handleCron() {
    this.logger.debug("🔁 매 분 45초에 실행됩니다.");
  }

  @Interval(10000) // 10초마다 반복
  handleInterval() {
    this.logger.debug("📌 10초마다 실행됩니다.");
  }

  @Timeout(5000) // 애플리케이션 시작 후 5초 뒤에 한 번 실행
  handleTimeout() {
    this.logger.debug("🚀 앱 시작 5초 후 한 번 실행됩니다.");
  }
}
```

<br>

## 🛠️ 주요 데코레이터 정리

| 데코레이터       | 설명                                                    |
| ---------------- | ------------------------------------------------------- |
| `@Cron(cronExp)` | [Cron 표현식](https://crontab.guru/) 기반으로 주기 실행 |
| `@Interval(ms)`  | 지정된 밀리초마다 반복 실행                             |
| `@Timeout(ms)`   | 지정된 시간 이후 단 한 번 실행                          |
