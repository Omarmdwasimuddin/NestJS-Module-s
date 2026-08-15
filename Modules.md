# NestJS Modules 

## Module আসলে কী?

**Module** হলো একটা class, যেটার উপরে `@Module()` decorator বসানো থাকে। এই decorator একটা metadata দেয়, যেটা দিয়ে Nest পুরো application-এর structure organize আর manage করে।

<img width="970" height="526" alt="image" src="https://github.com/user-attachments/assets/37f47c0e-694f-4abe-a62a-b9c2587bb282" />

প্রতিটা Nest application-এ অন্তত একটা module থাকতেই হয় — সেটাকে বলে **root module**। এটাই সেই starting point, যেখান থেকে Nest পুরো **application graph** বানায়। এই graph হলো একটা internal structure, যেটা দিয়ে Nest বুঝতে পারে কোন module আর কোন provider একে অপরের সাথে কীভাবে সম্পর্কিত (dependency)।

ছোট application হলে হয়তো শুধু root module দিয়েই চলে যায়, কিন্তু বেশিরভাগ ক্ষেত্রে multiple module ব্যবহার করাই ভালো — প্রতিটা module একটা নির্দিষ্ট, কাছাকাছি সম্পর্কিত (closely related) capability-কে একসাথে রাখে (encapsulate করে)।

---

## `@Module()` Decorator-এর প্রপার্টিগুলো

`@Module()` একটা object নেয়, যেখানে চারটা গুরুত্বপূর্ণ property থাকে:

| Property | কাজ |
|---|---|
| `providers` | এই module-এর যে provider-গুলো Nest injector দিয়ে instantiate হবে, এবং যেগুলো অন্তত এই module-এর ভেতরে share করা যাবে |
| `controllers` | এই module-এ থাকা controller-গুলো, যেগুলো instantiate করতে হবে |
| `imports` | যে module-গুলো import করা হচ্ছে, কারণ এই module-এর জন্য দরকারি provider সেগুলো export করে |
| `exports` | এই module-এর কোন provider-গুলো অন্য module-এ (যারা এটাকে import করবে) available করা হবে — provider নিজে অথবা শুধু তার token দিয়েও দেওয়া যায় |

**গুরুত্বপূর্ণ নিয়ম:** একটা module by default তার provider-গুলোকে **encapsulate** করে রাখে। মানে তুমি শুধু সেই provider-ই inject করতে পারবে যেটা:

- এই module-এরই অংশ, অথবা
- অন্য কোনো imported module থেকে explicitly **export** করা হয়েছে

একটা module থেকে export হওয়া provider-গুলো আসলে সেই module-এর **public interface / API**-এর মতো কাজ করে।

---

## Feature Module — একটা নির্দিষ্ট feature-কে গুছিয়ে রাখা

আমাদের উদাহরণে `CatsController` আর `CatsService` — দুইটাই একই domain (cats)-এর সাথে সম্পর্কিত। এগুলোকে একসাথে একটা **feature module**-এ রাখাই যুক্তিসঙ্গত।

Feature module একটা নির্দিষ্ট feature সংক্রান্ত সব code একসাথে organize করে, যাতে clear boundary থাকে এবং code বড় হলেও গোছানো থাকে (SOLID principle-এর সাথে সামঞ্জস্যপূর্ণ)।

```ts
// cats/cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> **Tip:** CLI দিয়ে module বানাতে চাইলে কমান্ড: `nest g module cats`

এরপর এই `CatsModule`-কে root module (`AppModule`)-এ import করে দিতে হবে:

```ts
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

### Directory Structure এখন এরকম দেখাবে

```
src
 └── cats
      ├── dto
      │    └── create-cat.dto.ts
      ├── interfaces
      │    └── cat.interface.ts
      ├── cats.controller.ts
      ├── cats.module.ts
      └── cats.service.ts
 ├── app.module.ts
 └── main.ts
```

---

## Shared Module — একই Instance সব জায়গায় শেয়ার করা

Nest-এ module-গুলো by default **singleton** — মানে একটা provider-এর একটাই instance বানে এবং সেটাই সব module-এর মধ্যে share হয়।

<img width="970" height="373" alt="image" src="https://github.com/user-attachments/assets/25bf6860-d252-4e5f-bd0a-489e806f6d9c" />

প্রতিটা module আসলে automatically একটা shared module — একবার বানানো হলে সেটা যেকোনো module পুনরায় ব্যবহার করতে পারে। ধরো তুমি `CatsService`-এর একটা instance একাধিক module-এর মধ্যে share করতে চাও — এর জন্য প্রথমে `CatsService`-কে `exports` array-তে যোগ করতে হবে:

```ts
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

এখন যে যে module `CatsModule` import করবে, তারা সবাই `CatsService`-এর **একই instance** পাবে।

### কেন এটা গুরুত্বপূর্ণ?

যদি তুমি প্রতিটা module-এ আলাদা করে `CatsService` register করতে (export না করে), তাহলে প্রতিটা module তার নিজের আলাদা instance পেত। এর ফলে:

- Memory বেশি খরচ হতো (একই service-এর একাধিক copy তৈরি হতো)
- State inconsistency হতে পারত — যদি service-টার ভেতরে কোনো internal state থাকে, সেটা module ভেদে আলাদা হয়ে যেত

`CatsService`-কে `CatsModule`-এর ভেতরে encapsulate করে export করলে সব module একই instance ব্যবহার করে — এতে memory কম লাগে, আর behavior predictable থাকে। এটাই NestJS-এ modularity আর dependency injection-এর একটা বড় সুবিধা।

---

## Module Re-exporting

একটা module শুধু তার নিজের provider export করতে পারে তা না, সে যেই module import করেছে সেটাকেও আবার **re-export** করতে পারে:

```ts
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

এখানে `CommonModule`-কে `CoreModule` নিজে import করছে এবং একই সাথে export-ও করছে — ফলে যে module `CoreModule` import করবে, সে সরাসরি `CommonModule`-এর provider-ও পাবে।

---

## Module-এর ভেতরে Dependency Injection

একটা module class নিজেও provider inject করতে পারে (যেমন configuration-এর প্রয়োজনে):

```ts
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

তবে মনে রাখতে হবে — **module class নিজে কখনো provider হিসেবে inject করা যায় না**, কারণ এতে circular dependency তৈরি হয়।

---

## Global Module

যদি একটা particular module প্রায় সব জায়গায় import করতে হয়, সেটা একটু ঝামেলার হয়ে যায়। Angular-এ provider-রা global scope-এ register হয় (একবার define করলেই সব জায়গায় পাওয়া যায়), কিন্তু Nest by default provider-দের **module scope**-এর ভেতরে encapsulate করে রাখে — encapsulating module import না করলে সেই provider ব্যবহার করা যায় না।

যদি কিছু provider সব জায়গায় out-of-the-box available রাখতে চাও (যেমন helper, database connection ইত্যাদি), তাহলে সেই module-কে `@Global()` decorator দিয়ে **global** বানানো যায়:

```ts
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

`@Global()` module সাধারণত শুধু একবার register করা উচিত, আর সেটা root module বা core module থেকেই করাই ভালো। এরপর যেকোনো module `CatsService` ব্যবহার করতে চাইলে `imports` array-তে `CatsModule` আনার দরকার পড়বে না — এটা সবখানেই পাওয়া যাবে।

> **সতর্কতা:** সব কিছু global বানিয়ে ফেলা ভালো practice না। Global module boilerplate কমায় ঠিকই, কিন্তু `imports` array ব্যবহার করে module-এর API নিয়ন্ত্রিতভাবে অন্য module-এ available করাই বেশি ভালো — এতে structure এবং maintainability ভালো থাকে, এবং অপ্রয়োজনীয় coupling এড়ানো যায়।

---

## Dynamic Module — Runtime-এ Configure করা যায় এমন Module

**Dynamic module** দিয়ে এমন module বানানো যায় যেটা runtime-এ configure করা যায়। কোনো option বা configuration-এর উপর ভিত্তি করে flexible, customizable module দরকার হলে এটা কাজে লাগে।

```ts
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
  exports: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

> **Tip:** `forRoot()` method synchronously অথবা asynchronously (Promise-এর মাধ্যমে) — দুইভাবেই dynamic module return করতে পারে।

এখানে `DatabaseModule` by default `Connection` provider দেয় (`@Module()`-এর মেটাডেটায় লেখা), কিন্তু `forRoot()`-এ পাস করা `entities` আর `options`-এর উপর ভিত্তি করে আরও provider (যেমন repository) generate করে দেয়।

**মনে রাখার বিষয়:** dynamic module থেকে return হওয়া properties গুলো `@Module()`-এ দেওয়া base metadata-কে **override না করে extend করে** — অর্থাৎ statically declare করা `Connection` provider আর dynamically তৈরি হওয়া provider — দুটোই export হয়।

### Dynamic Module-কে Global বানানো

```ts
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

### ব্যবহার করার নিয়ম

```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

### Re-export করতে চাইলে

যদি এই dynamic module-কে আবার re-export করতে চাও, `exports` array-তে `forRoot()` call না করে সরাসরি module নাম দিলেই হবে:

```ts
@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

---

## সংক্ষেপে — কোনটা কখন ব্যবহার করবে

| Concept | কখন ব্যবহার করবে |
|---|---|
| **Feature Module** | একই domain-এর controller ও service একসাথে গুছিয়ে রাখতে |
| **`exports`** | একটা provider-এর instance অন্য module-এর সাথে share করতে |
| **Module Re-export** | Import করা module-কে আবার নিজের consumer-দের কাছে available করতে |
| **`@Global()`** | Helper/database connection-এর মতো provider সব জায়গায় সহজলভ্য রাখতে (কিন্তু কম ব্যবহার করাই ভালো) |
| **Dynamic Module (`forRoot()`)** | Runtime-এ configuration অনুযায়ী module customize করতে |
