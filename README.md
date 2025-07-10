<nb-card accent="danger">
<nb-card-header class="d-flex flex-row justify-content-between">
  <h5 class="title-animation title-heading text-uppercase my-auto p-2">Gestion Segment</h5>
</nb-card-header>
  <nb-card-body>
    <div class="row">

      <!-- 🎛️ Colonne gauche - Configuration -->
      <div class="col-md-6">
        <nb-card class="border shadow-sm mt-3 equal-height-card ">
          <nb-card-header style="background-color: #3E5858; color: white;">
            <nb-icon icon="settings-outline"></nb-icon> &nbsp;
            Configuration du segment
          </nb-card-header>

          <nb-card-body>

            <label class="form-label fw-bold text-dark">Segment principal</label>
            <nb-form-field fullWidth>
              <nb-select fullWidth
                         placeholder="Choisir un segment"
                         [(selected)]="selectedSegment"
                         (selectedChange)="onSegmentChange($event)">
                <nb-option *ngFor="let seg of segments" [value]="seg.id">
                  {{ seg.libelle }}
                </nb-option>
              </nb-select>
            </nb-form-field>

            <label class="form-label fw-bold text-dark mt-4">Zone géographique</label>
            <nb-form-field fullWidth>
              <nb-select fullWidth
                         placeholder="Choisir une zone"
                         [(selected)]="selectedAreaId"
                         (selectedChange)="onAreaChange($event)">
                <nb-option *ngFor="let area of areas" [value]="area.id">
                  {{ area.libelle }}
                </nb-option>
              </nb-select>
            </nb-form-field>

            <label class="form-label fw-bold text-dark mt-4">Champs liés</label>
            <nb-form-field fullWidth>
              <nb-select fullWidth
                         placeholder="Sélectionner des champs"
                         [(selected)]="selectedFieldIds"
                         multiple>
                <nb-option *ngFor="let field of filteredFields" [value]="field.id">
                  {{ field.libelle }}
                </nb-option>
              </nb-select>
            </nb-form-field>

          </nb-card-body>
        </nb-card>
      </div>

      <!-- 📋 Colonne droite - Tableau de synthèse -->
      <div class="col-md-6">
        <nb-card class="shadow-sm mt-3 equal-height-card ">

          <!-- Header -->
          <nb-card-header style="background-color: #3E5858; color: white;">
            <nb-icon icon="clipboard-outline"></nb-icon> &nbsp;
            Synthèse & validation
          </nb-card-header>

          <!-- Body avec layout vertical -->
          <nb-card-body class="equal-height-body">

            <div>
              <!-- TABLEAU -->
              <table class="table table-hover table-bordered table-sm mb-4">
                <thead class="table-light">
                <tr>
                  <th>Segment</th>
                  <th>Zone</th>
                  <th>Champ</th>
                  <th class="text-center">Actions</th>
                </tr>
                </thead>
                <tbody>
                <tr *ngFor="let item of summaryTable">
                  <td>{{ item.segment }}</td>
                  <td>{{ item.area }}</td>
                  <td>{{ item.field }}</td>
                  <td class="deleteIcon text-center">
                    <nb-icon icon="delete-bin-line"
                             (click)="onReject(item)"
                             class="text-danger"
                             style="cursor: pointer;"></nb-icon>
                  </td>
                </tr>
                </tbody>
              </table>

              <div *ngIf="summaryTable.length === 0" class="text-muted text-center">
                Aucune donnée sélectionnée pour le segment choisi.
              </div>
            </div>

            <!-- BOUTONS collés en bas -->
            <div class="d-flex justify-content-end gap-3 mt-2">
              <button nbButton status="success" size="medium" (click)="onValidateAll()" outline>
                ✅ Valider tout
              </button>
              <button nbButton status="danger" size="medium" outline (click)="onCancelAll()">
                ❌ Annuler tout
              </button>
            </div>

          </nb-card-body>
        </nb-card>

      </div>

    </div>

  </nb-card-body>
  <nb-card-footer class="mt-4"></nb-card-footer>
</nb-card>
