<nb-card *ngFor="let segment of segments">
  <nb-card-header>{{ segment.libelle }}</nb-card-header>

  <nb-card-body>
    <nb-table>
      <thead>
        <tr>
          <th>Libelle Area</th>
          <th>Risques</th>
          <th>Poids</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let area of segment.areas">
          <td>{{ area.libelle }}</td>
          
          <td>
            <nb-select placeholder="Choisir un risque" [(selected)]="selectedRisques[area.id]">
              <nb-option *ngFor="let field of area.fieldConfigurations"
                         *ngIf="field.risqueValueList"
                         *ngFor="let item of field.risqueValueList.items"
                         [value]="item.id">
                {{ item.libelle }}
              </nb-option>
            </nb-select>
          </td>

          <td>
            <div *ngIf="getSelectedWeights(area)?.length">
              <ul>
                <li *ngFor="let w of getSelectedWeights(area)">
                  L: {{ w.weightL }} / ML: {{ w.weightMl }} / MH: {{ w.weightMh }}
                </li>
              </ul>
            </div>
          </td>
        </tr>
      </tbody>
    </nb-table>
  </nb-card-body>
</nb-card>
