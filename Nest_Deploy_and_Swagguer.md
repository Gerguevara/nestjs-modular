#Configuraciones para deploymen

#habilitar cors 
 `app.enableCors();`
 `await app.listen(process.env.PORT || 3000);`
- corse pede recibir un strings de que dns especifico va a recibir
- algunos servidores piden configurar le comando start o crear un archivo script que lo haga
- recordar que tambien piden variables entorno
- el comando de nest que se usta para producion es `npm run start:prod` este comando es el que se pone en archivo  	de script.
# Nest with Swagger

https://docs.nestjs.com/openapi/introduction

Para instalar utilizamos uno de los siguiente comandos dependiendo  del motor
elegido ya sea fastify o swagguer


`$ npm install --save @nestjs/swagger swagger-ui-express`
` npm install --save @nestjs/swagger fastify-swagger`

luego se debe hacer esta configuracion en el archivo main.ts

import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
`import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';`
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
    }),
  );

 ` const config = new DocumentBuilder()`
  `  .setTitle('API')`
  `  .setDescription('PLATZI STORE')`
   ` .setVersion('1.0')`
   ` .build();`
 ` const document = SwaggerModule.createDocument(app, config);`
 ` SwaggerModule.setup('docs', app, document);`
  await app.listen(3000);
}
bootstrap();


esta configura creara nuestra documentacion de la url de la api  localhost:3000/docs

================================================================================================
#Agregar informacion dtos en la pagina de coumentacion de swagguer de documentacion

primeramente deben terner la terminacion .dto.ts

luego debemos habilitar el pluguin de swagguer que lee los dto en el archivo `nest.cli.json`, agregando
la opcion compiler options 
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
 ` "compilerOptions": {`
  `  "plugins": ["@nestjs/swagger/plugin"]`
  }
}

y luego en los dtos al utilizar partial types ya no no debe ser tomado de 
`@nestjs/mapped-types` sino ahora debe vernia de swagguer


-nota swagguer genera archivos estatiocos asi que si se cambia la configurarcion lo mejor es elminar la carpeta dist
`rm -rf dist`
===================================================================================================
# Decoradores en swagguer

#Info extra en los dtos 
para ello utilizamo el decorador ApiProterty de esta forma:
`import { PartialType, ApiProperty } from '@nestjs/swagger';`

export class CreateUserDto {
  @IsString()
  @IsEmail()
 ` @ApiProperty({ description: 'the email of user' })`
  readonly email: string;
}

# Agrupar enpoints por controladores
 importamos estos paguetes 
 `import { ApiTags, ApiOperation } from '@nestjs/swagger';`


y los colocamos en los metodos de esta forma:

en el controler aplicamos este decorador y eso automaticamente crea una agrupacion 

`@ApiTags('products')`
@Controller('products')
export class ProductsController {... }


y para agrega runa descripcion a cada controle utilizamos apiOperations de esta forma

` @ApiOperation({ summary: 'List of products' })`
  getProducts(
    @Query('limit') limit = 100,
    @Query('offset') offset = 0,
    @Query('brand') brand: string,
  ) {...}
  
  
  
