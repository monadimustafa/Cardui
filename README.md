 @GetMapping("/requests")
    @Operation(summary = "Get all requests",
            description = "This resource is used to get all SCC request",
            responses = {
                    @ApiResponse(responseCode = "200", description = "Successful operation, check payload content")
            })
    public ResponseEntity<Page<SccRequestDto>> getAllRequest(@RequestParam(name = "pageNumber", defaultValue = "0", required = false) int pageNumber,
                                                             @RequestParam(name = "pageSize", defaultValue = "10", required = false) int pageSize,
                                                             @RequestParam(name = "sortValue", defaultValue = "id", required = false) String sortValue,
                                                             @RequestParam(name = "sortDirection", defaultValue = "DESC", required = false) String sortDirection,
                                                             @RequestParam(name = "requestRef", required = false) final String requestRef,
                                                             @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) LocalDateTime createdAtFrom,
                                                             @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) LocalDateTime createdAtTo,
                                                             @RequestParam(name = "createdAt", required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) final LocalDateTime createdAt,
                                                             @RequestParam(name = "id", required = false) String id,
                                                             @RequestParam(name = "customerNumber", required = false) String customerNumber,
                                                             @RequestParam(name = "displayName", required = false) String displayName,
                                                             @RequestParam(name = "codeAgence", required = false) String codeAgence,
                                                             @RequestParam(value = "accountNumber", required = false) String accountNumber,
                                                             @RequestParam(value = "requestStatus", required = false) String requestStatus,
                                                             @RequestParam(value = "soldCompte", required = false) BigDecimal soldCompte,
                                                             @RequestHeader(name = "X-USER", required = false) String userEmail) {
        SccRequestSearchDto sccRequestSearchDto = SccRequestSearchDto.builder()
                .requestRef(requestRef)
                .createdAt(createdAt)
                .createdAtFrom(createdAtFrom)
                .createdAtTo(createdAtTo)
                .id(id)
                .customerNumber(customerNumber)
                .displayName(displayName)
                .codeAgence(codeAgence)
                .accountNumber(accountNumber)
                .requestStatus(requestStatus)
                .soldCompte(soldCompte)
                .build();

        Page<SccRequestDto> portalRequests = iSccRequest.findRequestByCriteria(sccRequestSearchDto, PageRequest.of(pageNumber, pageSize, Sort.by(Sort.Direction.DESC,"createdAt")));
        return ResponseEntity.ok(portalRequests);
    }

    public Page<SccRequestDto> findRequestByCriteria(SccRequestSearchDto sccRequestSearchDto , Pageable pageable){


        CriteriaBuilder cb= em.getCriteriaBuilder();
        CriteriaQuery<SccRequest> cq=cb.createQuery(SccRequest.class);
        Root<SccRequest> root= cq.from(SccRequest.class);

        List<Predicate> andPredicate=new ArrayList<>();

        if (sccRequestSearchDto != null){
             if (sccRequestSearchDto.getRequestRef() != null) {
                Predicate matriculePredicate=cb.like(root.get("requestRef"),"%"+sccRequestSearchDto.getRequestRef()+"%");
                andPredicate.add(matriculePredicate);
                 if (sccRequestSearchDto.getCreatedAtFrom() != null && sccRequestSearchDto.getCreatedAtTo() != null){
                     Predicate createdAtPredicate=cb.between(root.get("createdAt"),sccRequestSearchDto.getCreatedAtFrom(),sccRequestSearchDto.getCreatedAtTo());
                     andPredicate.add(createdAtPredicate);}
             }if (sccRequestSearchDto.getId() != null){
                Predicate prenomPredicate=cb.like(root.get("id"),"%"+sccRequestSearchDto.getId()+"%");
                andPredicate.add(prenomPredicate);
            }if (sccRequestSearchDto.getCustomerNumber() != null) {
                Predicate affectationFinaleActuellePredicate = cb.like(root.get("bankAcountAggregate").get("customerNumber"), "%" + sccRequestSearchDto.getCustomerNumber() + "%");
                andPredicate.add(affectationFinaleActuellePredicate);
            }if (sccRequestSearchDto.getDisplayName() != null){
                Predicate affectationN3Predicate=cb.like(root.get("bankAcountAggregate").get("displayName"),"%"+sccRequestSearchDto.getDisplayName()+"%");
                andPredicate.add(affectationN3Predicate);
            }
            if (sccRequestSearchDto.getCodeAgence() != null){
                Predicate affectationN3Predicate=cb.like(root.get("bankAcountAggregate").get("codeAgence"),"%"+sccRequestSearchDto.getCodeAgence()+"%");
                andPredicate.add(affectationN3Predicate);
            }
            if (sccRequestSearchDto.getAccountNumber() != null){
                Predicate affectationN3Predicate=cb.like(root.get("bankAcountAggregate").get("accountNumber"),"%"+sccRequestSearchDto.getAccountNumber()+"%");
                andPredicate.add(affectationN3Predicate);
            }
            if (sccRequestSearchDto.getSoldCompte() != null){
                Predicate affectationN3Predicate=cb.like(root.get("bankAcountAggregate").get("soldCompte"),"%"+sccRequestSearchDto.getSoldCompte()+"%");
                andPredicate.add(affectationN3Predicate);
            }
            if (sccRequestSearchDto.getRequestStatus() != null){
                Predicate affectationN3Predicate=cb.equal(root.get("requestStatus"), RequestStatus.valueOf(sccRequestSearchDto.getRequestStatus()));
                andPredicate.add(affectationN3Predicate);
            }


            cq.where(cb.and(andPredicate.toArray(new Predicate[0])));

            return getRequest(cq,pageable);

        }else {
            return getRequest(cq,pageable);
        }

    }
    private Page<SccRequestDto> getRequest(CriteriaQuery<SccRequest> cq, Pageable pageable) {
        TypedQuery<SccRequest> tq= em.createQuery(cq);
        tq.setFirstResult((int) pageable.getOffset());
        tq.setMaxResults(pageable.getPageSize());
        int totalRows=sccRepo.findAll().size();
        List<SccRequestDto> requestList =mapper.toListDtos(tq.getResultList());
        Page<SccRequestDto> requestDtoPage=new PageImpl<>(requestList, pageable,totalRows);
        return requestDtoPage;
    }
