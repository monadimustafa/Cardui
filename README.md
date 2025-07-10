<nb-card accent="primary" class="border-0 shadow rounded-3 overflow-hidden">
  <!-- 🔥 HEADER PRINCIPAL -->
  <nb-card-header class="d-flex align-items-center justify-content-between bg-primary text-white p-3">
    <h5 class="m-0 fw-bold text-uppercase">Gestion Segment</h5>
    <nb-icon icon="layers-outline" pack="eva" class="fs-4"></nb-icon>
  </nb-card-header>

  <nb-card-body class="p-4">
    <div class="row gy-4">

      <!-- 🎛️ Colonne gauche - Configuration -->
      <div class="col-md-6">
        <nb-card class="shadow-sm rounded-3 border-0">
          <nb-card-header class="bg-secondary text-white fw-bold">
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
          <nb-card-header class="bg-secondary text-white fw-bold">
            <nb-icon icon="clipboard-text-outline" pack="eva" class="me-2"></nb-icon>
            Synthèse & Validation
          </nb-card-header>

          <nb-card-body class="p-4">
            <!-- Tableau -->
            <div class="table-responsive rounded border">
              <table class="table table-hover align-middle text-center">
                <thead class="bg-light text-dark sticky-top">
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
              <button nbButton status="success" size="medium" (click)="onValidateAll()" class="rounded-pill">
                ✅ Valider tout
              </button>
              <button nbButton status="danger" size="medium" outline (click)="onCancelAll()" class="rounded-pill">
                ❌ Annuler tout
              </button>
            </div>
          </nb-card-body>
        </nb-card>
      </div>

    </div>
  </nb-card-body>
</nb-card>


nb-card {
  border-radius: 12px;
}

nb-card-header {
  font-size: 1rem;
}

.table th {
  background-color: #f8f9fa;
  color: #495057;
}

.table-hover tbody tr:hover {
  background-color: #f1f3f5;
}

button[nbButton] {
  border-radius: 20px;
  font-weight: 500;
}
