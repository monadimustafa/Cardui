public Page<SccRequestDto> findRequestByCriteria(SccRequestSearchDto sccRequestSearchDto , Pageable pageable) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<SccRequest> cq = cb.createQuery(SccRequest.class);
    Root<SccRequest> root = cq.from(SccRequest.class);

    List<Predicate> andPredicate = new ArrayList<>();

    if (sccRequestSearchDto != null) {
        if (sccRequestSearchDto.getRequestRef() != null) {
            Predicate requestRefPredicate = cb.like(root.get("requestRef"), "%" + sccRequestSearchDto.getRequestRef() + "%");
            andPredicate.add(requestRefPredicate);
        }
        if (sccRequestSearchDto.getCreatedAtFrom() != null && sccRequestSearchDto.getCreatedAtTo() != null) {
            Predicate createdAtPredicate = cb.between(root.get("createdAt"), sccRequestSearchDto.getCreatedAtFrom(), sccRequestSearchDto.getCreatedAtTo());
            andPredicate.add(createdAtPredicate);
        }
        if (sccRequestSearchDto.getId() != null) {
            Predicate idPredicate = cb.like(root.get("id"), "%" + sccRequestSearchDto.getId() + "%");
            andPredicate.add(idPredicate);
        }
        if (sccRequestSearchDto.getCustomerNumber() != null) {
            Predicate customerPredicate = cb.like(root.get("bankAcountAggregate").get("customerNumber"), "%" + sccRequestSearchDto.getCustomerNumber() + "%");
            andPredicate.add(customerPredicate);
        }
        if (sccRequestSearchDto.getDisplayName() != null) {
            Predicate displayNamePredicate = cb.like(root.get("bankAcountAggregate").get("displayName"), "%" + sccRequestSearchDto.getDisplayName() + "%");
            andPredicate.add(displayNamePredicate);
        }
        if (sccRequestSearchDto.getCodeAgence() != null) {
            Predicate codeAgencePredicate = cb.like(root.get("bankAcountAggregate").get("codeAgence"), "%" + sccRequestSearchDto.getCodeAgence() + "%");
            andPredicate.add(codeAgencePredicate);
        }
        if (sccRequestSearchDto.getAccountNumber() != null) {
            Predicate accountNumberPredicate = cb.like(root.get("bankAcountAggregate").get("accountNumber"), "%" + sccRequestSearchDto.getAccountNumber() + "%");
            andPredicate.add(accountNumberPredicate);
        }
        if (sccRequestSearchDto.getSoldCompte() != null) {
            Predicate soldComptePredicate = cb.like(root.get("bankAcountAggregate").get("soldCompte"), "%" + sccRequestSearchDto.getSoldCompte() + "%");
            andPredicate.add(soldComptePredicate);
        }
        if (sccRequestSearchDto.getRequestStatus() != null) {
            Predicate requestStatusPredicate = cb.equal(root.get("requestStatus"), RequestStatus.valueOf(sccRequestSearchDto.getRequestStatus()));
            andPredicate.add(requestStatusPredicate);
        }

        cq.where(cb.and(andPredicate.toArray(new Predicate[0])));
    }

    // **Ajout du tri (sorting)**
    if (pageable.getSort().isSorted()) {
        List<Order> orders = new ArrayList<>();
        for (Sort.Order order : pageable.getSort()) {
            if (order.isAscending()) {
                orders.add(cb.asc(root.get(order.getProperty())));
            } else {
                orders.add(cb.desc(root.get(order.getProperty())));
            }
        }
        cq.orderBy(orders);
    }

    return getRequest(cq, pageable);
}
