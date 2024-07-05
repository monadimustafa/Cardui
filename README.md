 ngOnInit(): void {
    this.searchApplications();
        interval(5*60*1000).subscribe(() => {
        });
    this.userInfos.getUserInfos().subscribe(value => {
      this.role = value.role;
    });
  }
  searchApplications() {
    this.applications$ = this.service.getApplicationsByUser().pipe(tap(value => {
      value.forEach(value => {
        console.log("value " + value.appName);
        this.searchTasksForApplication(value).subscribe(); // <--- Subscribe to the observable
      })
    }))
  }

  searchTasksForApplication(param: Application): Observable<AppDataState<Task[]>> {
    return this.service.getTasksForApplication(param).pipe(
      map(response => ({ dataState: DataStateEnum.LOADED, data: response })),
      startWith({ dataState: DataStateEnum.LOADING }),
      catchError(err => of({
        dataState: DataStateEnum.ERROR,
        errorMessage: err.message,
        // this: this.showToast('Une erreur technique est survenue', "Erreur", "danger")
      }))
    );
  }
/*  searchApplications() {
    this.applications$ = this.service.getApplicationsByUser().pipe(tap(value => {
      value.forEach(value => {
        console.log("value "+value.appName);
        this.searchTasksForApplication(value);
      })
    }))
  }


  searchTasksForApplication(param:Application){
    this.service.getTasksForApplication(param).pipe(map(response => {
        return ({dataState: DataStateEnum.LOADED, data: response})
      }),
      startWith({dataState: DataStateEnum.LOADING}),
      catchError(err => of({
        dataState: DataStateEnum.ERROR,
        errorMessage: err.message,
        //this: this.showToast('Une erreur technique est survenue', "Erreur", "danger")
      })))
  }
  showToast(message: string, title: string, status: string) {
    return this.toastrService.show(message, title, {status, duration: 0});
  }*/
