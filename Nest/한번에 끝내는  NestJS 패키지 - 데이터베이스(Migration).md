![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br><br>

## Migration

<br>

- Migration은 데이터베이스 변경사항을 스크립트로 작성해서 반영한다. 통제된 상황에서 데이터베이스 스키마 변경 및 복구를 진행 할 수 있다.

<br>

![alt text](/Nest/image/migration.png)

<br>

![alt text](/Nest/image/migration2.png)

<br>

![alt text](/Nest/image/migration3.png)

<br>

## Migration Configuration : ormconfig.json

```json
{
  "type": "postgres",
  "host": "localhost",
  "port": 5432,
  "username": "test",
  "password": "test",
  "database": "test",
  "entities": ["src/entity/**/*.ts"],
  "entities": ["src/migration/**/*.ts"],
  "cli": {
    "entitiesDir": "src/entity",
    "migrationsDir": "src/migration"
  }
}
```

## Migration : 테이블 생성

### **RAW SQL**

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateMovieAndDirectorTables1634567890123
  implements MigrationInterface
{
  // 마이그레이션 진행
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            CREATE TABLE "director" (
                "id" SERIAL NOT NULL,
                "name" VARCHAR NOT NULL,
                "dob" DATE,
                "nationality" VARCHAR,
                PRIMARY KEY ("id")
            )
        `);

    await queryRunner.query(`
            CREATE TABLE "movie" (
                "id" SERIAL NOT NULL,
                "title" VARCHAR NOT NULL,
                "genre" VARCHAR,
                "directorId" INTEGER,
                PRIMARY KEY ("id"),
                FOREIGN KEY ("directorId") REFERENCES "director"("id") ON DELETE SET NULL
            )
        `);
  }

  // 마이그레이션 롤백
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "movie"`);
    await queryRunner.query(`DROP TABLE "director"`);
  }
}
```

### Migration API

```tsx
import {
  MigrationInterface,
  QueryRunner,
  Table,
  TableForeignKey,
} from "typeorm";

export class CreateMovieAndDirectorTables1634567890123
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    //... 생략

    await queryRunner.createTable(
      new Table({
        name: "movie",
        columns: [
          {
            name: "id",
            type: "int",
            isPrimary: true,
            isGenerated: true,
            generationStrategy: "increment",
          },
          {
            name: "title",
            type: "varchar",
          },
          {
            name: "genre",
            type: "varchar",
            isNullable: true,
          },
          {
            name: "directorId",
            type: "int",
            isNullable: true,
          },
        ],
      }),
      true
    );

    await queryRunner.createForeignKey(
      "movie",
      new TableForeignKey({
        columnNames: ["directorId"],
        referencedColumnNames: ["id"],
        referencedTableName: "director",
        onDelete: "SET NULL",
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable("movie");
    await queryRunner.dropTable("director");
  }
}
```

## Migration : 칼럼 추가

### **RAW SQL**

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class AddDateOfBirthToDirector1634567890124
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            ALTER TABLE "director"
            ADD "dateOfBirth" DATE
        `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            ALTER TABLE "director"
            DROP COLUMN "dateOfBirth"
        `);
  }
}
```

### Migration API

```tsx
import { MigrationInterface, QueryRunner, TableColumn } from "typeorm";

export class AddDateOfBirthToDirector1634567890124
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      "director",
      new TableColumn({
        name: "dateOfBirth",
        type: "date",
        isNullable: true,
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn("director", "dateOfBirth");
  }
}
```

## Migration : 칼럼 이름 변경

### **RAW SQL**

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class RenameNameToFullNameInDirector1634567890125
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            ALTER TABLE "director"
            RENAME COLUMN "name" TO "fullName"
        `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            ALTER TABLE "director"
            RENAME COLUMN "fullName" TO "name"
        `);
  }
}
```

### Migration API

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class RenameNameToFullNameInDirector1634567890125
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.renameColumn("director", "name", "fullName");
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.renameColumn("director", "fullName", "name");
  }
}
```

## Migration : 칼럼 타입 변경

### **RAW SQL**

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class ChangeEmailTypeInDirector1634567890126
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            ALTER TABLE "director"
            ALTER COLUMN "email" TYPE TEXT
        `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            ALTER TABLE "director"
            ALTER COLUMN "email" TYPE VARCHAR
        `);
  }
}
```

<br>

### Migration API

```tsx
import { MigrationInterface, QueryRunner, TableColumn } from "typeorm";

export class ChangeEmailTypeInDirector1634567890126
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.changeColumn(
      "director",
      "email",
      new TableColumn({
        name: "email",
        type: "text",
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.changeColumn(
      "director",
      "email",
      new TableColumn({
        name: "email",
        type: "varchar",
      })
    );
  }
}
```

<br>

## Migration : Relationship 작업

### **RAW SQL**

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateGenreAndMovieGenreTables1634567890127
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
            CREATE TABLE "genre" (
                "id" SERIAL NOT NULL,
                "name" VARCHAR NOT NULL,
                PRIMARY KEY ("id")
            )
        `);

    await queryRunner.query(`
            CREATE TABLE "movie_genres_genre" (
                "movieId" INTEGER NOT NULL,
                "genreId" INTEGER NOT NULL,
                PRIMARY KEY ("movieId", "genreId"),
                FOREIGN KEY ("movieId") REFERENCES "movie"("id") ON DELETE CASCADE,
                FOREIGN KEY ("genreId") REFERENCES "genre"("id") ON DELETE CASCADE
            )
        `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "movie_genres_genre"`);
    await queryRunner.query(`DROP TABLE "genre"`);
  }
}
```

### Migration API

```tsx
import {
  MigrationInterface,
  QueryRunner,
  Table,
  TableForeignKey,
} from "typeorm";

export class CreateGenreAndMovieGenreTables1634567890127
  implements MigrationInterface
{
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: "genre",
        columns: [
          {
            name: "id",
            type: "int",
            isPrimary: true,
            isGenerated: true,
            generationStrategy: "increment",
          },
          {
            name: "name",
            type: "varchar",
          },
        ],
      }),
      true
    );

    //...생략
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable("movie_genres_genre");
    await queryRunner.dropTable("genre");
  }
}
```

## Migration CLI 커맨드

```markdown
# Migration 파일 생성하기

npx typeorm migration:generate -n <MigrationName>

# Migration 실행하기

npx typeorm migration:run

# Migration 되돌리기

npx typeorm migration:revert
```
