# Desafio Dio Desenvolvendo seu Blog com as Tecnologias de Java e Angular



**Projeto completo e abrangente para desenvolver um blog com Java e Angular:**



**Objetivo:** Criar um blog funcional usando as tecnologias Java e Angular.



#### **Recursos:**

- Java
- Angular
- Banco de dados (por exemplo, MySQL, PostgreSQL)
- Servidor web (por exemplo, Apache Tomcat, Nginx)



### **Passos:**



#### **1. Configurar o ambiente de desenvolvimento:**

- Instale o Java Development Kit (JDK) e o Angular CLI.
- Crie um novo projeto Angular usando o comando `ng new blog-app`.
- Configure um banco de dados e um servidor web.



#### **2. Criar o back-end Java:**

- Crie uma classe de modelo para representar as postagens do blog.
- Crie um repositório para armazenar e recuperar postagens do banco de dados.
- Crie um serviço para gerenciar a lógica de negócios do blog.



#### **3. Criar o front-end Angular:**

- Crie componentes para exibir a lista de postagens, a postagem individual e o formulário de criação de postagem.
- Use serviços Angular para se comunicar com o back-end Java.
- Adicione estilos e layout ao blog usando CSS e HTML.



#### **4. Integrar o back-end e o front-end:**

- Use o Angular HttpClient para fazer chamadas de API para o back-end Java.
- Mapeie os dados recebidos do back-end para os modelos de dados do Angular.
- Exiba os dados do blog no front-end Angular.



#### **5. Implantar o blog:**

- Compile o aplicativo Angular usando o comando `ng build`.
- Implante o aplicativo Angular em um servidor web.
- Configure o servidor web para servir o aplicativo Angular.



### **Códigos de exemplo:**



### **Classe de modelo Java para postagens de blog:**

java



```java
public class Post {
    private Long id;
    private String title;
    private String content;
    private Date createdAt;
    // getters and setters
}
```



### **Serviço Angular para gerenciar postagens de blog:**

typescript



```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class PostService {

  constructor(private http: HttpClient) { }

  getPosts() {
    return this.http.get<Post[]>('api/posts');
  }

  getPost(id: number) {
    return this.http.get<Post>(`api/posts/${id}`);
  }

  createPost(post: Post) {
    return this.http.post<Post>('api/posts', post);
  }

  updatePost(post: Post) {
    return this.http.put<Post>(`api/posts/${post.id}`, post);
  }

  deletePost(id: number) {
    return this.http.delete(`api/posts/${id}`);
  }
}
```



### **Componente Angular para exibir a lista de postagens:**

typescript



```typescript
import { Component, OnInit } from '@angular/core';
import { PostService } from '../post.service';

@Component({
  selector: 'app-post-list',
  templateUrl: './post-list.component.html',
  styleUrls: ['./post-list.component.css']
})
export class PostListComponent implements OnInit {

  posts: Post[];

  constructor(private postService: PostService) { }

  ngOnInit(): void {
    this.postService.getPosts().subscribe(posts => this.posts = posts);
  }
}
```



### **Observação:**

 Este é apenas um exemplo de projeto e pode ser adaptado para atender às suas necessidades específicas.







### **Mais códigos para o projeto de blog Java e Angular:**



### **Módulo Angular para gerenciar o estado do blog:**

typescript



```typescript
import { NgModule } from '@angular/core';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';

import { postReducer } from './state/post.reducer';
import { PostEffects } from './state/post.effects';

@NgModule({
  imports: [
    StoreModule.forFeature('posts', postReducer),
    EffectsModule.forFeature([PostEffects]),
  ],
})
export class BlogModule {}
```



### **Reducer Angular para gerenciar o estado das postagens do blog:**

typescript



```typescript
import { Action, createReducer, on } from '@ngrx/store';
import { PostActions } from './post.actions';
import { Post } from '../models/post';

export interface PostState {
  posts: Post[];
  loading: boolean;
  error: string | null;
}

const initialState: PostState = {
  posts: [],
  loading: false,
  error: null,
};

const postReducer = createReducer(
  initialState,
  on(PostActions.getPosts, state => ({ ...state, loading: true })),
  on(PostActions.getPostsSuccess, (state, { posts }) => ({ ...state, loading: false, posts })),
  on(PostActions.getPostsFailure, (state, { error }) => ({ ...state, loading: false, error })),
  on(PostActions.createPost, state => ({ ...state, loading: true })),
  on(PostActions.createPostSuccess, (state, { post }) => ({ ...state, loading: false, posts: [...state.posts, post] })),
  on(PostActions.createPostFailure, (state, { error }) => ({ ...state, loading: false, error })),
  on(PostActions.updatePost, state => ({ ...state, loading: true })),
  on(PostActions.updatePostSuccess, (state, { post }) => ({ ...state, loading: false, posts: state.posts.map(p => p.id === post.id ? post : p) })),
  on(PostActions.updatePostFailure, (state, { error }) => ({ ...state, loading: false, error })),
  on(PostActions.deletePost, state => ({ ...state, loading: true })),
  on(PostActions.deletePostSuccess, (state, { id }) => ({ ...state, loading: false, posts: state.posts.filter(p => p.id !== id) })),
  on(PostActions.deletePostFailure, (state, { error }) => ({ ...state, loading: false, error })),
);

export function reducer(state: PostState | undefined, action: Action) {
  return postReducer(state, action);
}
```



### **Efeitos Angular para gerenciar ações assíncronas relacionadas a postagens de blog:**

typescript



```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, switchMap } from 'rxjs/operators';
import { of } from 'rxjs';

import { PostService } from '../services/post.service';
import { PostActions } from './post.actions';

@Injectable()
export class PostEffects {

  constructor(
    private actions$: Actions,
    private postService: PostService
  ) {}

  getPosts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(PostActions.getPosts),
      switchMap(() =>
        this.postService.getPosts().pipe(
          map(posts => PostActions.getPostsSuccess({ posts })),
          catchError(error => of(PostActions.getPostsFailure({ error })))
        )
      )
    )
  );

  createPost$ = createEffect(() =>
    this.actions$.pipe(
      ofType(PostActions.createPost),
      switchMap(({ post }) =>
        this.postService.createPost(post).pipe(
          map(post => PostActions.createPostSuccess({ post })),
          catchError(error => of(PostActions.createPostFailure({ error })))
        )
      )
    )
  );

  updatePost$ = createEffect(() =>
    this.actions$.pipe(
      ofType(PostActions.updatePost),
      switchMap(({ post }) =>
        this.postService.updatePost(post).pipe(
          map(post => PostActions.updatePostSuccess({ post })),
          catchError(error => of(PostActions.updatePostFailure({ error })))
        )
      )
    )
  );

  deletePost$ = createEffect(() =>
    this.actions$.pipe(
      ofType(PostActions.deletePost),
      switchMap(({ id }) =>
        this.postService.deletePost(id).pipe(
          map(() => PostActions.deletePostSuccess({ id })),
          catchError(error => of(PostActions.deletePostFailure({ error })))
        )
      )
    )
  );
}
```



Esses códigos fornecem uma implementação mais completa do gerenciamento de estado e das ações assíncronas relacionadas às postagens do blog no aplicativo Angular.
