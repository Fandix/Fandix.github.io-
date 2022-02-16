---
title: Day33. Communicating with backend services using HTTP
date: 2021-10-03 10:35:20
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在現代的網頁中絕大部分會需要與 server 互相溝通，無論是從 server 獲取商品的資料用於顯示在畫面中，還是畫面中的設定要儲存回 server 的設定都需與後端 server 互相溝通，Angular 提供了一個 client 端的 HTTP API，那便是 `@angular/common/http` 中的 `HttpClient` service class，該怎麼使用就繼續往下看吧!

![https://ithelp.ithome.com.tw/upload/images/20210905/201247675b7AjP33Yf.jpg](https://ithelp.ithome.com.tw/upload/images/20210905/201247675b7AjP33Yf.jpg)

<!-- more -->

# An overview of HTTP

HTTP 是一種允許`獲得數據`（例如 HTML 文檔）的協議，他是一種 client 端與 server 端的協議是數據交換的基礎，這意味著通常 request 是由接收方（ client 端）發起的，從而獲得不同的數據或檔案，從 client 發送的消息稱為 `request` 而 server 端回傳的數據稱為 `response`。

![https://ithelp.ithome.com.tw/upload/images/20210905/20124767FRlBCLRtNR.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767FRlBCLRtNR.png)

## HTTP request methods

HTTP 定義了一組 request method 用於 client 端對 server 端的所有操作，他們分別是

- **GET**：GET request 用於`請求數據`
- **HEAD**：和 GET request 一樣用於請求數據，但 response 沒有內容
- **POST**：用於向指定 server 提交數據，會導致狀態更改或對 server 產稱副作用（更改數據）
- **PUT**：用於將有效的數據替換掉目標資源的當前值
- **DELETE**：用於刪除指定資源
- **CONNECT**：用於建立到目標資源的 server 通道
- **OPTION**：用於描述目標資源的通選項


# Setup for server communication

大概介紹完 HTTP 與 HTTP method 後，接著回到 Angular 的內容，要在 Angular 中使用 HttpClient 之前需要先導入 `HttpClientModule`，大多數操作都會將他到入到 AppModule 中。

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule,
  ],
  declarations: [
    AppComponent,
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule {}
```

接著可以在需要使用到 httpClient 的地方將它注入到 component 中

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class ConfigService {
  constructor(private http: HttpClient) { }
}
```


# Requesting data from a server

首先介紹的是使用 HttpClient.get() method 從 server 中獲取數據，這個同步的 method 會發送一個 HTTP request，並在收到 responses 後會返回一個 `Observable`，responses 的類型會依據 request 中的 `observe` 和 `responseType` 決定，他接收兩個參數，一個是 server 的路徑 url 第二個是用於設置 request 的 option object。

```typescript
options: {
  headers?: HttpHeaders | {[header: string]: string | string[]},
  observe?: 'body' | 'events' | 'response',
  params?: HttpParams|{[param: string]: string | number | boolean | ReadonlyArray<string | number | boolean>},
  reportProgress?: boolean,
  responseType?: 'arraybuffer'|'blob'|'json'|'text',
  withCredentials?: boolean,
}
```

其中重要的是 `observe` 與 `responseType`，`observe` 會決定要返回多少 response，`responseType` 會指定返回 response 的格式。

所以如果要接收一個 JSON 形式的數據的話，需要將 `get()` 的 option 設置為 `{ observe: 'body', responseType: 'json' }` ，不過這些選項是預設值就算不加也會傳遞一樣的設置，下面來舉個例子，目標是獲得這一組數據

```json
{
  "heroesUrl": "api/heroes",
  "textfile": "assets/textfile.txt",
  "date": "2021-09-03"
}
```

1. 在 app.component.ts 中注入 HttpClient service

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { HttpClient } from '@angular/common/http';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      heros!: any;
      constructor(private http: HttpClient) {}

      ngOnInit() {
        
      }
    }
    ```

2. 接著在 app.component.ts 的 `ngOnInit()` 中使用 get 獲取數據，這邊我是有建一個 server，如果不想建的話也可以跟官方文檔一樣把假數據放在專案的別的位置

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { HttpClient } from '@angular/common/http';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      heros!: any;
      constructor(private http: HttpClient) {}

      ngOnInit() {
        this.http.get('http://localhost:3000/heros').subscribe((res: any) => {
          this.heros = res[0];
        });
      }
    }
    ```

3. 接著在 template 中把獲得的數據顯示在上面

    ```html
    <div class="content">
      <h4>HeroUrl: {{heros.herosUrl}}</h4>
      <h4>textFile: {{heros.textFile}}</h4>
      <h4>date: {{heros.date}}</h4>
    </div>
    ```
    
![https://ithelp.ithome.com.tw/upload/images/20210905/20124767SQIcmtqBlD.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767SQIcmtqBlD.png)

## Requesting a typed response

可以構造 HttpClient request 來聲明獲得的 reponse 數據類型，可以更好的對獲得的數據進行處理，收先先定義 responses 的資料型態

```typescript
export interface Config {
  heroesUrl: string;
  textfile: string;
  date: any;
}
```

將這個資料型態定義給 HttpClient.get() 

```typescript
ngOnInit() {
  this.http.get<Config[]>('http://localhost:3000/heros').subscribe((res) => {
    this.heros = res[0];
  });
}
```

## Reading the full response

在前面的範例中沒有設定 HttpClient.get() 的 option object，因為他預設就會獲得 `JSON` 格式的資料，不過有時候 server 會返回特殊的 header 或狀態用於指定某些特殊的條件，可以使用 `observe` 設定需要獲取完整的數據

```typescript
 ngOnInit() {
  this.http
    .get<Config[]>('http://localhost:3000/heros', { observe: 'response' })
    .subscribe((res) => {
      console.log(res);
    });
}
```

如果要獲取完整數據的話，就會連 header, status.... 都會獲得

![https://ithelp.ithome.com.tw/upload/images/20210905/20124767Bqun7G1JtE.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767Bqun7G1JtE.png)

## Requesting non-JSON data

除了向上面一樣 get 一組 `JSON` 型態的檔案之外，還可以隨著後端 server 回傳的數據類型不同 get 的 option 進行操作，比如今天後端傳遞的是一個 `string` 類型的數據，如果不去設定 request 的 option 的話就會出錯，因為預設你是會獲得 `JSON` 格式的數據但實際上拿到的卻是 `String`，這時就需要更改 option

```typescript
this.http
  .get('http://localhost:3000/heroName', { responseType: 'text' })
  .subscribe((res) => console.log(res));
```

將 `responseType` 更改為 `text` 代表獲得的 responses 會是 string 形式。


# Handling request errors

向 server 發送 request 的行為一定不會每一次都成功，所以當 request 失敗的話 HttpClient 將會返回一個 `error object` 而不是成功的 response，所以在我們撰寫處理 response 的 method 時也應該要同時加入`處理錯誤`的動作，所以當發生錯誤時可以獲得`失敗的原因`，這一方面可以讓我們開發者知道哪裡出了問題，也可以通知使用者發生了錯誤而不是空白畫面給他，甚至某些情況下還得再次發送 request 等等的錯誤處理。

## Getting error details

當 request 數據失敗的話，應用程式應該向使用者提供有用的反饋告入使用這發生了什麼事，是不是操作不當或是系統有問題等等，通常必較常會發生兩種錯誤：

- Server 後端可能會拒絕 client 的 request，這時會返回帶有`狀態` 的代碼（ 404, 500... ）的 HTTP response，這就是 error response
- Client 端可能會出錯，比如說阻止 request 完成的網路錯誤或 RxJS 拋出的錯誤，這屬於前端的錯誤，這類型的錯誤狀態會是 `0`，error obejct 的 property 會包含一個 `ProgressEvent`，他可以提供更多的錯誤信息。

HttpClient 在其 HttpErrorResponse 中會捕獲這兩種錯誤，可以檢查錯誤的原因，所以將剛剛的 get() 加上錯誤處理

```typescript
this.http
  .get('http://localhost:3000/heros', { observe: 'response' })
  .pipe(
    catchError(this.handleError)
).subscribe((res: any) => this.heros = res.body[0])
```

```typescript
handleError(error: HttpErrorResponse) {
  if (error.status === 0) {
    console.error('An error occurred', error.error);
  } else {
    console.error(`Back-end returned code ${error.status}, body was: ${error.error}`);
  }
  return throwError ('Something bad happened; Please try again later.')
}
```

## Retrying a failed request

有時候遇到 response 錯誤只是暫時的，可以再試一次說不定就會恢復正常，例如網路中斷就會導致 request 失敗，但是一但網路回歸正常後便可以再次 request，這時就可以使用 RxJS 提供的重試操作符 `retry()` 將會自動重新 request，次數可以隨意設定

```typescript
this.http
  .get('http://localhost:3000/heros', { observe: 'response' })
  .pipe(
    catchError(this.handleError),
    retry(3)
).subscribe((res: any) => this.heros = res.body[0])
```


# Sending data to a server

除了從 server 獲得數據之外，HttpClinet 還支持其他 HTTP method，比如 PUT, POST 和 DELETE，可以使用它們來修改 server 的數據，下面將會做一個 Todo 的範例，UI 會填入代辦事項並將待辦事項寫入 server 中，也可以對他進行更改與刪除。

## Post todo data to server

第一步要建立一個 Todo 的畫面並將在畫面中輸入的內容利用 `HttpClient.post` 傳遞給 server

1. 首先利用 Angular CLI 創建一個 component

    ```bash
    ng generate component todo-form
    ```

2. 在 todo-form.component.ts 中建立 todo 的 form model（記得要在 AppModule 中添加 `ReactiveFormsModule` 喔）

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';

    @Component({
      selector: 'app-todo-form',
      templateUrl: './todo-form.component.html',
      styleUrls: ['./todo-form.component.css']
    })
    export class TodoFormComponent implements OnInit {
      myForm = this.fb.group({
        title: ['', Validators.required],
        description: ['', Validators.required]
      })
      
      constructor(private fb: FormBuilder) { }

      ngOnInit(): void {}
    }
    ```

3. 接著在 todo-form.component.html 中添加輸入元件並綁定 FormControl

    ```html
    <form [formGroup]="myForm" (submit)="onSubmit()">
      <div class="input-area">
        <label for="title">Title </label>
        <input
          id="name"
          type="text"
          class="form-control"
          formControlName="title"
        />
      </div>
      <div class="input-area">
        <label for="description">Description </label>
        <textarea
          id="name"
          type="text"
          class="form-control"
          formControlName="description"
        ></textarea>
      </div>

      <button class="btn btn-success" type="submit">Add Todo</button>
    </form>
    ```
    
![https://ithelp.ithome.com.tw/upload/images/20210905/20124767NnxhXx7B2g.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767NnxhXx7B2g.png)

1. 在 todo-form.component.ts 中注入 HttpClient 並新增 post method

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';
    import { HttpClient, HttpHeaders } from '@angular/common/http';

    @Component({
      selector: 'app-todo-form',
      templateUrl: './todo-form.component.html',
      styleUrls: ['./todo-form.component.css']
    })
    export class TodoFormComponent implements OnInit {
      myForm = this.fb.group({
        title: ['', Validators.required],
        description: ['', Validators.required]
      })
      id = 0;
      httpOptions = {
        headers: new HttpHeaders({
          'Access-Control-Allow-Origin': '*',
          'Content-Type': 'application/json'
        }),
      };

      constructor(private fb: FormBuilder, private http: HttpClient) { }

      ngOnInit(): void {}

      onSubmit() {
        const body = {
          id: this.id,
          title: this.myForm.value.title,
          description: this.myForm.value.description
        }

        this.http
          .post('http://localhost:3000/todo', body, this.httpOptions)
          .subscribe((todo: any) => this.todoList = todo);

        this.id++;
      }
    }
    ```

    在 onSubmit() 中添加 `HttpClient.post` method，將 Todo 中的數據傳遞給 Server，這邊新增了一個 property `id`，用於後面要指定更改或刪除哪一個 todo item，現在點擊畫面中的 Add todo 不會有任何動作，不過可以打開瀏覽器的 Network 確實可以看到有一個 POST 的 method。

![https://ithelp.ithome.com.tw/upload/images/20210905/20124767AVDHOaTtMH.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767AVDHOaTtMH.png)

## Get todo list from server

要確認是否確實有將數據傳送給 server，可以使用上面介紹的 `HttpClient.get` method 取得 todo 的資料

1. 在 todo-form.component.ts 中的 OnInit() 中使用 HttpClient.get 獲得數據

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';
    import { HttpClient, HttpHeaders } from '@angular/common/http';

    @Component({
      selector: 'app-todo-form',
      templateUrl: './todo-form.component.html',
      styleUrls: ['./todo-form.component.css']
    })
    export class TodoFormComponent implements OnInit {
      todoList: any[] = [];                                    // (1)
     
      constructor(private fb: FormBuilder, private http: HttpClient) { }

      ngOnInit(): void {
        this.http.get('http://localhost:3000/todo').subscribe((todo: any) => {
          this.todoList = todo;                                // (2)
        })
      }

    	// ...
    }
    ```

    - (1): 新增一個 property 用於存放從 server 獲得的數據
    - (2): 在 ngOnInit 中使用 HttpClient.get 獲得 server 數據並存放在 todoList 中
2. 使用 Angular CLI 創建一個 component 用於顯示獲得的 todo 數據

    ```bash
    ng generate component todo-list
    ```

3. 在 todo-list.component.ts 中使用 `@Input()` 裝飾 property 代表從父層傳遞的數據

    ```typescript
    import { Component, Input } from '@angular/core';

    @Component({
      selector: 'app-todo-list',
      templateUrl: './todo-list.component.html',
      styleUrls: ['./todo-list.component.css']
    })
    export class TodoListComponent {
      @Input() todoList: any;
      constructor() { }
    }
    ```

4. 在 todo-list.component.html 中把獲得的 todo 數據顯示在畫面中

    ```html
    <div *ngFor="let todo of todoList" class="todo-content">
        <div class="todo-item">
            <div class="title">Title: {{ todo.title }}</div>
            <div class="desc">Description: {{ todo.description }}</div>
        </div>
        <div class="optionBtn">
            <button type="button" class="btn btn-success">E</button>
            <button type="button" class="btn btn-danger">X</button>
        </div>
    </div>
    ```

    這邊先新增了兩個 `<button>` 用於對單一 todo item 進行刪除或更改

5. 在 todo-form.component.html 中使用 todo-list 的 select 並使用 property binding 綁定 todo 數據

    ```html
    <div class="todo-list">
      <app-todo-list [todoList]="todoList"></app-todo-list>
    </div>
    ```
    
![img](https://i.imgur.com/EgXMfKL.gif)

## Delete todo item

在畫面中可以順利看到傳遞給 server 的數據後，接著要對這些數據進行刪除

1. 將 todo-list.component.ts 中新增一個 `@Output` 用於將點擊刪除事件傳遞給父層，這邊我的設計是將所有 http 動作都放在同一個 component 中

    ```typescript
    import { Component, Input, Output, EventEmitter } from '@angular/core';

    @Component({
      selector: 'app-todo-list',
      templateUrl: './todo-list.component.html',
      styleUrls: ['./todo-list.component.css']
    })
    export class TodoListComponent {
      @Input() todoList: any;
      @Output() deleteEvent = new EventEmitter<number>();
      constructor() { }

      onDelete(todoItem: any) {
        this.deleteEvent.emit(todoItem.id);
      }
    }
    ```

    這邊向父層傳遞被選中的 todo item 的 id，用於刪除指定 id 的數據

2. 在 todo-form.component.ts 中新增 onDelete() method 用於調用 `HttpClient.delete` method

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';
    import { HttpClient, HttpHeaders } from '@angular/common/http';

    @Component({
      selector: 'app-todo-form',
      templateUrl: './todo-form.component.html',
      styleUrls: ['./todo-form.component.css']
    })
    export class TodoFormComponent implements OnInit {
      httpOptions = {
        headers: new HttpHeaders({
          'Access-Control-Allow-Origin': '*',
          'Content-Type': 'application/json'
        }),
      };

      constructor(private fb: FormBuilder, private http: HttpClient) { }
    	// ...
    	
      onDelete(id: number) {
        this.http
          .delete(`http://localhost:3000/todo/${id}`, this.httpOptions)
          .subscribe((todo: any) => this.todoList = todo);
      }

    	// ...
     }
    ```

    將被選中的 todo item 的 id 做為參數加在 url 上，讓 server 端可以獲得指定的 Id 並對刪除指定的 todo item。

3. 在 todo-form.component.html 中綁定 `@Output()` 事件

    ```html
    <div class="todo-list">
      <app-todo-list 
    		[todoList]="todoList" 
    		(deleteEvent)="onDelete($event)"
    	></app-todo-list>
    </div>
    ```
    
![img](https://i.imgur.com/K8lTVkw.gif)

## Put todo data to update server

完成刪除功能後，接著要利用 `HttpClient.put` method 更改 server 中的 todo 數據，不過要做到這一點首先要先建立一個畫面，用來顯示目前的 todo 內容，對他的內容更改後才能送出 PUT 更新 server 中的數據

1. 利用 Angular CLI 創建一個 component 用於顯示被選中的 todo item 內容

    ```bash
    ng generate component edit-panel
    ```

2. 在 edit-panel.component.ts 中建立 Todo 的 form model

    ```typescript
    import { Component, OnInit, Input } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';

    @Component({
      selector: 'app-edit-panel',
      templateUrl: './edit-panel.component.html',
      styleUrls: ['./edit-panel.component.css']
    })
    export class EditPanelComponent implements OnInit {
      myForm = this.fb.group({
        id: [''],
        title: ['', Validators.required],
        description: ['', Validators.required]
      })
      constructor(private fb: FormBuilder) { }

      ngOnInit(): void {}
    }
    ```

3. 在 todo-list.component.ts 中建立 onEdit() method，他和 onDelete() 一樣傳遞被選中的 Todo id 給父層

    ```typescript
    import { Component, Input, Output, EventEmitter } from '@angular/core';

    @Component({
      selector: 'app-todo-list',
      templateUrl: './todo-list.component.html',
      styleUrls: ['./todo-list.component.css']
    })
    export class TodoListComponent {
      @Input() todoList: any;
      @Output() openPanelEvent = new EventEmitter<number>();
      constructor() { }
    	//... 
    	
      onEdit(todoItem: any) {
        this.openPanelEvent.emit(todoItem.id);
      }
    }
    ```

4. 在 todo-form.component.ts 中建立 onOpenEditPanel() method，這邊使用 `HttpClient.get` 獲取被選中的 todo item 數據，這邊多加入一個 property 用於決定是否開啟 edit-panel 畫面

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';
    import { HttpClient, HttpHeaders } from '@angular/common/http';

    @Component({
      selector: 'app-todo-form',
      templateUrl: './todo-form.component.html',
      styleUrls: ['./todo-form.component.css']
    })
    export class TodoFormComponent implements OnInit {
      todoItem: any;                                          // (1)
      isEditPanelOpen = false;                                // (2)
      constructor(private fb: FormBuilder, private http: HttpClient) { }
    	// ...
      
      onOpenEditPanel(id: number) {                           // (3)
        this.http.get(`http://localhost:3000/todo/${id}`)
        .subscribe((todo: any) => {
          this.todoItem = todo;
          this.isEditPanelOpen = true;
        });
      }
    }
    ```

    - (1): 新增一個 property 用於獲得指定的 todo item 數據
    - (2): 新增一個 property 用於決定是否開啟 edit-panel 畫面
    - (3): 新增一個 method 用於調用 `HttpClient.get`，這邊和 delete 一樣將 id 傳遞給 url 做為參數，server 會利用 url 的參數回傳指定 id 的數據

    這邊記得要在 todo-form.component.html 中將 Output event 綁定

    ```html
    <div class="todo-list">
      <app-todo-list 
    		[todoList]="todoList" 
    		(deleteEvent)="onDelete($event)" 
    		(openPanelEvent)="onOpenEditPanel($event)"
    	></app-todo-list>
    </div>
    ```

5. 將獲得的 todo 數據透過 property binding 綁定給 edit-panel 

    ```html
    <div *ngIf="isEditPanelOpen && todoItem" class="edit-panel">
        <app-edit-panel [todoItem]="todoItem"></app-edit-panel>
    </div>
    ```

    這邊加上 `*ngIf` 只有在 isEditPanelOpen 為 true 和 todoItem 有數據時才會顯示畫面

6. 由於需要將被選中的 todo 數據顯示在畫面中，所以要在 edit-panel.component.ts 的 ngOnInit() 中使用 `setValue` 設定表單的預設值

    ```typescript
    import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';

    @Component({
      selector: 'app-edit-panel',
      templateUrl: './edit-panel.component.html',
      styleUrls: ['./edit-panel.component.css']
    })
    export class EditPanelComponent implements OnInit {
      @Input() todoItem: any;
      constructor(private fb: FormBuilder) { }

      ngOnInit(): void {
        this.myForm.setValue(this.todoItem);
      }
    }
    ```
    
![img](https://i.imgur.com/cAovjJr.gif)

1. 和 delete 一樣在 edit-panel.component.ts 中利用 `Output()`  將更改後的數據傳遞給父層

    ```typescript
    import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';

    @Component({
      selector: 'app-edit-panel',
      templateUrl: './edit-panel.component.html',
      styleUrls: ['./edit-panel.component.css']
    })
    export class EditPanelComponent implements OnInit {
      @Output() editEvent = new EventEmitter<any>();
      
      constructor(private fb: FormBuilder) { }

    	// ...
      onSubmit() {
        this.editEvent.emit(this.myForm.value);
      }  
    }
    ```

2. 在 todo-form.component.ts 中新增一個 method 用於調用 `HttpClient.put` 將更新的數據傳遞給 server 更新數據

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { FormBuilder, Validators } from '@angular/forms';
    import { HttpClient, HttpHeaders } from '@angular/common/http';

    @Component({
      selector: 'app-todo-form',
      templateUrl: './todo-form.component.html',
      styleUrls: ['./todo-form.component.css']
    })
    export class TodoFormComponent implements OnInit {
      isEditPanelOpen = false;

      constructor(private fb: FormBuilder, private http: HttpClient) { }

    	// ...
      onEdit(todoItem: any) {
        this.http.put(`http://localhost:3000/todo/${todoItem.id}`, todoItem, this.httpOptions)
        .subscribe((todo: any) => {
          this.todoList = todo;
          this.isEditPanelOpen = false;
        });
      }
    }
    ```

    這邊和 delete 一樣將 id 傳遞給 url 做為參數，server 會利用 url 的參數更改指定的數據內容
    
![img](https://i.imgur.com/qOsOT8n.gif)


# 結論

本章介紹了基本的 HTTP 概念與 HTTP method 這是 client 與 server 溝通的方法，可以使用 get 從 server 端獲得數據、可以使用 post 將 client 端的數據傳給 server、可以使用 delete 刪除 server 的數據也可以使用 Put 更改 server 的數據。

最後介紹了如何利用 Angular 提供的 HttpClient 達成上面提到的所有功能，由於上面提到的範例與官方文檔的不一樣，我是自己寫 back-end server，所以有些地方可能會比較複雜，如果有看不懂的問題可以在下面留言問我喔！

Angular 的 HTTP 因為篇章也比較長，所以也會拆成兩部分講解，所以明天還會介紹 Angular Http 的一些其他功能與細節，那就明天見吧！


# Reference

- [Angular.io - Communicating with backend services using HTTP](https://angular.io/guide/http#adding-and-updating-headers)
- [An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)