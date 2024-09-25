private float calculateFraisCC(ProcessCCDTO processCCDTO)
    {
        float commissionHT = 0;
        float commissionTTC = 0;

        List<CodeBicCCDTO> codeBicCCDTOSWidthLiblleAndNatureFrais = getAllCodeBicCC().stream().filter(
                codeBicCCDTO -> codeBicCCDTO.getLibelle().equals(processCCDTO.getCodeBICBanqueBEN()) &&
                        codeBicCCDTO.getNatureFrais().contains(processCCDTO.getNatureDesFrais())).collect(Collectors.toList());

        for(CodeBicCCDTO codeBicCCDTO : codeBicCCDTOSWidthLiblleAndNatureFrais){
            System.out.println("codebic "+codeBicCCDTO.getLibelle());
            if(processCCDTO.getTypeMessage().contains("202")){
                commissionTTC = codeBicCCDTO.getFraisMT202();
            }
            else{
                if(codeBicCCDTO.getType().equals("tarificationLori")){

                    commissionHT = Math.min(
                            Math.max(
                                    codeBicCCDTO.getMin(),
                                    (Integer.parseInt(processCCDTO.getMontantSansFrais()) * codeBicCCDTO.getTaux())
                            ) , codeBicCCDTO.getMax())+
                            60;

                    commissionTTC = commissionHT + (commissionHT * 10) / 100;

                }
                else{
                    if(Integer.parseInt(processCCDTO.getMontantSansFrais()) >= codeBicCCDTO.getMin() && Integer.parseInt(processCCDTO.getMontantSansFrais()) <= codeBicCCDTO.getMax()){
                        commissionTTC = codeBicCCDTO.getCoutUnitaire();
                    }
                    else{
                        commissionTTC = 0;
                    }
                }
            }
        }
        return commissionTTC;
    }
