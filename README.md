public ResponseEntity<List<ApplicationDto>> getPortalRequestForRpa(
            @RequestHeader(name = "X-USER", required = false) String userEmail) throws Exception {
        log.info("[Portal - getPortalRequestForRpa] Start resource Analytics Ressource");
        PortalUser portalUser = authorizationService.authorize(userEmail, Action.AUDIT_PILOTAGE);
        List<ApplicationDto> applicationDtos = portalUser.getApplications().stream()
                .map(userMapper::fromApplicationToDTO)
                .collect(Collectors.toList());

        List<ApplicationDto> response = restTemplateConsumer.getRequests(applicationDtos, userEmail);

        calculateCountTask(response);

        return ResponseEntity.ok(response);
    }

    public void calculateCountTask(List<ApplicationDto> response) {
        for (ApplicationDto applicationDto : response) {
            int count = 0;
            for (PortalRequestDTOExtern portalRequestDTOExtern : applicationDto.getPortalRequestDTOExternList()) {
                count += portalRequestDTOExtern.getCount();
            }
            applicationDto.setCountTask(count);
        }
    }
