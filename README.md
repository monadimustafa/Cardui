<nb-card accent="primary" class="border-0 shadow rounded-3 overflow-hidden">
  <!-- 🔥 HEADER PRINCIPAL -->
  <nb-card-header class="d-flex align-items-center justify-content-between" 
                  style="background-color: #CE4529; color: #fff; padding: 1rem;">
    <h5 class="m-0 fw-bold text-uppercase">Gestion Segment</h5>
    <nb-icon icon="layers-outline" pack="eva" class="fs-4"></nb-icon>
  </nb-card-header>

  <nb-card-body class="p-4">
    <div class="row gy-4">

      <!-- 🎛️ Colonne gauche - Configuration -->
      <div class="col-md-6">
        <nb-card class="shadow-sm rounded-3 border-0">
          <nb-card-header style="background-color: #2A4746; color: #fff;" class="fw-bold">
            <nb-icon icon="settings-2-outline" pack="eva" class="me-2"></nb-icon>
            Configuration du segment
          </nb-card-header>

          <nb-card-body class="p-4">
            <!-- Segment -->
            <div class="mb-3">
              <label class="form-label fw-semibold text-dark">Segment principal</label>
              <nb-select
                placeholder="Choisir un segment"
                [(selected)]="selectedSegment"
                (selectedChange)="onSegmentChange($event)"
                status="primary"
                fullWidth>
                <nb-option *ngFor="let seg of segments" [value]="seg.id">
                  {{ seg.libelle }}
                </nb-option>
              </nb-select>
            </div>

            <!-- Zone -->
            <div class="mb-3">
              <label class="form-label fw-semibold text-dark">Zone géographique</label>
              <nb-select
                placeholder="Choisir une zone"
                [(selected)]="selectedAreaId"
                (selectedChange)="onAreaChange($event)"
                status="primary"
                fullWidth>
                <nb-option *ngFor="let area of areas" [value]="area.id">
                  {{ area.libelle }}
                </nb-option>
              </nb-select>
            </div>

            <!-- Champs liés -->
            <div>
              <label class="form-label fw-semibold text-dark">Champs liés</label>
              <nb-select
                placeholder="Sélectionner des champs"
                [(selected)]="selectedFieldIds"
                multiple
                status="primary"
                fullWidth>
                <nb-option *ngFor="let field of filteredFields" [value]="field.id">
                  {{ field.libelle }}
                </nb-option>
              </nb-select>
            </div>
          </nb-card-body>
        </nb-card>
      </div>

      <!-- 📋 Colonne droite - Synthèse -->
      <div class="col-md-6">
        <nb-card class="shadow-sm rounded-3 border-0">
          <nb-card-header style="background-color: #2A4746; color: #fff;" class="fw-bold">
            <nb-icon icon="clipboard-text-outline" pack="eva" class="me-2"></nb-icon>
            Synthèse & Validation
          </nb-card-header>

          <nb-card-body class="p-4">
            <!-- Tableau -->
            <div class="table-responsive rounded border">
              <table class="table table-hover align-middle text-center">
                <thead style="background-color: #F8F9FA; color: #2A4746;">
                  <tr>
                    <th>Segment</th>
                    <th>Zone</th>
                    <th>Champ</th>
                    <th>Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr *ngFor="let item of summaryTable">
                    <td>{{ item.segment }}</td>
                    <td>{{ item.area }}</td>
                    <td>{{ item.field }}</td>
                    <td>
                      <button nbButton ghost status="danger" size="tiny" (click)="onReject(item)">
                        <nb-icon icon="trash-2-outline"></nb-icon>
                      </button>
                    </td>
                  </tr>
                  <tr *ngIf="summaryTable.length === 0">
                    <td colspan="4" class="text-muted text-center">
                      Aucune donnée sélectionnée pour le segment choisi.
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>

            <!-- Boutons actions -->
            <div class="d-flex justify-content-end gap-2 mt-4">
              <button nbButton style="background-color: #CE4529; color: #fff; border-radius: 50px;" 
                      size="medium" (click)="onValidateAll()">
                ✅ Valider tout
              </button>
              <button nbButton outline status="danger" size="medium" class="rounded-pill" 
                      (click)="onCancelAll()">
                ❌ Annuler tout
              </button>
            </div>
          </nb-card-body>
        </nb-card>
      </div>

    </div>
  </nb-card-body>
</nb-card>
