import { PDFDocument, rgb } from 'pdf-lib';

async generateClosureRequest(listSelectedAccount: BankAccountAggregateDto[]): Promise<Uint8Array> {
    const pdfDoc = await PDFDocument.create();
    const page = pdfDoc.addPage([600, 800]);

    const { width, height } = page.getSize();
    const fontSize = 12;
    let textY = height - 60; // Position Y de départ

    // Supposons que listSelectedAccount contient plusieurs comptes
    const account = listSelectedAccount[0];

    // **Titre : Objet et Date**
    page.drawText('Objet : Demande de clôture de compte', {
        x: 150,
        y: textY,
        size: 14,
        color: rgb(0, 0, 0),
    });
    textY -= 20;

    page.drawText(`Date de création : ${new Date().toLocaleDateString()}`, {
        x: 150,
        y: textY,
        size: fontSize,
        color: rgb(0, 0, 0),
    });
    textY -= 30;

    // **Informations du client**
    const clientText = [
        `Je soussigné(e) Mme – Mlle – M. ${account.displayName},`,
        `Titulaire de la CIN : ${account.cin},`,
        `Autorise la SOCIETE GENERALE MAROCAINE DE BANQUE à fermer mon compte`,
        `Numéro de compte : ${account.rib}`,
    ];

    clientText.forEach(line => {
        page.drawText(line, { x: 50, y: textY, size: fontSize, color: rgb(0, 0, 0) });
        textY -= 20;
    });

    textY -= 20; // Espacement avant le tableau

    // **Tableau RIB et Motif**
    page.drawText('RIB', { x: 50, y: textY, size: fontSize, color: rgb(0, 0, 0) });
    page.drawText('Motif de clôture', { x: 250, y: textY, size: fontSize, color: rgb(0, 0, 0) });
    textY -= 20;

    listSelectedAccount.forEach(value => {
        page.drawText(value.rib, { x: 50, y: textY, size: fontSize, color: rgb(0, 0, 0) });
        page.drawText(value.motifCloture, { x: 250, y: textY, size: fontSize, color: rgb(0, 0, 0) });
        textY -= 20; // Décaler la ligne suivante
    });

    textY -= 30; // Espacement après le tableau

    // **Texte de clôture**
    const closingText = [
        "Je m’engage en outre, à accomplir, s’il y a lieu, toute formalité que la SOCIETE GENERALE",
        "Marocaine de Banques jugera utile au bon déroulement de l’opération de clôture et renonce",
        "à toutes les facilités qui ont pu m’être octroyées sur ce compte.",
        "",
        "Nous notons votre engagement à accomplir, s’il y a lieu, toute formalité que la SOCIETE GENERALE",
        "MAROCAINE DE BANQUES jugera utile au bon déroulement de l’opération de clôture ainsi que",
        "votre renonciation à toutes les facilités qui ont pu être octroyées sur le compte sus-indiqué.",
        "",
        "Nous vous rappelons que suite à votre demande de clôture de compte, vous devez procéder",
        "immédiatement au changement de domiciliation de vos éventuels prélèvements automatiques",
        "précédemment domiciliés sur ce compte.",
    ];

    closingText.forEach(line => {
        page.drawText(line, { x: 50, y: textY, size: fontSize, color: rgb(0, 0, 0) });
        textY -= 20;
    });

    textY -= 40; // Espacement avant la signature

    // **Signature**
    page.drawText('Signature :', { x: 50, y: textY, size: fontSize, color: rgb(0, 0, 0) });

    // Générer le PDF
    const pdfBytes = await pdfDoc.save();
    return pdfBytes;
}
