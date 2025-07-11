  selectedRisques: { [areaId: number]: number } = {}; // area.id → risque.id
  selectedWeights: { [areaId: number]: number } = {}; 


  getSelectedWeights(area: Area): RisqueWeight[] {
    let weights: RisqueWeight[] = [];
    area.fieldConfigurations.forEach(fc => {
      if (fc.risqueWeights) {
        weights = weights.concat(fc.risqueWeights);
      }
    });
    return weights;
  }

  /**
   * ✅ Appelé quand un risque est sélectionné
   */
  onRisqueChange(areaId: number, selectedRisqueId: number): void {
    this.selectedRisques[areaId] = selectedRisqueId;
    console.log(`Risque sélectionné pour area ${areaId}: ${selectedRisqueId}`);
  }

  /**
   * ✅ Appelé quand un poids est sélectionné
   */
  onWeightChange(areaId: number, selectedWeightId: number): void {
    this.selectedWeights[areaId] = selectedWeightId;
    console.log(`Poids sélectionné pour area ${areaId}: ${selectedWeightId}`);
  }

  /**
   * 🚀 Soumettre toutes les sélections
   */
  onSubmit(): void {
    console.log('Sélections risques:', this.selectedRisques);
    console.log('Sélections poids:', this.selectedWeights);
    // 👉 Appel API pour enregistrer ici
  }
}
