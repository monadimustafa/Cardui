<nb-form-field>
                <nb-select-label>BIC Donneur d'ordre <span class="text-danger">*</span>
                </nb-select-label>
                <nb-select
                  fullWidth
                  id="bicDonneurOrd"
                  formControlName="bicDonneurOrd"
                  selectrequired
                  [ngClass]="{ 'status-danger': submitted && form['bicDonneurOrd']?.errors }"
                >
                  <!-- Afficher la valeur sélectionnée uniquement si elle existe -->
                  <nb-option *ngIf="selectedBic" [value]="selectedBic">
                    {{ selectedBic }}
                  </nb-option>

                  <!-- Charger la liste des codes BIC si selectedBic est vide ou null -->
                  <ng-container *ngIf="selectedBic == null || selectedBic === undefined">
                    <nb-option *ngFor="let paramCode of codesBicsLibelle" [value]="paramCode">
                      {{ paramCode }}
                    </nb-option>
                  </ng-container>

                </nb-select>
              </nb-form-field>
 ngOnInit() {
    this.rowUuid = this.route.snapshot.params['uuid']
    this.getProcessByUser();
    if (this.rowUuid) {
      this.duplicateOrder();
    } else {
      this.initForm();
    }
    forkJoin({
      typeMessage: this.processClientService.getTypeMessage(),
      ccStatut: this.processClientService.getCCStatut(),
      natureFrais: this.processClientService.getNatureFrais(),
      compteCollecteur: this.processClientService.getCompteCollecteur(),
      codesBicsLibelle: this.processClientService.getDistinctLibelleCodeBic(),
    }).subscribe(({codesBicsLibelle,typeMessage, ccStatut, natureFrais, compteCollecteur}) => {
      this.typeMessageCCS = typeMessage;
      this.statutCC = ccStatut;
      this.natureFraisCC = natureFrais;
      this.compteCollecteurCC = compteCollecteur;
      this.codesBicsLibelle = codesBicsLibelle;

      this.selectedBic = this.saveFormCC.get('bicDonneurOrd')?.value || null;

    })



  }
