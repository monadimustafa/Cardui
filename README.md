ngOnInit() {
    this.rowUuid = this.route.snapshot.params['uuid']
    this.getProcessByUser();
    if (this.rowUuid) {
      this.duplicateOrder();
    } else {
      this.initForm();
    }
    forkJoin({
      typeMessage: this.processClientService.getTypeMessage(),
      ccStatut: this.processClientService.getCCStatut(),
      natureFrais: this.processClientService.getNatureFrais(),
      compteCollecteur: this.processClientService.getCompteCollecteur(),
      codesBicsLibelle: this.processClientService.getDistinctLibelleCodeBic(),
    }).subscribe(({codesBicsLibelle,typeMessage, ccStatut, natureFrais, compteCollecteur}) => {
      this.typeMessageCCS = typeMessage;
      this.statutCC = ccStatut;
      this.natureFraisCC = natureFrais;
      this.compteCollecteurCC = compteCollecteur;
      this.codesBicsLibelle = codesBicsLibelle;

    })
  }
