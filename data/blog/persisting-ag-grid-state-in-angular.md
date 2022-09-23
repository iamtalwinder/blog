---
title: 'Persisting AG grid state in Angular'
date: 2023-09-27T15:32:14Z
lastmod: '2023-09-27'
tags: ['angular', 'ag-grid', 'typescript']
draft: false
summary: 'How to save and load ag grid filter, column and pagination state in angular using localstorage.'
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/persisting-ag-grid-state-in-angular
---

[See complete code on github](https://github.com/iamtalwinder/angular-ag-grid-state-persistence-blog)

## Overview


<TOCInline toc={props.toc} exclude="Overview" toHeading={2} />

## Setting up the project

To get started, run the following commands to setup new project:

```bash
ng new angular-ag-grid-state-persistence-blog --style scss --routing true
cd angular-ag-grid-state-persistence-blog
npm start
```

## Setup ag-grid

Install ag-grid packages:

```bash
npm install --save ag-grid-community
npm install --save ag-grid-angular
```

Copy the following code to `src/app/app.component.html`:

```html:src/app/app.component.html
<ag-grid-angular
   style="width: 100%; height: 100%"
   class="ag-theme-alpine"
   [columnDefs]="columnDefs"
   [defaultColDef]="defaultColDef"
   [rowData]="rowData$ | async"
   (gridReady)="onGridReady($event)"
   [paginationAutoPageSize]="true"
   [pagination]="true"
 ></ag-grid-angular>   
```

Copy the following code to `src/app/app.component.ts`:

```ts:src/app/app.component.ts
import { HttpClient } from '@angular/common/http';
import { Component } from '@angular/core';
import { ColDef, GridReadyEvent } from 'ag-grid-community';
import { Observable } from 'rxjs';

@Component({
 selector: 'app-root',
 templateUrl: './app.component.html',
 styleUrls: ['./app.component.scss']
})
export class AppComponent {

  public columnDefs: ColDef[] = [
    { field: 'athlete' },
    { field: 'age' },
    { field: 'country' },
    { field: 'year' },
    { field: 'date' },
    { field: 'sport' },
    { field: 'gold' },
    { field: 'silver' },
    { field: 'bronze' },
    { field: 'total' },
  ];

 public defaultColDef: ColDef = {
  sortable: true,
  filter: true,
  resizable: true,
  enableValue: true,
  enablePivot: true,
  enableRowGroup: true
 };
 
 public rowData$!: Observable<any[]>;
 public gridApi!: GridApi;
 public gridColumnApi!: ColumnApi;

 constructor(private http: HttpClient) {}

 onGridReady(params: GridReadyEvent) {
  this.gridApi = params.api;
  this.gridColumnApi = params.columnApi;

  this.rowData$ = this.http
    .get<any[]>('https://www.ag-grid.com/example-assets/olympic-winners.json');
 }
}
```

Copy the following code to `src/app/app.module.ts`:

```ts:src/app/app.module.ts
import { HttpClientModule } from '@angular/common/http';
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AgGridModule } from 'ag-grid-angular';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule,
    AgGridModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

Copy the following code to `src/styles.scss`:

```scss:src/style.scss
@import 'ag-grid-community/styles/ag-grid.css';
@import 'ag-grid-community/styles/ag-theme-alpine.css';

html, body {
   height: 100%;
   width: 100%;
   padding: 5px;
   box-sizing: border-box;
   -webkit-overflow-scrolling: touch;
}
```

The result should look like this:

![Setup result](/static/images/blog/persisting-ag-grid-state-in-angular/1-setup-result.png "AG grid setup result").


## The approach 

AG grid fire events on grid state changes. Some of the events are listed below:

- `onFilterChanged`
- `onSortChanged`
- `onColumnVisible`
- `onColumnPinned`
- `onColumnResized`
- `onColumnMoved`
- `onColumnRowGroupChanged`
- `onColumnValueChanged`
- `onColumnPivotChanged`
- `onColumnPivotModeChanged`

We will save the grid state in local storage in each of the events and load the state on `onFirstDataRendered` event.

## Saving the state


### Add the local storage service

Add a localstorage service to save and retrieve data.

Copy the following code to `src/app/sevices/local-storage.service.ts`:
```ts:app/sevices/local-storage.service.ts
import { Injectable } from "@angular/core";

@Injectable()
export class LocalStorageService {
  private readonly localStorage: Storage;

  constructor() {
    this.localStorage = window.localStorage; 
  }

  setItem(key: string, value: any): void {
    this.localStorage.setItem(key, JSON.stringify(value));
  }

  getItem(key: string): any {
    const value: any | undefined = this.localStorage.getItem(key);
    
    if (value) {
      return JSON.parse(value);
    }

    return value;
  }
}
```

Copy the following code to `src/app/sevices/index.ts`:

```ts:app/sevices/index.ts
export * from './local-storage.service';
```

Provide the service in the app module:

```ts:src/app/app.module.ts
...
import { LocalStorageService } from './services';

@NgModule({
  ...
  providers: [LocalStorageService],
  ...
})
...
```

### Add persistence keys

```ts:src/app/constants.ts
export const AG_GRID_FILTER_PERSISTENCE_KEY: string = 'ag-grid-filter-persistence-key';
export const AG_GRID_COLUMN_PERSISTENCE_KEY: string = 'ag-grid-column-persistence-key';
export const AG_GRID_GROUP_PERSISTENCE_KEY: string = 'ag-grid-group-persistence-key';
export const AG_GRID_PAGINATION_PERSISTENCE_KEY: string = 'ag-pagination-filter-persistence-key';
export const AG_GRID_PIVOT_PERSISTENCE_KEY: string = 'ag-grid-pivot-persistence-key';
```
### Add event listeners

The below code will start listening to state change events and store all the relevant state in localstorage.

```html:src/app/app.component.html
<ag-grid-angular
  style="width: 100%; height: 100%"
  class="ag-theme-alpine"
  [columnDefs]="columnDefs"
  [defaultColDef]="defaultColDef"
  [rowData]="rowData$ | async"
  [paginationAutoPageSize]="true"
  [pagination]="true"

  (gridReady)="onGridReady($event)"

  (filterChanged)="onFilterChanged()"
  (sortChanged)="onSaveGridColumnState()"
  (columnVisible)="onSaveGridColumnState()"
  (columnPinned)="onSaveGridColumnState()"
  (columnResized)="onSaveGridColumnState()"
  (columnMoved)="onSaveGridColumnState()"
  (columnRowGroupChanged)="onSaveGridColumnState()"
  (columnValueChanged)="onSaveGridColumnState()"
  (columnPivotChanged)="onSaveGridColumnState()"
  (columnPivotModeChanged)="onSavePivotModeState()"
  (paginationChanged)="onPaginationChanged()"
 ></ag-grid-angular>   
```

```ts:src/app/app.component.ts
import { HttpClient } from '@angular/common/http';
import { Component } from '@angular/core';
import { ColDef, ColumnApi, GridApi, GridReadyEvent } from 'ag-grid-community';
import { Observable } from 'rxjs';
import { 
  AG_GRID_COLUMN_PERSISTENCE_KEY, 
  AG_GRID_FILTER_PERSISTENCE_KEY, 
  AG_GRID_GROUP_PERSISTENCE_KEY, 
  AG_GRID_PAGINATION_PERSISTENCE_KEY, 
  AG_GRID_PIVOT_PERSISTENCE_KEY,
} from './constants';
import { LocalStorageService } from './services';

@Component({
 selector: 'app-root',
 templateUrl: './app.component.html',
 styleUrls: ['./app.component.scss']
})
export class AppComponent {

  public columnDefs: ColDef[] = [
    { field: 'athlete' },
    { field: 'age' },
    { field: 'country' },
    { field: 'year' },
    { field: 'date' },
    { field: 'sport' },
    { field: 'gold' },
    { field: 'silver' },
    { field: 'bronze' },
    { field: 'total' },
 ];

 public defaultColDef: ColDef = {
  sortable: true,
  filter: true,
  resizable: true,
  enableValue: true,
  enablePivot: true,
  enableRowGroup: true
 };
 
 public rowData$!: Observable<any[]>;
 public gridApi!: GridApi;
 public gridColumnApi!: ColumnApi;

/**Prevent writing into state, before the localstorage state has been restored */
 private isGridStateRestored: boolean = false;

 constructor(
  private http: HttpClient,
  private localStorageService: LocalStorageService,
) {}

 onGridReady(params: GridReadyEvent) {
  this.gridApi = params.api;
  this.gridColumnApi = params.columnApi;

  this.rowData$ = this.http
    .get<any[]>('https://www.ag-grid.com/example-assets/olympic-winners.json');
 }


 onFilterChanged() {

  if (!this.isGridStateRestored) {
    return;
  }

  const filterState: any = this.gridApi.getFilterModel();

  this.localStorageService.setItem(AG_GRID_FILTER_PERSISTENCE_KEY, filterState);
 }

 onSaveGridColumnState() {

  if (!this.isGridStateRestored) {
    return;
  }

  const columnState: any = this.gridColumnApi.getColumnState();
  const groupState: any = this.gridColumnApi.getColumnGroupState();

  this.localStorageService.setItem(AG_GRID_COLUMN_PERSISTENCE_KEY, columnState);
  this.localStorageService.setItem(AG_GRID_GROUP_PERSISTENCE_KEY, groupState);
 }

 onSavePivotModeState() {

  if (!this.isGridStateRestored) {
    return;
  }

  const isPivotMode: boolean = this.gridColumnApi.isPivotMode();

  this.localStorageService.setItem(AG_GRID_PIVOT_PERSISTENCE_KEY, isPivotMode);
 }

 onPaginationChanged() {

  if (!this.isGridStateRestored) {
    return;
  }

  const pageNumber: Number = this.gridApi.paginationGetCurrentPage();

  this.localStorageService.setItem(AG_GRID_PAGINATION_PERSISTENCE_KEY, pageNumber);
 }
}
```

**Note**:

`isGridStateRestored` check is important here. The purpose of this check is to prevent writing into state, 
before the localstorage state has been restored.

## Restoring the state

```html:src/app/app.component.html
<ag-grid-angular
  ...
  (firstDataRendered)="onFirstDataRendered()"
  ...
  
 ></ag-grid-angular>   
```


```ts:src/app/app.component.ts
...
@Component({
 selector: 'app-root',
 templateUrl: './app.component.html',
 styleUrls: ['./app.component.scss']
})
export class AppComponent {

  ...

 constructor(
  private http: HttpClient,
  private localStorageService: LocalStorageService,
) {}

  onFirstDataRendered() {
    const filterState: any = this.localStorageService.getItem(AG_GRID_FILTER_PERSISTENCE_KEY);

    if (filterState) {
      this.gridApi.setFilterModel(filterState);
    }

    const columnState: any = this.localStorageService.getItem(AG_GRID_COLUMN_PERSISTENCE_KEY);
    const groupState: any = this.localStorageService.getItem(AG_GRID_GROUP_PERSISTENCE_KEY);

    if (columnState) {
      this.gridColumnApi.applyColumnState({
        state: columnState,
        applyOrder: true,
      });
    }

    if (groupState) {
      this.gridColumnApi.setColumnGroupState(groupState);
    }

    const isPivotMode: boolean = !!this.localStorageService.getItem(AG_GRID_PIVOT_PERSISTENCE_KEY);

    this.gridApi.setPivotMode(isPivotMode);

    const pageNumber: number = this.localStorageService.getItem(AG_GRID_PAGINATION_PERSISTENCE_KEY);

    if (pageNumber !== null || pageNumber !== undefined) {
      this.gridApi.paginationGoToPage(pageNumber);
    }

    this.isGridStateRestored = true;
  }
  ...
} 

```

## Conclusion

Now you know how to persist and load data for a single grid. If you have multiple grids in you application 
you can abstract this logic in dedicated component, and use that instead. And for state either you can provide 
unique key for each grid instance(which don't change on refresh), or save state based on the route. 



