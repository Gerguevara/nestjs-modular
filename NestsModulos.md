# Modulos NestJS

#Comando para generar modulos

`Nest g mo nombre-modulo`
Esto genera automaticamente la carpeta y el modulo dentro

Arquitectura modula

---Modulo
        -Servicios
        -Controllers
        -DTOs
        -Entities
 No todos los entities necesitan un modulo precisamente, es mejor
  persanlos segun el funciomiento que tendran los controlllers
  
  Tambiebn se puede crear una carpeta common en funcion de cosas que 
  todos los modulos puede requerir
  
==========================================================================================

# Estructura Basica de un modulo

@Module({
  imports: [ProductsModule],
  controllers: [CustomerController, UsersController],
  providers: [CustomersService, UsersService],
   exports: [UsaerService],
})
export class UsersModule {}


- Imports: pueden ser otros modulos
- controllers: todos los controles que este modulo utiliza
- Providers: todos los servicios que la appa utiliza
- Exports: puedes importar los servicios de este modulo a otros modulos

=============================================================================================

# interaccion entre modulos

-No hay ningun problema si un modulo rquiere utilizar clases de otro moidulo,
 estas solo son clases
 
-Para utilizar la logia de un servicio que esta en otro modulo, es necesario importarlo, por ello
se pone en el key imports , sino dara error al momento de compilar, asi mismo es necesario que el otro modulo
el exportado exporte esos servicios en su key exports

==================================================================================================
# Singleton aplicado a service
EL patron singleton de nest esta enfocado a los servicios para poder aplicar este patron
simplemente basta con que tenga el decorador @inyectable que todos los servicios traen por defecto,
luego solo falta inyectarlo en un constructor

Recordar que un servicio debe siempre pertencer a un modulo ya sea un creado por nosotros o el appmopdule
y solamente pertenecer a un modulo

Los servicios pueden ser inyectados en otros servicios, solo debe mantener el cuidaod de tener 
dependencia circular es decir servicio a al b y b al a.
===================================================================================================

# Tipos de providers

- useClass: es el modulo que usamos por defecto para decirle al modulo que necesitamso inyectar un servicio

 Basicamente hacemos esto providers: [CatsService], pero Nest lo interpreta como esto

 providers: [
  {
    provide: CatsService,
    useClass: CatsService,
  },
 ] 
 
# useValue: que es comno sueno un valor proveido en el modulo este literalmente puede utilizar
  cualquier valor de ts, como ejemplo un api key para compartir en varios servicios, este podria 
  ser inyectado por medio de inyeccion ojo providey es su key y useValue su valor real, en este 
  caso detectamos en que ambiente estamos

const API_KEY = '12345634';
const API_KEY_PROD = 'PROD1212121SA';

@Module({  
  providers: [
    AppService,
 `   {`
 `     provide: 'API_KEY',`
`      useValue: process.env.NODE_ENV === 'prod' ? API_KEY_PROD : API_KEY,`
 `   },`
  ],
})
export class AppModule {}
	
y para inyectarlo utilizamos directemente el decorador @inject y le damos su nombre de provide

import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class AppService {
  constructor(@Inject('API_KEY') private apiKey: string) {}
  getHello(): string {
    return `Hello World! ${this.apiKey}`;
  }
}

extra para decir que ambien queremos correr podes usar el comando NODE_ENV=pro npm run start:dev

# UseFactory: este provider nos permite fabricarlo de forma asyncrona , este tipo de provider relentiza
  el arranque del servicio y controlador por lo tanto no es buena idea usarlo para peticciones http a apis
  su mejor uso es para realizar conexiones a bases de datos.
  
@Module({
  imports: [HttpModule, UsersModule, ProductsModule],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provide: 'TASKS',
  `    useFactory: async (http: HttpService) => {`
        const tasks = await http
          .get('https://jsonplaceholder.typicode.com/todos')
          .toPromise();
        return tasks.data;
      },
      inject: [HttpService],
    },
  ],
})
export class AppModule {}



@Injectable()
export class AppService {
  constructor(
 
    @Inject('TASKS') private tasks: any[],
  ) {}
  getHello(): string {
    console.log(this.tasks);
    return  null
  }
}
================================================================================================

# GlobalModule

Ojo primeramente este modulo no es el mismo que appModule, todo lo que se inyecte en este modulo 
se puede utilizar en toda la aplicacion y no es necesario importarlo en otros submodulos
Se genera con el mismop comandop de los modulos normal 'ng g mo nombre-modulo', la unica diferencia de 
este molulo es que utiliza el decorador `@Global` para decir que es global y ya, pero ojo que lo que
se desee compartir debe estar en la key export sin embargo no se debe exportar ya en otros modulos solo
usar sus valores

`import { Module, Global } from '@nestjs/common';`
`const API_KEY = '12345634';`
`const API_KEY_PROD = 'PROD1212121SA';`
`@Global()`
`@Module({`
 ` providers: [`
  `  {`
  `    provide: 'API_KEY',`
    `  useValue: process.env.NODE_ENV === 'prod' ? API_KEY_PROD : API_KEY,`
 `   },`
 ` ],`
  `exports: ['API_KEY'],`
`})`
`export class DatabaseModule {}`

Este modulo puede ayudar a solucionar problemas de dependencia circular
===================================================================================================

#Config Module

Este modulo lo podemos utilizar para tener variables de entorno concexion a DB, api keys etc.

Para utilizar intalamos el modulo de nest 'npm i --save @nestjs/config' que contiene el paque de dotenv npm 
para utilizar variables de entorno.

para usar simplemente creaomor archivos .env y dentro ponemos las variables

- DATABASE_NAME = my_db
- API_NAME = sajdhasdjasjdhashdlhdsa

y para utilizar importamos el este configModule en otro modulo, le indicamos en el .forRoot de
configuracion el archivo que debe leer y tambien si es un modulo global

`import { Module, HttpModule, HttpService } from '@nestjs/common';`
`import { ConfigModule } from '@nestjs/config';`
`@Module({`
 ` imports: [`
  `  ConfigModule.forRoot({`
    `  envFilePath: '.env',`
   `   isGlobal: true,`
  `  }),`
   ` HttpModule,   `
 ` ],`
 ` controllers: [AppController],`
 ` providers: [ AppService ],`
`})`
`export class AppModule {}`


para utilizar en lo podemos solicitar por medio de otro servicio por medio del configService propio de la libreria instalada de esta manera

import { Injectable, NotFoundException, Inject } from '@nestjs/common';
`import { ConfigService } from '@nestjs/config';`
...

@Injectable()
export class UsersService {
  constructor(
    private productsService: ProductsService,
 `   private configService: ConfigService,`
  ) {}


  findAll() {
  `  const apiKey = this.configService.get('API_KEY');`
  `  const dbName = this.configService.get('DATABASE_NAME');`
    console.log(apiKey, dbName);
    return this.users;
  }
}
=====================================================================================================
#Manejo de multiples ambientes (arvhicos .env) 
// recoradr los archivos siempre deben ser ignorados en git
ejemplo prod.env stag.env dev.env

un ejmplo para manejarlo es creat una clase en un archivo ts que lo manje por ejemplo

enviromets.ts  con esta constante

`export const enviroments = {`
 ` dev: '.env',`
 ` stag: '.stag.env',`
 ` prod: '.prod.env',`
`};`

luego lo llamamos en el modulo donde tenemos el cofigModule de esta forma

import { Module, HttpModule, HttpService } from '@nestjs/common';
`import { ConfigModule } from '@nestjs/config';`
`import { enviroments } from './enviroments';`
@Module({
  imports: [
`    ConfigModule.forRoot({`
`      envFilePath: enviroments[process.env.NODE_ENV] || '.env',`
`      isGlobal: true,`
   }),
    HttpModule,
   UsersModule
  ],
  controllers: [AppController],
  providers: [  AppService ],`
})
export class AppModule {}                              


aca le estamos diciento que tome el anviente de nove (el que se pasa con comando y que segun el ambien resuelva
haciendolo como un tipo de buiqueda index con el valor en el array resuelva que archivo .env tomar 
y si no que tome el por defect .env

y para utilizar otra forma mas apropiada de hacerlo es importar el configSerive en el otro servicio que lo necesitemos, llamarlo en el constructror

 `constructor(   private configService: ConfigService) {}`
 
 y luego utilizar get para tener su valor, de forma opcional puede darsele un tipado
  
 `const apiKey = this.configService.get<strign>('API_KEY');`
===========================================================================================================

#Agragando tipado para prevenir tipos (buena practica pero opcional)

creamos un archivo config.ts (nombre opcional) con el siguiente contenido: 

`import { registerAs } from '@nestjs/config';`
`export default registerAs('config', () => {`
 ` return {`
  `  database: {`
   `   name: process.env.DATABASE_NAME,`
   `   port: process.env.DATABASE_PORT,`
  `  },`
 `   apiKey: process.env.API_KEY,`
 ` };`
`});`


-y para utilizar, ya no utilizariamos configService sino configType que es de la misma libreria
igualmente inyectado, pero muy importante antes ponerlo en el modulo donde se utilizara,
generalmente es un mmodulo global en la key load, esto es solo para dar tipado

`import config from './config';`

@Module({
  imports: [
`    ConfigModule.forRoot({`
`      envFilePath: enviroments[process.env.NODE_ENV] || '.env',`
`     load: [config],`
`     isGlobal: true,`
    }),   
  ],
  controllers: [AppController],
  providers: [
    AppService
  ],
})
export class AppModule {}


- y para utilizar ya propiamente en un servicio

`import config from './config';`

@Injectable()
export class AppService {
  constructor(
    // @Inject('API_KEY') private apiKey: string, 
   ` @Inject(config.KEY) private configService: ConfigType<typeof config>,`
  ) {}
  getHello(): string {
  `  const apiKey = this.configService.apiKey;`
  `  const name = this.configService.database.name;`
    return 'Hello World! ${apiKey} ${name}';
  }
}
============================================================================================================
# Validos variables de Ambien Externs
Para esto utilizamos un paque externo a Nest llamado Joi
`npm install joi`
para utilizar, le decimos en el modulo don estamos las variables de mabien que utilice esta validacion asi:

`import * as Joi from 'joi';`

 imports: [
    ConfigModule.forRoot({
      envFilePath: enviroments[process.env.NODE_ENV] || '.env',
      load: [config],
      isGlobal: true,
      validationSchema: Joi.object({
        API_KEY: Joi.number().required(),
        DATABASE_NAME: Joi.string().required(),
        DATABASE_PORT: Joi.number().required(),
      }),
    }),
    HttpModule,
    UsersModule   
  ],


