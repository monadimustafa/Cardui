    private List<ProcessCCDTO> checkTempProvisionWithEnoughBalance() {
        log.info("Start Service checkTempProvisionWithEnoughBalance at : {}", LocalDateTime.now());

        List<TempProvision> tempProvisions = tempProvisionRepository.findByCheckInAfterAndExecutedIsFalse(LocalDateTime.now().minusDays(5));
        log.info("Number of element matching 5 days criteria : {}", tempProvisions.size());

        List<ProcessCCDTO> processCCList = new ArrayList<>();
        for (TempProvision tempProvision : tempProvisions) {
            try {
                log.info("Start Calling Account Service with params: Account Number: {}, Branch Code: {}", tempProvision.getNCompte(), tempProvision.getBranchCode());
                AccountDetailDTO accountDetailDTO = accountFeignClient.getAccountDetails(tempProvision.getNCompte(), tempProvision.getBranchCode(), "000");
                BigDecimal availableBalance = accountDetailDTO.getAvailableBalance();
                ProcessCC processCC = processCCService.findFirstByUuid(tempProvision.getProcessUuid());
                processCC.setAvailableBalance(availableBalance != null ? availableBalance.toString() : "-1");

                BigDecimal orderValue = new BigDecimal(processCC.getMontantSansFrais());
                BigDecimal accountValue = new BigDecimal(processCC.getAvailableBalance());

                if (orderValue.compareTo(accountValue) > 0) {
                    log.info("Account Without enough balance: {}, Required Balance: {}, Found balance: {} "
                            , tempProvision.getNCompte(), processCC.getMontantSansFrais(), processCC.getAvailableBalance());
                    tempProvisionRepository.save(TempProvision.builder()
                            .id(tempProvision.getId())
                            .availableBalance(availableBalance)
                            .branchCode(tempProvision.getBranchCode())
                            .checkIn(tempProvision.getCheckIn())
                            .processUuid(processCC.getUuid())
                            .nCompte(tempProvision.getNCompte())
                            .overFiveDays(tempProvision.isOverFiveDays())
                            .build());
                } else {
                    log.info("Account With enough balance: {}, Required Balance: {}, Found balance: {} "
                            , tempProvision.getNCompte(), processCC.getMontantSansFrais(), processCC.getAvailableBalance());
                    TempProvision updateTempProvision = tempProvisionRepository.save(TempProvision.builder()
                            .id(tempProvision.getId())
                            .availableBalance(availableBalance)
                            .branchCode(tempProvision.getBranchCode())
                            .checkIn(tempProvision.getCheckIn())
                            .executed(true)
                            .nCompte(tempProvision.getNCompte())
                            .overFiveDays(tempProvision.isOverFiveDays())
                            .processUuid(tempProvision.getProcessUuid())
                            .build());
                    log.info("updateTempProvision: {}", updateTempProvision.toString());
                    processCCList.add(portalProcessMapper.fromProcessCCToDTO(processCC));
                }
            } catch (Exception e) {
                log.error("Exception Occurred when calling Customer Service for Account: {}", tempProvision.getNCompte());
                log.error(e.getMessage());
                TempProvision exceptionTempProvision = tempProvisionRepository.save(TempProvision.builder()
                        .id(tempProvision.getId())
                        .availableBalance(tempProvision.getAvailableBalance())
                        .branchCode(tempProvision.getBranchCode())
                        .checkIn(tempProvision.getCheckIn())
                        .executed(tempProvision.isExecuted())
                        .nCompte(tempProvision.getNCompte())
                        .overFiveDays(tempProvision.isOverFiveDays())
                        .processUuid(tempProvision.getProcessUuid())
                        .build());
                log.info("exceptionTempProvision: {}", exceptionTempProvision.toString());

            }
        }

        return processCCList;
    }
