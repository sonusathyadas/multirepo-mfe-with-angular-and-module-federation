# Build multi-repo micro-frontend using Angular and Webpack 5 Module Federation

Angular is one of the popular Single Page Application framework. It is commonly used to build monolithic frontend applications. It supoorts the lazy loading of modules that allows browser to load the feature modules on demand. Angular internally uses Webpack as the bundling tool. Micro-frontend is a pattern that allows you to build the frontend applications as individual applications(remote) that can be integrated with in a shell(host) application. Module federation plugin of the Webpack allows you to load these micro-frontend applications in to a shell application. 

This tutorial helps you to build Micro-frontends using Angular and Webpack Module Federation. 

## Prerequisites 
    *) Angular CLI : Version 14.0.0 (use exact version)
    *) Module Federation library for Angular (@angular-architects/module-federation): 14.3.0 (compatible to Angular 14.0.0)
    *) Node 16.x or later
    * Visual Studio Code

## Build first remote(Micro-frontend) application (Auth-MFE)
1) Open the command terminal and create an Angular workspace for first micro-frontend application. Run the following command to create a workspace for `auth-app-remote` .
    
    ```bash
    ng new auth-app-remote --create-application false --skip-tests
    ```

2) Switch to the workspace folder and create a project inside the `auth-app-remote`. 
    ```bash
    cd auth-app-remote
    ng g app auth-mfe --skip-tests --routing 
    ```

3) Add a home component to the application. 
    ```bash
    ng g c components/home --project auth-mfe
    ```

4) Create a feature module to the `auth-mfe` project.
    ```bash
    ng g module --project auth-mfe --routing auth
    ```

5) Add a component `login` to the `auth` feature module.
    ```bash
    ng g c --project auth-mfe --module auth auth/components/login
    ```
6) Updates the `auth-routing.module.ts` file with the following route configuation.
    ```typescript
    const routes: Routes = [
        {
            path:'login',
            component:LoginComponent
        }
    ];
    @NgModule({
      imports: [RouterModule.forChild(routes)],
      exports: [RouterModule]
    })
    export class AuthRoutingModule { }
    ```

7) Update the `app-routing.module.ts` file with the following routing configuration.
    ```typescript
    const routes: Routes = [
        {
            path:'',
            component:HomeComponent,
            pathMatch:'full'
        },
        {
            path:'auth',
            loadChildren:()=>import('./auth/auth.module').then(m=>m.AuthModule)
        }
    ];
    
    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

8) Install the `bootstrap` and `jquery` libraries to the project.
    ```bash
    npm install bootstrap@5.2.2 jquery --save
    ```

9) Open the `app.component.html` file and replace the existing code with the following code.
    ```html
    <nav class="navbar navbar-expand-lg bg-primary bg-body-tertiary" data-bs-theme="dark">
        <div class="container-fluid">
          <a class="navbar-brand" href="#">Auth MFE</a>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav">
              <li class="nav-item">
                <a class="nav-link active" aria-current="page" href="#">Home</a>
              </li>
              <li class="nav-item">
                <a class="nav-link" routerLink="auth/login">Login</a>
              </li>
            </ul>
          </div>
        </div>
    </nav>
    <div class="container">
        <router-outlet></router-outlet>
    </div>
    ```
10) Run and test the application.
    ```bash
    ng serve --port 4200 -o
    ```
### Configure the application with module federation
11) Install the Module federation plugin to the application. Install the supported version of the `@angular-architects/module-federation` based on the Angular CLI. In this demo, we are using **Angular 14.0.0**, so the supported version of module federation plugin is `14.3.0`.

    ```bash
    ng add @angular-architects/module-federation@14.3.0 --type remote --project auth-mfe --port 4200
    ```
12) Open the `webpack.config.js` file, and update the `name` and `exposes` properties.
    ```javascript
    const { shareAll, withModuleFederationPlugin } = require('@angular-architects/module-federation/webpack');
    
    module.exports = withModuleFederationPlugin({
    
      name: 'auth-mfe',
    
      exposes: {
        './Module': './projects/auth-mfe/src/app/auth/auth.module.ts',
      },
    
      shared: {
        ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
      },
    
    });

    ```

13) Run and test the application.
    ```bash
    ng serve --port 4200 -o
    ```

## Build second remote(Micro-frontend) application (Book-MFE)

1) Open the command terminal and create an Angular workspace for another micro-frontend application. Run the following command to create a workspace for `book-app-remote` .
    
    ```bash
    ng new book-app-remote --create-application false --skip-tests
    ```

2) Switch to the workspace folder and create a project inside the `book-app-remote`. 
    ```bash
    cd book-app-remote
    ng g app book-mfe --skip-tests --routing 
    ```

3) Add a home component to the application. 
    ```bash
    ng g c components/home --project book-mfe
    ```

4) Create a feature module to the `book-mfe` project.
    ```bash
    ng g module --project book-mfe --routing book
    ```

5) Add a component `book-list` to the `book` feature module.
    ```bash
    ng g c --project book-mfe --module book book/components/book-list
    ```
6) Updates the `book-routing.module.ts` file with the following route configuation.
    ```typescript
    const routes: Routes = [
        {
            path:'list',
            component:BookListComponent
        }
    ];
    @NgModule({
      imports: [RouterModule.forChild(routes)],
      exports: [RouterModule]
    })
    export class BookRoutingModule { }
    ```

7) Update the `app-routing.module.ts` file with the following routing configuration.
    ```typescript
    const routes: Routes = [
        {
            path:'',
            component:HomeComponent,
            pathMatch:'full'
        },
        {
            path:'books',
            loadChildren:()=>import('./book/book.module').then(m=>m.BookModule)
        }
    ];
    
    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

8) Install the `bootstrap` and `jquery` libraries to the project.
    ```bash
    npm install bootstrap@5.2.2 jquery --save
    ```

9) Open the `app.component.html` file and replace the existing code with the following code.
    ```html
    <nav class="navbar navbar-expand-lg bg-primary bg-body-tertiary" data-bs-theme="dark">
        <div class="container-fluid">
          <a class="navbar-brand" href="#">Book MFE</a>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav">
              <li class="nav-item">
                <a class="nav-link active" aria-current="page" href="#">Home</a>
              </li>
              <li class="nav-item">
                <a class="nav-link" routerLink="books/list">Books</a>
              </li>
            </ul>
          </div>
        </div>
    </nav>
    <div class="container">
        <router-outlet></router-outlet>
    </div>
    ```
10) Run and test the application.
    ```bash
    ng serve --port 4200 -o
    ```
### Configure the application with module federation
11) Install the Module federation plugin to the application. 

    ```bash
    ng add @angular-architects/module-federation@14.3.0 --type remote --project book-mfe --port 4300
    ```
12) Open the `webpack.config.js` file, and update the `name` and `exposes` properties.
    ```javascript
    const { shareAll, withModuleFederationPlugin } = require('@angular-architects/module-federation/webpack');
    
    module.exports = withModuleFederationPlugin({
    
      name: 'book-mfe',
    
      exposes: {
        './Module': './projects/book-mfe/src/app/book/book.module.ts',
      },
    
      shared: {
        ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
      },
    
    });

    ```

13) Run and test the application.
    ```bash
    ng serve --port 4300 -o
    ```

## Create a Shell(Host) Application (BookStore-Host)
1) Open the command terminal and create an Angular workspace for a shell application. Run the following command to create a workspace for `bookstore-host` .
    
    ```bash
    ng new bookstore-host --create-application false --skip-tests
    ```

2) Switch to the workspace folder and create a project inside the `bookstore-host`. 
    ```bash
    cd bookstore-host
    ng g app bookstore-shell --skip-tests --routing 
    ```

3) Add a home component to the application. 
    ```bash
    ng g c components/home --project bookstore-shell
    ```

4) Update the `app-routing.module.ts` file with the following routing configuration.
    ```typescript
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes } from '@angular/router';
    import { HomeComponent } from './components/home/home.component';
    
    const routes: Routes = [
        {
            path:'',
            component:HomeComponent,
            pathMatch:'full'
        }
    ];
    
    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

5) Install the `bootstrap` and `jquery` libraries to the project.
    ```bash
    npm install bootstrap@5.2.2 jquery --save
    ```

6) Open the `app.component.html` file and replace the existing code with the following code.
    ```html
    <nav class="navbar navbar-expand-lg bg-primary bg-body-tertiary" data-bs-theme="dark">
        <div class="container-fluid">
            <a class="navbar-brand" href="#">Bookstore</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
                aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link active" aria-current="page" routerLink="">Home</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    <div class="container">
        <router-outlet></router-outlet>
    </div>
    ```
7) Run and test the application.
    ```bash
    ng serve --port 5000 -o
    ```
### Configure the shell application with module federation
8) Install the Module federation plugin to the application. 

    ```bash
    ng add @angular-architects/module-federation@14.3.0 --type host --project bookstore-shell --port 5000
    ```
9) Open the `webpack.config.js` file, and update the `remote` property.
    ```javascript
    const { shareAll, withModuleFederationPlugin } = require('@angular-architects/module-federation/webpack');

    module.exports = withModuleFederationPlugin({
    
      remotes: {
        "auth-mfe": "http://localhost:4200/remoteEntry.js",    
        "book-mfe": "http://localhost:4300/remoteEntry.js"    
      },
    
      shared: {
        ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
      },
    
    });
    ```
10) Open the `app-routing.module.ts` file and update the routes for loading the `auth` and `book` micro-frontends.
    ```typescript
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes } from '@angular/router';
    import { HomeComponent } from './components/home/home.component';
    
    const routes: Routes = [
        {
            path:'',
            component:HomeComponent,
            pathMatch:'full'
        },
        {
            path:'auth',
            loadChildren:()=>import('auth-mfe/Module').then(m=>m.AuthModule)
        },
        {
            path:'books',
            loadChildren:()=>import('book-mfe/Module').then(m=>m.BookModule)
        }
    ];
    
    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

11) Update the `app.component.html` to add the hyperlinks for invoking `book list` and `login` components from the micro-frontends.
    ```html
    <nav class="navbar navbar-expand-lg bg-primary bg-body-tertiary" data-bs-theme="dark">
        <div class="container-fluid">
            <a class="navbar-brand" href="#">Bookstore</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
                aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link active" aria-current="page" routerLink="">Home</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" routerLink="books/list">Books</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" routerLink="auth/login">Login</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    <div class="container">
        <router-outlet></router-outlet>
    </div>
    ```
12) Run and test the application.
    ```bash
    ng serve --port 5000 -o
    ```
