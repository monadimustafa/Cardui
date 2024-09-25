List<CodeBicCCDTO> codeBicCCDTOSWidthLiblleAndNatureFrais = getAllCodeBicCC().stream()
    .filter(codeBicCCDTO -> codeBicCCDTO.getLibelle().equals(processCCDTO.getCodeBICBanqueBEN()) &&
            codeBicCCDTO.getNatureFrais().contains(processCCDTO.getNatureDesFrais()))
    .collect(Collectors.toList());

if (codeBicCCDTOSWidthLiblleAndNatureFrais.isEmpty()) {
    System.out.println("Aucun élément ne correspond aux critères de libellé et nature des frais.");
    return 0;
}

for (CodeBicCCDTO codeBicCCDTO : codeBicCCDTOSWidthLiblleAndNatureFrais) {
    System.out.println("codebic " + codeBicCCDTO.getLibelle());
    System.out.println("Min: " + codeBicCCDTO.getMin() + " Max: " + codeBicCCDTO.getMax());

    try {
        int montantSansFrais = Integer.parseInt(processCCDTO.getMontantSansFrais());

        if (processCCDTO.getTypeMessage().contains("202")) {
            commissionTTC = codeBicCCDTO.getFraisMT202();
        } else {
            if (codeBicCCDTO.getType().equals("tarificationLori")) {
                commissionHT = Math.min(
                        Math.max(codeBicCCDTO.getMin(), montantSansFrais * codeBicCCDTO.getTaux()),
                        codeBicCCDTO.getMax()
                ) + 60;

                commissionTTC = commissionHT + (commissionHT * 10) / 100;
            } else {
                if (montantSansFrais >= codeBicCCDTO.getMin() && montantSansFrais <= codeBicCCDTO.getMax()) {
                    commissionTTC = codeBicCCDTO.getCoutUnitaire();
                } else {
                    commissionTTC = 0;
                }
            }
        }
    } catch (NumberFormatException e) {
        System.out.println("Erreur lors de la conversion du montant sans frais : " + e.getMessage());
        commissionTTC = 0;
    }
}
return commissionTTC;
