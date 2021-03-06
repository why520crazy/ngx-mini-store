## ngx-mini-store

A mini, yet powerful state management library for Angular.
>Sharing state between several components as mini and powerful. 


## Installation

```
npm install ngx-mini-store --save
# or
yarn add ngx-mini-store --save
```

## Usage & Example
### 1. Define Store and State
```
import { Store, Action } from 'ngx-mini-store';
import { of } from 'rxjs';
import { tap } from 'rxjs/operators';

export interface TaskInfo {
    id: number;
    title: string;
}

interface TasksState {
    tasks: TaskInfo[];
}

export class TasksStore extends Store<TasksState> {

    constructor() {
        super({
            tasks: []
        });
    }

    @Action()
    fetchTasks() {
        const apiMockTasks: TaskInfo[] = [{ id: 1, title: 'Todo 1' }];
        return of(apiMockTasks)
            .pipe(tap((tasks) => {
                const state = this.snapshot;
                state.tasks = tasks;
                this.next(state);
            }));
    }

    @Action()
    addTask(title: string) {
        return of({ id: 100, title: title }).pipe(tap((task) => {
            const state = this.snapshot;
            state.tasks = [...state.tasks, task];
            this.next(state);
        }));
    }
}

```

### 2. Import and use NgxMiniStoreModule in you AppModule, and use `forRoot` set stores

```
import { NgxMiniStoreModule } from 'ngx-mini-store';
import { TasksStore } from './tasks.store.ts'; 

@NgModule({
  imports: [
    NgxMiniStoreModule.forRoot([TasksStore])
  ]
})
export class AppModule {}
```

### 3. Component inject TasksStore and use store select get observable data.

```
import { Component, OnInit, OnDestroy } from '@angular/core';
import { TasksStore, TaskInfo } from '../tasks-store';
import { Observable, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-tasks',
  templateUrl: `
<div>Tasks:</div>
<div *ngFor="let task of this.tasks$ | async">
  {{task.id}} - {{task.title}}
  <button (click)="updateTask(task)">Change</button>
</div>

<hr>
<input [(ngModel)]="addTaskTitle" placeholder="please input task title" (keydown.enter)="addTask(addTaskTitle)">
<button (click)="addTask(addTaskTitle)">Submit</button>

  `
})
export class TasksComponent implements OnInit, OnDestroy {

  tasks$: Observable<TaskInfo[]>;

  unsubscribe$ = new Subject();

  constructor(public store: TasksStore) {
    this.tasks$ = this.store.select((state) => {
       return state.tasks;
    });
  }

  ngOnInit(): void {
    this.store.fetchTasks().subscribe((tasks) => {

    });
  }

  updateTask(task: TaskInfo) {
    task.title += `1`;
  }

  addTask(title: string) {
    if (title) {
      this.store.addTask(title);
      this.addTaskTitle = '';
    }
  }

  ngOnDestroy() {
    this.unsubscribe$.next();
    this.unsubscribe$.complete();
  }
}

```


