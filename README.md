<nb-card *ngFor="let segment of segments" class="mb-4 shadow-sm rounded-3 border-0">
  <!-- Header du Segment -->
  <nb-card-header 
    style="background-color: #CE4529; color: white; font-weight: bold; font-size: 1.1rem;">
    <nb-icon icon="layers-outline" class="me-2"></nb-icon>
    {{ segment.libelle }}
  </nb-card-header>

  <!-- Body avec tableau -->
  <nb-card-body class="p-3">
    <div class="table-responsive rounded border">
      <table class="table table-hover align-middle text-center mb-0">
        <thead style="background-color: #F8F9FA; color: #2A4746;">
          <tr>
            <th>Libelle Area</th>
            <th>Risque</th>
            <th>Poids</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let area of segment.areas">
            <!-- Libelle Area -->
            <td class="fw-semibold">{{ area.libelle }}</td>

            <!-- Select Risques -->
            <td>
              <nb-select placeholder="Choisir un risque" 
                         [(selected)]="selectedRisques[area.id]"
                         status="primary"
                         fullWidth>
                <nb-option *ngFor="let field of area.fieldConfigurations" 
                           *ngIf="field.risqueValueList"
                           *ngFor="let item of field.risqueValueList.items" 
                           [value]="item.id">
                  {{ item.libelle }}
                </nb-option>
              </nb-select>
            </td>

            <!-- Select Poids -->
            <td>
              <nb-select placeholder="Choisir un poids" 
                         [(selected)]="selectedWeights[area.id]"
                         status="info"
                         fullWidth>
                <nb-option *ngFor="let weight of getSelectedWeights(area)" 
                           [value]="weight.id">
                  L: {{ weight.weightL }} / ML: {{ weight.weightMl }} / MH: {{ weight.weightMh }}
                </nb-option>
              </nb-select>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </nb-card-body>
</nb-card>
nb-card {
  border-radius: 12px;
}

nb-card-header {
  font-size: 1.1rem;
}

.table-hover tbody tr:hover {
  background-color: rgba(42, 71, 70, 0.08);
}

nb-select {
  border-radius: 8px;
}

.nb-select button {
  border-radius: 8px;
}
