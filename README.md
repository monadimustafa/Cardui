  async generateClosureRequest(listSelectedAccount: BankAccountAggregateDto[]): Promise<Uint8Array> {
    const pdfDoc = await PDFDocument.create();
    const page = pdfDoc.addPage([600, 800]);

    const { width, height } = page.getSize();
    const fontSize = 12;

    // Supposons que listSelectedAccount contient plusieurs comptes
    const account = listSelectedAccount[0];

    // objet et date
    page.drawText('Objet : Demande de cloture de compte',{
      x : 150,
      y: height - 60,
      size: 14,
      color: rgb(0,0,0),
    });
    // objet et date
    page.drawText(`Date de création : ${new Date().toLocaleDateString()}`,{
      x : 150,
      y: height - 80,
      size: fontSize,
      color: rgb(0,0,0),
    });
    // Text
    page.drawText(`Je soussigné Mme – Mlle – MR. ${account.displayName} titulaire de la CIN ${account.cin}.<br>
                        autorise la SOCIETE GENERALE MAROCAINE DE BANQUE à fermer mon compte sous le<br>
                        numéro ci-après :<br>`,{
      x : 150,
      y: height - 80,
      size: fontSize,
      color: rgb(0,0,0),
    });

// Tableau RIB et motif
    const tableY = height - 200;
    page.drawText('RIB', { x: 50, y: tableY, size: fontSize, color: rgb(0, 0, 0), });
    page.drawText('Motif de clôture', { x: 250, y: tableY, size: fontSize, color: rgb(0, 0, 0), });

    listSelectedAccount.forEach(value => {
      page.drawText(`${value.rib}`, { x: 50, y: tableY - 20, size: fontSize });
      page.drawText(`${value.motifCloture}`, { x: 250, y: tableY - 20, size: fontSize });
    })

// text
    page.drawText('Je m’engage en outre, à accomplir, s’il y a lieu, toute formalité que la SOCIETE GENERALE Marocaine de Banques<br>' +
                       'jugera utile au bon déroulement de l’opération de clôture et renonce à toutes les facilités qui ont pu m’être octroyées<br>' +
                       'sur ce compte.<br><br><br>' +
      'Nous notons votre engagement à accomplir, s’il y a lieu, toute formalité que la SOCIETE GENERALE MAROCAINE<br>' +
      'DE BANQUES jugera utile au bon déroulement de l’opération de clôture ainsi que votre renonciation à toutes les<br>' +
      'facilités qui ont pu être octroyées sur le compte sus indiqué.<br><br><br>' +
      'Nous vous rappelons que suite à votre demande de clôture de compte, vous devez procéder immédiatement au<br><br><br>' +
      'changement de domiciliation de vos éventuels prélèvements automatiques précédemment domicilié sur ce compte<br><br>',{
      x : 150,
      y: height - 60,
      size: 14,
      color: rgb(0,0,0),
    });
    // Signature
    page.drawText('Signature :', { x: 50, y: tableY - 100, size: fontSize, color: rgb(0, 0, 0), });
    // Générer le PDF
    const pdfBytes = await pdfDoc.save();
    return pdfBytes;
  }
}
