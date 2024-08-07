<nb-card accent="primary" class="controls">
  <nb-card-header>
    <h5 class="title-animation card-title text-uppercase my-auto p-2">Charger la demande de clôture du compte</h5>
  </nb-card-header>
  <nb-card-body>
    <form [formGroup]="loadFileFormGroup" novalidate>
    <div class="d-flex justify-content-center box">
      <div class="w-50">
        <div class="upload-container">
          <div class="upload-box" id="drop-area">
            <label for="fileElem" class="m-auto w-100">
              <input (change)="onFileSelected($event)" type="file" id="fileElem" #fileElem
                                    accept="image/png, image/jpeg, application/vnd.openxmlformats-officedocument.wordprocessingml.document, application/pdf"
            >
              <div class="upload-box-img">
                <nb-icon icon="file-excel-2-line">
                </nb-icon>
              </div>
              <div class="upload-box-description">
                <p>Faites glisser et déposez un fichier ici ou <span
                  class="fw-bold text-danger">choisissez un fichier</span><br>
                  <span class="text-muted">Pdf, docx, jpg, png (10 MB Max) par fichier</span></p>
              </div>
            </label>
          </div>
          <br>
          <div *ngIf="isUploading" class="upload-progess d-flex justify-content-between shadow d-flex flex-row">
            <div class=" col-lg-1 upload-progress-icon my-auto">
              <nb-icon icon="file-text-line"></nb-icon>
            </div>
            <div class="col-md-8 col-lg-10 upload-progress-values my-auto">
              <div class="file-name fw-bold">{{fileName}}</div>
              <div class="file-size text-muted">{{fileSize}}</div>
              <div class="progress-bar w-100">
                <div [ngStyle]="{width: (progress)+'%'}" class="progress-bar-value"></div>
              </div>
            </div>
            <div (click)="onRestMask()" class="col-lg-1 cancel-icon my-auto">
              <nb-icon icon="close-line"></nb-icon>
            </div>
          </div>
        </div>
      </div>
    </div>
    </form>
  </nb-card-body>
  <nb-card-footer>
    <div class="d-flex flex-row justify-content-between">
      <button (click)="cancel()" type="button" class="btn btn-outline-danger">Abandonner</button>
      <button (click)="createRequest()" type="button" class="btn btn-dark">Soumettre</button>
    </div>
  </nb-card-footer>
</nb-card>

onFileSelected($event: any) {
    this.sendFile = true;
    this.selectedFile = $event.target.files[0];
    if (this.selectedFile) {
      this.isUploading = true
      this.fileSize = this.utilsService.formatBytes(this.selectedFile.size, 2)
      this.fileName = this.selectedFile.name;
      const reader = new FileReader();
      reader.onload = (e) => {
      };
      reader.onprogress = (e) => {
        this.progress = Math.round((e.loaded / e.total) * 100);
        this.utilsService.delayExecution(1500, () => {
          this.sendFile = true;
          console.log("sndFile boolean "+this.sendFile)
        });
      };
      reader.readAsDataURL(this.selectedFile);
    }
  }

  sendFiles(id: number) {
        const formData = new FormData();
        formData.append('file', this.selectedFile, this.selectedFile.name);
        this.demandeService.initRequest(formData,id).pipe(
          map(response => {
            //this.utilsService.delayExecution(1000, () => {
            this.initProcessFormGroup();
            this.isUploading = false
            if (response.status == "REJECTED") {
              this.utilsService.displayError("Erreur", "Le fichier chargé contient des erreurs !")
            } else {
              this.utilsService.displaySucess("Succès ", "Fichier envoyé pour traitement")
            }
            //});
            this.openDialog(response);
            return ({dataState: DataStateEnum.LOADED, data: response})
          }),
          startWith({dataState: DataStateEnum.LOADING}),
          catchError(err => of({
            dataState: DataStateEnum.ERROR,
            // errorMessage: err.message,
            // this: this.utilsService.displayError(err.error.message, 'Erreur'),
          }))).subscribe((appState) => (this.sccRequest$ = appState));
    }
  get form() {
    return this.loadFileFormGroup.controls;
  }

  initProcessFormGroup() {
    this.loadFileFormGroup = this.fb.group({
      motif: ["", Validators.required]
    })
  }

  openDialog(data: SccRequest) {
    const buttonsConfig: NbWindowControlButtonsConfig = {
      minimize: false,
      maximize: false,
      fullScreen: false,
      close: false,

    }
  }

  export interface DemandeClosAccountDTO {
  clientDto ?: Client;
  ribs ?: string[];
  motif ?: string;
  date ?: Date;
}
export interface Client{
  codeClient?: string,
  nomOrRs?: string,
  cinOrRc?: string,
  agence?: string,
  rib: string
}
