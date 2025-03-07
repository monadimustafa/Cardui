import { PDFDocument, rgb } from 'pdf-lib';ll

async function generateClosureRequest(listSelectedAccount: BankAccountAggregateDto[]): Promise<Uint8Array> {
    const pdfDoc = await PDFDocument.create();
    const page = pdfDoc.addPage([600, 800]);

    const { width, height } = page.getSize();
    const fontSize = 12;
    let textY = height - 60; // Position de départ

    // **Titre : Objet et Date**
    page.drawText('Objet : Demande de clôture de compte', { x: 150, y: textY, size: 14, color: rgb(0, 0, 0) });
    textY -= 20;

    page.drawText(`Date de création : ${new Date().toLocaleDateString()}`, { x: 150, y: textY, size: fontSize, color: rgb(0, 0, 0) });
    textY -= 30;

    // **Informations du client**
    const account = listSelectedAccount[0];
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

    // **Tableau avec bordures**
    const tableX = 50;
    const tableY = textY;
    const rowHeight = 25;
    const columnWidths = [180, 320]; // Largeur pour RIB et Motif
    const tableWidth = columnWidths.reduce((a, b) => a + b, 0);
    const tableHeight = (listSelectedAccount.length + 1) * rowHeight;

    // Bordure du tableau
    page.drawRectangle({
        x: tableX,
        y: tableY - tableHeight,
        width: tableWidth,
        height: tableHeight,
        borderColor: rgb(0, 0, 0),
        borderWidth: 1,
    });

    // En-têtes
    page.drawText('RIB', { x: tableX + 10, y: tableY - 5, size: fontSize, color: rgb(0, 0, 0) });
    page.drawText('Motif de clôture', { x: tableX + columnWidths[0] + 10, y: tableY - 5, size: fontSize, color: rgb(0, 0, 0) });

    // Lignes horizontales pour séparer les lignes
    for (let i = 0; i <= listSelectedAccount.length; i++) {
        const yPos = tableY - (i * rowHeight);
        page.drawLine({
            start: { x: tableX, y: yPos },
            end: { x: tableX + tableWidth, y: yPos },
            color: rgb(0, 0, 0),
            thickness: 1,
        });
    }

    // Lignes verticales pour séparer les colonnes
    page.drawLine({
        start: { x: tableX + columnWidths[0], y: tableY },
        end: { x: tableX + columnWidths[0], y: tableY - tableHeight },
        color: rgb(0, 0, 0),
        thickness: 1,
    });

    // Remplissage du tableau
    listSelectedAccount.forEach((value, index) => {
        const yPos = tableY - ((index + 1) * rowHeight) - 5;
        page.drawText(value.rib, { x: tableX + 10, y: yPos, size: fontSize, color: rgb(0, 0, 0) });
        page.drawText(value.motifCloture, { x: tableX + columnWidths[0] + 10, y: yPos, size: fontSize, color: rgb(0, 0, 0) });
    });

    textY -= tableHeight + 40; // Ajuster la position après le tableau

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
