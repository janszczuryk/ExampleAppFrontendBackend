# Example App - Frontend & Backend

## Instructions

1. **Backend**

    Scaffold backend nest.js project:
    ```bash
    npx @nestjs/cli@latest new nest-backend
    cd nest-backend
    ```
    
    Install dependencies:
    ```bash
    npm install --save pg typeorm @nestjs/typeorm
    ```
    
    Run database container with PostgreSQL:
    ```bash
    docker run -d \
      --name postgres \
      -e POSTGRES_USER=postgres \
      -e POSTGRES_PASSWORD=postgres \
      -e POSTGRES_DB=postgres \
      -e PGDATA=/var/lib/postgresql/data/pgdata \
      -v ${PWD}/db:/var/lib/postgresql/data \
      -p '5432:5432' \
      postgres
    ```
    
    Edit `app.module.ts` to import TypeORM:
    ```ts
    import { Module } from '@nestjs/common';
    import { TypeOrmModule } from '@nestjs/typeorm';
    
    @Module({
      imports: [
        TypeOrmModule.forRoot({
          type: 'postgres',
          host: 'localhost',
          port: 5432,
          username: 'postgres',
          password: 'postgres',
          database: 'postgres',
          autoLoadEntities: true,
          synchronize: true,
        }),
      ],
    })
    export class AppModule {}
    ```
    
    Edit `main.ts` to support CORS HTTP headers:
    ```bash
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      app.enableCors();
      await app.listen(3000);
    }
    bootstrap();
    ```
    
    Generate user module:
    ```bash
    npx @nestjs/cli@latest generate resource user
    ```
    
    Edit `user.entity.ts` to use TypeORM decorators:
    ```ts
    import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';
    
    @Entity()
    export class User {
      @PrimaryGeneratedColumn()
      id: number;
    
      @Column()
      firstName: string;
    
      @Column()
      lastName: string;
    }
    ```
    
    Edit `user.module.ts` to import TypeORM user entity:
    ```ts
    import { Module } from '@nestjs/common';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { User } from './entities/user.entity';
    import { UserService } from './user.service';
    import { UserController } from './user.controller';
    
    @Module({
      imports: [TypeOrmModule.forFeature([User])],
      controllers: [UserController],
      providers: [UserService],
    })
    export class UserModule {}
    ```
    
    Edit `user.controller.ts` to redefine new get endpoint:
    ```ts
    import { Controller, Get } from '@nestjs/common';
    import { UserService } from './user.service';
    
    @Controller('user')
    export class UserController {
      constructor(private readonly userService: UserService) {}
    
      @Get()
      findAll() {
        return this.userService.findAll();
      }
    }
    ```
    
    Lastly edit `user.service.ts` to provide data to the controller:
    ```ts
    import { Injectable } from '@nestjs/common';
    import { InjectRepository } from '@nestjs/typeorm';
    import { Repository } from 'typeorm';
    import { User } from './entities/user.entity';
    
    @Injectable()
    export class UserService {
      constructor(
        @InjectRepository(User)
        private userRepository: Repository<User>,
      ) {}
    
      findAll(): Promise<User[]> {
        return this.userRepository.find();
      }
    }
    ```
   
2. **Frontend**

   Scaffold frontend Vue.js with TypeScript project via Vite:
   ```bash
   npm create vite@latest vue-frontend -- --template vue-ts
   cd vue-frontend
   ```
   
   Install npm dependencies for the first time:
   ```bash
   npm install
   ```
   
   Create `UserList.vue` component:
   ```vue
   <script setup lang="ts">
   import { ref } from 'vue'
   
   const userList = ref([
     { id: 1, firstName: 'John', lastName: 'Doe' },
   ]);
   </script>
   
   <template>
     <p><strong>Users list</strong></p>
     <table>
       <thead>
         <tr><th>ID</th><th>First name</th><th>Last name</th></tr>
       </thead>
       <tbody>
       <tr v-for="(user, i) in userList" :key="i">
         <td>{{ user.id }}</td><td>{{ user.firstName }}</td><td>{{ user.lastName }}</td>
       </tr>
       </tbody>
     </table>
   </template>
   
   <style scoped>
   table {
     border: 1px solid white;
     border-collapse: collapse;
   }
   table td, table th {
     border: 1px solid white;
     padding: 10px;
   }
   </style>
   ```
   
   Include `UserList.vue` component inside main app component - `App.vue`:
   ```vue
   <script setup lang="ts">
   import UserList from './components/UserList.vue'
   </script>
   
   <template>
       <UserList/>
   </template>
   ```
   
   Start development server for preview (Hot Reload included):
   ```bash
   npm run dev
   ```
   
   Edit `UserList.vue` component to fetch user list data from the backend:
   ```vue
   <script setup lang="ts">
   import { onMounted, ref } from 'vue'
   
   const userList = ref([]);
   
   onMounted(async () => {
     const response = await fetch('http://localhost:3000/user');
     userList.value = await response.json();
   });
   </script>
   
   <template>
     <p><strong>Users list</strong></p>
     <table>
       <thead>
       <tr><th>ID</th><th>First name</th><th>Last name</th></tr>
       </thead>
       <tbody>
       <tr v-for="(user, i) in userList" :key="i">
         <td>{{ user.id }}</td><td>{{ user.firstName }}</td><td>{{ user.lastName }}</td>
       </tr>
       </tbody>
     </table>
   </template>
   
   <style scoped>
   table {
     border: 1px solid white;
     border-collapse: collapse;
   }
   table td, table th {
     border: 1px solid white;
     padding: 10px;
   }
   </style>
   ```
