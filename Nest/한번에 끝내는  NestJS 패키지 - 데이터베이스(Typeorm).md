![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

## 설치 및 설정

설치

```tsx
pnpm i @nestjs/config joi @nestjs/typeorm typeorm pg
```

.env 파일 세팅

```
# dev = 개발환경
# prod = 배포환경

 ENV = dev

 # DB

DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5556
DB_USERNAME=postgres
DB_PASSWORD=ymyj0603
DB_DATABASE=postgres

```

app.module 코드 작성

```tsx
import { Module } from "@nestjs/common";

import { MovieModule } from "./movie/movie.module";
import { TypeOrmModule } from "@nestjs/typeorm";
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [
    ConfigModule.forRoot(),
    TypeOrmModule.forRoot({
      type: process.env.DB_TYPE as "postgres",
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      entities: [],
      synchronize: true, // 개발할때만 true 이고, 운영시에는 바꿔야함
    }),
    MovieModule,
  ],
})
export class AppModule {}
```

Joi로 유효성 검사 코드 작성

```jsx
import { Module } from '@nestjs/common';

import { MovieModule } from './movie/movie.module';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        ENV: Joi.string().required(),
        DB_TYPE: Joi.string().required(),
        DB_HOST: Joi.string().required(),
        DB_PORT: Joi.number().required(),
        DB_USERNAME: Joi.string().required(),
        DB_PASSWORD: Joi.string().required(),
        DB_DATABASE: Joi.string().required(),
      }),
    }),
    TypeOrmModule.forRoot({
      type: process.env.DB_TYPE as 'postgres',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      entities: [],
      synchronize: true, // 개발할때만 true 이고, 운영시에는 바꿔야함
    }),
    MovieModule,
  ],
})
export class AppModule {}

```

해당 코드처럼 Joi로 유효성 검사를 할 경우 .env파일에서 오타가 나더라도 잘못된 경우 바로 Config validation error가 발생하여 오류가 있다는것을 명시해준다.

```jsx
import { Module } from '@nestjs/common';

import { MovieModule } from './movie/movie.module';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        ENV: Joi.string().valid('dev', 'prod').required(),
        DB_TYPE: Joi.string().valid('postgres').required(),
        DB_HOST: Joi.string().required(),
        DB_PORT: Joi.number().required(),
        DB_USERNAME: Joi.string().required(),
        DB_PASSWORD: Joi.string().required(),
        DB_DATABASE: Joi.string().required(),
      }),
    }),
    TypeOrmModule.forRoot({
      type: process.env.DB_TYPE as 'postgres',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      entities: [],
      synchronize: true, // 개발할때만 true 이고, 운영시에는 바꿔야함
    }),
    MovieModule,
  ],
})
export class AppModule {}

```

또한 특정값만 오는게 가능하도록 .valid()를 통해 설정을 해줄수 있다.

하지만 Typeorm에서 위의 코드처럼 직접적으로 process.env를 사용하는 경우는 드물다

Typeorm에서는 직접적으로 process.env를 사용하지 않고 configService를 사용하여 다음과같이 환경변수를 받아온다.

```jsx
import { Module } from '@nestjs/common';

import { MovieModule } from './movie/movie.module';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        ENV: Joi.string().valid('dev', 'prod').required(),
        DB_TYPE: Joi.string().valid('postgres').required(),
        DB_HOST: Joi.string().required(),
        DB_PORT: Joi.number().required(),
        DB_USERNAME: Joi.string().required(),
        DB_PASSWORD: Joi.string().required(),
        DB_DATABASE: Joi.string().required(),
      }),
    }),
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: configService.get<string>('DB_TYPE') as 'postgres',
        host: configService.get<string>('DB_HOST'),
        port: configService.get<number>('DB_PORT'),
        username: configService.get<string>('DB_USERNAME'),
        password: configService.get<string>('DB_PASSWORD'),
        database: configService.get<string>('DB_DATABASE'),
        entities: [],
        synchronize: true,
      }),
      inject: [ConfigService],
    }),

    // TypeOrmModule.forRoot({
    //   type: process.env.DB_TYPE as 'postgres',
    //   host: process.env.DB_HOST,
    //   port: parseInt(process.env.DB_PORT),
    //   username: process.env.DB_USERNAME,
    //   password: process.env.DB_PASSWORD,
    //   database: process.env.DB_DATABASE,
    //   entities: [],
    //   synchronize: true, // 개발할때만 true 이고, 운영시에는 바꿔야함
    // }),
    MovieModule,
  ],
})
export class AppModule {}

```

## Repository

- Repository는 지정한 Entity에 대한 CRUD 쿼리를 할수 있게 해준다.
- Typeorm에 정의된 메서드를 사용해서 직접 SQL을 사용하지 않더라도 데이터 관리를 할수 있다.

ex)

```jsx
const userRepository = dataSource.getRepository(user);
const user = await userRepository.findOneBy({
  id: 1,
});
user.name = "Code Factory";
await userRepository.save(user);
```

## create()

- create() 메서드는 객체를 생성하는 역할을 한다.
- create()는 save()와 다르게 데이터베이스에 데이터를 생성하지 않고 “객체”를 생성만 한다는것을 꼭 기억해야 함!!

```jsx
const user = repository.create();
const user = respository.create({
  id: 1,
  firstName: "Timber",
  lastName: "Saw",
});
```

## save()

- save() 메서드에 저장할 Entity를 입력해주면 저장 할 수 있다.
- create()와 다르게 실제 데이터베이스에 저장이 된다.
- 만약에 이미 Row()가 존재한다면 (Primary Key 값으로 구분) 업데이트 한다. (주의해야함!!)
- 여러 객체를 한번에 저장도 가능하다.

```jsx
await repository.save(user);
await repository.save([category1, category2, category3]);
```

## upsert()

- update와 insert를 합친게 upsert() 이다.
- 데이터 생성 시도를 한 후 만약에 이미 존재하는 데이터라면 업데이트를 진행한다.
- save() 와는 다르게 upsert()는 하나의 transaction에서 작업이 실행된다.
  - 아이디가 없는 상태면 그냥 저장, 아이디가 들어오면 아이디가 있는지 확인후 존재하지 않으면
    업데이트 함 (save)
  - upsert는 해당 과정을 한번에 모두 해줄수 있다.

```jsx
await repository.upsert(
  [
    { externalId: "abc123", firstName: "Code" },
    { externalId: "bca321", firstName: "Factory" },
  ],
  ["externalId"]
);

/**
	executes
	INSERY INTO user
	VALUES
			(externalId = abc123, firstName = Code ),
			(externalId = cba321, firstName = Factory ),
	ON CONFLICT (externalId) DO UPDATE firstName = EXCLUDED.firstName

**/
```

## delete()

- Row를 삭제할때 사용된다.
- 대체적으로 Primary Key를 사용해서 삭제한다.
- 원하다면 findOptionsWhere 조건으로 여러 값을 삭제 할 수도 있다.

```jsx
await repository.delete(1);
await repository.delete([1, 2, 3]);
await repository.delete({ firstName: "Timber" });
```

## softDelete(), restore()

- softDelete()는 비영구적으로 삭제하는 기능이다.
- restore()를 실행하면 softDelete() 했던 Row를 복구 할 수 있다.

```jsx
// 삭제
await repository.softDelete(1);

// 복구
await repository.restore(1);
```

## update()

- 첫번째 파라미터에 검색 조건을 입력해준다.
- 두번째 파라미터에 변경 필드를 입력해준다.

```jsx
// UPDATE user
// SET category = ADULT
// WHERE age = 18
await repository.update(
	{ age: 18 }
	{ category: "ADULT" }
)

// UPDATE user
// SET firstName = Code Factory
// WHERE id = 1
await repository.update(1, {
	firstName: "Code Factory"
})
```

## find() , findOne(), findAndCount()

- find() : 해당되는 Row를 모두 반환한다.
- findOne() : 해당되는 첫번째 Row를 반환한다. 없을경우 null
- findAndCount() : 해당되는 Row와 전체 갯수를 반환한다.

```jsx
const rows = await repository.find({
  where: {
    firstName: "Code Factory",
  },
});

const rows = await repository.findOne({
  where: {
    firstName: "Code Factory",
  },
});

const [rows, count] = await repository.findAndCount({
  where: {
    firstName: "Code Factory",
  },
});
```

## exists()

- 특정 조건의 Row가 존재하는지 boolean 값을 반환 받을 수 있다.

```jsx
const exists = await repository.exists({
  where: {
    firstName: "Timber",
  },
});
```

## preload()

- preload()는 데이터베이스에 저장된 값을 Primary Key 기준으로 불러오고 입력된 객체의 값으로 프로퍼티를 덮어 쓴다.
- 덮어쓰는 과정에서 데이터베이스에 업데이트 요청이 보내지지는 않는다.

```jsx
const partialUser = {
  id: 1,
  firstName: "Code Factory",
  profile: {
    id: 1,
  },
};

const user = await repository.preload(partialUser);
```

## FindOptions

- 모든 find 관련된 API는 FindOptios를 아규먼트로 받는다.
- FindOptions는 어떤 값들을 불러올지 필터링하는 역할을 한다.
- FindOptions의 정확한 TS 타입 명칭은 FindOneOptions와 FindManyOptions로 정의되어 있다.
- FindManyOptions는 FindOneOptions를 상속받고 skip, take 두가지 프로퍼티가 더 존재한다.

```jsx
const rows = await repository.find({
  where: {
    firstName: "Code Factory",
  },
});

const row = await repository.findOne({
  where: {
    firstName: "Code Factory",
  },
});

const [rows, count] = await repository.findAndCount({
  where: {
    firstName: "Code Factory",
  },
});
```

## FindOptions

```jsx
export interface FindOneOptions<Entity = any> {
	select? : FindOptionsSelect<Entity>
	where? : FindOptionsWhere<Entity>[]
	relations?: FindOptionsRelations<Entity>
	order?: FindOptionsOrder<Entity>
	cache?: boolean | number
}
```

```jsx
export interface FindManyOptions<Entity = any>
extends FindOneOptions<Entity> {
	skip? : number
	take? : number
}
```

<br>

**Select : 불러올 Column을 지정할수 있다.**

**Where : 필터링할 조건을 설정 할 수 있다.**

**Relations : 불러올 관계 테이블을 지정 할 수 있다.**

**Order : 정렬을 지정 할 수 있다.**

**Cache : 캐싱 기간을 지정 할 수 있다.**

<br>

## Where Property

```jsx
// 기본 사용법

const users = await userRepository.find({
  where: { isActive: true },
});
```

```jsx
// 다중조건 사용법

const users = await userRepository.find({
  where: [
    { firstName: "John", lastName: "Doe" },
    { firstName: "Jane", lastName: "Smith" },
  ],
});
```

```jsx
// 중첩 사용법

const users = await userRepository.find({
  where: {
    isActive: true,
    profile: { age: MoreThan(25) },
  },
});
```

## Order Property

```jsx
// 단일 정렬 사용법

const users = await userRepository.find({
  order: { firstName: "ASC" },
});
```

```jsx
// 복수 정렬 사용법

const users = await userRepository.find({
  order: { lastName: "ASC", firstName: "DESC" },
});
```

## Relation Property

```jsx
// 기본 사용법

const users = await userRepository.find({
	relations: ["profile" "photos"],
});
```

## Select Property

```jsx
// 기본 사용법
const users = await userRepository.find({
	select: ["firstaNaem, "lastName"],
});
```

## Cache Property

```jsx
// 기본 사용법

const users = await userRepository.find({
  cache: true,
});
```

```jsx
// 기간 직접 정의

const users = await userRepository.find({
  cache: 60000,
});
```

## FindManyOptions

Skip : 정렬 후 스킵할 데이터 갯수를 정할 수 있다.

Take : 처음 몇개의 데이터를 불러올지 정할 수 있다.

```jsx
const users = await userRepository.find({
  skip: 10,
  take: 5,
});
```

## Eqeual Operator

- 같은 값을 찾을때 사용

```jsx
const users = await userRepository.find({
  where: { age: Equal(25) },
});
```

## Not Operator

- 아닌값을 찾을때 사용

```jsx
const users = await userRepository.find({
  where: { age: Not(25) },
});
```

## LessThan & LessThanOrEqual Operator

- 적은값 & 적거나 같은 값을 찾을때 사용한다.

```jsx
const users = await userRepository.find({
  where: { age: LessThan(30) },
});

const users = await userRepository.find({
  where: { age: LessThanOrEqual(30) },
});
```

## MoreThan & MoreThanOrEqual Operator

- 적은값 & 적거나 같은 값을 찾을때 사용한다.

```jsx
const users = await userRepository.find({
  where: { age: MoreThan(20) },
});

const users = await userRepository.find({
  where: { age: MoreThanOrEqual(20) },
});
```

## Between

- 사이 값을 찾을때 사용

```jsx
const users = await userRepository.find({
  where: { age: Between(20, 30) },
});
```

## Like & ILike

- 스트링에 매칭되는 값을 찾을때 사용한다. Like는 대소문자 구분할때, ILike는 대소문자 구분하지 않을때 사용한다.

```jsx
const users = await userRepository.find({
  where: { firstName: Like("Co%") },
});

const users = await userRepository.find({
  where: { firstName: ILike("co%") },
});
```

## In

- 리스트에 매칭되는 값을 찾는다.

```jsx
const users = await userRepository.find({
  where: { age: In(20, 25, 30) },
});
```

<br>

### ArrayContains & ArrayContainedBy & ArrayOverlap Operator

- ArrayContains: 엔티티 리스트 값이 타겟 리스트와 완전 똑같은 경우를 필터링
- ArrayContainedBy: 엔티티 리스트 값이 타겟 리스트 안에 모두 포함되는 경우를 필터링
- ArrayOverlap: 엔티티 리스트 값이 타겟 리스트와 겹치는 경우를 필터링

```jsx
const users = await userRepository.find({
  where: { roles: ArrayContains(["admin"]) },
});

const users = await userRepository.find({
  where: { roles: ArrayContainedBy(["admin", "user"]) },
});

const users = await userRepository.find({
  where: { roles: ArrayOverlap(["admin", "guest"]) },
});
```

<br>

## ArrayContains

```jsx
// Record 1
{
	id: 1,
	tags: ["admin", "user", "manager"]
}

// Record 2
{
	id: 2,
	tags: ["user", "guest"]
}
```

```jsx
const users = await getRepository(User).find({
  tags: ArrayContains(["user", "guest"]),
});
```

- Record2의 Tags가 완전히 같으니 Record2만 반환

<br>

## ArrayContainedBy

```jsx
// Record 1
{
	id: 1,
	tags: ["admin", "user", "manager"]
}

// Record 2
{
	id: 2,
	tags: ["user", "guest"]
}
```

```jsx
const users = await getRepository(User).find({
  tags: ArrayContainedBy(["admin", "user", "guest", "manager"]),
});
```

- Record1과 Record2의 Tags들이 모두 필터 리스트에 포함되기 때문에 둘다 반환

## ArrayOverlap

```jsx
// Record 1
{
	id: 1,
	tags: ["admin", "user", "manager"]
}

// Record 2
{
	id: 2,
	tags: ["user", "guest"]
}
```

```jsx
const users = await getRepository(User).find({
  tags: ArrayOverlap(["guest", "admin"]),
});
```

- Record1의 Tags와 Record2의 Tags들이 모두 필터 리스트와 겹치는 부분이 있기 때문에 둘다 반환

## IsNull

- Null인 값을 필터링할때 사용

```jsx
const users = await userRepository.find({
  where: { profile: IsNull() },
});
```

## Or

- OR로직 연산할때 사용 (하나만 해당)

```jsx
const loadedPosts = await dataSource.getRepository(Post).findBy({
  title: Or(Equal("About #2"), ILike("About%")),
});
```

## And

- AND로직 연산할때 사용 (둘다 만족해야함)

```jsx
const loadedPosts = await dataSource.getRepository(Post).findBy({
  title: And(Not(Equal("About #2")), ILike("About%")),
});
```

## 복잡한 레포지토리 쿼리

```jsx
const users = await userRepository.find({
  where: [
    { isActive: true, age: MoreThan(25) },
    { firstName: "John", age: LessThan(50) },
  ],
  order: { firstName: "ASC" },
  relations: ["profile"],
  select: ["firstName", "lastName"],
  skip: 0,
  take: 10,
  cache: true,
  loadRelationIds: true,
  loadEagerRelations: false,
  withDeleted: false,
});
```

## 각종 통계

- count() : 해당되는 갯수를 반환한다
- sum() : 해당되는 칼럼 값을 모두 더한다
- average() : 해당되는 칼럼 값의 평균을 구한다
- minimum() : 해당되는 칼럼 값의 최소치를 반환한다
- maximum() : 해당되는 칼럼 값의 최대치를 반환한다

```jsx
const count = await repository.count({
  where: {
    firstName: "Code Factory",
  },
});

const sum = await repository.sum("age", { firstName: "Timber" });

const average = await repository.average("age", { firstName: "Timber" });

const minimum = await repository.minimum("age", { firstName: "Timber" });

const maximum = await repository.maximum("age", { firstName: "Timber" });
```

## Entity Embedding

엔티티 임베딩(Entity Embedding)은 다른 클래스(속성들을 가진 클래스)를 하나의 엔티티에 포함시켜 데이터베이스 컬럼을 효율적으로 정의하고 재사용성을 높이는 기법입니다.

<br>

![alt text](/Nest/image/Image6.png)

<br>

- `User` 테이블과 `Employee` 테이블 모두 `nameFirst`와 `nameLast`라는 두 개의 컬럼을 가지며, 이는 `Name` 클래스에서 정의된 `first`와 `last` 속성에 해당합니다.
- 즉, `Name` 클래스는 데이터베이스의 두 개 컬럼(`nameFirst`, `nameLast`)으로 확장되어 각 엔티티에 매핑됩니다.

<br>

![alt text](Image7.png)

<br>

## Entity Inheritance

엔티티 상속은 기본 클래스에 정의된 공통 속성을 여러 서브클래스에서 상속받아 사용하는 것입니다. 이렇게 함으로써 코드의 중복을 줄이고 유지보수성을 높일 수 있습니다. 특히, 공통된 속성을 가진 여러 엔티티들이 있을 때, 이러한 구조를 활용하면 효율적으로 데이터 모델을 정의할 수 있습니다.

<br>

![alt text](/Nest/image/Image8.png)

<br>

- `Photo`와 `Post` 클래스는 `Content` 클래스를 상속(`extends`)합니다.
- 이로 인해 `Photo`와 `Post`는 `Content`에 정의된 `id`, `title`, `description` 속성을 모두 상속받습니다.

<br>

![alt text](/Nest/image/Image9.png)

<br>

### **Single Table Inheritance**

<br>

![alt text](/Nest/image/Image10.png)

<br>

<br>

![alt text](/Nest/image/Image11.png)

<br>

## Relationships

<br>

![alt text](/Nest/image/Image12.png)

<br>

### **Relationships Annotation 적용하기**

- 첫번째 파라미터에는 타입을 반환하는 함수를 입력한다. (Class Transformer Type과 같은 개념)
- 두번째 파라미터에는 첫번째 파라미터에 입력한 클래스의 칼럼중 하나를 입력한다. 이 칼럼은 서로 관련지을 프로퍼티여야 한다.
- 예를들어 ManyToOne 관계이니 Photo 테이블에 user_id 칼럼이 생성되며 user 테이블과 관계가 형성된다.
- 특정 Photo와 관련있는 user는 photo.user로 불러올 수 있고 user와 관련있는 photo들은 user.photos로 불러 올 수 있다.

<br>

```jsx
@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @ManyToOne(() => user, (user) => user.photos)
  user: User;
}
```

```jsx
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @OneToMany(() => Photo, (photo) => photo.user)
  photos: Photo[];
}
```

<br>

![alt text](/Nest/image/image13.png)

<br>

## OneToOne Relationship

- OneToOne Relationship도 마찬가지로 Annotation을 원하는 프로퍼티에 정의해주면 된다.
- ManyToOne은 상대의 레퍼런스를 갖는 테이블이 명확하다.
- OneToOne은 두 테이블 누가 레퍼런스를 들고 있어도 상관이 없기 때문에 어떤 테이블이 레퍼런스를 들고 있을지 명시해줘야한다.
- @JoinTable Annotation을 사용해서 어떤 프로퍼티가 레퍼런스를 들고 있을지 정해 줄 수 있다.
- @JoinTable은 꼭 한쪽에만 적용해야한다. 둘 모두 적용하는건 불가능하고 의미도 없다.

<br>

```jsx
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  gender: string;

  @Column()
  photo: string;

  @OneToOne(() => User, (user) => user.profile)
  user: User;
}
```

```jsx
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => profile)
  @JoinColumn()
  profile: Profile;
}
```

<br>

## ManyToMany Relationship

- ManyToMany Relationship도 OneToOne Relationship와 마찬가지로 @JoinTable Annotation을 한쪽에 적용 해줘야한다.
- 중간 테이블이 생성될때 @JoinTable이 적용된 테이블 이름이 먼저 위치하게 된다.

<br>

```jsx
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Question)
  questions: Question[];
}
```

```jsx
@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  text: string;

  @ManyToMany(() => Category)
  @JoinTable()
  categories: Category[];
}
```

<br>

## Query Builder (** 매우중요 **)

- 복잡하지 않은 일반적인 쿼리를 실행할 떄는 Repository를 사용하는게 편리하다.
- 조금 더 복잡한 쿼리를 실행 해야하거나 다이나믹하게 쿼리를 만들어가야 할 경우 Query Builder를 사용해야한다.

<br>

```jsx
const users = await userRepository
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.profile", "profile")
  .where("user.isActive = :isActive", { isActive: true })
  .orderBy("user.firstName", "ASC")
  .skip(10)
  .take(5)
  .getMany();
```

<br>

## Query Builder의 5가지 실행 타입 : 1 SELECT

```jsx
const movie = await dataSource
  .createQueryBuilder()
  .select("movie")
  .from(Movie, "movie")
  .leftJoinAndSelect("movie.detail", "detail")
  .leftJoinAndSelect("movie.director", "director")
  .leftJoinAndSelect("movie.genres", "genres")
  .where("movie.id = :id", { id: 1 })
  .getOne();
```

<br>

## Query Builder의 5가지 실행 타입 : 2 INSERT

```jsx
await dataSource
  .createQueryBuilder()
  .insert()
  .into(Movie)
  .values([
    {
      titile: "New Movie",
      genre: "Action",
      director: director,
      genres: genres,
    },
  ])
  .execute();
```

<br>

## Query Builder의 5가지 실행 타입 : 3 UPDATE

```jsx
await dataSource
  .createQueryBuilder()
  .update(Movie)
  .set({ title: "Updated Title", genre: "Drama" })
  .where("id = :id", { id: 1 })
  .execute();
```

<br>

## Query Builder의 5가지 실행 타입 : 4 DELETE

```jsx
await dataSource
  .createQueryBuilder()
  .delete()
  .from(movie)
  .where("id = :id", { id: 1 })
  .execute();
```

<br>

## Query Builder의 5가지 실행 타입 : 5 RELATIONS

```jsx
const genres = await dataSource
  .createQueryBuilder()
  .relation(Movie, "genres")
  .of(1) // Movie id
  .loadMany();
```

### getOne(), getMany(), select()

```jsx
// 단일 Row만 가져올때
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .select(["user.id", "user.firstName", "user.lastName"])
  .getOne();
```

```jsx
// 복수 Row 가져올때
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .select(["user.id", "user.firstName", "user.lastName"])
  .getMany();
```

<br>

## where()

```jsx
// 하나의 필터링 조건 적용
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .where("user.isActive = :isActive", { isActive: true })
  .getMany();
```

```jsx
// 다수의 필터링 조건 적용
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .where("user.fistName = :firstName", { firstName: "John" })
  .andWhere("user.lastName = :lastName", { lastName: "Doe" })
  .getMany();
```

<br>

## orderBy()

```jsx
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .orderBy("user.lastName", "ASC")
  .addOrderBy("user.firstName", "DESC")
  .getMany();
```

<br>

## skip() take()

```jsx
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .skip(10) // 11번째부터 데이터 가져옴
  .take(5) // 5개 가져옴
  .getMany();
```

<br>

## Join()

```jsx
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .innerJoinAndSelect("user.profile", "profile") // innerJoin 이기 때문에 null값이 있는것은 select 되지않음
  .getMany();
```

```jsx
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .getMany();
```

<br>

## Aggregation()

```jsx
const userCount = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .select("COUNT(user.id)", "count")
  .getRawOne();
```

<br>

## Subquery()

```jsx
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select("subUser.id")
      .from(User, "subUser")
      .where("subUser.isActive = :isActive", { isActive: true });
    return "user.id IN " + subQuery;
  })
  .setParameter("isActive", true)
  .getMany();
```
