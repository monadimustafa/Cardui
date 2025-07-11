getSelectedWeights(area: Area): RisqueWeight[] {
  const selectedRisqueId = this.selectedRisques[area.id]; // le risque sélectionné pour cet area
  let weights: RisqueWeight[] = [];

  area.fieldConfigurations.forEach(fc => {
    if (fc.risqueValueList && fc.risqueValueList.items) {
      const selectedRisqueItem = fc.risqueValueList.items.find(item => item.id === selectedRisqueId);
      if (selectedRisqueItem && selectedRisqueItem.risqueWeights) {
        weights = weights.concat(selectedRisqueItem.risqueWeights);
      }
    }
  });

  return weights;
}
