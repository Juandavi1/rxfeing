### RxFeign (Request en Sevidor y Cliente) JS

#### Motivacion
realizar peticiones por medio de anotaciones

#### Instalacion 
```sh
$ yarn add axios rxjs rxfeign

or

$ npm install --save axios rxjs rxfeign
```

```typescript
import { Get , Client,PathParam, Query, HttpObservable } from 'rxfeign';
```

#### Anotaciones
   - *@Client*  
   recibe dos tipos de parametro, un string o un objeto
        * string : url base (http://domain/)
        * objecto : permite definir una configuracion global para todos los request dentro de esta clase. este objeto definido de la siguiente manera:

```typescript
    export interface FeignConfig {
        url?: string;
        headers?: { [key: string]: any };
        timeout?: number;
        withCredentials?: boolean;
        adapter?: AxiosAdapter;
        auth?: AxiosBasicCredentials;
        responseType?: string;
        xsrfCookieName?: string;
        xsrfHeaderName?: string;
        maxContentLength?: number;
        maxRedirects?: number;
        httpAgent?: any;
        httpsAgent?: any;
        proxy?: AxiosProxyConfig | false;
    }
```   
   ejemplo: 
```typescript
    @Client({
        url: 'https://jsonplaceholder.typicode.com/posts/',
        headers:{}, 
        timeout:99
    })
    export class User{}
    
    o
    
    @Client('https://jsonplaceholder.typicode.com/posts/')
    export class User{}
```
   - *@Config*  
   esta anotacion recibe por parametro el mismo objeto que se define en la anotacion *@Client*
   pero esta configuracion solo aplicara para el metodo anotado, si hay atributos igual que en *@Client* entonces seran sobrescritos. En el siguiente ejemplo, el timeout del request sera *1*.
   
```typescript
    @Client({
     url: 'https://jsonplaceholder.typicode.com/posts/',
     timeout:99 // sobrescrito
    })
    export class Post{
        
        @Get()
        @Config({
            timeout: 1
        })
        public findById(
            @PathParam() id: number
        ): HttpObservable<any>
    }
```
   
   - *@Get/@Post/@Put/@Patch/@Delete*   
   recibe por parametro la url que ira junto con la url base y el segundo parametro sera
    la clase en la que se encuentra.
   
```typescript
    @Client('https://jsonplaceholder.typicode.com/posts/')
    export class Post {
    
      constructor(){}
    
      @Get('{id}')
      public findById(
        @PathParam('id') id: number,
      ): HttpObservable<PostModel> {}
    }
```
   - *@PathParam*       
   recibe por parametro el string que contiene el id que coincide con la expresion en la url como se    ve en el ejemplo   anterior. Este parametro es opcional, si no se define entonces se agregara al     final de la url. 

```typescript
    @Client('https://jsonplaceholder.typicode.com/posts/')
    export class User {
    
      constructor(){}
    
      @Get('{id}')
      public findById(
        @PathParam('id') id: number,
      ): HttpObservable<PostModel> {}
    }
    
    es equivalente a 
    
    @Client('https://jsonplaceholder.typicode.com/posts/')
    export class User {
    
      constructor(){}
    
      @Get()
      public findById(
        @PathParam() id: number,
      ): HttpObservable<PostModel> {}
    }
```
   
   
   - *@Query*  
   recibe por parametro un string con el nombre que ira el query en la url.  
   el resultado del siguiente ejemplo seria :
   __/1?attr=holaa__
   
```typescript
    @Get('{id}')
    public findById(
        @PathParam('id') id: number, // 1
        @Query('attr') value: number|string // holaa
    ): HttpObservable<PostModel> {}
```
   si el parametro que se pasa es un objeto, se mapeara cada atributo valor en la url 
   y no se define el string que se pasa por parametro. 
   la salida del siguiente ejemplo seria la siguiente:  
   __/1?attr=holaa&name=lord&id=99__
   
```typescript

    class PostModel{
        constructor(
            public name: string,
            public id: string
        ){}
    }
    
    @Get('{id}')
    public findById(
        @PathParam('id') id: number, // 1
        @Query('attr') value: number|string // holaa
        @Query() value: PostModel // new PostModel('lord',99)
    ): HttpObservable<PostModel> {}
```
    
   - *@Body*  
   Esta anotacion no recibe parametros, el objeto que se reciba por parametro en el metodo 
   sera el body de la peticion. 
   
   Ademas de eso, tambien esta la anotacion *@PathParamProperty* la cual permite 
   definir el parametro que ira en la url atra vez del objecto que ira en el body. 
   La salida del siguiente ejemplo sera la siguiente:   
   __url/88__  
```json
{
  "id": 88,
  "name": "Bodyy"
}
```

```typescript

    class PostModel{
    
        @PathParamProperty()
        public id: string
        
        public name: string
        
        constructor(id, name){
            this.name = name
            this.id = id
        }
    }
    
    @Post('{id}')
    public create(
        @Body object: PostModel // new PostModel(88,"Bodyy")
    ): HttpObservable<PostModel> {}
```

   - *@Mapper*:  
   Es el map de rxjs. Lo que retorne esta funcion, sera lo que retorna el metodo con la anotacion. 
   
```typescript
    export class Class {
        public static map(body): number{
            return body.id as number
        }
    
        @Mapper(Class.map)
        @Post('{id}')
        public create(
            @Body object: PostModel // new PostModel(88,"Bodyy")
        ): HttpObservable<number> {}
    }
```

   - *@Before*:  
   Este metodo permite interceptar el request antes de enviar la peticion y hacer lo que se quiera.
   se llamara antes que los interceptores (ya veremos los interceptores).  
   el metodo que se pasa por parametro recibe un objeto de tipo Request que esta definido de la siguiente manera: 
   
```typescript
    export interface Request_ {
        readonly method: string,
        body: any,
        readonly headers: HeadersHttp,
        readonly url: string
    }
```
```typescript
    export class Class {
        public static before(req: Request_): Request_{
            req.headers.set("otro","header")
            req.body = {
                ...body,
                otro:"atributo en el body"
            }
            return req
        }
    
        @Before(Class.before)
        @Post('{id}')
        public create(
            @Body object: PostModel // new PostModel(88,"Bodyy")
        ): HttpObservable<number> {}
    }
```

   - *@Headers*  
   Permite establecer headers especificos para ese request. recibe un objeto.
```typescript        
        @Headers({
            "ApiKey":"anwb0ted4132"
        })
        @Post('{id}')
        public create(
            @Body object: PostModel // new PostModel(88,"Bodyy")
        ): HttpObservable<number> {}
```

   - *@HandlerError*  
   Este metodo que se pasa por parametro recibe una instancia de tipo *__AxiosError__* y debe retornar la instancia que sera lanzada como excepcion.
   se espera que el usuario tenga establecido un manejador de excepciones global.
   se puede lanzar una excepcion custom heredando de __HttpRequestException__
   si no se define esta anotacion y hay un error entonces se lanzara una excepcion de tipo __HttpRequestException__
   el cual esta definido de la siguiente manera: 

```typescript        
        export class HttpRequestException {
            constructor(
                public error: string,
                public statusCode: number,
                public message: string,
            ) {}
        }
```
   
   
```typescript
    export class Class {
        
        public static error<MyClass extends HttpRequestException>(error: AxiosError): MyClase {
            if(error.response.status === 404){
                return new NotFoundHttp(...)
            }else{
                return new MyClass(...)
            }
        }
    
        @HandlerError(Class.error)
        @Post('{id}')
        public create(
            @Body object: PostModel // new PostModel(88,"Bodyy")
        ): HttpObservable<number> {}
    }
```

   - *addInterceptor*  
   Este es un array el cual admite callbacks que reciben por parametro una clase que implemente la interfaz __HttpInterceptor__. 
   Los interceptors se deben agregar en el main de la aplicacion. 
   Estos interceptores se llaman en todos los request que se realicen y despues de la anotacion *@Before* que este definida en los metodos. Es similar a la anotacion @Before solo que esta forma es global, todos los request pasaran por ahi.

```typescript
    
    import { addInterceptors } from 'rxfeign';


    export class Main implements HttpInterceptor {
        
        public start(): void{
            addInterceptors(Main)
        }
        
        public intercep(req: Request_): Request_{
            console.log(req.url)
            return req
        }
    }
```
