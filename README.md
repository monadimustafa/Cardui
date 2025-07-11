<!-- Select Risques -->
<td>
  <nb-select placeholder="Choisir un risque" 
             [(selected)]="selectedRisques[area.id]"
             (selectedChange)="onRisqueChange(area.id, $event)"
             status="primary"
             fullWidth>
    <nb-option *ngFor="let item of area.risqueValueList.items" [value]="item.id">
      {{ item.libelle }}
    </nb-option>
  </nb-select>
</td>

<!-- Select Poids -->
<td>
  <nb-select placeholder="Choisir un poids" 
             [(selected)]="selectedWeights[area.id]"
             (selectedChange)="onWeightChange(area.id, $event)"
             status="info"
             fullWidth>
    <nb-option *ngFor="let weight of getSelectedWeights(area)" [value]="weight.id">
      L: {{ weight.weightL }} / ML: {{ weight.weightMl }} / MH: {{ weight.weightMh }}
    </nb-option>
  </nb-select>
</td>
