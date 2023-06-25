---
title: 如何在nest中使用typeorm.md
date: 2023-06-25 11:36:37
tags: [Nest]
index_img: /img/nest.svg
---
# 如何在nest中使用typeorm

## 安装相关依赖

```bash
pnpm install @nestjs/typeorm typeorm mysql2 -S
```

## 创建测试 entity

```bash
nest g resource user
```

修改 entity user 的属性

```ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({
    length: 50,
  })
  name: string;
}

```

## 在入口app.module中添加 typeOrmModule 模块

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    UserModule,
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'typeorm_mysql',
      synchronize: true,
      logging: true,
      entities: ['./**/entities/*.ts'],
      poolSize: 10,
      migrations: [],
      subscribers: [],
      connectorPackage: 'mysql2',
      extra: {
        authPlugin: 'sha256_password',
      },
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

## 在对应的实体service中实现CURD

- 注入entityManager的方式

```ts
// user.service.ts

import { Injectable } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { InjectEntityManager } from '@nestjs/typeorm';
import { EntityManager } from 'typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UserService {
  // 注入entityManager
  @InjectEntityManager()
  private manager: EntityManager;

  create(createUserDto: CreateUserDto) {
    this.manager.save(User, createUserDto);
  }

}
```

- 注入 Repository 的方式

```ts
// user.module.ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';
import { User } from './entities/user.entity';
import { TypeOrmModule } from '@nestjs/typeorm';
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}


// user.service.ts
import { Injectable } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { InjectRepository } from '@nestjs/typeorm';
import { EntityManager, Repository } from 'typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UserService {
  // 注入Repository
  @InjectRepository(User)
  private userRepository: Repository<User>;

  findAll() {
    return this.userRepository.find();
  }
}

```
