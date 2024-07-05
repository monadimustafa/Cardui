import { BehaviorSubject, Observable, of } from 'rxjs';
import { catchError, concatMap, map, startWith, tap, toArray } from 'rxjs/operators';

data$: BehaviorSubject<AppDataState<Task[]>> = new BehaviorSubject<AppDataState<Task[]>>({
  dataState: DataStateEnum.LOADING
});
applications$!: Observable<Application[]>;

searchApplications() {
  this.applications$ = this.service.getApplicationsByUser().pipe(
    tap(applications => {
      this.data$.next({ dataState: DataStateEnum.LOADING });
      
      applications.pipe(
        concatMap(application =>
          this.searchTasksForApplication(application).pipe(
            map(taskState => taskState.data)
          )
        ),
        toArray() // Collect all results into an array
      ).subscribe({
        next: allTasks => {
          const combinedTasks = allTasks.flat();
          this.data$.next({ dataState: DataStateEnum.LOADED, data: combinedTasks });
        },
        error: err => {
          this.data$.next({
            dataState: DataStateEnum.ERROR,
            errorMessage: err.message
          });
        }
      });
    })
  );
}

searchTasksForApplication(param: Application): Observable<AppDataState<Task[]>> {
  return this.service.getTasksForApplication(param).pipe(
    map(response => ({ dataState: DataStateEnum.LOADED, data: response })),
    startWith({ dataState: DataStateEnum.LOADING }),
    catchError(err => of({
      dataState: DataStateEnum.ERROR,
      errorMessage: err.message
    }))
  );
}
